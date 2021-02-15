---
layout: post
title: 5 Functional Programming Side Projects To Deepen Your Functional Programming Skills
date: 2021-02-14 23:26
summary: Practice Makes Perfect
categories: functional-programming scala software-development
tags: functional-programming scala software-development
---

_![Photo by Clint Patterson](https://images.unsplash.com/photo-1562813733-b31f71025d54?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1349&q=80)_

One of the best ways to improve coding is to start coding. Mastering any skills require an extensive amount of work and study. 

To quote Malcolm Gladwell, "10000 hours is the magic of greatness". 

Functional programming is hard. One of the reasons is because there aren't many companies out there that adopt functional programming. Suppose you are one of the lucky ones who works in a company that advocates functional programming practices. In that case, you will assume a lot of best ways that often different than the regular object-oriented or web development jobs.

Therefore, just like training your muscle by going to the gym every day and work on it - to be sound in understanding functional programming principles, you need to do the work and write as much application and force yourself to think functionally.

Without further ado, here are five side project ideas that you can do to improve your functional programming knowledge. It doesn't matter what kind of programming language you are using - either Scala, Haskell or JavaScript. The point of writing these applications is to learn about functional programming. Therefore you need to strip out all the OOP thoughts and adopt functional design principles on these projects.


## 1. Creating TODO Application in Tagless Final Style
Todo Application is like the hello world for side projects. All the tutorials that I've seen are usually done in Todo Application, but it uses various technology stack. One of the great things about the Todo Application is that it can be done in a wide range of possible ways. You can straightforwardly make a Todo Application- just with a CLI and no backend. However, you can also make Todo Application that uses a backend and a great UI. 

In this Todo Application, you should start simply by creating Todo Application that can interact with a CLI - in a tagless final way. If you get stuck, look at other tutorials to figure how they design their tagless final application. I will put the Todo Application scala that I often reference to learn about Tagless final on the reference [here](https://www.youtube.com/playlist?list=PLJGDHERh23x-3_T3Dua6Fwp4KlG0J25DI).

### What will you learn? 
You will start to realize all the things that can go wrong with a tagless final. For instance, you tend to put a lot of the stuff in a `Sync` method. However, if you are tracking your effect as a `Sync`, it can be misleading and dangerous - because it can basically do anything, including IO calls.

You will also learn how to structure your application. You can structure your application like how you structure your React or other Java application. However, Scala SBT has its own style of structuring the application. I remember setting up SBT from scratch for the first time is quite challenging. SBT has its own syntax to construct the dependency tree. Therefore, it will be beneficial to structure your file so that one project can easily depend on each other.

## 2. Creating Your own Rate Limiter Library
Writing rate limiter library application from scratch is a great way to learn about Cats library concurrency primitive. 

Writing a rate-limiting library can be quite tricky because of how you need to account for concurrent modification. However, by adopting the cat effect or other functional libraries, such as FS2 [2], you can see the power of writing functionally in a concurrent environment.

### What will you learn? 
At some point, you might want to create an application to do a particular task. Functional code for an operation is usually generic. You will learn how to play around with functions - how to create a function to be composable. You will also learn about how other library authors design their applications. Usually, they will make their operation in a type class pattern. Implicit are generally created as an instance of the type class. Instances of a type class should be placed in the companion object of the type to avoid any import tax (unnecessary import on the top of the file for implicit value).

## 3. Implement Raft Consensus Algorithms in any Functional Language
This is essentially the best way for anyone who wants to understand concurrency and distributed systems. There is various blog post out there that implement the raft algorithm in their favorite languages. Therefore, you can learn about distributed systems and learn how to use functional programming in a distributed systems environment. 

There are a couple of websites and articles that you can reference to including the regular Scala [implemention](https://github.com/pablosmedina/ckite), and the one that uses [ZIO](https://ariskk.com/scala-raft-zio).

### What will you learn? 
Implementing each of the Raft Consensus Algorithm is a good way to learn about distributed systems. You don't really worry about mutating values because everything is immutable. Functions are also more compositional. If you want to create a function that polls for a stream infinite times and stop every 2 seconds, you can do something like this with the FS2 library:

```scala
Stream.repeatEval(IO(println("Hello World"))).through(stopAfter(2.seconds))
```
Imagine if you need to do this in a concurrent environment, it will be more lines of code and a much harder intermediate state that you need to think to do a simple task like this.

## 4. Transform Any Regular Library in a Functional Way
Writing a great library needs a deep understanding of the languages and also some art. You will need to think a lot of the client's perspective on how you want them to use the library. Writing a functional library will force you to learn about commonly available design patterns.


### What will you learn? 
Learning typical pattern in functional programming library. You will first write your library in an object-oriented way, and that is okay! However, you will reference some parts of the library from other blog posts or other libraries. Then, you start to learn how other authors construct their library and the philosophy behind it.

Learning how to transform any regular library into a functional way can be as easy as writing a wrapper for AWS SDK. You can see from the perspective of what sort of flexibility and power you want to give your client. Should we abstract the `sqsClient` from the client or provide that authority to the client? How do we create a modular code so that new parameters can be easily added? Can you implement this function with other more primitive functions?

I learn about functional programming in this way. My team told me to create an internal functional library. It forces me to read functional programming books, look into other popular functional programming libraries, and see the philosophy behind designing the library.


## 5. Write Data Processing Application 
This is the side project that I have done to learn more about [FS2](https://edward-huang.com/functional-programming/etl/scala/stream-processing/fs2/2020/06/02/how-to-write-data-processing-application-in-fs2/) and [Akka Actors system](https://edward-huang.com/akka/scala/programming/functional-programming/reactive/2020/05/30/how-to-write-a-simple-data-processing-application-with-akka-actors/). If you want to know how I did it, check out the link on both. 

### What will you learn? 
Data processing applications can be as easy as creating a primary and worker pool, building pipelines to process each data, and write to a file. 

There are two ways of doing it - first, you can try to use Akka stream or actors to write the application. Second, you can try to use FS2 to write the application Once you finish writing both applications for a text file. Try making those operations as generic as possible so you can use them in any kind of big data application. 

You learn about Akka actor's internal message systems. You will understand how to create an actor so that it is easy to reason about. One of the problems of creating an actor system application is that it is hard to read. You will need good enough documentation so that the next developer can quickly debug any issue. 

FS2 library is itself challenging. I remember I read through the guide in FS2 and still am confused about FS2. The library has a steep learning curve, but it is compelling. FS2 can help you write many significant data processing logic, such as polling from an SQS queue or persisting a large amount of data to a Database. It has its own internal queue that you can use to throttling or rate-limit the file you write. Its concurrency primitives are very compositional in that it is easy to write a hard logic in a simple, understandable operation.


## Closing
Functional programming is not all about academia. You can develop all these applications with Haskell and learn about category theory along the way.

Thanks for reading. I hope you gained new ideas to start learning about functional programming.

