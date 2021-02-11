---
layout: post
title: 4 Clean Code Principles for Functional Programming
date: 2021-02-10 23:52
summary: Your future self will thank you later for readable code
categories: programming scala functional-programming
tags: programming scala functional-programming
---

<img src="{{site.baseurl}}/images/4-clean-code-principles-for-functional-programming/Clean Code.png" alt="Clean Code"/>


There are many principles and design patterns when it comes to writing imperative code. For instance, the gang of four books mentioned 23 different ways to write a well-maintained object-oriented code. You must hear of the SOLID principles, the DRY principles, and many other acronyms for principles of writing clean code. However, some of the imperative code principles might not apply anymore when writing functional programming code. 

Don't get me wrong - SOLID and a gang of four design pattern principles are very useful in constructing readable code. However, software engineers can adopt regulations specific to functional programming for writing readable code - especially creating your code around pure functions. Since available programming promotes immutability and pure functions, the way you think about structuring your code is different from writing imperative code. 

There is a strategy pattern in object-oriented programming because there is no way in object-oriented programming for you to create a lambda function. I'll pull the classic shape class, where each respective shape will print the name of that shape.

<script src="https://gist.github.com/edwardGunawan/d016bb016a4cce21135c2703126cbc21.js"></script>


If there is a higher-order function, you can create a lambda function that passes it to `SomeClass`. The above strategy pattern can be condensed to a few lines of code:

<script src="https://gist.github.com/edwardGunawan/63e6dd5f129b4ac3b30e8f032c8eacfc.js"></script>

In this article, I want to share 4 general principles of writing readable code that you can imply when writing your functions. 

Disclaimer - these are not hard rules by any means, and if you have any other pattern that you see to write readable and better code, don't forget to share it in the comment section below!

There are KISS and DRY principles, which are sound principles to take from imperative to functional code. I won't mention it here since you can find it on many of the internet out there.

## Don't Nest Excessively
I have this problem myself. Because there is no state in functional programming code - most of the code a series of expressions, having multiple nested expressions seems right. It is easier to write when you are on the flow. Another thing about nesting excessively is that we tend to want to generalized things to make them reusable (DRY principles), but then it hurts the readability of the code. However, when you go back and read it, you often are confused about what happened in your code. Take a look at the below code:

<script src="https://gist.github.com/edwardGunawan/ba7b529b51a86f1ec7486478769be84a.js"></script>

In this case, we can refactor it smaller function with clear names like this:

<script src="https://gist.github.com/edwardGunawan/c8bd1279412dbd54ffdd94215555f761.js"></script>

Although you don't know what the function is doing, at least it is easier on the eyes.

Sometimes functional programming decomposes the code very minimal to a single line. One of the reasons is that that is the most basic primitive of the operation that cannot be implemented with some other operators. Try doing this exercise of trying to decompose functions into multiple smaller ones - you will see that your code can be more composable. 
 
## Don't Get Rid of Intermediate Variables
One of the gripes of other people think about functional programming, or Scala is people who love one-liner. The fact that each expression returns a value, we can chain them together to write our function. Although we need to type more keystroke, in the future, you might thank me later when there is some bug in your code. 

There are multiple generic operations in Scala that help you write code in one [line](https://gist.github.com/mkaz/d11f8f08719d6d27bab5). Let's take the above example. Inside `incrementByOneAndMultipleByTwoWith(x:Int)`, I don't get rid of intermediate value. It will look something like this:

<script src="https://gist.github.com/edwardGunawan/17bbba952df45e64e7d4f26bc02984a6.js"></script>


However, having an intermediate value can help you understand the code and faster to debug your codebase.

### Don't Use Pattern Matching If there is an Equivalent Standard Library for that Function
One of the reasons why pattern matching is cool is that you can literally do anything with it. If I want to get the value inside the option, I can do pattern matching to get the option's value. If I want to reverse an integer array, I can do pattern matching to change an integer array. 

However, its advantages also serve as its disadvantages. Because it is too generic, you broke the DRY principles by re-inventing the code that exists. Instead of using pattern matching to get value within an Option monad, you can get used `getOrElse` - which serves the same purpose. Instead of using pattern matching to reverse an array of integers, you can just use the standard library `reverse`.

Remember, not all rules are hard rules - there is also an exception when you use pattern matching. If the operation you want to apply hasn't been seen in the standard library, you _need_ to use it. If you play around with your function and pattern matching is much easier to read, you will pick pattern matching instead.


### Isolate your I/O
Writing functional doesn't mean that all of your function needs to be pure. Writing functionally is knowing how to push the IO to the end of the boundary. We want to isolate the impure calls to one place- usually, we said we like to call it at the end of the world. 

Some functional programming languages, like Haskell, are enforced within the language itself. However, when it is not enforced, try putting all the IO calls to the end of your program's boundary, or a little box can help make your code more maintainable. You suddenly can unit test your code quickly because those functions are pure.

Let's take one example:

<script src="https://gist.github.com/edwardGunawan/e4341917eb05fd1f5cb7b7ba919e0d79.js"></script>


We can further push the boundary of IO to the edge by separate out the IO and just return a String:

<script src="https://gist.github.com/edwardGunawan/1fd3762e71fd6f3588f4460357a4b1ca.js"></script>

Then we can call it like this:

<script src="https://gist.github.com/edwardGunawan/81ac185959001507e74ca8bb4046eac2.js"></script>


In the above example, we can test the `contest` function quickly because `contest` becomes a pure function right now.


That's it! I hope you get some guidelines and principles for writing readable, functional code. Remember, these principles are more like guidelines rather than rules. I hope you find this article helpful. If you find any hidden pattern or ways to create a more modular code, please let the world know by commenting on it below!
