---
layout: post
title: How Do you Generate an Incremental Global Unique in a Distributed System
date: 2021-01-07 09:26
summary: Create a monotonically increasing unique function in a distributed system
categories: distributed-system programming architecture system-thinking
tags: distributed-system programming architecture system-thinking
---


<img src="{{site.baseurl}}/images/how-do-you-generate-an-incremental-global-unique-in-a-distributed-system/Global Counter.png" alt="Gloabl Counter Pic">

Have you ever wonder how you can implement `getUUID`? We often set random IDs for making an entry in a table. The most common way is to create a `UUID.randomUUID`. However, many programming languages and cloud infrastructure have provided us with UUID that sometimes we took the function for granted. 

## What is a globally unique identifier?
A globally unique identifier is a function that will generate a different output each time you call that function. It will not create the same function on the same machine, another machine, or even your lifetime. It needs to be unique.

In this article, I would like to walk through the thought process of creating an incrementing globally unique identifier. 

## First Intuition
The first intuition for creating a unique identifier is to use the time of the day, timestamp, or some random generator with an initial seed in it.

This will work perfectly fine in a single machine. However, if we want to create a globally unique identifier on a different device, each machine might generate the UUID simultaneously and at the same moment. It won't work.

We need to find a way to differentiate the unique ID based on per machine, and they're a couple of ways to do it.

We can append the newly generated timestamp with the machine IP address.

We can append the newly generated timestamp with the machine(server) ID.

We can append the newly generated timestamp with the CPU serial number.

Is that good enough to generate a globally unique identifier in a distributed system?

#### What if one machine evokes the function twice at the same nanosecond? In that scenario, we will have a duplicate timestamp in the device.
To solve that issue, we can create an incremental counter or append a few bits after the timestamp to generate the machine's unique id.

#### Clock doesn't monotonically go forward in a computer. If the computer is using NTP ([Network Time Protocol](https://en.wikipedia.org/wiki/Network_Time_Protocol)), and if the clock is skewed, it will rollback wards to fix it to the right time.
We can do a gradual roll backward to solve the issue and not do sudden changes in time.

#### Generating the time of day for a unique identifier can be slow, is there a faster way to generate time?
Instead of a time of day, we can use a global atomic counter by appending the `computer_id`. 

Since the global atomic counter is generated in memory, if the computer is a reboot, the counter is also rebooted. Therefore, we will end up with repetition.

We will need to have some data store to store those counter. However, it won't be efficient if we need to retrieve the counter each time from the disk. To avoid writing to a disk in each counter, we can write the first `N'th` counter to the disk. Let's say 1000.

We wrote the counter to the disk when it incremented every 1000. If the computer is a reboot, we can retrieve the latest counter from the disk and do `+ 1000` on that counter. If the previously-stored counter is stored in the disk, that means that the last counter that we generated before the computer is reboot hasn't reached the next 1000. If the next counter developed before the computer is reboot has reached the next 1000, it will be a newly updated counter in the disk.


## How about Incremental Global Counter?
We have solved generating unique identifiers in a distributed system. However, this article's title is "How do you create an __incremental__ global counter in a distributed system". 

That means, not only does the function needs to be unique on all machine, but also monotonically increasing. 

If we use the time of the day, it will be tricky because there is no central clock that all the machine is referring to. Each device has its internal clock, and we cannot use that to give order to our unique Id.

We can create a service that generates a unique identifier explicitly. Therefore, all the machine that wants to make a unique identifier will call the Id machine, and it will return an ID that is monotonically increasing. 

Since we need to make network calls at each counter, it will kill the system's performance. Also, what if that server dies? Then all the services will also be affected.

Should we partition the server so that it is available?

However, if we partition the server, how can we guarantee that the Id server will return a unique monotonical increasing Id?

Another way to mitigate the Id server is to have an even generated counter and even developed counter. If the odd generated Id server dies, we can fetch from the even server, and vice versa.

As you think, through all these problems, you realized that creating a monotonically increasing counter in a distributed system is challenging. The more you think about the solution to this problem, the more you think are some of this problem worth solving for generating an incremental Id?

We do have to think about what is more critical, efficiency, or accuracy. Unfortunately, having an incremental global counter serves for real questions. Id generation needs to be in a single location. There is no way to generate distributed unique IDs and maintain the monotonicity property.

The centralized solution is the only solution for an ever-increasing counter because you want to be consistent. It is the trade-off of consistency and availability in the CAP theorem.

## Lesson Learned
Sometimes the simplest solution is good enough. Having a server that generates a monotonically increasing server is good enough.

