---
layout: post
title: Three Use Case When You Don't Need Asynchronous Communication In Your Microservice
date: 2021-04-07 23:27
summary: Cross off these three scenario before you want to use asynchronous communication for your service
categories: best-practice distributed-system
tags: best-practice distributed-system
---

![Photo by 1 1](https://images.unsplash.com/photo-1579202645762-5de0cd37c52f?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=668&q=80)

Asynchronous communication becomes widely used in processes and systems - especially in Microservices. You see, people blog about how they can make certain operations asynchronous. All the newest web application frameworks and new programming languages are created to make asynchronous programming much easier to reason about. There are tons of great benefits of implementing asynchronous communication - your program may run faster, your client may be happier, and so on.

However, with great benefit and great hype, there is also a downside of doing asynchronous communication. One of the disadvantages is that it is very hard to debug when there is a problem. Your program becomes complex. 

For instance, adding a message queue to the system will require a dead letter queue to store failure messages. If your program needs to be in order, that will require more storage and other compute resources. Asynchronous communication at the code level can make the program hard to read. One example is the JavaScript callback hell. 

Then, there is another situation in which you should avoid asynchronous work because doing so can slow down your engineer development time.

In this article, I want to share three reasons you can cross before implementing asynchronous communication in your application.

## What is Asynchronous Communication
Before proceeding further, I want to describe asynchronous communication and patterns so that we are on the same page. In brief, asynchronous communication is when clients receive a delivery message right away when sending a request to the server. This delivery message doesn't have the response yet but just acknowledging that the server received it. 

The client can do other tasks and trust the server to receive a notification on the accurate response.

A couple of patterns are often used in asynchronous communication. The most widely known is the callback function. These callback functions can be implemented at the service level or at the programming level (callback event). Another widely-used communication pattern in microservice is message queues. Having a publisher-subscriber pattern and using a message broker to decouple the communication between two services.


## Updating a Process that is  Dependent Upon Another
It is a bad idea to use asynchronous communication to update data dependent upon another because your data can be out of sync. 

Imagine that you have an update field in your applications that can update the user information on an e-commerce site. Doing the operations requires you to update user service and order service, and that ordered service depends on user service. 

In this case, the synchronous computation may be preferred since it is much easier to reason when one service goes wrong. 

If you use asynchronous communication, you will need to account for rolling back operations. Moreover, you also have to deal with data consistency within your application. Therefore, depending on your application's use case, this may be a good idea to be synchronous instead of asynchronous.

## Your Service doesn't Have Any Heavy Computation Resources
Does your system do any heavy computations such as compression, heavy recursion, or heavy encoding in memory?

You don't need a message queue between two services if one service wants to get an attribute in the database from another service. 

Usually, the first load on a web application may require an extended amount of data from the server to render the web. However, there is no need to implement service callback operations for a simple fetch to get the user by its Id. At this point, you are over-optimizing your application.

We often choose the asynchronous way of doing simple computations. For example, we often try to optimize loops by making them asynchronous to avoid blocking. 

However, asynchronous loops are necessary when there are many iterations or complex operations within the loop. When you are iterating a simple task, such as looping through an array, there is no reason to overcomplicate things using a complex recursive function.

One tip to know if you need to leverage async on your computations when you know that it is so slow that it can be a problem for your entire system. One way to test that is by doing a performance test on your application. When you realized certain operations are taking up a lot more time, the first resort is caching, then if caching doesn't help, you can use async as a last resort. One of the reasons why async is the last resort is because async creates complexity in the system and when bugs are not obvious. Therefore, by optimizing the performance of using async, you also create a hidden tax of complexity.


## No Strict SLA
Even if an application has many CPU bonds but doesn't have strict SLA, using asynchronous communication through services just overcomplicate things. 

For instance, it is okay to use synchronous calls for an application that does batch processing for an ETL pipeline that doesn't have a strict response time. Doing async IO call in these situations will result in pre-mature optimization and may over-complicate the system.

A rule of thumb here is that if efficiency is not a requirement, you almost always want to go for simplicity. Simplicity doesn't mean it is easy. To make it simple, you may need to do a lot of work. That work may include creating abstraction to make the application maintainable.


## Closing
Overly optimize applications can create complexity. Therefore, only try to optimize if it is necessary. 

If you can get anything from this article, it is this - use asynchronous communication when necessary. Always think about simplicity. For instance, there are many ways to write Fizz Buzz, and you can make the algorithm really complex and really performant. However, if the application's intent is to call "Fizz" on the number divisible by three and "Buzz" on the number that is divisible by five, you don't need to use use a callback to implement that algorithm. It doesn't need to have a nanosecond speed of performance.

The same goes for implementing your microservices. A message queue is necessary if it can solve the bottleneck in the system. Some new technology or paradigm was created to solve specific problems. If the system doesn't have that problem, it creates complexity that will hurt us in the long run. Hopefully, these checklists can help you identify the right decisions for your application.

## Resources
- [Asynchronous Communication — Methods and Strategies - DZone Performance](https://dzone.com/articles/asynchronous-communication-methods-and-strategies)
- [When to Use (and Not to Use) Asynchronous Programming: 20 Pros Reveal the Best Use Cases – Stackify](https://stackify.com/when-to-use-asynchronous-programming/)
