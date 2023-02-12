# Publisher - Subscriber Pattern in Azure - Part 2

\*\*Introduction : \*\*

In part 1, we saw the publisher-subscriber pattern in Azure, useful scenarios in which the pattern is implemented, and different approaches to implementing the pattern in azure.

In this blog post, I will be implementing the pattern using one of the approaches.

\*\*Prerequisites: \*\*

1. Please go through [https://exploreazurecloud.com/publisher-subscriber-pattern-in-azure-part-1](Link) before you read this blog post for a better understanding.
    
2. If you want to try implementing this pattern practically, You need to have an Azure subscription, create an Azure Free Account using this link [https://azure.microsoft.com/en-in/free/?ref=microsoft.com&utm\_source=microsoft.com&utm\_medium=docs&utm\_campaign=visualstudio](Link)
    

\*\*Practical Implementation of the Pattern using Azure Logic Apps and Azure Service Bus: \*\*

We discussed different messaging brokers while discussing the pattern in the earlier post. However, for demo purposes, I will be choosing Azure service bus as the message broker and azure logic app as the integration component.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642831154217/S_p6L2GHN.png align="left")

For simplicity purposes, let's not consider any data transformation in the publish logic app i.e you will have an HTTP trigger receiving the message and pushing the message to a service bus.

\*\*Publish Logic App: \*\*

1. For publishing the message on the service bus, you need to create a new service bus namespace if not created already and then add a queue under the same. Use this link to create the Service Bus Namespace and the Queue. [https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-quickstart-portal](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-quickstart-portal)
    
2. Once Created, you will have a service bus created with a queue like below.
    

![image-9.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627797816075/4b29WhaTS.png align="left")

3 . Now that we have the service bus created, the publish logic app is created like below with the request message sent to the created service bus queue. In order to simplify things, we do not have any data transformations.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653718758331/BpLlDuAvU.png align="left")

\*\*Subscribe Logic App: \*\*

1\. Subscribe Logic App contains a service bus trigger in peek-lock mode, which peeks at the messages coming on the service bus queue.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653721143470/ucvhAlnQ0.png align="left")

2\. It is always best practice to complete the message of a queue in a peek-lock mode at the beginning itself. If you have the complete message in a queue action in the end, if your logic app fails intermittently in the middle due to some action failure and if the service bus lock token duration expires, it will lead to multiple processing of the same messages on the queue. We will not delve deeper into this concept here as it is out of context. Please refer to this excellent article explaining how one can reliably process messages from the service bus with logic apps by completing the message in the beginning and also handling resubmit scenarios gracefully.

[https://yourazurecoach.com/2020/11/05/reliably-processing-messages-from-service-bus-with-logic-apps/](Link)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653721220351/0h-H8XVzu.png align="left")

3\. As the intention of the blog is to explain the publish-subscribe pattern practically, the remaining process of the logic app is kept simple.

4\. Once the service bus trigger reads the message, the read message which is in base64encoded format is decoded and parsed to the original JSON message. This is then passed as input to the dummy api i.e [https://reqres.in/](Link)

```plaintext
json(base64ToString(triggerBody()?['ContentData']))
```

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653720814226/BwCodOOWF.png align="left")

5.\\ Then the API response is sent as input to another logic app.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653723719050/qhV-3JF9p.png align="left")

\*\*Conclusion : \*\*

With this, the publisher-subscriber pattern scenario is practically implemented using Logic Apps and Service Bus where a system A ( Publish Logic App) publishes a message on the Service Bus Queue and a system B consumes the message and processes it further.

Even the event-driven architecture is built on the publish-subscribe model and with the Azure Resources / Custom Events acting as publishers, Event Grid acting as a message broker to send these events to multiple consumers i.e Azure Function, Azure Logic app endpoints, Service Bus Queues, Webhooks, etc. This approach is also discussed in [https://exploreazurecloud.com/publisher-subscriber-pattern-in-azure-part-1](Link) , one can go through that and practically implement the same.