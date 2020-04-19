---
layout: post
title: How To Test Code With Side Effect In A Pure Way In Scala
date: 2020-04-19 16:36
summary: Abstracting out into Type Constructor to make code test simpler
categories: scala functional-programming cats
tags: scala functional-programming cats unit-test
---

<img src="{{site.baseurl}}/images/how-to-test-code-with-side-effect-in-a-pure-way-in-scala/Testing Async Code.png" alt="Async Code Pictures">

A unit test is one of the most basic tests when we create our application. It is to test each component or function as a unit - given an input, insert input to the function, and assert the output. However, we encounter much complexity within a unit test that has to do with complex IO or side effects.

A good rule of thumb is to create a mock of that side effect that returns the value that we want. However, by doing so, we need to extract out a single component to a function to mock that operation. It can be cumbersome.

In this blog, I would like to share about how you can test asynchronous code with a single few tweaks in your function - abstracting it over type constructor.

Let's dive into an example to illustrate what I mean.

Note: We use category type classes and `cats` library later in the code if you are not familiar with category type classes <a href="https://en.wikipedia.org/wiki/Type_class" target="_blank">here</a> is a brief description of what it is. 

## Problem
Imagine we have a `DBClient`, and we want to have a `DBService` to do some operation that has an interaction with Database, using `DBClient`.

Here is `DBClient` implementation:
```scala
trait DBClient {
  def get(url:String):Future[Int]
}
```

Here is the `DBService` implementation: 
```scala
class DBService(dbClient:DBClient) {
  def sumAllPrice(urls:List[String]): Future[Int] = Future.traverse(urls)(dbClient.get).map(_.sum)
}
```

Now, if we want to test `sumAllPrice` we can create a stub of `DBClient`.
```scala
class TestDBClient extends DBClient {
  override def get(url:String): Future[Int] = Future.successful{1}
}
```

How can we test `sumAllPrice` in the unit test?

Since it is asynchronous, we need to either have a test case that can receive the asynchronous result, however, if there is also a way to mitigate this problem by making the `DBClient` more generic and abstract it out to a type constructor.


## Action
We can solve this in a couple of ways.
The first one is to refactor the code into a type constructor. In the second one, I want to change the code to a type class pattern.

## Abstracting Over Type Constructor
We make `DBClient` to receive a type constructor `F[_]` type. 

```scala
trait DBClient[F[_]] {
  def get(url:String):F[Int]
}

```

Note: You need to import <a href="https://stackoverflow.com/questions/6246719/what-is-a-higher-kinded-type-in-scala" target="_blank">Higher Kinded Types </a> in your application.

It means that `get(url:String)`  returns any constructor type. It can be `Future[Int]` or a `List[Int]`. 

We use `Cats` <a href="https://typelevel.org/cats/typeclasses.html" target="_blank">library</a> to generate an asynchronous code for the production version and synchronous code for the test version.

`Cats` library has a Monad type `id` which allow types to wraps into a type constructor without changing their meaning:
```scala
package cats

type Id[A] = A
```

We have `TestDBClient` trait which uses for unit testing, and `ProdDBClient` trait which uses for the main code:
```scala
import cats.Id
trait TestDBClient extends DBClient[Id]

trait ProdDBClient extends DBClient[Future]
```

Then, we abstract the `DBService` over type constructor too.

```scala
import cats.implicits._

class DBService[F[_]:Applicative](dbClient:DBClient[F]) {
  def sumAllPrice(urls:List[String]): F[Int] = urls.traverse(dbClient.get).map(_.sum)
}
```

`F[_]: Applicative` is syntactic sugar, context-bound, for having an `implicit` value of `ap: Applicative[F]`. 

The above function is the same as `class DBService[F[_]](dbClient:DBClient[F])(implicit ap:Applicative[F])`

Here we make the type constructor be an Applicative because `traverse` only works on a sequence of values that has an `Applicative`. In the context of `Future` it has an `Applicative`, and it results in `List[Future[Int]]`. However, by abstracting over type constructor, `List[F[Int]]`, we need to prove to the compiler that the value has an `Applicative` when passing into the function.

In this case, we make the type constructor in `DBClient` to not bound to any specific context so that it can easily use in other services. However, we restrict the context of `DBService` because it needs to have `Applicative` to do `traverse` operation.

The second method is abstracting over type constructor but using type class pattern.

## Using Type Class 
There are 3 things that we need to do to define a Type class:
1. Type Class
2. Type Instances
3. Interface Syntax, Interface Object 

[WTF is a Type Class?](https://edward-huang.com/functional-programming/2020/01/02/wtf-is-a-type-class/)

We will define `DBClient` as a type class:
```scala
trait DBClient[F[_]] {
  def get(url:String): F[Int]
}
```


Then we will defined the instances. The instance object is where we put the `ProdDBClient` and `TestDBClient`.
```scala
object DBClientInstances {
  implicit val getFutureInstance: DBClient[Future] = new DBClient[Future] {
    override def get(url: String): Future[Int] = ???
  }

  implicit val getIdInstance:DBClient[Id] = new DBClient[Id] {
    override def get(url: String): Id[Int] = ???
  }
}
```

Lastly, we create interface object, `DBService` and inject our instances in the `sumAllPrice`:
```scala
object DBService {
  def sumAllPrice[F[_]:Applicative](urls:List[String])(implicit dbClient:DBClient[F]): F[Int] = urls.traverse(dbClient.get).map(_.sum)
}
```

We also restrict our type constructor to have an `Applicative` to use `traverse`. 

Why didn't we use `Monad` over here and `Applicative` instead?
It is because `Monad` is more restricted, the subtype of `Applicative`, in the type class hierarchy, and for the current function `Applicative` can do the job. We don't need to restrict the incoming element to `Monad` as we can have a broader range of behavior with `Applicative`, and fewer laws to obey (no `flatMap`). Therefore, the caller can make a broader range of behavior when implementing `DBService`.

## Takeaway
- We can test asynchronous code by abstracting our application with a type constructor.
- When abstracting your application with type constructor, it is an excellent practice to make the type constructor have a minimal restriction for behavior that needed for the current implementation. For instance, `DBClient` is not restricted to having any context, whereas `DBService` is restricted to have `Applicative` because we want to be able to use `traverse` in the type constructor.

All the source code in this tutorial are <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/asynchronousTesting" target="_blan">here</a>.
