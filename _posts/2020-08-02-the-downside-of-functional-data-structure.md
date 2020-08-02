---
layout: post
title: The Downside of Functional Data Structure
date: 2020-08-02 19:39
summary: It is not performant enough
categories: functional-programming scala stream tech algorithm
tags: functional-programming scala stream tech algorithm
---

![Photo by Kelly Sikkema](https://images.unsplash.com/photo-1507925921958-8a62f3d1a50d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1355&q=80)

Functional programming creates a lot of benefits in creating a distributed system. You will deal with fewer bugs in a concurrent environment because everything is immutable. It also helps us [local reasoning and composition](https://www.inner-product.com/posts/fp-what-and-why/) - we can construct applications and programs like lego building blocks that will produce a predictable outcome. With having a predictable outcome, we can understand code to scale with the size of the codebase. 

However, there is also a downside if you strictly make everything "pure" and immutable. For instance, in my [previous articles](https://edward-huang.com/functional-programming/scala/2020/04/14/how-to-implement-functional-list-in-scala/), having a functional data structure, such as List, can be [costly](https://www.lihaoyi.com/post/BenchmarkingScalaCollections.html). Let's take the below as an example:
```scala
List(1,2,3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2)
```

Each of these operations above will make its input and create a new intermediate List on the output. The expression above for the `map` function will introduce a new `List(2,3,4,5)`. Then, it goes to `filter` function and filters out all the odd numbers, which produces a new List, `List(2,4)` that gets a pass to the last `map` operation. Each transformation gets to produce a shortlist that gets used as an input to the next transformation and is then immediately discarded. If we manually produce a trace for its output evaluation, it will be something like this:

```scala
List(1,2,3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2)

List(2,3,4,5).filter(_ % 2 == 0).map(_ * 2)

List(2,4).map(_ * 2)

List(4, 8)
```

You must be thinking right now - the syntax is compositional and succinct, but the performance and memory usage are terrible. Ideally, we will want to fuse the sequence transformation on each element like this in one go. We don't want to wait for all the List to finished transforming the first element to have its second transformation. We also don't want to create a temporary list element in the middle of each change.

How do we do that?

We can change the structure to a while loop and for a loop. However, we want to persist in the compositional structure of the transformation rather than writing monolithic circuits. One way of doing this is to make your data structure _non-strict_ (lazy).


## What is non-strict (lazy)?

Before we talk about what is a non-strict, I want to explain what is strict.

The formal definition of strictness is - "If an expression's evaluation runs forever or throws an error instead of returning a definite value, we say that the expression doesn't terminate, or that it evaluates to _bottom_. A function `f` is strict if the expression `f(x)` evaluates to bottom for all x that value to bottom."

Simply put, if a function is strict, it means that function will evaluate all of its arguments. Let's take an example:

```scala
def add[A](a:A, b:A): A = a + b

add(1,2)
```

Function `add` will received an evaluated arguments of `1` and `2`. If we do something like this:

```scala
add(sys.error("something wrong"), sys.error("something wrong")) 
```
It evokes the arguments _first_ before function `add` has a chance to evaluate the value of its arguments.

Strictness is the norm in programming languages, and indeed most programming language support functions that expects its arguments to be evaluated.

Non-strictness means that the function can choose _not_ to evaluate one or more of its arguments first when evoking that function.

Non-strictness is widely used in programming languages, and you most certainly familiar with the concept. One example is when a short-circuit happens in a boolean operation, `&&` and `||`. Although the boolean operation is a built-in syntax within the language, we can think of it as a function that chooses not to evaluate the next one, depending on the result of the first evaluation. 

```scala
false && (1 + 1 == 2)
```

The following expression will not be evaluated because the first expression returns a `false`.

If you would like to create an operation of `&&` in scala it will be something like this:

```scala
def and(a: () => Boolean, b: () => Boolean): Boolean = if(!a()) {
  false
} else if(!b()) {
  false
} else {
  true
}

```

The argument that we pass unevaluated has a `() => ` syntax, usually, it is called a [_thunk_](https://en.wikipedia.org/wiki/Thunk). The `() => ` means that you pass in a function with zero arguments. Then, inside the function, `a()` evokes that function.

Since this has been widely used in Scala, we have more beautiful syntax, `: =>`, which is usually called _pass by name. 

Therefore, the above function can be translate to this:
```scala
def and(a: => Boolean, b: => Boolean): Boolean = 
if(!a) {
  false
} else if(!b) {
  false
} else {
  true
}
```

Note that these evaluated functions are not cached. Therefore, if you call them multiple times, it will get evoked numerous times. To memoized the value that is evaluated, you have to use `lazy val`. It assessed the value lazily once and memoized the result after.


## Lazy List (Stream)
Now you know the definition of non-strictness and Laziness. Let's solve the first example by re-create our List to "Lazy" ones.


### Intuition
We will first define the definition of the Lazy List. Then, we will create `foldRight` functions, the generic function of transforming values, and `foldRight` to create `map` and `filter` for our Lazy List. Lastly, I will explain how the programs flow and why it is much more performant to use Lazy List than the regular immutable List.


### Definition of Lazy List

Let's defined Lazy List:

```scala
sealed trait LazyList[+A]

case class Cons[A](head: () => A, tail: () => LazyList[A]): LazyList[A]

case object End extends LazyList[Nothing]

object LazyList {
  def cons[A](head: => A, tail: => LazyList[A]): LazyList[A] = {
    lazy val h = head
    lazy val t = tail
    Cons(() => h, () => t)
  }
  
  def empty[A]: LazyList[A] = End
}

```

The above implementation is a simple example of a Lazy List. The `cons` and `empty` is a smart constructor for caching any object instantiation. 

This is similar to the regular List, except everything is a thunk. This way, we need to force arguments to evaluate the function definition.


## Separate program Description and Evaluation
Now that you see how everything is non-strict, we can separate the description and its evaluation. In my previous article, [_How to Write Data processing Application in FS2_](https://edward-huang.com/functional-programming/etl/scala/2020/06/02/how-to-write-data-processing-application-in-fs2/), all values inside the Stream are a description of the program, until there is `unsafeRunSync`. 

A significant theme of Functional programming is about separation concerns. For instance, functions as first-class describe some computation in the body but will execute if we supply the arguments to those functions. We used `Option` to capture the program's description that there is some error that occurred. However, it leaves the decisions of what to do with those errors as its _separate concern_. With Lazy List, we build up the computation of a sequence of elements without evaluating them, only running those elements when we need them.

This has a significant advantage: we can describe an abundant expression more than we need and evaluate only a portion of it. You will see that in the example of `foldRight` below.

### Implement foldRight

Let's define the implementation of `foldRight`:

```scala
sealed trait LazyList[+A] {
  def foldRight[B](z: => B)(f: (A, => B) => B): B = this match {
    case Cons(h, t) => f(h(), t().foldRight(z)(f))
    case Empty => z
  }
}

```

One major difference with the regular `foldRight` in List is that the `B` is a pass-by name. It means that the function `t().foldRight(z)(f)` will be evaluated depends on the function `f`. In regular list, it will be a recursive call to `t().foldRight(z)(f)` until it hits base case. Over here, it will depend on how the `f` function is implemented. 

Let's take an example of defining a function `exists` which will check if the value exist in the List:

```scala
def exists[A](lst:LazyList[A], p: A => Boolean): Boolean = lst.foldRight(false)((a, b) => p(a) || b )
```

This function will traverse through the list until `p(a)` returns true. Once `p(a)` returns true, it won't evaluate `b` and short-circuit. We cannot do that with a strict function `foldRight` since it will recursively call `t().foldRight(z)(f)` until the function hits a base case.

### Implement Map and Filter
Back to the first example above:
```scala
List(1,2,3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2)
```

Let's define `map` and `filter` functions.

By implementing a generic recursive function `foldRight`, we can apply `map` and `filter` using `foldRight` in a non-strict manner.

```scala
sealed trait LazyList[+A] {
  def map[B](f: A => B): B = foldRight[LazyList[B]](empty[B])((a,b) => cons(f(a), b))
  
  def filter(f: A => Boolean): LazyList[A] = foldRight[LazyList[B]](empty[B])((a,b) => if(f(a)) cons(a, b) else b)
}
```

When we defined `map` and `filter` in a non-strict way, the implementation doesn't __fully generate__ the answer. Even when the function unfolds, it will just do enough work to generate the requested elements.

Let's trace layer on layer how our first example works:

Since we are using `LazyList`, we need to execute a `toList` function that will force evaluation on the subsequent `LazyList`.

```scala

sealed trait LazyList[+A] {
  // ... map, foldRight, and filter implementation here
  
  def toList:List[A] = this match {
    case Cons(h, t) => h() :: t().toList
    case Empty => Nil
  }

}

object LazyList {
  // all the smart ctor such as cons, empty is here
  apply[A](a:A*): LazyList[A] = if(a.isEmpty) empty else cons(a.head, apply(a.tail: _*))
}


```

Mimicking the above example with `LazyList`:
```scala
LazyList(1,2,3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2).toList
```

Tracing on how this program runs:
```scala

LazyList(1,2,3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2).toList

Cons(2, LazyList(2,3,4).map(_ + 1)).filter(_ % 2 == 0).map(_ * 2).toList

Cons(2, LazyList(2,3,4).map(_ + 1).filter(_ % 2 == 0)).map(_ * 2).toList

Cons(4, LazyList(2,3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2)).toList

4 :: LazyList(2,3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2).toList

4 :: Cons(3, LazyList(3,4).map(_ + 1)).filter(_ % 2 == 0).map(_ * 2).toList

4 :: LazyList(3,4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2).toList

4 :: Cons(4, LazyList(4).map(_ + 1)).filter(_ % 2 == 0).map(_ * 2).toList

4 :: Cons(4, LazyList(4).map(_ +1).filter(_ % 2 == 0)).map(_ * 2).toList

4 :: Cons(8, LazyList(4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2)).toList

4 :: 8 :: LazyList(4).map(_ + 1).filter(_ % 2 == 0).map(_ * 2).toList

....

```

Now, the `filter` and the `map` transformation are interleaved - it will add one with the `map` on that element, and test with `filter` to see if that element is divisible by 2, and multiply that output by 2. All the aspects inside LazyList are not divided by two; then, it will keep calling the next element until it creates a `Cons(a, b)`. The `toList` function is to force to evaluate the subsequent component if it is another Lazy List.

Note: If you run `val lst = LazyList(2,2,2).map(_ + 1).filter(_ % 2 == 0)`. Then, `print(lst)` it will evaluate the entire List without needing to foce evaluation on all the elements in the Lazy List.

The incremental nature of Lazy List transformation also has significant consequences on the memory usage. Because the intermediary List is not generated, the transformation only requires enough working memory to store and transform the current elements. Thus, the garbage collector can reclaim the space allocated for the values emitted by the map, and as soon as the filter determined, it is not needed.


## Takeaway
- By evaluating data structure in a non-strict manner, we can separate the description with its evaluation.
- Using the non-strict function, we can optimize the transformation and only transform the required elements in the List.

