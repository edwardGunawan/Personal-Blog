---
layout: post
title: Database vs Stream Processing - What it means for the Future of Data Management
date: 2021-02-04 23:22
summary: As Marc Anderson said, "Software is eating the world."
categories: distributed-systems architecture stream-processing
tags: distributed-systems architecture stream-processing
---


![infoQ](https://res.infoq.com/articles/whats-the-next-step-for-data-management/en/resources/35image001-1605277240254.jpg)
When we think about data, we often think that data give human the insight to make better decisions. Therefore, an application such as a database exists to interact with the UI application to know the current state.

However, data can help us make better decisions and help other software make better decisions, thus removing humans from the system's critical path.

Think about Netflix - their business started as a rental DVD service, you go to Netflix to rent a DVD, and they will send the DVD via postal service. In 2021, Netflix has become a pure software platform that lets users watch any movies with a click of a button.

Airbnb initially started as a website like Craiglist without a payment provider - you connect with the host via the website, and you physically paid the host afterward. In 2021, Airbnb has become a pure software platform that lets users rent any house and experience with a click of a button.

Businesses are becoming more automated - the business process becomes more like an autopilot.

We also started to see many applications where the user-interface is the smaller part of the whole package. We see companies create software to serve our end-user, human, and develop systems to talk to another system.

When we think about data management systems, there are two things: stream processing and database. Understanding the difference between the two can help us know how we can create better and efficient systems for ourselves.

## Active and Passive Data
Database and stream processing are similar in that they are containers that store data. However, they are different in how their data is moving.

You will use Cassandra if you want to store a vast amount of data for the amount of memory that the database is accumulated. If you're going to store data that contains relationships between entities, you can use a graph database like Neo4J; if you want to have an active query functionality, you can use elastic search.

This database provides answers to our questions about data. They are built to interact with the user that contains a UI. Therefore, the data that stores within this database is passive - it waits for you to do some processing on them, and the query that we do on them are active. If we - humans, stop interacting with the database at a specific time, it won't do anything.

On the other hand, Stream processing is asynchronous. Their end-user are not us - human - but also other software. They also store data just like the database does, but the data is active. The action that is usually triggered is based on the information itself. They help us automate the process within our system. In other words, the query (action) is passively waiting to be triggered.


## Where you have been vs. Where you are now
The database helps us answers the questions regarding where we are at the current state of our application. A database table represents the current state at a single point in time. You have no idea what is happening previously. An analogy for English grammar will be a database table is more like a noun. It helps tell you where you currently are.


On the other hand, an event stream is more like a verb. Event streams answer the history of what happened to the state. It records the move in your phone, the hover of your mouse, or the progress of your order. It records every single action into an event that can be replay back in time. You get more information regarding what happens. If it is a chess game, you can know whether the opponent's move is a smart move or not.


## Mutability vs. Immutability
We can think of Stream processing and Database table act as the same purpose but with different properties. An event stream is a special kind of table that is immutable - you cannot change it. The database table is usually mutable where it enables us to insert, delete and update its elements.

Each event in the event stream is like a fact. If you order an item and you want to cancel it. The system will append the cancelation state on top of the order state - you can rewind your history to see what you have done. It is like a ledger. One of the benefits of append-only is because the data that we interact with are moving fast. By the time you want to update the current state, another service has already consumed it and does some action. Therefore, it needs to create a compensating events chain to propagate all those actions to its subscribers.

On the other hand, the database will update the state from order to cancel. You can edit the row on where the data is moving. It gives a source of truth to the system on the current state of the application. 

Database and Event Stream serves applications that serve a different purpose with a diverse audience. For Databases, we, the human, are its end users. Therefore, it makes sense to design a database as synchronous, passive data and mark it as a single truth source. For Event Streams, the end-user is not humans - they are software. If the end-user is not human, we can leverage asynchronicity to optimize our resources. The data becomes the ones that trigger other software to do specific actions. 

Although event stream and database behave in different use-cases, the team doesn't need to adopt either one or the other. A lot more applications need both stream and database to gather better insights into its internal systems. This means that we see it a lot more in modern applications where we need to handle synchronous and asynchronous.  Hence, more enterprise applications use the hybrid approach of the stream-oriented database. There are several active-database such as MongoDB, CouchbaseDB, and RethinkDB. There are also stream processors that can serve as a database table, such as Flink and Hazelcast Jet. No matter which hybrid approach you are using for your applications, knowing the difference can help us understand what it means to have either one of our systems' architecture.


## Resource
- [Beyond the Database, and beyond the Stream Processor: What's the Next Step for Data Management?](https://www.infoq.com/articles/whats-the-next-step-for-data-management/)
- [Video Version](https://www.youtube.com/watch?v=wTI6zj9Xbn4)


