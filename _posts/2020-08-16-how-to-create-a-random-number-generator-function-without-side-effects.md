---
layout: post
title: How to create a random number generator function without Side Effects
date: 2020-08-16 21:33
summary: Functional State to the rescue
categories: scala functional-programming monad programming
tags: scala functional-programming monad programming
---

_![Photo by dylan nolte](https://images.unsplash.com/photo-1518688248740-7c31f1a945c4?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

Must you be thinking about this title? Is that even possible? The random generator has been illustrated as an example of side-effects. Each time when we call `random.nextInt` twice, the value will not be the same. We don't know `nextInt` besides, we know that it generates random values. Let's look at a regular imperative API that relies on side effects:

```scala
val random = new scala.util.Random

random.nextInt

random.nextInt // this produces another new value

```

In this article, we will first discuss why we want a random generator without side effects. Then, we change this API so that it is referentially transparent. By the end of this, you will know the gotcha on how to create a functional imperative program with having pure states.

## Why even bother?
We know that a random generator will always generate random values. However, this becomes a trouble when we incorporate "randomness" into our program. 

Imagine you need were developing a program that relies on random numbers to do AB Test two different features. However, you encounter trouble when doing unit-test. How do you unit test this application, since it creates a side effect? 

Besides, you might be thinking, any sort of bugs in the concurrent environment will be tough to reproduce. Since you don't know how to reproduce the problem, you have a hard time explaining to your senior management or manager why there is a considerable failure in your system at 2 AM on Tuesday.

How do you create randomness in your program that can be easily unit-test and, most importantly, able to reproduce bugs reliably?

## Understanding Randomness
Before we tackle the problem, we have to understand how do "randomness" generated. Let's revisit the regular random generator API from the standard scala library:

```scala
val random = new scala.util.Random

random.nextInt
random.nextInt
```

We don't know how `nextInt` function is implemented. However, we know that when we call `random.nextInt`, there is some internal state inside there that gets updated. Once it updates, it returns a new value. Because the update of the country is performed as side-effects, the function is not referentially transparent.

From the observation above, we can deduce two facts. First, there is no "pure" randomness in programs. They are all functions that produce "pseudo" random to the caller. The reason why the caller thinks that it is random is that they don't expose their internal state to the caller. It gives the caller a number that they cannot reproduce. Thus, it is "random". Second, we can roughly understand how `nextInt` is implemented. When we instantiate the `random` number, it provides an internal seed value to the random generator. Then, each time we evoke `nextInt`, it will use some algorithms, updates its internal state, and return a new value.

## Show me how to solve this!
Yes, we will get there! But I want to show you how we can derive to that purely functional state so that you can implement this derivation not only for this particular problem but also for other issues with a state within.

We know that each time we call a random generator, we can provide its random generator with a seed value. That seed value will use some algorithm to compute from the current state to the next state. Therefore, to reproduce the "same" generator, we need to have the same seed with that same state to reproduce the same future state.

However, in the regular random number generator, once the next state is generated, the previous state is destroyed. This becomes hard to produce the same result. Since we are given seed value, we need to keep track of how many times the `nextInt` or `nextDouble` is called to reproduce the same result.

Fortunately, there is a way to make the random generator pure without needing to keep track of the functions' counts. __The key to recovering referential transparency is to make updating the state explicit__. Don't update the state as a side-effect, simply return the new state along with the result that we are generating.

In the example of a random generator, instead of mutating the state inside the function _in place_, we return a _new_ state and the "random" number generated back to the caller. 

Let's re-create a random generator. For this random generator, we will build on top of scala random numbers util by passing in the seed value to `scala.util.Random(seed)`. Then, compute the next seed value, and return the new random generator to the caller.

```scala
case class Random(seed:Int) {
  def nextInt:(Int, Random) = {
    val rng = new scala.util.Random(seed)
    val res = rng.nextInt
    val newSeed = Random(seed+1)
    (res, newSeed)
  } 
}
```

We can run this random generator by calling the example below:
```scala
val rng = Random(1)
val (newValue, nextRandom) = rng.nextInt
val (newValue1, nextRandom1) = nextRandom.nextInt

// the two value will be different because we compute a random number
println(s"new value ${newValue}. newValue1: ${newValue1}.")


val (sameValue1, sameNextRandom1) = nextRandom.nextInt
// both value will be the same below because it uses nextRandom to compute the nextInteger
println(s"newValue1: ${newValue1}. sameValue1: ${sameValue1}")
```


We return the new state with the old state unmodified. We separate the new state's computation with communicating _this_ new state to the rest of the program. There is no internal state memory being used. We are merely giving the next state to the caller, and let the caller have complete control to decide what to do with that new state. 

We can use this pattern in a lot of programs that contain a global or mutable state. The key to making a simple state API is to return _that_ new state to the caller and let the caller have full control over the state.

I will give one more example for a class with an internal state, and how you can refactor it to be pure.

Each time when you see a class that has an internal state like this:

```scala
class Foo {
  private var map : FooState = ???
  def put: Bar
  def write: Int
}
```

Suppose that `put` and `write` will mutate the `map` state in some ways.

You can translate the above class by making explicit the transition from one state to the next by changing to this:
```scala
trait Foo {
  def put: (Bar, FooState)
  def write: (Int, FooState)
}
```

Now you notice that if you want to do sequence a computation in a practical way, it will be awkward and tedious. Therefore, we can abstract these computations into a [State monad](https://edward-huang.com/scala/functional-programming/algorithm/2020/03/11/how-to-construct-an-immutable-queue/). However, that will be a post for another day.

## Main Takeaway
- The key to recovering referential transparency is to separate the concerns of the next state's computation with the communication of that state to the rest of the program.
- By making stateful API pure, we transition from one state to the next explicit to the caller and delegate that decision to the caller.

