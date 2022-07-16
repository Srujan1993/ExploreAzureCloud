## Asynchronous Request-Reply Pattern in Azure - Part 2

**Introduction : **

In part 1, we saw Asynchronous Request-Reply Pattern in Azure, useful scenarios in which the pattern is implemented, and different approaches to implementing the pattern in Azure.

In this blog post, I will implement the pattern using one of the approaches.

**Prerequisites :**

1) Please go through  [https://exploreazurecloud.com/asynchronous-request-reply-pattern-in-azure-part-1](Link) before you read this blog post.
2) If you want to try implementing this pattern practically, You need to have an Azure subscription, create an Azure Free Account using this link
[https://azure.microsoft.com/en-in/free/?ref=microsoft.com&utm\_source=microsoft.com&utm\_medium=docs&utm\_campaign=visualstudio](https://azure.microsoft.com/en-in/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
3) Make Sure that you have Visual Studio installed in your machine along with Azure development tools selected in the Visual Studio Installer.
4) In order to develop Azure Functions using Visual Studio, use this link as a reference  [https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-vs ](Link)


**Practical Implementation of the Pattern using Azure Functions : **

In part 1, we saw multiple approaches to implement this pattern i.e using Azure Functions, Azure Logic Apps, and also a combination of Azure Functions and Azure Logic Apps.  

For this demo, we will implement the pattern using Azure Functions. I referred to the GitHub sample  [https://github.com/mspnp/cloud-design-patterns/tree/master/async-request-reply/src](Link)  mentioned in the MSDN documentation to demonstrate this pattern. 

For Simplification purposes, let us consider Postman as the Client in place of a web browser. 


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1629516152328/dGSvODV_1G.png)




-  We will have three functions implemented within this pattern, the first function receives the request from the postman and then sends a reply with 202 status code and status endpoint URL which the client i.e postman asynchronously and also sends the request message to the service bus queue for further processing. Here is the code snippet for the function.


```
public static class AsyncProcessingWorkAcceptor
    {
        [FunctionName("AsyncProcessingWorkAcceptor")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = null)] Customer customer,
            [ServiceBus("outqueue", Connection = "ServiceBusConnectionAppSetting")] IAsyncCollector<Message> OutMessage,
            ILogger log)
        {
            if (String.IsNullOrEmpty(customer.id) || String.IsNullOrEmpty(customer.customername))
            {
                return new BadRequestResult();
            }

            string reqid = Guid.NewGuid().ToString();

            string rqs = $"http://{Environment.GetEnvironmentVariable("WEBSITE_HOSTNAME")}/api/RequestStatus/{reqid}";

            var messagePayload = JsonConvert.SerializeObject(customer);
            Message m = new Message(Encoding.UTF8.GetBytes(messagePayload));
            m.UserProperties["RequestGUID"] = reqid;
            m.UserProperties["RequestSubmittedAt"] = DateTime.Now;
            m.UserProperties["RequestStatusURL"] = rqs;

            await OutMessage.AddAsync(m);

            return (ActionResult)new AcceptedResult(rqs, $"Request Accepted for Processing{Environment.NewLine}ProxyStatus: {rqs}");
        }
    }
``` 



- Second function is a service bus queue trigger which polls the message from the queue and then writes it to the blob storage having sas signature location.


```
public static class AsyncProcessingBackgroundWorker
{
    [FunctionName("AsyncProcessingBackgroundWorker")]
    public static void Run(
        [ServiceBusTrigger("outqueue", Connection = "ServiceBusConnectionAppSetting")]Message myQueueItem,
        [Blob("data", FileAccess.ReadWrite, Connection = "StorageConnectionAppSetting")] CloudBlobContainer inputBlob,
        ILogger log)
    {
        // Perform an actual action against the blob data source for the async readers to be able to check against.
        // This is where your actual service worker processing will be performed.

        var id = myQueueItem.UserProperties["RequestGUID"] as string;

        CloudBlockBlob cbb = inputBlob.GetBlockBlobReference($"{id}.blobdata");

        // Now write the results to blob storage.
        cbb.UploadFromByteArrayAsync(myQueueItem.Body, 0, myQueueItem.Body.Length);
    }
}
``` 

- The Last function is exposed as a GET method with status endpoint URL and two optional query parameters added in the function.

      
1. One query parameter is **OnComplete** and the other one is **On pending**.
2. If the caller wants the redirect URI of the blob storage as a response if the input blob exists in the status endpoint URL, then the caller has to pass **Redirect** as a value where as if the caller wants to file to be downloaded and returned directly, then the caller has to pass **Stream** as value.
3. Lets's say the blob request is still not completed i.e the blob is still not fully available on the endpoint location, the caller can pass **Accepted** as value to **OnPending** query parameter if he wants a 202 status code and a self-referencing Location header pointing to the endpoint URL. 
4. Caller can pass **Synchronous** as value to **OnPending** query parameter to call the retry logic to check for the blob existence , call the OnComplete logic if the blob exist after some time interval else send blob not found response back to the caller.


