# Publisher-Subscriber Pattern in Azure - Part 1

**Introduction : **

   Publisher-Subscriber pattern commonly called the pub-sub pattern is one of the most popular messaging paradigms where we enable applications to announce events to multiple interested consumers asynchronously, without coupling the senders to the receivers. 

  A typical publish-subscribe channel has one input channel that splits into multiple output channels, one for each subscriber. When an event/message is published into the channel, it delivers a copy of the message to each of the output channels.  A message broker typically takes care of sending the events or messages to output channels. 

Within Azure, we have multiple message brokers with Azure Service Bus, Azure Event Grid, Azure Event Hubs, and Apache Kafka connector being the most widely used. 

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640351572829/IWK9efM2A.png)


**Context and Problem:**

 While working with cloud-based solutions and distributed systems, we often need to send messages/events to the other components or the downstream systems. We want these communications to be asynchronous so that the message sender doesn't wait for any response especially when the message needs to be consumed by different subscribers. 

 Introducing a specific message queue channel per each consumer will not help in scaling the solution and the consumer might be interested in all the messages published but specific messages only.

**Solution:**

In a nutshell, we need to decouple the senders from receivers/subscribers by introducing an asynchronous messaging layer. The sender can announce the events to the interested consumers without knowing their identities.

The answer for this is the **Publisher-Subscriber pattern**.

Within ***Microsoft Azure***, 

- If we are dealing with messages i.e packet of data, then we can use **Azure Service Bus** to set up the asynchronous messaging subsystem.


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642319393658/41cwiXznli.png)

- It is a message that notifies something about a change or an action it is treated as an Event and we use **Azure Event Grid** for such scenarios


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642826940003/jIBlMlNwV.png)



Let's try to evaluate some scenarios where the pub-sub pattern can be useful.

***Scenario 1 : ***

While we design and build cloud-based solutions, we come across some scenarios where we receive some events/ messages from a system and those messages need to be sent to multiple target systems/consumers in real-time or near real-time. We may have to check the metadata or based on some of the content in the message we need to message to different consumers using a single message channel. In such cases, the pub-sub pattern will be of great help.

***Scenario 2 : ***

one of the main pillars in an Event-Driven Architecture is a pub-sub pattern. The event-driven architecture will have event producers who send events which in turn are listened to by the event subscribers. The messaging infrastructure associated with an event grid takes care of the subscriptions. After an event is published, it is sent to each subscriber. Once an event is received, it cannot be replayed i.e a new subscriber will receive the upcoming messages but not the old ones which other subscribers have received.

**Approaches to implementing this pattern in Azure:**

***Approach 1: ***

One can implement a pub-sub pattern using a combination of Azure Logic Apps and Service Bus Queues / Topics 


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642831154217/S_p6L2GHN.png)


***Approach 2: ***

We can also implement a pub-sub pattern using a combination of Azure Functions and Service Bus Queues/ Topics.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642831336714/-1XiuDfI3.png)

***Approach 3: ***

If we are dealing with event-driven architecture, one can build integrations using a combination of Azure Event Grid Topics with Event Subscriptions, Logic Apps, Functions thereby leveraging the pub-sub pattern.   


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642832079859/ihpUVFK9p.png)


**Conclusion : **

In the next part, I will be practically implementing the pattern using one of the approaches mentioned above, will also explain some benefits and design considerations.

**References: **

 
- [https://docs.microsoft.com/en-us/azure/architecture/patterns/publisher-subscriber
](Link) 

 
- [https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven](Link) 


- ***Book Reference***: Enterprise Integration Patterns by Gregor Hohpe and Bobby Woolf 




