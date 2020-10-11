---
layout: post
title: How to Write Data to a File with FS2
date: 2020-10-11 12:16
summary: A More Performant Function
categories: fs2 functional-programming programming scala
tags: fs2 functional-programming programming  stream-processing scala
---

![Photo by Aaron Burden](https://images.unsplash.com/photo-1471107340929-a87cd0f5b5f3?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1266&q=80)


Last week, I tried to investigate raw data from Dead Letter Queue (DLQ) at work. One of the functionalities that I want to do is poll sources from the SQS DLQ and write those data in a file for further investigation. I thought using FS2 will be a great use-case for this.

FS2 is a light-weight streaming library that can help you do a lot of data processing functionality. Often, we want to get some data from upstream and write them to a file for any investigation.

I want to share how we can create a simple [type class](https://edward-huang.com/functional-programming/2020/01/02/wtf-is-a-type-class/) for writing data to a file system with FS2.

We will break it down by the simplest use-case of writing any value to a file. We will then explore how we can incorporate Queue to decrease the back-pressure of writing to a file.

Let's start by defining our function.

In the fs2 [guide](https://fs2.io/guide.html), there is already a sample example of how we want to write to an external file system.

<script src="https://gist.github.com/edwardGunawan/3748ae8afe5873c5f7222ea7f26fd915.js"></script>


The code specified that we will read all from the file and do filtering on some function `fahrenheitToCelsion`. Then,  encode it into a byte and write it to the `testdata/celsius.txt`.

`through` will combine one stream to another, and `text.utf8Encode` returns a `Pipe[F[_], I+, O-]` which is equivalent to `Stream[F,I] => Stream[F,O]`.

Therefore, we got our first questions answered!

We can write our initial function that takes in an upstream and write it to file. Let's create `toFile(fileName:String, upstream:Stream[IO, String]): Stream[IO, Unit]`:

<script src="https://gist.github.com/edwardGunawan/1ef0a8ebb6e175099f109a43ef531dfc.js"></script>


We need a blocker for `writeAll` because it is an operation that will `block` the thread. Therefore, cats-effect provide a dedicated thread pool `Blocker[IO]` to explicitly handle blocking [operations](https://typelevel.org/cats-effect/concurrency/basics.html#blocking-threads). 

So far it looks great. However, we can make `toFile` as a `Pipe` since it needs `upstream:Stream[IO,String]` as a dependency. Let's refactor our `toFile` to return a `Pipe`:

<script src="https://gist.github.com/edwardGunawan/e5c1a73f2ce1647cd80d26fa639cad04.js"></script>


Then, we can run this operation like this:

<script src="https://gist.github.com/edwardGunawan/0eff09c815825a6ca449a8e8745c9672.js"></script>


Now that we have `toFile` essential operation of writing a file let's expand more by using Queue to alleviate the back-pressure of writing to the file.

Essentially, the caller will want to call `write(item: String)`, and the function will handle writing those items to a file. Let's start by defining our function arguments:

<script src="https://gist.github.com/edwardGunawan/1fe895a5a82c557df0e2b3e02de34ed0.js"> </script>

We want to create a way when each time the caller calls `write(item)`, it will concurrently write the item to a queue, and there will another thread that concurrently polls the value from that Queue and write those value to the file. 

How can we create that internal Queue?

Fortunately, FS2 has a concurrent package with a queue implemented that helps create an internal queue. 

There are two portions when implementing the Queue - enqueue and dequeue. 

Based on your application, sometimes, you want to leave either one of the portions to the caller to have full control over it.

In this scenario, we will encapsulate `enqueue` in the equivalent of `write` to the caller and implement dequeue by writing the file's value.

## Dequeue from FS2 Queue
Let's implement our `dequeue` method first. The code snippet below is equivalent to `constantPoll` from the Queue and pipe the stream will `toFile` function that we created earlier.

<script src="https://gist.github.com/edwardGunawan/87642892a80a227fa8823d01fd37dcd8.js"> </script>


We create a Queue of `Queue[IO, Option[Either[Throwable, String]]]`. 

Then, we create a Resource that will close the Queue if there is no more value from the upstream by `enqueue1` with a `None` type. This is often the [workaround](http://www.aimplicits.com/posts/2018-05-09-fs2-queue-stop/) if you are not using `NoneTerminatedQueue`. 

The value inside of `Stream.bracket` is downstream. It will terminate if it gets a `None`, meaning we have no more amount left in the Queue. 


`write(item:String)` enqueue the `item` to the internal `Queue`:

<script src="https://gist.github.com/edwardGunawan/ef600394f8e6f7063ece20b70513cf0a.js"> </script>


Finally, we can wire the enqueue and dequeue together by spawning dequeue into another thread while letting our caller access the `WriteToFile` instances.

<script src="https://gist.github.com/edwardGunawan/08dad4deb478ea7c81e1f986ce9b33b0.js"> </script>


The key here is the `start` method, where it will spawn a fiber and run the `constantPoll` in the background. If you remove the `start` the operation will be sequential.


We refactor our function by letting the user invoke `WriteToFile.create`. The user will need to supply the Queue and the file destination - something like this:

<script src="https://gist.github.com/edwardGunawan/2fd1a592dfd4e3f03f8f01b3ebdc689f.js"> </script>


Therefore, our `WriteToFile` function looks like this:

<script src="https://gist.github.com/edwardGunawan/0f42265170ff0751dac959051999631c.js"> </script>

Then, we can write a simple program to leverage our implementation like this:

<script src="https://gist.github.com/edwardGunawan/756a3594fb37946672b4b5414d860c43.js"> </script>

Finally, execute the program:
```scala
program.unsafeRunSync
```

## Conclusion

First, we create the `toFile` method that takes a chunk of stream, encode it to byte code and write it to the file.

Then, we try to create a wrapper around `WriteToFile` that encapsulates all the complexity and uses FS2 primitive, `Queue[F,A]`, on the background to handle loads of data. The caller can call `write(item: String): F[Unit]` and do the job for them.

Lastly, we wired everything and untangled the complex logic wired our `WriteToFile` trait to `toFile` and created a library that enables the user to write any value to an external file.

We leveraged the FS2 API to write a complex logic with a little line of code. Concurrently creating an internal queue will be challenging and error-prone without all the combinator of the API.


All source codes are <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/fs2/src/main/scala/WriteToFile" target="_blank">here</a>. 


