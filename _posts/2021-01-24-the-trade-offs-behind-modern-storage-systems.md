---
layout: post
title: The Trade offs Behind Modern Storage Systems
date: 2021-01-24 23:18
summary: The algorithms behind modern storage systems in the lense of RUM conjecture
categories: distributed-system
tags: distributed-system
---

<img src="{{site.baseurl}}/images/the-trade-offs-behind-modern-storage-systems/RUM Conjecture.png" alt="Rum Conjecture">

The database has always been the center of an application. If you are building a web application, the database is a critical component of your application. You need to render the data from storage. You need to have a storage engine that keeps a single source of truth in your application state. With the vast amount of data that we can consume on the internet, creating an efficient algorithm behind this data storage is essential. 

In 1970, the database researchers optimized database algorithm to minimize random access on the disk. Fifty years later, we want to optimize the storage engine on disk and reduce the random access on memory. Therefore, to create a great user experience and more accurate data prediction, we want to optimize our storage engine based on specific criteria.

What are the criteria that we usually optimized our storage engine? 

As I started my distributed systems journey, I saw a paper called the RUM Conjecture. The article states that we usually want to optimize our algorithm and data structure for the storage engine in Read Optimized, Write (Update) Optimized, and Memory Optimized. 

While you want to have a perfect solution for Read, Update, and Memory efficiency in your storage engine, we cannot have all three. We will need to compromise our design. If we want to have Read optimized storage engine, we sacrifice update and memory. This conjecture is similar to the famous CAP Theorem. 

In this article, I want to explain more about RUM (Read, Update, Memory) overhead and some examples of data storage optimized on a certain one of the three.

## Read Overhead
Read overhead is the total amount of data to read your primary data. If the location you store your data is random, you need to do more operations to find the data. If you want to optimize on the Read, you will need to create an index of auxiliary data to have fast retrieval. This auxiliary data is what causes the read overhead. 

If you create a QuadTree to do a faster read on the nearby location, the total amount of data that you read will be all quads that you traverse and nearby data that you get from the database. You need to read all these data to get _that_ nearby location. The read overhead can be measured by the ratio of all the data over the primary data you want to search.

## Update Overhead
Update overhead is the amount of access required to update your primary data. An update overhead occurs when a storage engine needs to perform write on any other data location to update your preliminary data fully. For instance, if you want to do an update on a database that contains an index, you need to do an update on the index alongside your database.

## Memory Overhead
Memory overhead is the amount of memory occupied when storing your primary data. It includes needing to store auxiliary data alongside the main ones. It is often the tradeoffs in data structure between speed and memory. You will occupy memory as a result of optimizing for faster Read and write. The memory you occupied is an additional memory for storing the primary data.

Let's think about some scenarios of why we can't optimize all three simultaneously. To better illustrate the compromise between the three, let's check out systems and hypotheses optimized on all three spectrums.

## Read Optimized
The Read optimized usually performed low read overhead. An example will be a Hash-based index. You can create a hash-based index on top of your storage engine to create a faster read. For reading, you need to call a hash function to locate the primary data. However, you will also need to update your primary data and the hash function when it comes to updating. Sometimes, you have to repartition your hash function to prevent any collisions. In this case, the update will suffer. Suppose you are accounting for collisions in your hash-based function. In that case, you will need to create overhead for your memory to store additional empty block to decrease the probability of collision. In this case, your memory suffers too.

Some examples of read optimized storage system are hash-based index for constant read access and B-Trees and Skip lists for logarithmic time access.


## Update Optimized
Optimizing on updates meaning, you want to make sure the update overhead is as low as possible. To create a low update overhead, we can store the updated data in some conditional auxiliary space like a buffer that will eventually flush into the bulk data. We will append every update, effectively forming an ever-increasing log. 

Although we minimize the update overhead, we increase the read overhead and the memory to store the buffer. If we want to read our data, we may need to do multiple merges to get the data's current state. We also need to re-organize our data each time the buffer is flushed to the main storage. If we re-organize our data to minimize the read overhead, we are increasing our update overhead.

Some examples for update optimized systems are [LSM Tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree), Partitioned B Trees, and FD Tree.

## Memory Optimized
Like Read and Update optimized, memory-optimized try to decrease the memory overhead by minimizing the memory required to access and update the primary data. The system usually tries to optimize them by compressing the data or storing the data in a dense array. To do a read, we need to scan through all the values or decompress the data. To do an update, we need to decompress and update the data.

Some examples of memory optimize applications are lossy index structures like [Bloom Filters](https://en.wikipedia.org/wiki/Bloom_filter), Count-min sketches, lossy encodings, and [Bitmaps](https://en.wikipedia.org/wiki/Bitmap).


If we want to optimizes on all three, is that mean we are doomed? Application is becoming more complex, and the demand for accessing, updating, and storing data is higher than before.

## Building RUM Access Methods
Although the tradeoffs exist, many storage engine applications try to be creative by balancing the RUM tradeoffs with a change in hardware, workload patterns, and applications.

Therefore, data management techniques variants that can create a dynamic way of optimizing access method. For instance, we can build a multi-access strategy by navigating partly in the RUM space and covering the whole space in aggregate. We can also tune to reduce the RUM overheads depending on the data access pattern.

## Conclusion
Data storage usually are optimized based on Read, update, and memory. You can use an optimized data structure based on specific properties—for instance, using LSM Tree to optimize update operation and B-Tree for a read operation. However, you cannot combine these three properties to create a data structure equal to every RUM aspect and every single overhead. 

Now that you understand RUM overhead, you can use this knowledge to evaluate the databases that are best suitable for your application and your workload at the moment. The RUM Conjecture gives you a basic rule of thumb to assess the database's overhead that you will use for your new applications and its alternatives.

I hope you find the explanation useful! If you have any other questions, feel free to comment below so that others can also learn from them. If you have any comments and other new findings, please also mention them here to learn from you!

## Resource
There are some great resources about RUM Conjecture. We are barely scratching the surface on the algorithm behind modern storage systems. If you are curious to learn more, check out the resources below!
- [RUM Conjecture](https://stratos.seas.harvard.edu/files/stratos/files/rum.pdf)
- [Algorithms behind Modern Storage Systems - YouTube](https://www.youtube.com/watch?v=wxcCHvQeZ-U)
