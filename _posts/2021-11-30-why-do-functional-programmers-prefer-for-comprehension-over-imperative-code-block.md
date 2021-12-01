---
layout: post
title: Why Do Functional Programmers Prefer For-Comprehension Over Imperative Code Block
date: 2021-11-30 22:07
summary: Short Answer - Readability
categories: scala functional-programming
tags: scala functional-programming
---

"Why do you wrap everything into an effect and put a for over here?" a Senior engineer in another team commented on my second code review. I looked into the code, and the first thing that popped up on my mind was, "Because it is readable."

Readability is very subjective. Someone from the world of imperative programming and used to the Java 8 API Stream-like syntax will say that sequencing your program with for-comprehension will make it harder to read. On the other hand, functional programmers will view Java 8 API Stream-like syntax harder to trace through the effect of your code. Hence, I try to find a more objective point of view on why it is readable.

## Looks like a Pseudo Code
For-comprehension is very similar to Haskell monad comprehension.

For Comprehension is syntactic sugar for `flatMap`. 

`flatMap` represents sequential computations, and it is the main trait of Monad. Hence, the designer Scala uses monads for collection operations. 

Monad comprehension in Scala was designed to look like imperative `for` loops in some generic C/pseudo-code-like language. Why? So that it looks like imperative sequential side-effecting code block like a C-style language. The left-array for assignment is typical in imperative pseudo-code. Hence, it creates better readability.

However, Scala and Haskell differ within the comprehension syntax - it can do more than perform the two monadic operations `join` and `bind` (or `map` and `flatMap`).

In Scala, a `for` Comprehension without a `yield` translates into `foreach`. `foreach` is an imperative iteration that produces side-effect. A `yield` can have a guard, `yield bar if baz`, which translates into filtering elements.

That reason alone doesn't give a good reason why you would prefer `for` Comprehension over generic Java code block. However, since for-comprehension is equivalent to monad comprehension, you can ensure that your code runs sequentially in that effect.

## Enforces Sequential Operation
Unless you use `IO`, having multiple [effects](https://edward-huang.com/functional-programming/scala/monad/2020/06/21/what-is-effect-or-effectful-mean-in-functional-programming/) inside your functions is hard to debug without for-comprhension. 

Let's take an example if you have a function that sequence multiple API calls:

```scala
def computeSequentially = {
  val api1Future = Future {...}
  val api2Future = Future {...}
  val api3Future = Future {...}
  
  api1Future.flatMap {api1 => 
   api2Future.flatMap { api2 => 
    api3Future.map{api3 =>
     //do something here
    }
   }
  }
}
```

From the code above, you may see that you sequence multiple API calls with `flatMap`. However, you are making a parallel call instead of doing a sequential call. 

Future is eager. That means once you instantiate the value, it will trigger a thread and execute what is inside of Future.

Compared to wrapping each effect within the for-comprehension, it enforces any effect to execute sequentially.

```scala
def computeSequentially = for {
  api1 <- Future { ... }
  api2 <- Future { ... }
  api3 <- Future { ... }
} yield( /* do something */ )
```

Another benefit of using for-comprehension with effects is that another developer can easily take a quick look at the codebase and understand the intent of your program without understanding how Future works. You don't need to guess if this block is running sequentially vs. in parallel.

The above code looks very simple because we are just doing simple API fetch calls. However, what happens if we have a nested effect? Such as calling an API that will return `Future[Option[Int]]` or `Future[Future[Future[_]]]`?

## Increase Readability with Nested Effect
Without for-comprehension, you see something like this:

```Scala
def fetchAPi2(someValue: Int): Future[Option[Int]]
def fetchApi3(someOtherValue: Int): Either[Throwable, Result]

def computeApi = {
  fetchApi2(1).flatMap{someValueMaybe => 
    someValueMaybe.map{
      fetchApi3(someOtherValue) match {
        case Left(throwable) => ???
        case Right(result) => ???
      }
    }.getOrElse(0)
  }

}

```

Then, if there are even more sequential calls, the line of the above code will go towards the right side - like a callback hell.

With for-comprehension, it helps flatten the amount of nested call-on effects. 

```scala
import cats.data._

type FutureOption[A] = OptionT[Future, A]
type FutureOptionEither[A] = EitherT[FutureOption, Throwable, A]

def computeApi = {
  val futureEitherOption = for {
    someValue <- FutureOptionEither(fetchAp2(1))
    someOtherValue <- FutureOptionEither(fetchApi3(someValue))
   } yield ()
   
   futureEitherOption.value match {
     case Left(throwable) =>  ???
     case Right(result) => ???
   }
 }
```

We use the `cats.data` collection on monad transformer and use for-comprehension to compute multiple nested affect calls. The for-comprehension helps decrease the number of nested calls and make it look like a sequential call. 

This rail-way-oriented programming. With rail-way-oriented programming, you can separate all error handling on all effects in one place - keeping the main logic like a pseudo code. 

## It is all based on Your Style Guide
Convincing the benefit of for-comprehension vs. regular code-block in that PR as it is all based on your team's style guide.

If all developers believe that writing a regular code-block is more readable than for-comprehension, writing code-block is preferable. 

In my case, another developer also commented, "plus 1" on his comments. However, I don't fully agree with the explanation. The team prefers to write code in an imperative code block style instead of `for` Comprehension. 

I ended up making changes to the preferred style guide.

## Source:
[programming languages - Why does Scala name monadic composition as "for comprehension"? - Software Engineering Stack Exchange](https://softwareengineering.stackexchange.com/questions/307565/why-does-scala-name-monadic-composition-as-for-comprehension)
