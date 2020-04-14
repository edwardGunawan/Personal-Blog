---
layout: post
title: How To Implement Functional List In Scala
date: 2020-04-14 17:07
summary: How to implement functional data structure in Scala
categories: functional-programming scala
tags: funcitonal-programming scala
---

![Photo by drmakete lab](https://images.unsplash.com/photo-1452696193712-6cabf5103b63?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1267&q=80)


Functional programming has been getting more and more popular these days. A critical distinction about functional programming from imperative is that you cannot update variables or modify mutable data structures. There can be a lot of <a href="https://alvinalexander.com/scala/fp-book/benefits-of-functional-programming/" target="_blank">benefits</a> on having all data immutable.

However, with great benefits, it also comes with some cost. For instance, it is a higher learning curve to adapt this paradigm into your team, and also another paradigm shift in how you define your data structure.

Many questions arise when we need to define a functional data structure - how do we define data structure and operate on them? How do we make every data structure pure?

In this article, I like to share how you can define a widely used data structure in scala, List, which is equivalent to a singly-LinkedList in Java. We start with the thought process on how you can think about data structure as immutable and proceed to define `next` function, which chains one node to the next in List.

## Intuition
The functional data structure operates on using pure function - it cannot change the data in place or do any side effects. Therefore, creating a functional list for an empty list (Nil) is the same for its immutability as an integer 3 or 4.When we do an operation on an integer, such as `3+4`, we get a new integer `7`. We didn't change the value of integer 3 or integer 4 because it is immutable. 

The functional List is the same. 

When we concatenate 2 list together, `List(1) ++ List(2)`, we will return a new List of `List(1,2)` without mutating `List(1)` or `List(2)`.

Does that mean there is much deep copy from the previous two operations to the new ones?

Surprisingly, it doesn't. 

Since every single element is immutable, we can reuse it. When we concatenate two lists together, we return a new list by having one list pointing to the other. It is called <a href="https://stackoverflow.com/questions/16430376/what-does-sharing-refer-to-in-the-implementation-of-a-functional-programming-lan" target="_blank">data sharing</a>. By doing so, we can implement functions much more efficiently by not having to worry about following code modifying our data.

Usually, the data structure is recursive - they have a base case, when the data structure is empty, such as `Nil` for empty List, `Leaf` for trees. Then, they have another type that represents the data structure that consists of value. In the case of List, it is `Cons`, which contains a single member variable and a function that connects the current Node to the next. In the case of Binary Tree, `Branch` is equivalent to the Node in the tree that has `left` and `right` that connects to the next `Branch`. We use this intuition to build our functional List.

## Implementation

How do we implement a List?

In Java, we create a class LinkedList and an inner class node that operates inside the LinkedList. The Node has a `next` members variable that points to the next Node.

We do the same when creating a functional List by creating the LinkedList, which is a `MyList`.

```scala
sealed trait MyList[+A]  {
    // create a right associative function
    def ::[B >: A](a:B):MyList[B]
}
```

We create top-level MyList that Empty List, `Nil`, and `Cons` will implements. The reason for creating `sealed trait` is to make our List pattern matchable later with `Cons` and `Nil`. Another reason is to create a sum type <a href="https://edward-huang.com/functional-programming/2019/12/30/what-is-an-adt-algebraic-data-types/" target="_blank">ADT</a>.


Let's create `Nil`:
```scala
case object Nil extends MyList[Nothing] {
  override def ::[B >: Nothing](a: B): MyList[B] = Cons(a, Nil)
}
```

And `Cons`:
```scala
case class Cons[+A](head:A, tail:MyList[A]) extends MyList[A] {
  override def ::[B >: A](a: B): MyList[B] = Cons(a, head :: tail)
}
```

Notice that both `Cons` and `Nil` both extend `MyList`. The head in `Cons` is equivalent to the value, Node, in Java, and `tail` refer to the next in the Node in Java.

Another observation from the above code is that there is a recursive nature when we are defining `::`. When we define `5 :: List(3)`, the `Cons`  recursively call `::` on the `tail` until it hits the base case `Nil`. `Nil` l create the `Cons` by attaching the parameter in `::` to `Cons(a,Nil)`. 

The recursive nature in List appears the same in other functional data structures as well. The base type,`Nil`, calls the intermediary type, `Cons`, and constructs the base element from the intermediary type. The intermediary function recursively is calling itself, and let the base type take care of the base scenario. Both of the types extend a trait that consists of the data structure.

Let's create `MyList` for data constructor of our new List: 
```scala
object MyList{
  def apply[A](a:A*): MyList[A] = if(a.isEmpty) Nil else Cons(a.head, apply(a.tail: _*))
}
```

Then, we can try testing our List in the main function:
```scala
object Main extends App {
  val a = 5 :: Nil
  val b = 7 :: 6 :: a
  println(b)

}
```

## Main Takeaway
- The functional data structure uses data sharing instead of doing a deep copy each time you do the operation with them.
- The data structure is recursive. There always be the base type and the intermediary type that connect the container to form a new structure.
- Usually, the base type creates its own by implement the function from the intermediary type. In the case of List, `Nil` creates `::` from `Cons`.

All the source code on this tutorial are <a href="https://github.com/edwardGunawan/Blog-Tutorial/blob/master/ScalaTutorial/functionalDS/src/main/scala/MyList.scala" target="_blank">here</a>.
