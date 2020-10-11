---
layout: post
title: How to Write Data Processing Application in FS2
date: 2020-06-02 00:06
summary: Processing Application with Stream
categories: functional-programming ETL scala stream-processing fs2
tags: functional-programming ETL scala stream-processing fs2
---
_![Photo by Cam Adams](https://images.unsplash.com/photo-1453503795393-c496eee08c98?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1352&q=80)_

In my previous [article](https://edward-huang.com/akka/scala/programming/functional-programming/reactive/2020/05/30/how-to-write-a-simple-data-processing-application-with-akka-actors/), I discussed how to write a data processing application with classic Akka actors.

In this article, I would like to switchgear and used a functional programming streaming library, FS2, to process HTTP logs from a CSV file to get the numbers of HTTP status we have in the log file.

I found that learning FS2, in general, is a bit hard, especially from a background where you never heard of stream. Therefore, before we start diving into the codebase, I would like to begin by defining a Stream, and some necessary Stream processing in FS2. 

Note: I will be using FS2 with effect type cats.effect IO in the code below. If you are interested to know more about cats.effect `IO`, check out their [documentation](https://typelevel.org/cats-effect/datatypes/io.html).

## What is a Stream?
Stream, as in Wikipedia, means a continuous flow of something, becoming from a single source, a liquid, air, or gas. In computing, it said that it transmit or receive (data) over the internet as a steady, continuous flow.

The simple term for it that I could think of will be an iterator but on steroids. That means there will be an infinite number of containers that you will need to iterate. In OOP, you can think of it as an iterator that will iterate finite or infinite amount of array. While you iterate through all those elements, you can do any sort of crazy things and interact with the outside world.

The stream has a lot of useful features that List doesn't. 

First, Each of the elements in a Stream is lazily evaluated, meaning when writing the code, it is not _eagerly_ evoked until you command to execute them. This feature is a big deal, because it enables load, transform, and write big data sets without blowing up your heap.

Second, the stream doesn't mutate the data like how List will do. If you want to map an element `A` to an element `B`, it doesn't alter the original value of `A`. It helps if you are operating big data sets and files in a concurrent environment.

Lastly, you can do a lot of operation in Streams of data like doing IO in the intermediate processor combining it with another stream.

## Basic Stream operation
FS2 has a type:`Stream[F[_], O]`. It means you get a value stream that emits `O` type. The `F[_]` is the effect type. The effect type is implying that if you have some sort of interaction with the outside world, writing to DB or doing IO. Usually, this is an `IO` type. If you don't have any effect, a basic calculation that only affects your CPU is often the `Pure` type.

You can compose a stream like how you would compose a `List`:
```scala
Stream(1,2,3)
```

When you create the `Stream` above, it only creates a `Pure` type constructor of a stream. It doesn't run the program yet; you are describing your application. That means you need to explicitly execute it by either converting it to a List, by doing `Stream(1,2,3).toList`.

### Creating an Effectful Stream
You can create a Stream with effect in it and execute it:
```scala
Stream(1,2,3).covary[IO].compile.toList.unsafeRunSync()
```

The above code describes you lift this stream to an `IO` effect type. Then, you want to execute the stream into an effect type by calling `compile`. Then, we convert the effect to a `List`, by running `toList`. By then, the nature of the stream is `IO(List(1,2,3))`. We still need to lift the IO by running `unsafeRunSync` to get the `List(1,2,3)`.

### Resource Safety
FS2 provided `brackets` or `resources` to interact with files. These are similar to [cats effect](https://typelevel.org/cats-effect/datatypes/resource.html) `IO`, which can allocate resources and release them afterward, regardless of any problem that happens within the allocation.

```scala
Stream.brackets {
  IO {
    new BufferedReader(new FileReader(new File("yourfile.txt")))
  }
}(f => IO(f.close()))
```

The above code describes an operation on how to acquire the resource and release them afterward.

### Dealing with Infinite Stream
To create an infinite amount of constant stream `constant`. To limit the amount of `chunks` or value, we want to pull from the source, `take`:
```scala
Stream.constant(42).take(5).toList
 // res0: List(42,42,42,42,42)
```
It will take the five elements from an infinite pure stream that emits 42.

### Concurrent Processing
To create multiple concurrent processing, we can use `parEvalMap(nWorker:Int)(f:A => F[A1])`. It also has the same syntax for people who are familiar with Akka stream `mapAsync`. 

It creates `nWorker` thread and process the stream. `parEvalMap` will return to downstream in order. Under the hood, it uses `fs2.concurrent.Queue` to execute concurrently and maintaining the request to the downstream.


There are a lot more operations that you can use in FS2. If you are interested, check out this [guide](https://fs2.io/documentation.html).

Now, we can go forth and conquer on processing data with FS2.

## Thought Process
We want to get the data from the `weblog.csv`. We sanitized the incoming data to filter out all the invalid IP. Then, we want to send it to the workers and process the data, converting it into a `Log` instance. Lastly, we count the HTTP status in the logs and write it to the `out.txt` file.


<img src="{{site.baseurl}}/images/how-to-write-data-processing-application-in-fs2/Stream Illustration.png" alt="Stream Illustration">


## Execution

Let's start from top to bottom. Here is the primary function:

```scala
object Main extends IOApp with Processor {

  def run(args: List[String]): IO[ExitCode] = {

    processData("/weblog.csv", "/out.txt", 100).compile.drain.map(_ => ExitCode.Success)
  }
}
```

`processData` will provide a `source` file, `/weblog.csv`, and the `sink` file, `/out.txt` with the number of parallelisms it will generate. Then, after describing the function `processData`, we will need to execute it by calling `compile.drain`. 

`drain` removes all the values from the stream and just execute it. Once all the amounts are emitted from the stream, we will get a single `Unit` in the result, because we will be writing it to the `out.txt` inside `processData` function.

We will use `Stream.resource` to acquire and release resources:
```scala
Stream.resource(Blocker[IO]).flatMap {  blocker =>
  val inResource = getClass.getResource(in)
  val outResource = getClass.getResource(out)
  io.file
    .readAll[IO](Paths.get(inResource.toURI), blocker, 4096)
    .through(text.utf8Decode)
    .through(text.lines)
    ..... // our processing logic here
    .through(text.utf8Encode)
    .through(io.file.writeAll(Paths.get(outResource.toURI), blocker))
  
}

```

The code above creates an `inResource and `outResource` to read and write data to the file. It reads the data from the file line by line, by first decoding the bytes to UTF8, and then encoding it again to UTF8 and writes incrementally to `out.txt`.

This is line of logs will look like:
```
10.128.2.1,[29/Nov/2017:06:58:55,GET /login.php HTTP/1.1,200
```

There is a `...` which we will fill in for all the operations that we will do in the thought process.

Before that, this is the `Log` and `Date` model that each worker will convert into:
```scala
  case class Log(ip: String, time: Date, url: String, status: String)
  case class Date(year: Int, month: String, date: Int, time: String)
```

### Processing Logic
#### Filter invalid IP
Once we get each line from the file, we will filter out the invalid IP address:
```scala
filter(isValidIp) // filter out valid IP
```

`isValidIp` function looks like this:
```scala
def isValidIp(line: String): Boolean = {
    val ipRegex: Regex = """.*?(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3}).*""".r
    ipRegex.pattern.matcher(line.split(",")(0)).matches()
  }
```

#### Send lines to Workers
We send each of the filtered lines to the workers so they can convert them to `Log` model.

```scala
.parEvalMapUnordered(parallelism)(convertToLog)
```

`convertToLog` function looks like this:
```scala
def convertToLog(line: String): IO[Log] = line.split(",").toList match {
    case ip :: time :: url :: status :: _ =>
      IO(Log(ip, convertToDate(time), url, status))
  }
```

#### Count HTTP Status Code
Once the worker finished converting to Logs, it will go through the folding phase. Inside the folding phase, we group the number of HTTP status to a Map with `status -> number of status`.
```scala
flatMap { m =>
          Stream.fromIterator[IO](m.keys.map { key =>
            s"Status : ${key} has a total of ${m(key)} amount "
          }.iterator)
        }
```

#### All Together
To sum up if we put all the logic together, it will become something like this:
```scala
io.file
        .readAll[IO](Paths.get(inResource.toURI), blocker, 4096)
        .through(text.utf8Decode)
        .through(text.lines)
        .filter(isValidIp)
        .parEvalMapUnordered(parallelism)(convertToLog)
        .fold(Map.empty[String, Int]) { (map, currLog) =>
          val updatedStatus = map.getOrElse(currLog.status, 0) + 1
          map + (currLog.status -> updatedStatus)
        }
        .flatMap { m =>
          Stream.fromIterator[IO](m.keys.map { key =>
            s"Status : ${key} has a total of ${m(key)} amount "
          }.iterator)
        }
        .through(text.utf8Encode)
        .through(io.file.writeAll(Paths.get(outResource.toURI), blocker))
```


We can also do fold with the outside of the stream then write to the `out.txt` file. In that sense, we will need to acquire and release the resource for that writeAll operation.

## Summary
In these articles, we touch upon complicated Stream processing library, FS2.

We discuss what a Stream is and how it is different than List. We also talk about the essential operation of the stream. Then, we dive deep into how you can create a data processing application with FS2.

Here is the full source [code](https://github.com/edwardGunawan/Data-Processing/blob/master/dataprocessingfs2/src/main/scala/com/notetoself/example/Processor.scala).

If you are interested to learn more about FS2, check out these resources:
- [A streaming library with a superpower: FS2 and functional programming](https://www.freecodecamp.org/news/a-streaming-library-with-a-superpower-fs2-and-functional-programming-6f602079f70a/)
- [Jakub Kozłowski—A Sky Full of Streams](https://www.youtube.com/watch?v=oluPEFlXumw)
- [Declarative Control Flow with fs2 Stream – Fabio Labella](https://www.youtube.com/watch?v=YSN__0VEsaw)



