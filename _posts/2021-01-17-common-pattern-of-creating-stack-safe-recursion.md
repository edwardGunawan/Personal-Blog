---
layout: post
title: Common Pattern of Creating Stack-Safe Recursion
date: 2021-01-17 20:55
summary: Create your recursion in a stack safe way with Tampolining
categories: functional-programming programming scala optimization
tags: functional-programming programming scala optimization
---

![Photo by Iza Gawrych](https://images.unsplash.com/photo-1586010135736-c16373adf060?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1305&q=80)

Recursion is often used when we want to compute combination values and other iterative values. Having a recursion in your function can decrease the amount of code that you write in your function. However, a lot of the time, recursive calls can be hard to deal with when you have extensive input. 

Let's take an example of calling Fibonacci. 

<script src="https://gist.github.com/edwardGunawan/d01b2f247a06a0a43eadceb65b68bea6.js"></script>


Having a recursive call of `fib(n-1)` and `fib(n-2)` is readable. However, when we input `fib(1000000)`. It throws stack overflow.

In a regular recursive call, we can optimize the value by making it tail-recursive. Usually, in scala with annotation of `@tailRec` to tell the compiler to static check if the current function that you wrote is tail-recursive or not. 

The problem with this is that tail-recursive sometimes is hard to construct, and some function with just regular code and cannot complete tail recursion. Consider creating Fibonacci in a tail-recursive manner.

This post is inspired by the book _Functional Programming in Scala_ by Runar Bjarnason. He tried to explain the `IO` and give some examples of creating a generic tail-recursive value for the function that we construct. This kind of tail-recursive call is called trampoline.

In this article, I would like to walk through why it is called trampoline and how we construct this trampoline function, with the example of Fibonacci, to make all function tail-recursive. 


The trick for making a stack-safe recursion is to move your recursion from the stack to the heap space. Basically, instead of letting the JVM run a recursion function, pushing the new frame to the pile each recursion is performed, we want to write the recursion so that, rather than the JVM, we have control of the execution.


In the above Fibonacci case, calling the `fib` itself gives that execution control to the JVM. It pushes the `fib(n-1)` to the call frame because we will need to calculate `fib(n-2)` and evaluate that value. If we simply [_reify_](https://en.wikipedia.org/wiki/Reification_(computer_science)#:~:text=Reification%20is%20the%20process%20by,a%20non%20computable%2Faddressable%20object.) our implementation instead of calling a function, it constructs a data type. 

For instance, instead of calling `fib(n-1)` we can try to wrap the function into a data type `Suspense(() => Function[Int, Int])` data types that will hold this value. Essentially, we are constructing a data structure instead of calling a function.

If we want to wrap our function call with `Suspense`, our return value will also need data types.

Essentially, the function call will be something like this:

<script src="https://gist.github.com/edwardGunawan/70f2979e39e0f0e0d871ca06e58d7b74.js"></script>

We will have a `Trampolining` type constructor that wraps our result value. 

If we return `Trampolining` on each function call, how do you actually chain those results together? 

The original values recursive value for `fib` function just return an `Int`.  To illustrate what I mean:

<script src="https://gist.github.com/edwardGunawan/5a2b79b5d784cf022dc98251969dbbd0.js"></script>


It works when we do `fib(n-1) + fib(n-2)` because both return an `Int`, and we can do an addition on it. However, if we return a `Trampolining` , how can we do _chain_ the result of `Trampolining[Int]` and `Trampolining[Int]`?
 
This sounds like a `map` or a `flatMap` if we want to sequence our computation together.
 
Our function will look something like this:
 
<script src="https://gist.github.com/edwardGunawan/fea1f195f6b8bdb4b89b91dea0f4e0da.js"></script>


However, we also reify `flatMap` and `map` to construct data types instead of calling the function `f`. We just want to wrap it into the data so that it can be built in the Heap.

Therefore, our data type `Trampolining` will have to have `flatMap` and `map` functionality.

Before I explain further on how we can make this tail recursive with `Trampoloining`, let's construct what `Trampolining` should look like:

<script src="https://gist.github.com/edwardGunawan/5d6caa0bfcd3feccaebf348147a2fe8e.js"></script>


The constructor will have `Return`(base case, which will return a single value), `Suspense` (suspending the function to execute them later), and `FlatMap` (to chain operations). 

We can re-write our existing recursive function in the following manner:
- If the original function returns an `A`, the new function should return a `Trampolining[A]`
- Each `return` value in the original function should be wrap in `Return`
- We want to `Suspense` the recursive call so that the program doesn't push to our stack frame
- Things that we do after the recursive call, such as the addition of `fib(n-1)` and `fib(n-2)`, will be wrapped in the `FlatMap`.


## New Fibonacci Recursive Call
Our fibonnaci recursive call right now looks like this:

<script src="https://gist.github.com/edwardGunawan/0f6c61ec4813d2fd201567d118f15eb4.js"></script>



## The Call to Tail Recursion

Now, the purpose of making this into a data constructor is to delegate the control of the program to the programmer. We then can create an interpreter to interpret this control flow.

The interpreter is where we use tail-recursive call on our program. 

Let's create our interpreter:

<script src="https://gist.github.com/edwardGunawan/7fd99a4a82c599998cb1780330111bbd.js"></script>


Note that in `Suspense` under `FlatMap` we need to wrap it with `FlatMap` to call the `f` function to execute that `flatMap` once the function call is finished. If you just create `run(x())` like the above, it will not execute the function after getting the value of `a`.

## Running through the Code
Let's execute our code with `fibTailRef(5)` and walk through what happened internally.

When calling `fibTailRef(5)`, it goes to the `else` statement. Through substituting our code with its definition, we can get something like this:

<script src="https://gist.github.com/edwardGunawan/d18ae95ddc075ed2b53c3c92218b8ff1.js"></script>


When we execute that constructor with our interpreter, `run`, it will go to the `FlatMap` - `Suspense` case:

<script src="https://gist.github.com/edwardGunawan/27b3fa2740673a87b51ce9c65e4ffa2a.js"></script>



It will execute a `FlatMap(a(), f)`. Therefore it will execute the `a()`:

<script src="https://gist.github.com/edwardGunawan/cfca6d4a9e5e0a57fd7c68aa36808b3e.js"></script>


At this point, we will evaluate our function `fibTailRef(4)` which gives us:
<script src="https://gist.github.com/edwardGunawan/378d85e3d5a18fb02c8b5cb117870dec.js"></script>

We substitute the definition of `fibTailRef(4)` above to the `run` method in graph 4:

<script src="https://gist.github.com/edwardGunawan/5f0eb155ed58e77a42004caf819a28f9.js"></script>


At this point, the `FlatMap` case is executed underneath the `FlatMap`. In this case, we re-associate our function from `a.flatMap f flatMap f1` to `a.flatMap(x => f(x).flatMap(f))` so it is tail recursive. The reason for this is because `a` will keep recursively going through until it hits base case `Return` or `Suspense` from the first two pattern match case, and it will call the `f` function after. 

This won't change the result because we are leveraging the associativity of the Monad's law.

The above, graph 5 after being called becomes like this:

<script src="https://gist.github.com/edwardGunawan/c5d8791b250150ac578e245aa31cd83d.js"></script>

Phew! It took me a while to trace through this. Hopefully, you get my point. We can recurse through the end of the value, which will have a `Return(1)` and `Return(0)`. Then, it does the same with the inner function `f`.

This structure has the same form as our call stack, except it is created in the Heap.

## Why trampoline?

As you can see, our stack doesn't grow and grow. We first call the `run` function, which pushes it to the stack. It will then go up to `FlatMap` for the first time by creating another `FlatMap` construct. When all the `Suspense` values pop the stack and go to the `fibWithRef(4)`. Then it goes to the `run` function. Then, it goes again to the `fibWithRef(3)`. Then it goes to `run` also. Until it constructs the values. 

We keep jumping back and forth from `run` and `fibWithRef`, like a trampoline. 


## Conclusion
The `Trampolining` data constructors that we created works for a lot of the recursive scenario. We wrap the value of our return statement to `Trampolining`. We wrap `Return` data constructor on the base case value that we return on our regular recursion call. Lastly, we do `flatMap` on the operation that we want to do on the values.


This has been used a lot that Cats has an Eval monad to construct values in a stack safe way. You can check out my previous [post](https://edward-huang.com/tech/scala/programming/functional-programming/2020/01/12/demystified-scala-eager-lazy-memoized-how-cats-eval-can-safe-your-recursive-stack-for-overflowing/) to see how we can use Eval to construct our recursive function.

If you want to see the source code, you can see them [here](https://github.com/edwardGunawan/Blog-Tutorial/blob/master/ScalaTutorial/tarmpoline/src/main/scala/Trampolining.scala).

Lastly, if you are interested to learn more about trampolining, here are a few reference material to know more about:
- [Stackless Scala with Free Monads](http://blog.higher-order.com/assets/trampolines.pdf)
- [Recursion and Trampolines in Scala · GitHub](https://gist.github.com/eamelink/4466932a11d8d92a6b76e80364062250)
- [How Trampoline Works in Scala](https://free.cofree.io/2017/08/24/trampoline/)
- [Trampolining and stack safety in Scala by Oleksii Avramenko](https://medium.com/@olxc/trampolining-and-stack-safety-in-scala-d8e86474ddfa)
