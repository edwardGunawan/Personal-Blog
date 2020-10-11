---
layout: post
title: Creating a Throttling System for Any Application with no more than 100 lines of Code
date: 2020-10-11 12:41
summary: With the help of functional programming and FS2
categories: fs2 functional-programming stream-processing scala programming
tags: fs2 functional-programming stream-processing scala programming
---

<img src="{{site.baseurl}}/images/create-a-throttling-system-for-any-application-with-no-more-than-100-lines-of-code/Throttling Diagram.png" alt="Throttling Diagram">



Applications are often design with a throttling mechanism involved. Sometimes, we want to limit the number of requests to improve our application's security and performance. Sometimes your application cannot respond to more than a certain number of connections. For instance, if you polled the message as fast as your can from a queue and did not limit the number of connection, your application will soon exhaust its connection fool and face Denial of Service error. In this scenario, having a mechanism that limits the amount of the number of applications being processed at the same time will help improve your application performance.

In this article, I want to share how we can create a throttling mechanism with FS2 concurrent primitive Queue.

Before we start, I want you to imagine an application that will continuously poll elements from upstream. Then, we leverage [FS2](https://fs2.io/guide.html) to handle throttling/back-pressure by providing the maximum buffer size that the resources can operate simultaneously. When the internal Queue is full, it won't enqueue the message until some of the tasks are finished. 


Essentially the user of the application can use it like this:
<script src="https://gist.github.com/edwardGunawan/ec2dbd413578953e203e46618f12083d.js"></script>


Therefore, when the maximum size exceeds 100, it will stop poll the element until the internal Queue has some space.

There are 2 parts to the process of creating this:
- Consumer is a type of class that will subscribe to an upstream by constantly enqueuing the value to an internal Queue. 
- Subscriber is a type class that will wrap the Consumer and dequeues the chunk from the internal Queue and process that value.

Depending on the use-case of the application, we can encapsulate the Subscriber portion or the Consumer portion. In this article, it will be the Consumer. Meaning the user can specify what upstream their function wants to poll from, and they get to access the result from the Subscriber. The [last](https://edward-huang.com/fs2/functional-programming/programming/scala/2020/10/11/how-to-write-data-to-a-file-with-fs2/) article was encapsulating the subscriber portion.


## Consumer
The Consumer will subscribe to the upstream. Therefore, we want a function that is like this:
<script src="https://gist.github.com/edwardGunawan/b1df56e6df443fd8fe2e07f727d2fae3.js"></script>


The function receives any value from `upstream` and `enqueue` the matter to the internal Queue.


Let's create the initial subscribe method. We need to create the instance of the `Consumer` type class by letting our caller initialized the internal Queue and inject them into the `Consumer` instance.
<script src="https://gist.github.com/edwardGunawan/17f2120cec233aef4765a5c250ce4398.js"></script>

We will use NoneTerminatedQueue for terminating the Queue once the upstream stops are sending a message to the Consumer. The Consumer can tell the Subscriber to stop the stream.

Does that sound like resource acquisition?

You are right! Essentially, we want to acquire a resource and want to guarantee that some cleanup action is run if the resource is acquired. Therefore, we will create the resource helper method for `subscribe`:

<script src="https://gist.github.com/edwardGunawan/b4f0536b2d17e13393db9cba183a636d.js"></script>


We will get the value from `upStream` and `enqueue1` to our internal Queue. Then, we will `compile.drain` and drain all the input coming from `upStream`. If all the information is a drain or any errors occur during the computation, the resource will clean up by `enqueue1` a `None` to our internal Queue (Subscriber). The Subscriber then stops its stream.

This is how we call the `Consumer`:
<script src="https://gist.github.com/edwardGunawan/de8c86d27266ea21d0783c6e1fa85512.js"></script>


`start` here will start a `fiber`. If you didn't put a `start`, then the entire process will be sequential, meaning it will enqueue all the value then dequeue. If our internal Queue is full, it will hang there. Therefore, having `start` will execute the `subscribe` in another IO thread.


## Subscriber

We want the `Subscriber` to _poll repeatedly_ and return a `Stream[F, A]` back to the caller.

Therefore, we can create a type class that has will `pollRepeat`:
<script src="https://gist.github.com/edwardGunawan/930e657de28803fe1eedf729c6619e76.js"></script>


Similar to `Consumer` we will need to create an instance of `Subscriber` by having the maximum queue size and the upstream as the parameter:
<script src="https://gist.github.com/edwardGunawan/29a5d47f91159160c486a2994b883663.js"></script>



We need to subscribe to the upstream, fire off the Subscriber into another thread, and instantiate the Subscriber.
<script src="https://gist.github.com/edwardGunawan/d0f23dc6d5efa030d4a0c4c6678aabc5.js"></script>


We create the `boundedNoneTerminated` internal Queue with the `maxBufferSize` that the caller provides. Then, we make our `Consumer` with the help of the `Consumer` type class, and subscribe to upstream and start off on another fiber. We return the Queue back so that we can wire it with the `Subscriber` instance.

Then as we create the `consumer`, we can connect the consumer with the `Subscriber`. Like this:
<script src="https://gist.github.com/edwardGunawan/c12a433e43dab12ebffb77507e3e5a6e.js"></script>


This is the program to use the subscriber instance and use it as any Throttling mechanism in your application:
<script src="https://gist.github.com/edwardGunawan/67b5b52440b2cb61e39a5777d2f06a26.js"></script>


You will need to call `unsafeRunSync` at the end:
```scala
subscriberExample.unsafeRunSync
```

## Conclusion
Adding a throttling mechanism can be challenging, especially if you need to do it in a concurrent environment. Luckily, with the help of FS2, constructing a throttling mechanism on any application can be done in a few lines of codes. 

We create the `Consumer` type class to subscribe to any source. Then, we use `Subscriber` to constantly `enqueue` and `dequeue` in a concurrent manner. We can guarantee the cleanup of the Queue data before we stop the stream with resource acquisition.

I hope you find this post useful in learning more about FS2, Scala, or Functional Programming in general. If there is anything that might cause any error to feel free to point it out so I can also learn from you.

All source code is in <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/fs2/src/main/scala/Subscriber" target="_blank">github</a>.
