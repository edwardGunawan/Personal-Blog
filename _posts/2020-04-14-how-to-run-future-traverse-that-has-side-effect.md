---
layout: post
title: How to run Future Traverse that has Side Effect
date: 2020-04-14 17:07
summary: Solving parallel programming problems
categories: scala functional-programming
tags: scala functional-programming
---

![Photo by Tomas Sobek](https://images.unsplash.com/photo-1531376842337-a13868cbed5b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1356&q=80)

Making your program asynchronous has been the go-to for performance and scalability. However, bugs and problem about asynchronous programming has been tough to solve.

A couple of days ago, I explained to one of my co-workers how Future works. We talked about how to execute a list of Future to create a Future of List.

As you may know, the operation itself has been used widely that standard Scala library has the function dedicated to solving the problem, `Future.traverse` and `Future.sequence`.

As we dive deep into the implementation of `traverse` and `sequence`, I wondered if foldLeft blocks when Future execute. 

After countless research and understanding of how Future works, I decided to post my findings and insights so that you can also solve some of the bugs when you dealt with the Future.

## Problem Statement
Let's consider a problem where you need to create a function getPrices, where you give a list of URL of prices of a coffee. You do a batch fetch. Aggregate the price of the coffee from all the URL and store the prices in an existing database. Return the aggregated data to the user. 

One condition is that the URL has duplication. Meaning, it may provide multiple of the same URL, which returns the same prices. Therefore, you also need to deal with that scenario.

```scala
case class Coffee(url:String, price:Int)

// assuming this the fetch method for coffee
def fetch(url:String): Future[Int] = ???

// write to DB for existing coffee
def writeToDB(coffee:Coffee): Future[Unit] = ???

// read value from database
def readfromDB(url:String): Future[Coffee] = ???

def isExistInDB(coffee:Coffee): Future[Boolean] = ???


// your function here
def batchCoffeePrice(coffeeUrls:List[String]): Future[List[Coffee]] = ???

```

When you write `Coffee` to the database, you need to check if the value exists in the database and write the value afterward. 

Assuming the price won't change for this article illustration purposes.

## Order of Execution

One way is to check if the value exists in the database. If it doesn't exist, fetch URL and write to a database. If it does exist, get the value from the database and return it to the user.

Therefore, execution be:
1. Check if the value exists in the database
2. If the value exists, get the value and loop through the next URL.
3. If the value doesn't exist, fetch URL and create a coffee instance.
4. Write the coffee instance to the database.

Therefore, the first instinct will be to do something like this:
```scala
def batchCoffeePrice(coffeeUrls:List[String]): Future[List[Coffee]] = Future.traverse(coffeeUrls){url =>
  isExistInDB(url).flatMap{ boolean =>
    if(boolean) {
      readFromDB(url)
    }
    else {
      fetch(url).flatMap{
        price =>  {
          val coffee = Coffee(url,price)
          writeToDB(coffee)
          coffee
        }
      }
    }
  }
}
```

Looks good, and the logic seemed to work. However, when calling `batchCoffeePrice`, it writes multiple of the same value to the database if the coffeeUrls is duplicate. 

That seems weird. Since `isExistInDB` should already check if it exists in the database then do the operation, why is it still having duplicate writes?

## Dive deep into Traverse
The problem relies on `Future.traverse`. Does Traverse execute the List of Future sequentially or in parallel?

If you look into the implementation of `Future.traverse`:
```scala
def traverse[A](initial:List[A])(f:A => Future[B]): Future[List[B]] = initial.foldLeft(List.empty[B]){(acc, currA) => 
  val res = f(currA)
  for{
    a <- acc
    b <- res
  } yield a :+ b
}
```

You noticed that Traverse implements in foldLeft, `val res = f(currA)` explains the bug that makes the function inconsistent.

The above traverse function iterates through all the List of Future without blocking. However, since the Future is __eager__, it depends on how you implement your `f` function. It will either be sequential or parallel.

There is no difference in the return statement `Future[List[B]]` because, at the end of the function, it returns all the resulted `f(currA)`. However, if you want to do something with `f(currA)`, you need to be aware, especially dealing with something that is eager and side effects, that it is in parallel.

If you want to deal with side effects within the Future, such as fetching a database, you need to execute the List of value in sequential because of __order__ matters.

To explain the code above, the function will trigger first, and wire to the for-comprehension after. Therefore, evoking all the value in `List[A]` in parallel. However, if we change the function signature to `lazy val res` like this:
```scala
def traverse[A](initial:List[A])(f:A => Future[B]): Future[List[B]] = initial.foldLeft(List.empty[B]){(acc, currA) => 
  lazy val res = f(currA)
  for{
    a <- acc
    b <- res
  } yield a :+ b
}
```

We solved the problem of making the asynchronous value run in sequential instead of parallel.

## Solving the Problem
To solve the problem, we implement foldLeft and traverse ourselves and make the value lazy.

If we want to refactor the code above and fit into `Future.traverse`, you need to check inside the `f` function `ifExistInDB` again.

```scala
def batchCoffeePrice(coffeeUrls:List[String]): Future[List[Coffee]] = Future.traverse(coffeeUrls){url =>
  isExistInDB(url).flatMap{ boolean =>
    if(boolean) {
      readFromDB(url)
    }
    else {
    for {
      price <- fetch(url)
      exist <- isExistInDB(url)
    } yield {
        val coffee = Coffee(url,price)
        if(exist) writeToDB(coffee) 
        coffee
      }
    }
  }
}
```


However, you might as well not use Future .traverse and use `foldLeft` and make the value sequential to create less IO in the function.

## Main Takeaway
- `Future.traverse` fits into executing a list that doesn't depend upon each other (or side effects within your Future). If you want to make the function inside Future .traverse to execute parallel, you can do Future .traverse.
- Future is eager by nature. Therefore, once you call `Future{something}`  it executes right away. In order to not make it eager, you can change it to an IO.
- Future.sequence is uses Future .traverse under the hood.
