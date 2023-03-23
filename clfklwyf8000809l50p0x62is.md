---
title: "Pipes and Filters Pattern in Azure - Part  1"
datePublished: Thu Mar 23 2023 04:21:54 GMT+0000 (Coordinated Universal Time)
cuid: clfklwyf8000809l50p0x62is
slug: pipes-and-filters-pattern-in-azure-part-1
tags: design-patterns, azure, cloud-native, integration-patterns

---

## Introduction

Pipes and Filters is one of the popular enterprise integration and cloud design patterns where in we divide a larger processing task into a sequence of smaller, independent processing steps (filters) that are connected by channels (pipes).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679288001425/66101676-7266-4ef7-b2ef-bba94d047b0a.png align="left")

Decomposing the task into smaller steps improves performance, scalability and reusability thereby allowing the smaller tasks to be scaled and deployed independently.

In Azure, we can utilize native message brokers like Azure Service Bus & Azure Event Grid as Channels and Azure Logic Apps/ Azure Functions to design an application using the Pipes-Filters pattern.

## Context and Problem

Let's assume we have an order processing application, this application can contain a variety of tasks with varying complexity while processing the information. One way of implementing this application is in a monolithic way where we place all the functions/tasks inside a single component reducing the scope for reuse and increasing the complexity of refactoring and code optimization.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679376906216/ecd5f5a1-5320-4ca2-b3de-050e7f850d9e.png align="center")

While processing the data in such applications, some of the tasks can become compute intensive which is demanded to be run on better hardware whereas some tasks are quite the opposite in terms of compute capacity. Order processing sequence might also change in the future.

We need to have a solution to address these problems and increase the possibilities for code reuse and independent scalability of the tasks.

## Solution

Pipes and Filters design pattern can be implemented to address the above problems by decomposing the tasks within a monolith into a set of separate components with each performing a single task.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679467196093/b739d045-c51f-4c4a-9df4-4d274610ec94.png align="center")

These tasks ( filters ) are connected through channels (pipes) providing the much-needed flexibility to spread the load and increase the throughput due to the opportunity of independent scalability. Below is a diagram depicting a few resource-intensive tasks in the pipes and filter setup. These filters being resource intensive can run on different machines taking advantage of scalability and elasticity in the process.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679467418115/3c8cc048-63ae-40ff-a2fc-6d7529fc28be.png align="center")

### Benefits

* Decomposability
    
* Scalability
    
* Reusability
    
* Elasticity
    

### Issues and Considerations

* Increased flexibility also introduces complexity especially if some filters are running on distributed servers. So, one needs to be cautious while implementing such filters.
    
* Messaging Channels Infrastructure ( Brokers) should be reliable as they are responsible for transmitting the data between different filters.
    
* If a filter in a pipeline fails after posting the message to the next filter, there is a high chance that another instance of the filter will run and post the same message leading to duplication of messages. To avoid this, the pipeline should be capable of eliminating and detecting duplicate messages. For Instance, we can enable duplicate detection property on Azure Service Bus to detect and remove duplicate messages.
    

### Scenarios to Implement the Pattern

* When you want the application processing to be broken down into a set of independent processing steps.
    
* A few steps with the application processing need specific scalability requirements
    
* Capability to add or remove steps within a processing application pipeline.
    

### Implementation Approaches using Microsoft Azure

Within Microsoft Azure, the pattern can be implemented using native message brokers such as Azure Event Grid / Azure Service Bus acting as Pipes ( Channels) and Azure Functions / Azure Logic Apps used as Filters.

**Approach 1 :**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679469670918/83b701da-5b99-42e9-8873-8063935c2a58.png align="center")

* An incoming message is sent on an Azure Service Bus (Pipe) and is consumed by an Azure Function ( Filter) which performs some business logic on the message ( API call from the message payload and some lookups).
    
* Once processed, Azure Function sends the new/enriched message to Service Bus (Pipe). Azure Logic App ( Filter) consumes the message from Service Bus, performs some data transformation and generates the fully transformed message.
    

**Approach 2 :**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679469731420/57c92684-7cfc-4037-8416-0b078bdf9c55.png align="center")

* A publisher pushes notification messages to an Azure Event Grid ( Pipe). Azure Event Grid Topic has a set of Event Grid Subscriptions with Subscribing Endpoints ( Logic app / Function app endpoint URL).
    
* Let's assume a message N1 is picked up by an Event Grid Topic Subscription, this topic subscription sends the message to a logic app which is a subscribing endpoint and this logic app then performs some business logic ( Might contain an orchestration logic to connect to different systems and gather data), sends the output to Azure Service Bus ( Pipe). Then there is a final filter logic app that takes this data and transforms the message to a target format. The final message is then sent to the destination (which can be an API or Message Broker )
    

## Conclusion

In the second part, I will be implementing the pipes and filter pattern practically through a use case.

### References:

[Pipes and Filters pattern - Azure Architecture Center | Microsoft Learn](https://learn.microsoft.com/en-us/azure/architecture/patterns/pipes-and-filters)

[Pipes and Filters - Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PipesAndFilters.html)

**Book Reference**: Enterprise Integration Patterns by Gregor Hohpe and Bobby Woolf