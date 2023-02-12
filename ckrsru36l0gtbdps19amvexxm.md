# Implement Claim Check Pattern in Azure - Part 2

**Introduction :**

In part 1, we saw what is a claim check pattern, useful scenarios in which the pattern is implemented, and different approaches to implementing the pattern in Azure. In this blog post, I will implement the pattern using one of the approaches.

**Prerequisites :**

1. Please go through [https://exploreazurecloud.com/2021/06/implement-claim-check-pattern-in-azure-part-1/](https://exploreazurecloud.com/2021/06/implement-claim-check-pattern-in-azure-part-1/) before you read this blog post.
    
2. If you want to try implementing this pattern practically, You need to have an Azure subscription, create an Azure Free Account using this link [https://azure.microsoft.com/en-in/free/?ref=microsoft.com&utm\_source=microsoft.com&utm\_medium=docs&utm\_campaign=visualstudio](https://azure.microsoft.com/en-in/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
    

**Practical Implementation of Claim Check Pattern in Azure using** **Azure Storage Blob**, **Azure Event Grid System Topic, and Azure Logic App :**

![image-33.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797246612/O2zh22k0_.png align="left")

1. Sign in to the Azure Account and then first create a Storage Account if not created already. Use this link and createt the same [https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal)
    
2. Once Created , Storage Account Overview page will look like the below .
    

![image-5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797255253/AKEvujDtP.png align="left")

3\. Next Step is to create a Blob Container where the Original Message will be uploaded by the Client . You can create it by clicking on the blob service tab and then you will have create container button . I already created eventcontainer for the same.

![image-6.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797264577/Z6i6CZx7U.png align="left")

4\. Now that we have a Blob Container where the file gets uploaded, We need to set up the Event Handling Mechanism to poll these messages from the blob container whenever a new message is uploaded.

5\. We have Azure Storage events that allow applications to react to events, such as the creation and deletion of blobs. We do not need complicated code or expensive and inefficient polling services. The best part is you only pay for what you use.

6\. If you are using the Event Grids for the first time in your subscription ,we have to verify if the Event Grid Resource Provider is registered. In order to check the same , please go to Subscription ---&gt; Settings -----&gt; Resource Providers and find Microsoft.EventGrid . It should be in Registered Status , if not Register it.

![image-7.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797276350/83jOHeM9M.png align="left")

![image-8.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797285279/nihnk7G56.png align="left")

7\. Before we Subscribe to the events for the Blob Storage , We need to create a messaging endpoint for the event message. I created a Logic App with HTTP trigger as messaging endpoint to consume the event message . Please fidn the screenshot below .

![image-9.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797294116/ECWv-oulc.png align="left")

8\. Now that we have a messaging endpoint , Let us setup the Event Subscription Configuration by going to Events Section under Azure Storage Account where the Blob Container is Created.

![image-11.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797306256/ya6suW_UU.png align="left")

9\. Click on the Event Subscriptions option , On the Event Subscription Page do the following steps.

a) Enter Name for the Event Subscription .

b) Enter a name for the system topic

![image-12.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797321381/Nuvg2w0-b.png align="left")

c) Once after you provide Event Name and System Topic Name in this page , you have Event Type Configuration Filter , Select Blob Created Event under that like mentioned below .

![image-13.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797331290/L-vY-xFgD.png align="left")

d) Under Endpoint Details, Select the Endpoint type as Webhook and copy the http request trigger URL from the Logic App Created in Step 6 and paste it under Subscriber Endpoint URL.

![image-14.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797341064/lkOXk-L4V.png align="left")

![image-15.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797351590/7C3GEq77p.png align="left")

![image-16.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797362958/14EBoEflG.png align="left")

e) Now go to filters tab under Event Subscription page, Under Filters, we need to enable Subject Filtering and Add the below filter under Subject begins with Property.

```plaintext
/blobServices/default/containers/eventcontainer/
```

General Syntax is : **/blobServices/default/containers/{blobcontainername}**

f) This filter is setup to send blob storage creation event information to the Azure System Event Grid only when the message is uploaded on the blob container named "Event Container" . In Practical Scenarios too , We usually expose a blob container to the Client using SAS URI to dump their messages. If we do not setup this Subject Filter , Event Grid will receive the message for any blob creates happening on any container in ths storage account.

![image-17.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797374714/hag3hq0Zs.png align="left")

Please be noted that Subject Ends with is not configured to any value . ".jpg" is actually blurred and it is a reference for the developer to indicate that this property is for filtering specific file extensions . I didn't add any specific value for this property.

10\. Once after all the configurations are finished , this is how the Event Subscription would look .

![image-18.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797384328/q0CR92Y9A.png align="left")

![image-19.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797393098/JcwKBMIy8.png align="left")

Filters Section :

![image-20.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797403013/t95_aJqNs.png align="left")

1. Now we are all set to test the pattern, upload a file into the blob container you created earlier. I am using a small file as this is a Proof of Concept. Ideally, this pattern is for dealing with large messages.
    

![image-21.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797412788/agXetY3EY.png align="left")

1. The Moment you dump a file into the blob container, it is treated as an Event and this Blob Creation Metadata flows to the System Event Grid and then to the Event Subscription.
    

![image-23.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797424077/3go3LTx-5.png align="left")

![image-24.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797435918/Hh8zCSWjQ.png align="left")

1. We can see the message is successfully picked up by the Event Subscription and it is delivered to the message endpoint i.e Azure Logic app. Let us go to Azure Logic App Run history for the confirmation.
    

![image-25.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797446437/5_u59QSoH.png align="left")

1. We can see from the run that Blob Metadata is received on the Azure Logic App Successfully.
    

![image-26.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797458077/8EDox75wj.png align="left")

Blob Event Data Received on the Logic App :

![image-28.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797472914/mvcwOr3on.png align="left")

1. Key Details like a **topic**, event subject filter, event type, **and blob uri** details are received from the Event Subscription to Logic App.
    
2. Now using **url attribute which contains blob container details**, we can easily retrieve the original message by connecting to the blob storage connector. So this step is added in the logic app.
    
3. Blob Content is retrieved using Blob Storage Connector.
    

![image-29.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797486101/9-nIv12GA.png align="left")

1. Finally a Compose Action is added to retrieve the Original Message of the Blob by Converting the application/octet-stream content to JSON using the below simple expression.
    

```plaintext
json(body('Get_blob_content_using_path'))
```

![image-30.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797495924/cA0Uir7gm.png align="left")

**Conclusion :**

With this, Practical implementation of a claim check pattern in Microsoft Azure is demonstrated using one of the approaches successfully.

**References :**

[https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview)

[https://docs.microsoft.com/en-us/azure/event-grid/blob-event-quickstart-portal?toc=/azure/storage/blobs/toc.json](https://docs.microsoft.com/en-us/azure/event-grid/blob-event-quickstart-portal?toc=/azure/storage/blobs/toc.json)