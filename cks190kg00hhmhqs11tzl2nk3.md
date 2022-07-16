## Asynchronous Request-Reply Pattern in Azure - Part 1

**Context and Problem:**
        

- In modern application development, it's normal for client applications i.e code running on a web client to depend on remote APIs to provide business logic.
- Backend Processing is usually decoupled from the front end host where the backend process becomes asynchronous but the front end still needs a clear response. 
- Backend APIs exposed might be directly related to an application or maybe shared services provided by a third party. These API calls happen over HTTPS protocol following REST Semantics.
- In the majority of the cases, API's for a client application is designed to respond quickly in the order of milliseconds however many factors can lead to response latency like below 

        1. Relative Geographic Location of the Caller and Backend
        2. Network Infrastructure
        3. Security Components
        4. Size of the request payload.
- These are some of the major factors leading to latency in the response.
-  In some scenarios, the work done by the backend API might be long-running i.e in the order of seconds, or might be a background process that gets executed in minutes or hours. In such cases, it is not feasible to wait for the work to complete before responding to the request. This is one of the potential problems for any synchronous request and reply pattern.
- Some Architecture solves these problems by introducing a message broker between the request and response stages. This separation is achieved using    [https://exploreazurecloud.com/implement-queue-based-load-leveling-pattern-in-azure-part-1](Link). This separation can allow the client process and the backend API to scale independently. But this separation also brings additional complexity when the client requires success notification, as this step needs to become asynchronous.
- Many of these considerations discussed above happens not only for client applications but also for the server to server REST API calls in distributed systems in a microservice architecture.

**Solution: **


- In order to implement this pattern, one can use HTTP polling. Polling usually happens on the client-side code as it can be hard to provide call-back endpoints or provide long-running connections. Even though callbacks are possible, it becomes cumbersome to manage required extra libraries and services which makes the solution complex.

- So Asynchronous request and reply patterns can be implemented like below 


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628262077215/kLHC9q9rQw.png)

-  The client sends a request and receives an HTTP 202 (Accepted) response.
- The client sends an HTTP GET request to the status endpoint. The work is still pending, so this call also returns HTTP 202.
- At some point, the work is complete and the status endpoint returns 302 (Found) redirecting to the resource.
- The client fetches the resource at the specified URL.

**Issues and Considerations: **

-  Please be noted that there are many ways to implement this pattern using HTTP and not all upstream systems follow the same semantics.
-  For Instance, a typical GET response doesn't return the status as 202. If we follow the standard REST Semantics, we should ideally receive status as 404 i.e Not found which actually makes sense as the response of the call is not present yet.
- A typical HTTP 202 response should send two key headers in the response to the client. One is the location that contains the URL the client should poll for the response status and the other one is retry-after which tells the client to retry after a specific point of time rather than overwhelming the backend with retries. 
- Not all solutions implement this pattern in the same way as some might include additional or alternate headers.
- Legacy Clients might not support this pattern, in such scenarios we need to introduce a facade between the Legacy Client and Asynchronous API which hides the asynchronous processing from the Client. The best example is to use an Azure Logic App as a facade layer between the Client and the API.


**Scenarios to implement this pattern:**

- One can use this pattern while implementing client-side code such as browser applications, where it's difficult to provide callback endpoints or using long-running connections leads to complexity.
- Service calls that need to integrate with legacy architectures that don't support modern technologies like webhooks.


**Approaches to implement this pattern:**

- one can implement this pattern is using a combination of Azure Functions/Azure Logic Apps, we can also use Azure Logic Apps in place of Azure Functions.

**Approach 1 :**

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628307962009/X7AMT-InH.png)

**Approach 2 :**

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628308121169/37o99_M_4l.png)

**Approach 3:** 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1628308263905/k9OR5mJta.png)


**References : **

 [https://docs.microsoft.com/en-us/azure/architecture/patterns/async-request-reply](Link) 

















 
   


     





