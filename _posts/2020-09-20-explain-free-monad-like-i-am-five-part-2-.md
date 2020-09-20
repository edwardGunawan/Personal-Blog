---
layout: post
title: Explain Free Monad Like I am Five (Part 2)
date: 2020-09-20 14:55
summary: Generalizing our Free Structure in Part 1
categories: functional-programming  scala programming monad
tags: functional-programming  scala programming monad
---
![Photo by Esther Jiao](https://images.unsplash.com/photo-1524337676612-18e579b62f4a?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

In [part one](https://edward-huang.com/functional-programming/scala/programming/monad/2020/09/06/explain-free-monad-like-i-am-five-part-1/) of this post, we talked about how Free Monad can build a monad from Functor. To do so, you can create a DSL like programs and generate another program. The program that you make becomes plain data structures that represent a domain logic. Once you composed the program, you can connect those programs with an interpreter. You can switch a different interpreter on dev and QA. You can also abstract out the implementation of your program or optimize them without changing your domain logic.


We started by creating a Todo application with these algebras:
```scala
case class Todo(id: Long, description: String, isFinished: Boolean)

sealed trait Action[T] extends Product with Serializable
case class Create(description: String) extends Action[Todo]
case class Find(description: String) extends Action[Option[Todo]]
case class Mark(id: Long) extends Action[Unit]
case class Delete(id: Long) extends Action[Option[Todo]]
case object Read extends Action[List[Todo]]
```

First, we created an algebra `FlatMap` and `Pure` and introduce what it's like to make a monad like programs by wrapping our Todo algebras into the "Free" data structure:
```scala
sealed trait Free[F[_], A] {
  import Free._

  def flatMap[B](fn: A => Free[F, B]): Free[F, B] = this match {
    case Free.Pure(a) =>
      fn(a)
    case Free.FlatMap(fa, fun) =>
      FlatMap(fa, fun andThen (a => a.flatMap(fn)))
  }
  def map[B](fn: A => B): Free[F, B] = flatMap(a => Pure(fn(a)))
}

object Free {
  final case class FlatMap[A, B, F[_]](fa: F[A], fun: A => Free[F, B]) extends Free[F, B]
  final case class Pure[F[_], A](a: A) extends Free[F, A]

}

```

Then, we created a way to "lift" these algebras to a "Free" monad by creating an implicit conversion of `Action[A] => Free[Action, A]`:
```scala
implicit def lift[A](fa: Action[A]): Free[Action, A] = FlatMap(fa, Pure.apply)
```

Lastly, we ended with connecting the program and the interpreter with our "Free" algebras that we construct. 
```scala
def runProgram[A](program: Free[Action, A]): A = program match {
    case Free.Pure(a) => a
    case FlatMap(fa: Action[A], fn: (A => Free[Action, A])) =>
      // execute the Action here

      val res = execute(fa)
      // thread the function into a new Free
      val newFree = fn(res)
      // execute the next function
      runProgram(newFree)
  }
```

Note: This is the short recap from "Explain Free Monad like I'm Five (Part one)" I explain my thought process on how I came up to these conclusions. If you are interested, check it out before reading further.

## Problem
The above solutions seem to work for constructing a program using Free Monad for a Todo Application. However, that solution will only work for that particular application. We cannot use our `runProgram` function or "lift" our algebras to a "Free" version. It will be quite tedious if we need to implement the `runProgram` for every program that we construct and also creating an implicit conversion each time we build a new set of algebras.


## Generalizing Lift
To generalized lift, we just need to abstract out `Action` to an `F[_]` type constructor:
```scala
implicit def lift[F[_], A](fa: F[A]): Free[F, A] = FlatMap(fa, Pure.apply)
```

However, if we do this, it will cost compile error because a lot of the times, we will want to lift concrete type constructors instead of the generic "sealed trait":
```scala
val create: Free[Create, String] = lift(Create("Check Homework"))
```

This will throw an error in for-comprehension because it cannot automatically lift the value to the `Action` sealed trait.

One way to do this is to create a "smart-constructor", by first setting the value as the generic algebra `Action`, and defined the function that abstract out regular input to a "free" version of it. 

```scala
type TodoAction[A] = Free[Action, A]

def create(description: String): TodoAction[Todo] = lift(Create(description))
def find(description: String): TodoAction[Option[Todo]] = lift(Find(description))
def mark(id: Long): TodoAction[Unit] = lift(Mark(id))
def delete(id: Long): TodoAction[Option[Todo]] = lift(Delete(id))
def read: TodoAction[List[Todo]] = lift(Read)
```

Remember that the return of the "Free" type is defined by the generalized algebra return type. For instance, `create` will return a `TodoAction[Todo]` because the algebra for `Create` returns an `Action[Todo]`: 

```scala
case class Create(description: String) extends Action[Todo]
```

Then we can construct our program with these smart constructors:
```scala
val program =
  for {
    todo <- create("Do Laundry")
    _ <- mark(todo.id)
    listOfTodo <- read
  } yield {
    println(listOfTodo)
  }
```

## Generalizing RunProgram
Referencing from the `runProgram` from the [previous](https://edward-huang.com/functional-programming/scala/programming/monad/2020/09/06/explain-free-monad-like-i-am-five-part-1/) article: 
```scala
def runProgram[A](program: Free[Action, A]): A = program match {
    case Free.Pure(a) => a
    case FlatMap(fa: Action[A], fn: (A => Free[Action, A])) =>
      // execute the Action here

      val res = execute(fa)
      // thread the function into a new Free
      val newFree = fn(res)
      // execute the next function
      runProgram(newFree)
  }
```

The current run program has an `execute` function tied to the ones with `Action`. Moreover, `Free[Action,A]` is restricted to the particular `Todo` application. How can we generalize this further to take any program of `F[_]`?

We can generalize the `runProgram` by taking a type constructor `F[_]`, but how will we generalize the `execute` function?

```scala
def runProgram[F[_], A](program: Free[F, A]): A = program match {
  case Free.Pure(a) => a
  case FlatMap(f: F[A], fn: (A => Free[F,A])) =>
    val res = execute(fa) // where is this execute come from?
    val newFree = fn(res)
    runProgram(newFree)
}
```

Somehow, the `execute` function should be a generic function that the user defines when creating the program. Therefore, it needs to be some `trait` that will need to pass in as an argument. If you look closely at the implementation above, the function `execute(fa)` takes in an `F[A]` and returns an `A`. That `A`, then, can be an input to the `fn` function. Therefore, we need to define the `execute` function that takes an `F[A]` and returns an `A`. Let's define the `execute` function inside an `Executor` trait:

```scala
trait Executor[F[_]] {
  def execute[A](fa: F[A]): A
}
```


We pass in the `Executor` trait class as an argument in `runProgram`, and call the `execute` within that `Executor` trait:
```scala
def runProgram[F[_], A](program: Free[F, A], executor: Executor[F]): A = program match {
  case Pure(a) => a
  case FlatMap(fa: F[A], f: (A => Free[F, A])) =>
    val res = executor.execute(fa)
    runProgram(f(res), executor)
}
```

Then, we can define our interpreter for Todo application in the `Executor`, `execute` function and pass the value into the `runProgram`:
```scala
runProgram(program, executor)
```

We finally can run `Free[F,A]` on any programs by providing the custom "execute" that interprets our algebra. However, can we generalized this even further?


## Generalizing Executor
The current `Executor` function takes in an `F[A]` and returns an `A`. However, this function is limited to the pure effect type. We can somehow generalize this further by generalizing its effect type. If you are unsure about what effect means, check out my article on [What is "effect" or "effectful" mean in Functional Programming? \| edward-huang.com](https://edward-huang.com/functional-programming/scala/monad/2020/06/21/what-is-effect-or-effectful-mean-in-functional-programming/).

This kind of transformation between functors is called [natural transformation](https://typelevel.org/cats/datatypes/functionk.html). Instead of having a signature of `F[A] => A`, which is equivalent to `F[A] => Id[A]`, we can have `F[A] => G[A]` where `G[_]` can be any type constructor.

Let's refactor our `Executor` function to take in 2 type constructor:

```scala
trait Executor[F[_], G[_]] {
  def execute[A](fa: F[A]): G[A]
}
```

Then, `runProgram` will take an additional type constructor `G[_]`:
```scala
def runProgram[F[_], G[_], A](program: Free[F, A], executor: Executor[F, G]): G[A] = program match {
  case Pure(a) => a
  case FlatMap(fa: F[A], f: (A => Free[F, A])) =>
    val res = executor.execute(fa)
    runProgram(f(res), executor) // something is not right here ....
}
```

You noticed that the above code would not compile. Initially, `execute` function returns `A`. However, after changing our `Executor` function to return `G[A]`, we cannot pass in the result to the `f` function in `FlatMap`.

We can resolve this issue by requiring `G` to be a Monad, and use `flatMap` to bind the value inside of `G` to `f`:
```scala
import cats.{Monad}

 def runProgram[F[_], A, G[_]](program: Free[F, A], executor: Executor[F, G])(implicit M: Monad[G]): G[A] =
    program match {
      case Pure(a) => M.pure(a)
      case FlatMap(fa: F[A], f: (A => Free[F, A])) =>
        val res = executor.execute(fa)
        M.flatMap(res)(a => runProgram(f(a), executor))
  }
```

## Conclusion
In this article, we learn how to generalize our application even further by generalizing our "Free" wrapper to use it in various applications.

First, we generalize the `lift` functions and introduce a "smart-constructor" to construct our program. Then, we generalized our `runProgram`, the interpreter that runs the program, by submitting an `Executor` trait. Lastly, we generalized our `Executor` trait further by introducing natural-transformation on our `Executor` trait.

The Free structure exists in the `Cats` [library](https://typelevel.org/cats/datatypes/freemonad.html) with different names. Hopefully, you can understand the concept of Free and how Free works. There are also Free Applicative, and the difference is in the `runProgram` and its "Free" algebra, where instead of being a Monad, it is an Applicative functionality.

All source code on this article can be viewed [here](https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/freeMonad/src/main/scala/GeneralizedFreeStructure).


Source and Reference:
[Free Monads Explained (pt 1). Building composable DSLs \| by Oleksii Avramenko | Medium](https://medium.com/@olxc/free-monads-explained-pt-1-a5c45fbdac30)
[Cats: FreeMonads](https://typelevel.org/cats/datatypes/freemonad.html)
