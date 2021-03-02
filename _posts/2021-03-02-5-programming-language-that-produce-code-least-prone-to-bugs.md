---
layout: post
title: 5 Programming Language That Produce Code Least Prone to Bugs
date: 2021-03-02 08:27
summary: Developer tend to write less bug prone code on these languages
categories: programming software-development
tags: programming software-development
---

![Photo by Markus Orth](https://images.unsplash.com/photo-1599900466892-13839a66b212?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=635&q=80)

Bug usually occurs when there is a mismatch between what we expect the program to behave vs. the program actually acts. 

Most bugs aren't the fault of the language itself - instead, they are the fault of developers making mistakes in using the language. However, some programming languages have been designed for use not only for writing readable code but also to help a developer do the right thing. For instance, Tony Hoare said that inventing null has been a "[billion dollar mistake](http://lambda-the-ultimate.org/node/3186)". He said that he is trying to design the first comprehensive type system for references in an object-oriented language but couldn't resist the temptation to put in a null reference because it is easy to implement. The `null` cause numerous errors, vulnerabilities, and system crashes. More recent programming has used many declarations with non-null references, such as Option type in Scala.

There are a couple of programming languages that I think are less prone to bug. These programming languages all have these characteristics that increase developers' productivity and decrease developers' chance of writing error-prone code. 

These are a common feature in the language and its tool that helps decrease error-prone bugs.

## Type Safety
The more compile-time checking, the more bugs will be caught by the compiler- so in general - type safety is a good thing. If a program is evaluated in run-time, there are more unexpected errors occurred.

## Expressiveness
If the code is shorter and has less boilerplate/ incidental complexity, it's easier to spot a bug. If you require to write 100 lines to express an algorithm, your algorithm will usually be harder to read and hard to spot the bug.

## Immutability
A lot of hard bugs are due to complex interactions with mutable states. Languages that emphasize or advocate immutability usually tend to have fewer bugs produce by developers.

## Great Concurrency Support
The most challenging bug in programming is those bugs that cannot be reproduced. One of the reasons it cannot be produced is that it is unpredictable.  Concurrency problems are tough to detect and debug. Anything that requires manual locking is ultimately doomed to fail (this includes every OOP approach to concurrency).

Below is the list that I think the programming language that makes programmers write less error-prone code.

## Haskell
In the 80s, functional programming language might have been elegant, but it was treated as an impractical language. Many of the available programming languages don't have an open-source framework. Therefore, a group of researchers formed a committee to design and implement a new language used to research and teach functional programming - and Haskell is born.

Many people think that Haskell is a "safe" language because it provides a rich set of tools for writing correct code, coupled with defaults that encourage safer patterns. One of the benefits of the writing program in Haskell is its functional programming aspect and a vibrant type-system. 

All Haskell variables are default lazy evaluated. Therefore, the code that you write is predictable without having unexpected eager execution in the run-time. 

The most significant impact in Haskell is how it handles non-local effects. It helps you consider _every_ effects that you put in your function, backed by tools and libraries that let you express your effect management explicitly. If you don't know what an effect is, check out [this](https://edward-huang.com/functional-programming/scala/monad/2020/06/21/what-is-effect-or-effectful-mean-in-functional-programming/) article. 

When you write things in Haskell, you keep track of all your effect in your program - and separate pure domain code wrapped in an imperative layer handling input and out- which leads to less buggy code and unexpected results.

Haskell is a statically typed language and is very expressive. The language provides a facility through its type system to talk about effect and organization in the language. That means you can write your code based on the type category's principles and laws and make it an abstraction in your codebase. These abstractions help write more generic code and avoid writing primitives in your data type that can cause bugs and errors. Many libraries in other programming languages, such as refined types in Scala and other best conventions from other languages, come from Haskell. 

## Ada
In mid-1970, when the US Department of Defense and the UK Ministry of Defense sought to replace hundreds of specialized programming languages for embedded computer systems. Ada was designed as a single language to [run](https://hackaday.com/2019/09/10/why-ada-is-the-language-you-want-to-be-programming-your-systems-with/) the system in all the embedded military systems that performed the same or better performance reliability.

After that, they have targeted their language as a general-purpose language - by adding OOP while not losing the core values of reliability and maintainability.

Ada's current application includes military hardware, avionics, air-traffic control systems, and many satellites where small glitches can cause a significant impact.

Ada is designed so that bugs can be caught at compile-time as much as possible. That means it often takes longer to get a program of Ada compiles than Java. However, when it compiles, you can feel confident that whole classes' bugs will not manifest themselves when the program runs. 

Ada type system is strongly typed - it doesn't allow any form of implicit conversions. Therefore, the programmer must explicitly name closing statements for blocks and state the variable's range. For instance, if you want to take a bit of C code like this:
<script src="https://gist.github.com/edwardGunawan/59cd1f3757b3b7759f758bd3c28a1e36.js"></script>

If the code above is directly translated to Ada:

<script src="https://gist.github.com/edwardGunawan/08db2087f4f51e456a6a21dd6bc59163.js"></script>

You will see a nasty error- "Integer" and "Myint" are not the same. 

One of the benefits of this is fewer surprises - one variable who change the type definition, later on, will not suddenly make a thousand implicit conversion to explode throughout the codebase.

## Ruby
You must be thinking, "Ruby doesn't have the description character description that you describe above. How is this in the list?"

One of the reasons why Ruby is on the list is because I think its philosophy and principles helps human write less bug-prone code. 
 
One of the reasons Yukihiro Matsumoto created Ruby is for the purpose of "making developers happy". Therefore, the program includes high-level developer-friendly that is easy to understand in human language and some sound principles, such as Least Astonishment principles.

Although object-oriented language tends to produce more bug-prone in a concurrent environment, if we want to write simple logic into machine language, Ruby's high-level language can help clarify and understand syntax. It is a transparent language that can directly translate human ideas to code.

The founder created Ruby with these principles in mind - Principle of Least Surprise. Things work the way you would expect them to, with very few exceptional cases or exceptions. 

Ruby doesn't obscure the solution you write behind lots of syntax and the need to import on external libraries to get simple things done. With Ruby, you can write programs close to the problem domain rather than always mapping your ideas and designs down to low-level language. 

What Ruby gives is predictably creating a program.

## Scala
Martin Odersky, the found of Scala, wanted to design a language that is better than Java. Although Scala is a programming language with a mix of object-oriented and functional, it strongly advocates functional programming as the defacto in its source code.

Immutability is by default the standard and encouraged if you are writing code in Scala. A lot of the Scala community has also gone towards the Haskell way of writing code. A couple of popular libraries such as Cats and FS2 adopt functional programming in writing highly concurrent applications.

Scala has a strong and static type system that unifies algebraic data types with class hierarchies that enable types and path-dependent types to apply calculus to a concrete language design.

Many big data applications like Hadoop and Kafka are created with Scala because Scala has excellent concurrency API support, which is the key in parallelizing the processing of the large data set.

Duolingo (a famous language learning app) migrate their session generator feature to Scala because it is less prone to run-time [bugs](https://blog.duolingo.com/rewriting-duolingos-engine-in-scala/). Scala static typing makes it easier to catch bugs with a compiler. It is less verbose than Java, which makes it easier to work with the application. Moreover, the Scala programming language is scalable. Before they re-write their application in Scala, their service had an average downtime of 2 hours every quarter. After switching to Scala, the first few months bosted zero downtime.

There are many more case studies on companies that benefit from implementing sala. However, writing Scala code is less prone to errors because of its type-safe feature, advocating functional programming paradigm, and managed memory resources.


## Rust
Rust is the new language that has been voted as the "most loved" language in the StackOverflow developer survey. One of Rust's feedback is that its static type, ownership mode, and dependency management have increased developer productivity.

When you program with Rust, one of the great things is that interacting with error handling messages is robust enough to tell you what error you might not have handled yet.

For instance, if you write data that is reading from a string of `data.txt`, like this:

<script src="https://gist.github.com/edwardGunawan/832ccbc56cf0005b1f65045ed2973bda.js"></script>


You will get an error message like this:
```
error[E0308]: mismatched types note: expected type `std::string::String` found type `std::result::Result<std::string::String, std::io::Error>`
```

The above code will give a compiler error because the `fs::read_to_String` method may fail with an IO error: for example, the file may not exist, but the program is not handling this error case. Other compilers usually don't have error handling up-front - it is often tracked as issues arise. 

Rust uses the functional programming mantra of "making illegal states unrepresentable". This eliminates a whole class of bugs that could show up as "unchecked "exceptions - such as "NullPointerException", "IllegalArgumentException", "ConcurrentModificationException", and more. Rust will catch these on the compile time. These exceptions are usually the exception that programmers made when writing code rather than network failure. 

## Takeaway
Each programming language is designed for various purposes. Some languages are designed for easily used, while others are designed based on scalability purposes.

The more powerful the language type, the more bugs will be caught at the compile time itself. 

A language designed in concurrency can help decrease the amount of bug that is hard to find. 

No matter how good a programming language is in identifying a bug, it can only guide us to write code according to its philosophy. Ultimately, the developer will still need to be preventive on all possible scenarios to produce a robust application.

These are 5 of the programming languages that I think, from my experience, are produced less prone to bugs. If you have any questions or other programming language that you feel can top these five, please comment on them below, so the world knows :)


## Resource
- [What is the safest programming language? - Quora](https://www.quora.com/What-is-the-safest-programming-language)
- [What programming languages generally produce the least buggy code? - Quora](https://www.quora.com/What-programming-languages-generally-produce-the-least-buggy-code)
- [A Large-Scale Study of Programming Languages and Code Quality in Github](https://cacm.acm.org/magazines/2017/10/221326-a-large-scale-study-of-programming-languages-and-code-quality-in-github/fulltext)
- [Why is Haskell considered to be safe? - Quora](https://www.quora.com/Why-is-Haskell-considered-to-be-safe)
- [Programming Language Safety Score Mark 2 · deliberate software](http://deliberate-software.com/safety-rank-part-2/)
- [Why choose Rust?](https://www.passfort.com/news/why-choose-rust)
