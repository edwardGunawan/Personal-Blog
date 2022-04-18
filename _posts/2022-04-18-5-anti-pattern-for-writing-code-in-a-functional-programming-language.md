---
layout: post
title: 5 Anti Pattern for Writing Code in a Functional Programming Language
date: 2022-04-18 11:06
summary: No 1. Nested Asynchronous Function
categories: scala functional-programming
tags: scala functional-programming
---

Functional programming languages have gained so much traction in these past few years. 

Many people can see the benefit of writing code that contains features such as functions as a first-class citizen. They embrace immutability in a concurrent environment, running heavy computed tasks without worrying about some concurrency issue, and love to write generic code to be as DRY as possible.

I saw this as a good sign that functional programming language is becoming mainstream again. However, one of the hard parts of writing code in a functional programming language is its design pattern and anti-pattern, which are different from a regular programming language.

I often see engineers writing in a large codebase that I classified them anti-pattern. I had also committed to these anti-patterns when I initially wrote a production-ready application in a functional programming language. Since then, I have read many [books](https://www.google.com/search?q=parctical+fp&oq=parctical+fp&aqs=chrome..69i57j0i13j46i13j0i13l5j0i22i30l2.4661j1j1&sourceid=chrome&ie=UTF-8) about functional programming design patterns and books that have helped me create a more maintainable code. 

## Overly nested anonymous callback function.

An anonymous function can be good for code-reusability. However, too much anonymous function can hurt the eyes of those engineers who want to extend the functionality. Although DRY is the way to go, sometimes duplication is better than the wrong abstraction.

I have touched codebase where engineers will write a heavily concise and abstract method. The code is something like this:

<script src="https://gist.github.com/edwardGunawan/3b4323b746f4b66818db9810999e9eb0.js"></script>

Can you tell me what the definition of `buildRunner` is?

Then, this `buildRunner` is used in all action operations like authorize, capture, and void in the payment processor. I look at it for two days to finally understand what it is trying to do. 

This creates an abstract to be as DRY as possible on all the functions you write. However, having a nested anonymous callback can be tough for regular engineers to build a new feature or maintain. Most of the engineers will need a couple of days to understand what `buildRunner` is trying to do. 

The good thing about functional programming is that you can look at the function signature and know what it is trying to do right away. However, this function doesn't explain much about what it does. It further confuses engineers trying to make a change in the codebase.

Therefore, a good rule of thumb is not to use an anonymous function is possible. Instead, use a higher-order function.

If you want to use an anonymous function, please put a `type` on the top to make it easier to read. [http4s](https://http4s.org/) does this internally by wrapping its type-ins `Kleisli`. `Kleisli` in itself is an anonymous function that is `A => F[B]`. However, wrapping the anonymous function with a `type` definition on the top helps readability in your codebase.


## Pattern Matching to the Extreme

The first thing that we learn about the benefit of writing code in functional programming is the pattern-matching feature - it eliminates the ugly if-else statement that we often use in the common programming language. 

Pattern matching is nice if you only have a shortcode based. Things become more like a callback hell when you have more than two layers of [pattern matching](https://www.bmc.com/blogs/callback-hell/).


<script src="https://gist.github.com/edwardGunawan/f077e0e94edcac9028f9ff1651f743ba.js"></script>

Often, engineers who are new to writing code with functional programming language don't know how many built-in higher-order functions the language provides. Therefore, they default their function implementation through pattern matching and recursive function. 

Having nested case expression and recursive implementation in your function implementation causes hard to read and understand code. It takes a long time in PR comments, and harder to find bugs if there is one in the implementation.

One solution to writing nested pattern-matching case statements is to only care about the success case and leave the error scenario outside the function implementation. Moreover, use the built-in higher-order function provided by the library or language, `map` and `flatMap` if possible. It gives better ergonomics to your codebase, and you can identify right away where that error is being handled. 

The beauty of expressing your types in the function definition is that your function implementation no longer needs to handle all error case scenarios - the type system can propagate those to the caller - enforcing the function up the stack to handle those errors.

## Using Monad Transformer on The Interface

Using Monad Transformer when encountering the nested effect is very useful. In the above scenario, a Monad Transformer is another solution to solve the heavily nested effect - it helps make your API composable. However, we should not expose Monad Transformer onto the interface because it makes our API tight to a specific Monad Transformer. 

Let's make a more concrete example. The below interface can be `Future[Either[Throwable, String]]` instead of `EitherT[Future, Throwable, String]`.

<script src="https://gist.github.com/edwardGunawan/f74d5d025f0881b54fbe674c1f8c9e36.js"></script>

All the function that wants to use `someFunction` as an API will also need to use `EitherT`. 

What if it is a series of functions, and we see some function returns an `OptionT`?

We will need to call `value` a couple of times to return to our effect `Future`, unnecessary wrapping. 

Alternatively, we should make `someFunction` returns a `Future[Either[Throwable, String]]` and let the effect detects what constraint you will need in your program.

<script src="https://gist.github.com/edwardGunawan/b7976b6be3a109c125244052e47d4f40.js"></script>

In conclusion, having the purest form of effect is better than a monad transformer, as it doesn't lock services that use the API to use a monad transformer.


## Returning a Boolean Value On an API
Many APIs may return a single boolean value to indicate one logic or the other. The classic example, taken from [Practical Fp in Scala](https://www.google.com/search?q=parctical+fp&
oq=parctical+fp&aqs=chrome..69i57j0i13j46i13j0i13l5j0i22i30l2.4661j1j1&sourceid=chrome&ie=UTF-8), is the `filter` function.

<script src="https://gist.github.com/edwardGunawan/8d51fc5313b036b4e533612edeb2997f.js"></script>

What does' filter' actually do if you look at the function definition?

If the Predicate evaluates to `true`, it will discard the elements in the list. On the other hand, it can also mean if the Predicate evaluates to `true`, it will keep the elements in the list. 

That is ambiguous.

There is also `filterNot` in Scala, which has the same function definition but has a different name. I often saw many bugs occur within these two functions because engineers oversee the difference between the two.

We can improve this by wrapping an ADT (Algebraic Data Type) around the Predicate with meaningful values.

<script src="https://gist.github.com/edwardGunawan/189b5dc7631be686f442262d9fdf2740.js"></script>

This ADT helps us create a more specific function signature like this:

<script src="https://gist.github.com/edwardGunawan/26e5044a5a2ce399b51d6b748f57de05.js"></script>

Whoever uses this function will understand if they want to either keep the element or discard them.

<script src="https://gist.github.com/edwardGunawan/8b9d13833c2122d6f2070d4642d2f09c.js"></script>

To solve this problem on the filter class, you can always create an extension method, `filterBy` from the scala List trait.

<script src="https://gist.github.com/edwardGunawan/87d8e7ab1f117cd725fd3e742f796b8f.js"></script>

The key to avoiding confusion in boolean values is to use ADT to give meaningful values and extend those functions with the ADT. Nevertheless, this causes more boilerplate, but you get less confusion and bugs around constructing your application. 

Wrapping all boolean returned API with an ADT may be overkill. Hence, you can wrap the boolean returned API with an ADT in the critical component and be flexible in the rest of your application. It is a matter of agreement with your team members.

## Using Generic Data Structure in Your Trait

This statement may be controversial as, in regular software engineering practice, an interface should be as generic as possible to be extendable. It sounds great in theory but not in practice.

One example is `Seq` - a generic representation defined in the Scala standard library. It is very generic that `List`, `Vector`, and `Stream` all extend from `Seq`. This is a problem because each of these data structures behaves differently.

For example, we have a trait that returns a `Future[Seq[String]]`:

<script src="https://gist.github.com/edwardGunawan/bee566016e680a0df01837068ecb4bbd.js"></script>


Some engineers will call the function `fetchAll` and convert the `Seq` into a `List` with the `toList` function.

How do you know that it is safe to call `toList`? The interpreter may define the `Seq` as a `Stream`, and in this case, it will have a different semantic, and it may throw an exception on the caller's side.

Therefore, to decrease the number of surprises on the caller side, it is best to define a more specific type, such as `List`, `Vector`, `Stream`, depending on the application's goal and performance.


## Conclusion

The problem with these anti-patterns is that it is not an anti-pattern in a generic programming language. 

For instance, we are taught that writing abstraction is good, and it keeps your codebase DRY. However, the overly nested anonymous callback function can be hard to read. The best way to solve this problem is to duplicate the code to increase readability. API that returns a boolean value may not be a problem, and it exists in a lot of the API designs and applications. Nevertheless, implementing an API that returns a boolean doesn't give any clarity in terms of what those boolean mean. Furthermore, human minds often oversee small details in the documentation that can cause bugs in the implementation. 

Pattern matching is a powerful feature in the functional programming language, but it is overly generic. If you can find a better higher-order function to implement the function, you should use that instead.  

An overly generic data structure can increase ambiguity in the use of the API. Therefore, it is best to create a more specific type and make the function declaration as clear as possible to the call side.

I hope you avoid this anti-pattern when writing code in a functional programming language. Do you think these are anti-patterns? What is another anti-pattern you can think of when writing code in a functional programming language? Please comment them down below so I can also learn from them! 
