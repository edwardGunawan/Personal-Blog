---
layout: post
title: Stop Using Pattern Matching To Compose Your Program, Use These Functional Operations Instead
date: 2021-02-02 16:33
summary: Uses these 4 functional operation to help you construct declarative programs
categories: scala programming
tags: scala programming
---

![Photo by Patrick Fore](https://images.unsplash.com/photo-1505678137082-8a4376f90c86?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1350&q=80)


When people think about functional programming, two things come to mind. The first one is usually pure function; the second one is immutability. While these two are correct, the key to thinking in functional programming is to think in a declarative way.

A declarative way of thinking about programs is where you express a computation logic without describing its control flow. It is also how humans usually speak. Let me give an example - when you are thirsty, and you want to drink water, you don't tell others than "I want to fill up water on a cup, and chuck it on my mouth"; you tell them that I want to _drink_ water. You are essentially declaring to other people _what_ your intention is instead of _how_ you wish to achieve your purpose. The _what_ is declarative.

On the other hand, thinking in terms of imperatively is when you think about _how_ to accomplish the computation logic. Your focus is on the state of the application - you have a list of commands for the computer to perform. Usually, we see this a lot when we look at recipe books. It doesn't tell you _what_ the intention, but _how_ you want to achieve that intent. Imperative programming focuses on _how_ the program should operate. 

In terms of writing programs, they have various ways of writing programs. A declarative way of coding programs tends to be more recursive. Conversely, the imperative form of writing programs usually requires loops. For instance, to get the sum of a list, you will have for loop and add each item on the array to a variable. The declarative form of doing this will be to looping each element's recursive way, and on each component, we want to add each other.


While most of us usually started imperatively writing code, I want to show the other way of how you can write your code to be more declarative so that your code can be more descriptive. 

This article will not talk about both paradigm's pros and cons but to give you an idea of how you can program your program declaratively. These are four functional operations that I often used as I got into the functional programming journey that you can use in your tool belt to write programs declaratively. I will not talk about `map` and `filter` since it has been widely used in other functional languages.


## Fold
Fold takes data in one format and gives it back to you in another. Any data processor can condense a lot of the operations in all these programs to fold. For instance, reversing a list - the imperative way will be to do a two-pointer and a for a loop.

<script src="https://gist.github.com/edwardGunawan/397c2594bebd0ecca8e4dc7aecdafee7.js"></script>


The declarative way will be using `foldLeft`, like this:
<script src="https://gist.github.com/edwardGunawan/154a69ca64e3e70cd0c58b21ec936b13.js"></script>


Fold takes two arguments; the start value and a function. This function also takes two arguments; the accumulated value and the current item in the list. Therefore, you can describe what you want to do on each of the iterations by specifying that you want to append the `curr` to the rest of `acc` - which reverses the existing list. 

What if we want to stop in the middle of the list. How can we represent a while loop? 

## Take and Drop
Take and drop operation is what you expected it would do. If there is a list of elements, and you said you want to take the first two. You can do it like this:

<script src="https://gist.github.com/edwardGunawan/a556689f0a5f4a85bf48468e16bff1da.js"></script>


Conversely, if you want to drop the first two and want to take the rest - you can use `drop`:
<script src="https://gist.github.com/edwardGunawan/a9a6a7e51151798c5991d69e79f7d9ae.js"></script>


Certain algorithm operation in Scala might be hard to implement without a recursion. For instance, if you want to split a list into two parts based on some index, one way to do it is using a recursion or a for loop. Like this:

<script src="https://gist.github.com/edwardGunawan/0d27493601374cbf0e6703b590a546ce.js"></script>

You can also `take` and `drop` to take the `index` value and `drop` the index value :

<script src="https://gist.github.com/edwardGunawan/e5d2305c004c70500736b6bca183b6f0.js"></script>

## Zip
The zip function is to merge two sequential collections into pairs. This operation is beneficial when you want to make two separate collections into a pair, especially in a stream environment when you want to join both streams or make both streams run simultaneously.

If you want to do a stream of infinite fibonnaci sequence, you can call it with a zip function:

<script src="https://gist.github.com/edwardGunawan/813f52513dd78a1640688740d581c2a9.js"></script>

The Fibonacci stream zips two-stream - one including the head of the list, and the other doesn't include the head of the list. In this case, fibs will start with `[0,1,1,...]` and `fibs.tail` will start with `[1,1,2,....]`. We combine the two-stream together - `[(0,1), (1,1), (1,2), ...]`- and add them together. The result will be a list of the Fibonacci sequence.

## Unfold
Unfold is the opposite of fold. Instead of recursively condensing the sequence into a single value, it reverses the logic by taking the initial state and building both the following condition and the next value in the sequence to be generated.  


<script src="https://gist.github.com/edwardGunawan/48f9f63b576319f0482954b8dbd5e3e9.js"></script>

A first glance at the method signature might make you think how to unfold could be possibly useful. However, unfold is a very generic stream building function. The unfolding concept is [co-recursion](https://edward-huang.com/functional-programming/algorithm/programming/scala/2020/08/02/wtf-is-corecursion/), which lets you start from the base case to slowly build-up to the next value. We can try to use the Fibonacci sequence from the `zip` example to construct `unfold`:

<script src="https://gist.github.com/edwardGunawan/cc30869af33266656ccfcca942949931.js"></script>

The above function takes in the initial state, a tuple of zero and one, and execute the process to "unfold" the sequence. In each iteration, the returned tuple of type `Option[(A,S)]` means:
1. The first element `A` is the value to be added to the resulting sequence - in this case, it will be zero.
2. The second element `S` is the next _state_ value - in this case, `S` is another tuple (one, zero+one)
3. A return of `Some` signals that the function generates the new element. A `None` signals the "termination" for the sequence generation operation


I hope you get to put these functional operations under your tool belts to write programs in a more declarative way. We are just scraping the surface of available functions. Other active operations have been instrumental in describing the intent of your plan. For instance, FlatMap is a combination of map and then flatten the nested sequence; dropWhile and takeWhile are both functional operations on top of taking and drop, including additional predicate; zipWithIndex will zip the sequence with an index as a tuple. These operations are more powerful and build in a more general way.  Both paradigms have pros and cons, and each has its use cases. Using these functional operations helps you make the intention of your program more composable. It lets you state what you want to do with your program. On the other hand, using an imperative way of writing the program can help make your program more modular -  it improves the program's maintainability. Now that you know this functional technique of composing your program stop using regular recursion and use these declarative functions operations to make your code more readable.

