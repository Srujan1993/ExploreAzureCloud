# Implement Claim Check Pattern in Azure - Part 1

**Introduction :**

Claim Check Pattern is one of the most commonly used Enterprise Integration Patterns. Before we jump into the design pattern, let us try to understand what is claim check-in simple terminology.

Claim Check describes a mechanism where you store the original data in a well-known place while maintaining only a pointer to where the data is located.

**Context :**

Whenever we are working with Messaging Based Architectures, we come across scenarios wherein we need to receive, send and manipulate messages of huge size . Messages can be images, pdf, or chat history text documents.  Sending Such Large Messages directly to the message bus leads to slowing down the solution. Messaging Platforms are usually designed to handle smaller chunks of messages, so we have to come up with a workaround to handle messages of bigger size and that's where Claim Check Pattern Comes in Handy.

**Solution :**

Store the Message Payload to an external database, get the reference of the payload location in the database and send it to a Message Bus. This acts like a Claim Check as original data is stored in a place and the pointer of the data is maintained in the Message Bus, hence the pattern is named as Claim Check Pattern. Clients interested in receiving the message payload can subscribe to the message bus to retrieve the payload reference and access the payload accordingly.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627796672053/9ZdOkLNwB.png)

**Ways of Implementing Claim Check Pattern in Azure :**

This pattern can be implemented in several ways in Azure and using different technologies . However I will try to discuss about the approaches using the combination of Azure Storage Blob and Azure Event Grid .

**Automatic Claim Check Generation using Azure Storage Blob , Azure Event Grid , Azure Storage Queue and Azure Function :**

**Approach 1 :**

Sender drops the message payload into Azure Blob Storage Container and then Event Grid automatically generates a tag/reference and sends it to a supported message bus like Azure Storage Queue. The receiver application polls the Azure Storage Queue to get the message and then uses this stored reference data to download the payload message from Azure Storage Blob. Here the receiver application can be an Azure Logic App with a Queue Trigger or else an Azure Function with Queue Trigger.


![image-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627796730563/_L_VPKSzE.png)


![image-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627796766713/d91Ro22tb.png)

**Approach 2 :**

In Approach 1 , Event Grid was pushing the blob reference message to Storage Queue Endpoint . Instead Azure Function with Azure Event Grid Trigger can directly get the blob reference message there by taking advantage of Serverless nature of Azure Event Grid and Azure Functions.


![image-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627796789394/Krrd56ti4.png)

Approach 3 :

In this Approach , We use Azure Storage Events Which allow applications react to events whenever there is a creation or deletion of blobs . In this Scenario , We setup a System Event Grid Topic which reacts to Azure Blob Storage Create Events pointing to Blob Container where Client dumps the message and then Event Grid topic Subscription sends the message to LogicApp Endpoint exposed as Webhook.


![image-32.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627796852372/4qgSf4SXd.png)

**Manual Claim Check Generation using Azure Storage Blob**, **Azure Service Bus**, **and Azure Functions / Azure Logic Apps :**

The sender Application will upload the original message to Azure Blob Storage and sends the blob storage location details to Azure Service Bus Queue. Receiver Application polls the Service Bus Queue using Azure Functions or Azure Logic apps each time a new message is uploaded to blob storage by the sender and will access the original message using the location details send in the Queue.

**Design Considerations :**

1.  Make sure you have a condition to skip this pattern for smaller messages from the client and apply this pattern if and only if your message size is large so that we can reduce overhead and latency for smaller message sizes.
2.  Consider deleting the message data after it is consumed if you no longer need it as it might cost you some money in the long run if you are storing large messages in a blob .
3.  Consider having a logic to archive the orginal message if there are any failures in the midst of the pattern and you might have to correct the failure and resubmit the message. In such cases , move the original blob to a error blob.

In the Second Part, I will demonstrate a practical implementation of the Pattern using one of the approaches mentioned above.

**References :**

[https://docs.microsoft.com/en-us/azure/architecture/patterns/claim-check](https://docs.microsoft.com/en-us/azure/architecture/patterns/claim-check)

[https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html](https://www.enterpriseintegrationpatterns.com/patterns/messaging/StoreInLibrary.html)