```
public static class AsyncOperationStatusChecker
{
    [FunctionName("AsyncOperationStatusChecker")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "RequestStatus/{thisGUID}")] HttpRequest req,
        [Blob("data/{thisGuid}.blobdata", FileAccess.Read, Connection = "StorageConnectionAppSetting")] CloudBlockBlob inputBlob, string thisGUID,
        ILogger log)
    {

        OnCompleteEnum OnComplete = Enum.Parse<OnCompleteEnum>(req.Query["OnComplete"].FirstOrDefault() ?? "Redirect");
        OnPendingEnum OnPending = Enum.Parse<OnPendingEnum>(req.Query["OnPending"].FirstOrDefault() ?? "Accepted");

        log.LogInformation($"C# HTTP trigger function processed a request for status on {thisGUID} - OnComplete {OnComplete} - OnPending {OnPending}");

        // Check to see if the blob is present.
        if (await inputBlob.ExistsAsync())
        {
            // If it's present, depending on the value of the optional "OnComplete" parameter choose what to do.
            return await OnCompleted(OnComplete, inputBlob, thisGUID);
        }
        else
        {
            // If it's NOT present, check the optional "OnPending" parameter.
            string rqs = $"http://{Environment.GetEnvironmentVariable("WEBSITE_HOSTNAME")}/api/RequestStatus/{thisGUID}";

            switch (OnPending)
            {
                case OnPendingEnum.Accepted:
                    {
                        // Return an HTTP 202 status code.
                        return (ActionResult)new AcceptedResult() { Location = rqs };
                    }

                case OnPendingEnum.Synchronous:
                    {
                        // Back off and retry. Time out if the backoff period hits one minute
                        int backoff = 250;

                        while (!await inputBlob.ExistsAsync() && backoff < 64000)
                        {
                            log.LogInformation($"Synchronous mode {thisGUID}.blob - retrying in {backoff} ms");
                            backoff = backoff * 2;
                            await Task.Delay(backoff);
                        }

                        if (await inputBlob.ExistsAsync())
                        {
                            log.LogInformation($"Synchronous Redirect mode {thisGUID}.blob - completed after {backoff} ms");
                            return await OnCompleted(OnComplete, inputBlob, thisGUID);
                        }
                        else
                        {
                            log.LogInformation($"Synchronous mode {thisGUID}.blob - NOT FOUND after timeout {backoff} ms");
                            return (ActionResult)new NotFoundResult();
                        }
                    }

                default:
                    {
                        throw new InvalidOperationException($"Unexpected value: {OnPending}");
                    }
            }
        }
    }

    private static async Task<IActionResult> OnCompleted(OnCompleteEnum OnComplete, CloudBlockBlob inputBlob, string thisGUID)
    {
        switch (OnComplete)
        {
            case OnCompleteEnum.Redirect:
                {
                    // Redirect to the SAS URI to blob storage
                    return (ActionResult)new RedirectResult(inputBlob.GenerateSASURI());
                }

            case OnCompleteEnum.Stream:
                {
                    // Download the file and return it directly to the caller.
                    // For larger files, use a stream to minimize RAM usage.
                    return (ActionResult)new OkObjectResult(await inputBlob.DownloadTextAsync());
                }

            default:
                {
                    throw new InvalidOperationException($"Unexpected value: {OnComplete}");
                }
        }
    }
}

public enum OnCompleteEnum {

    Redirect,
    Stream
}

public enum OnPendingEnum {

    Accepted,
    Synchronous
}
``` 

**Demo:**

-  I hosted all these functions in an azure function app and also making sure we have one service bus queue and storage account created.
- Let's execute the first function which accepts the input message and then pushes the message to a queue and also sends an endpoint URL back in the response for the caller to poll later.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630125838621/O_l8X5-jA.png)


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630125899761/bgOBGMh_0.png)

-  Asynchronous work acceptor function is called from Postman to start the process.
-  A simple request is passed as input to the function and the function, in turn, returns 202 as status code response, also the endpoint URL is sent in the response as highlighted in the below screenshot. URL contains a unique GUID representing the message and at the same time, the message is sent to a service bus queue.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630127267762/Z1FlQwhhu.png)


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630127371929/52pxrnl-1.png)


- The second Function i.e Asynchronous Background worker polls the message from the queue and writes that message to blob storage with the id of the blob set to the GUID created in the first function.



![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630157350996/vBYzsh5rc.png)

- Now the client i.e Postman, in this case, triggers the final function i.e Asynchronous status checker to retrieve the data from the blob storage using GUID. As the query parameters are optional, didn't pass it explicitly.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630157078964/ZxRRxCJCM.png)



- So the caller was able to receive the response message successfully using the Status endpoint URL created in the first azure function.

- To keep it simple, I have used a simple JSON in the demo however these patterns will be of great use when the back end API is dealing with large messages, and this  Backend Processing is usually decoupled from the front end host where the backend process becomes asynchronous but the front end still needs a clear response.

-  Retry logic implemented in the Asynchronous Status Checker will come into the picture while dealing with large messages.


**References:**

 - [https://docs.microsoft.com/en-us/azure/architecture/patterns/async-request-reply](
Link) 

- **Github Sample** :  [https://github.com/mspnp/cloud-design-patterns/tree/master/async-request-reply](Link) 





