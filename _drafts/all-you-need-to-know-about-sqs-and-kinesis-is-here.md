---
layout: post
title: All you need to know about SQS and Kinesis is Here
date: 2020-05-30 10:25
summary: Identify each resources can help you design a reliable system, and save your resources time and money
categories: cloud AWS devops
tags: cloud AWS devops
---

<img src="{{site.baseurl}}/images/all-you-need-to-know-about-sqs-and-kinesis-is-here/SQS vs Kinesis.png" alt="SQS vs Kinesis">

When designing a reliable distributed system or a message system, the first things that come to mind when using AWS resources are SQS and Kinesis.

At first glance, Kinesis has a much powerful feature than SQS. One thing about Kinesis is that it can handle a large volume of data; you can replay messages, or have multiple consumers that are subscribing to your Kinesis Stream. However, Kinesis is also a costly tool, there are a lot of learning curves in learning about developing in Kinesis. It also depends on your applications, sometimes you don't need a massive lifting tool like Kinesis in your use, and SQS will work perfectly fine.

In this article, I would like to discuss Kinesis and SQS, showing you the benefits and drawbacks of Kinesis and SQS. Lastly, I will dive deep into the difference between the two systems and the best use cases for both services.

## Kinesis
Before there were stream data pipeline or Kinesis, the only way to handle big data application is using [Batch Processing](https://www.7wdata.be/business-analytics/batch-vs-real-time-data-processing/). 

The producers will aggregate all the file, put it into a Storage file, or Simple Storage (S3). Then, the consumer will do a batch processing by having a Cron job, i.e., once every day, to process each new batch of data.

There is a problem with this sort of big data operation. Using batch processing, you are not able to process data in real-time. Applications that require real-time analytics, such as the stock-market app, fraudulent detection, or even a clickstream, will always process data in the past.

Kinesis stream is a streaming pipeline that helps create real-time data streaming processing in your applications.

## Benefit of Kinesis Stream

### Multiple Consumers 
Once the producers publish new data to the pipeline, it can be consumed from various consumers immediately. 

It provides an ordering of the records (data) and the ability to read or replay the same order to multiple consumers.

Each consumer will have its checkpoint in the Kinesis iterator shards that keeps track of where they consume the data.


There is a feature, enhanced fan-out, where each consumer can receive its own 2MB/second pipe of reading throughput. 


### Real Time
Kinesis has the maximum throughput for data ingestion or processing. When producers publish the data to the Kinesis Stream, the consumer can often get the data in less than a second - regardless of how much data you write.

### Big Data
If you need to handle terabytes of data per day in a single stream, Kinesis can do that for you. You can push data from many data producers and store for later processing or real-time.

Note that Kinesis stores record for 24 hours by default, up to 168 hours.


## Drawback of Kinesis Stream

### Shard Management
You need to manage your shard for your data. Knowing whether or not you need to increment or decrement the number of shards based on how many consumers you have.

### Complicated Consumer and Producer Library
For the maximum performance, Kinesis provides the consumer and producer library. As a producer library, you deploy the C++ binary with a Java interface for reading and writing data records to Kinesis Stream. To add a consumer library, you need to implement a Java application that can communicate with other languages. Either way, it needs some time invested in developing and maintaining the library.

### Maintaining State
Kinesis allows each of its consumers reads from the stream independently. It requires each consumer to manage their own iterator/ checkpoint in the stream and how far they have read and process. The consumer metadata will then be stored in DynamoDB if you have multiple consumer reading from a Kinesis Stream that you will need to manage each customer's checkpoint to read from each shard in the Kinesis Stream.

## Simple Queue Service (SQS)
SQS helps decouples your microservices, distributed systems. 

Using SQS, you can send, store, or receive messages between software components at any volume. You don't need to fear that there will be any data loss on the components or requiring the service always to be available. You can scale the system separately by having a queue as the middle layer to transfer data between your service. You can also make each of your services perform discrete functionality to improve the system's scalability and reliability.

## Benefit of SQS
### Easy to use
The AWS SDK for SQS is straightforward to use. You don't need to create a producer or client library to interact with SQS.

### Read Throughput
SQS can quickly scale a large volume of data without user intervention. It allows you to dynamically increase the read throughput based on the amount of task that is reading from the queue. SQS buffers requests to handle spike on load.


## Drawbacks of SQS
### Create Your Checkpoint 
Without and configuration, there is a delay time attribute and the visibility timeout attribute that AWS preset.

Delay time attribute is the delay time from when the message arrived in SQS to the message is visible to the consumer.

Visibility timeout is when the consumer reads the message, that message is not visible to the other consumer that is reading it until the visibility timeout expires. The message will be visible again to the consumer. 

SQS doesn't have multiple consumers like Kinesis Stream, and the message delivers behavior at least once, where consumers might receive duplicate messages. 

Therefore, it is a best practice to delete the message after you received it.

### Message Replayability
Because consumers usually will delete the message once it processed, there is no message replayability in SQS. 

In comparison, each consumer in Kinesis will have its checkpoint to point where the latest records they processed, SQS serves a single iterator for all of its consumers.

## Use Cases for SQS
SQS is used if you want to decouple your software components or allocate tasks to multiple workers.

If you have any operation that has a substantial background task, you can send the background task to SQS to removed the point of failure from that service. For instance, request that needs a cumbersome process in the background before writing to DDB. That service is not able to handle a tremendous amount of requests all at the same time. Therefore, by putting each application in the SQS, we can easily handle a significant spike of traffic and don't need to worry about data loss if that service is down. Furthermore, SQS has a dead letter queue functionality that can store the data in a DLQ if it fails to process for 14 days.

## Use Cases for Kinesis
Kinesis can help you stream data in real-time. You can put the data in the Kinesis pipeline and let numerous consumers read the data from the stream if you have multiple consumers who need to do analytics in the data.

Kinesis is best used for logging events data, real-time analytics, search functionality, or fraud detection - anything that multiple consumers need to process or process in real-time

## Summary
There are various amount of tools and resources when developing applications in the cloud. The caveat of designing your application is to understand and do research on what your application use cases and know which tools are suitable for what purposes.

Consider Kinesis if you are doing something with a substantial amount of data in real-time or need to have multiple consumers subscribe to that data. If not, consider SQS.