---
layout: post
title: Explain Free Monad Like I am Five (Part 1)
date: 2020-09-06 13:10
summary: Constructing Complex Programs with simple data structures in a functional way
categories: functional-programming  scala programming monad
tags: functional-programming  scala programming monad
---
![Photo by Meagan Carsience](https://images.unsplash.com/photo-1558932129-58b0867c4985?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

In Cats documentation:
> A free monad is a construction that allows you to build a monad from any Functor.



Another definition says it is an idea to switch from effect functions (that might be impure) to plain data structures representing our domain logic.

Simply put, it is like a wrapper that wraps any ADT into a monadic structure DSL that you able to create a program out of it. It separates your program DSL with its interpreter so that you can pick and choose various interpreters.



This post will start with a simple Todo application (no concurrency and all that), from the imperative standpoint, and slowly transform it into a Free monad style.


## Creating Algebra 
```scala
case class Todo(id: Long, description: String, isFinished: Boolean)

sealed trait Action[T] extends Product with Serializable
case class Create(description: String) extends Action[Unit]
case class Find(description: String) extends Action[Unit]
case class Mark(id: Long) extends Action[Unit]
case class Delete(id: Long) extends Action[Unit]
case object Read extends Action[Unit]
```

The above, we created the `Todo` case class, with a simple CRUD. 

The initial way to embed DSL to other programming languages was by using ADT to encode every sentence in your programming languages as a form of the ADT.

## Imperative Program
We will want to do a series of operations such as `Create` a `Todo`. Then, we can `Find` the id belongs to the Todo List. `Read` the Todo List, to show all the results of the Todo list that we had, and Mark the Todo that we have finished.

The imperative way of doing such things can be to construct a List of these ADT like this:

```scala
val program = List(
    Create("Do Laundry"),
    Read,
    Find("Do Laundry"),
    Mark(0L)
  )
```


Therefore, the program above is just a _description_. To execute the plan, we need some executing _interpreter_ to run it.

Let's create an "execute" interpreter:
```scala
object Interpreter {
  import Dsl._
  var map = scala.collection.mutable.Map[Long, Todo]()
  var id = 0L

  def execute[T](action: Action[T]): T = {
    action match {
      case Read =>
        println(map.values.toList)
        ().asInstanceOf[T]
      case Find(id) =>
        println(map.get(id))
        map.get(id).asInstanceOf[T]
      case Mark(id) =>
        println(
          map
            .get(id)
            .flatMap(t => map.put(id, t.copy(isFinished = !t.isFinished)))
        )
        map
          .get(id)
          .flatMap(t => map.put(id, t.copy(isFinished = !t.isFinished)))
          .asInstanceOf[T]
      case Delete(id) =>
        println(s"removing ${map.get(id)}")
        map.remove(id).asInstanceOf[T]
      case Create(description) =>
        println(s"creating todo list ${description} in id ${id}")
        (map += (id -> Todo(id, description, false)))
        id += 1
        ().asInstanceOf[T]
    }
  }

}
```


We use pattern matching to do all kinds of action when we run the program's description above. This is where we all the side-effect and all the mutation happens. Therefore, we will loop through the program and run `execute` on each of the descriptions:

```scala
program.foreach(t => Interpreter.execute(t))
```

So far, so good. However, the ADT that we describe above is not very useful. We cannot, for instance, `Find` an id, and based on that id marked the Todo list. Ideally, we want to do a sequential operation, such as creating a Todo List, Reading all the list of todos, and marking the finished ones. 

We need to find a way to get the previous operation's value and do some other sequential process based on the prior operation's evaluated value. Sounds like a Monad, right?

Ideally, we want to do something like this:
```scala
val program = for {
  todo <- Create("Do Laundry")
  listTodos <- Read
  idZero <- Find(0L)
  _ <- Mark(todo.id)
} yield ()
```


The problem now is that since the program is not a List anymore, how do we create the interpreter? 

Since we are creating a general data structure on the program, we want to have some sort of "wrapper" to wrap these data structures with a Monadic bind to construct a monadic type of program.

## Big Rewrite

We need to return our algebra to "return" some value to capture it in the monadic bind:

```scala


case class Todo(id: Long, description: String, isFinished: Boolean)

sealed trait Action[T] extends Product with Serializable
                                                  // The return type
                                                    // vv
case class Create(description: String) extends Action[Todo]
case class Find(description: String) extends Action[Option[Todo]]
case class Mark(id: Long) extends Action[Unit]
case class Delete(id: Long) extends Action[Option[Todo]]
case object Read extends Action[List[Todo]]

```

Let's write a wrapper of our program so that it can have that monadic bind function. The monadic operation can be translated to something like this:

```scala
FlatMap(Create("Do Laundry"), todo => 
  FLatMap(Create("Clean Bedroom"), cleanBedroom => 
    FlatMap(Read, (listTodos) => 
      FlatMap(Find(0L), idZero => 
        Pure(Mark(idZero))
      )
    )
  )
)
```

We introduce `FlatMap` and `Pure` to bind our original algebraic type to Monad:
```scala
sealed trait Free[F[_], A]

case class FlatMap[F[_], A, B](fa: F[A], f: A => F[B]) extends Free[F,B]

case class Pure[F[_],A](fa: F[A]) extends Free[F, A]
```

Imagine that `F[_]` being like `Todo`, but it can be any type of `F[_]`. It doesn't have any constraint, and `FlatMap` and `Pure` are analogous to `flatMap` and `pure` in Monad, where it binds the context into a sequential operation.


Free is a recursive data structure where each subsequent computation can access the previous calculation. This is all we need to build the programs using a straightforward data structure that is _free_ to its interpretation.

How can we make the above `Free` ADT works with the for-comprehension above?

Ideally, we want to come into this conclusion: 
```scala
val program =
for {
  todo <- Create("Do Laundry")
  _ <- Mark(todo.id)
  listOfTodo <- Read
  } yield {
    println(listOfTodo)
  }

```

To do so:
1. Free has to be a monad. It needs to have some `flatMap` and `map` so that scala can detect and do "for-comprehension".
2. We want the program to do `flatMap` on _Free_, not the "Action" ADT that we defined. That leaves the action just a data structure that we can wire to our interpreter later on.

## Creating Free as a Monad
Let's construct Free function, which it needs to have a `map` and a `flatMap` method.
```scala
sealed trait Free[F[_], A] {
  def flatMap[B](func: A => Free[F,B]): Free[F,B] = this match {
    case FlatMap(fa, f) => FlatMap(fa, f andThen (a => a.flatMap(func))) // a is a Free[F[_], A] here
    case Pure(a) => func(a)
  }
  
  def map[B](func : A => B): Free[F,B] = flatMap(a => Pure(func(a)))
}
```

The code `flatMap` above recursively doing flatMap until it hits `Pure`, and apply the `func` to `a`. `f andThen (a => a.flatMap(func))` means it compose the function by applying the input to `f` first. The return value of `f(a)` is a `Free[F[_], B]`. The return value, then, is applied to the consequent function `a => flatMap(func)`. If you still don't get it, slap the above code into IntelliJ, and try solving the type yourself. The more you look at the input type and return type, the more you know how to create the function above.

Once we have the `map` and `flatMap` function ready, we can start constructing our program. 

However, how do we construct Free?

## Lifting Free stuff
We want to `lift` the `Action` to a `Free[F[_],A]`.

Let's create a `lift` function that will do that:
```scala
def lift[A](fa: Action[A]): Free[Action,A] = FlatMap(fa, Pure.apply)
```


Then, we can create our program like this:
```scala
val program = for {
  todo <- lift(Create("Do Laundry"))
  cleanBedroom  <- lift(Create("Clean Bedroom"))
  listTodos <- lift(Read)
  idZero <- lift(Find(0L))
  _ <- lift(Mark(idZero))
} yield ()
```

We can also just create a dsl to make it more readable way by creating an implicit conversion:

```scala
def lift[A](fa: Action[A]): Free[Action,A] = FlatMap(fa, Pure.apply)

val program =
  for {
    todo <- Create("Do Laundry")

    _ <- Mark(todo.id)
    listOfTodo <- Read

  } yield {
    println(listOfTodo)
  }

```

## Interpreter

Now that we have created our program, we also need to bind the program with an interpreter somehow.

Let's look back at the first interpreter that we defined:
```scala
 def execute[T](action: Action[T]): T = {
    action match {
      case Read =>
        println(map.values.toList)
        map.values.toList.asInstanceOf[T]
      case Find(description) =>
        println(map.values.find(t => t.description == description))
        map.values.find(t => t.description == description).asInstanceOf[T]
      case Mark(id) =>
        println(
          map
            .get(id)
            .flatMap(t => map.put(id, t.copy(isFinished = !t.isFinished)))
        )
        ().asInstanceOf[T]
      case Delete(id) =>
        println(s"removing ${map.get(id)}")
        map.remove(id).asInstanceOf[T]
      case Create(description) =>
        println(s"creating todo list ${description} in id ${id}")
        val todo = Todo(id, description, false)
        (map += (id -> Todo(id, description, false)))
        id += 1
        todo.asInstanceOf[T]
    }
  }

```

The above code looks good, but how do we connect the interpreter above with our Free Monad?

We want to do something like this - given the following `Free[F,A]` data structure, we want to traverse the `Free` structure, evaluating each step and thread the result to the next subsequent computation. We want to fold the `List` of the program description.


Ultimately, we will do the same for Free, by creating a pattern matching for `FlatMap` and `Return`:
```scala
def runProgram[A](free: Free[Action,A]): A = free match {
  case Pure(a) => a
  case FlatMap(fa, fn) => ???
}
```
I put a `???` above because we don't have any access to the regular interpreter that we created. If only if we also supply the interpreter as another argument to evaluate the `Action`:

```scala

 def execute[T](action: Action[T]): T = {
    action match {
      case Read =>
        println(map.values.toList)
        map.values.toList.asInstanceOf[T]
      case Find(description) =>
        println(map.values.find(t => t.description == description))
        map.values.find(t => t.description == description).asInstanceOf[T]
      case Mark(id) =>
        println(
          map
            .get(id)
            .flatMap(t => map.put(id, t.copy(isFinished = !t.isFinished)))
        )
        ().asInstanceOf[T]
      case Delete(id) =>
        println(s"removing ${map.get(id)}")
        map.remove(id).asInstanceOf[T]
      case Create(description) =>
        println(s"creating todo list ${description} in id ${id}")
        val todo = Todo(id, description, false)
        (map += (id -> Todo(id, description, false)))
        id += 1
        todo.asInstanceOf[T]
    }
}

def runProgram[A](program:Free[Action,A]): A = program match {
  case Pure(a) => a
  case FlatMap(fa, fn) =>
    // execute the Action here
    val res = interpreter(fa)
    // thread the function into a new Free
    val newFree = fn(res)
    // execute the next function
    runProgram(newFree, interpreter)
}
```

We can run our program with the existing interpreter like this:
```scala
runProgram(program)
```

## Conclusion
We have come a long way by first introducing the traditional imperative way of creating a program description. We realized that we have no access to the previously computed value within a regular' List' of description and cannot make a sequentially like computation that we want to.

Then, we created a _Free_ structure that enabled us to wrap our existing ADT into something more monadic. We introduce _Free_ ADT by having _FlatMap_ and _Pure_, which is analogous to monadic bind for _flatMap_ and _pure_. Besides, we also created a way to `lift` the `Action` type to a `Free` variety to comprehend them.

Lastly, we touch on using the existing interpreter that we created,`execute`, and execute the `Free` program. 

In part 2 of this series, I want to dive further into how we can generalize the `Free` ADT and create a _free_ structure to create a DSL in any program ultimately. Stay tuned!

Full source code is <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/freeMonad/src/main/scala" target="_blank">here</a>.
