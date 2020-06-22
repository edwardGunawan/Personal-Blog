---
layout: post
title: What is "effect" or "effectful" mean in Functional Programming?
date: 2020-06-21 23:28
summary: Hint - it is not side effect
categories: functional-programming scala monad
tags: functional-programming scala monad
---

![Photo by pawel szvmanski](https://images.unsplash.com/photo-1560419263-9032568994c9?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1351&q=80)

A lot of the time, when we discuss the effect, we usually talk about side-effect. However, as I study more and more into functional programming and reading more and more functional programming books, I noticed many times "Effect" or "Effectful" had been widely said in the FP community when describing abstract things. 

I dig a little deeper into what an "Effect" or "Effectful" means and put that in this blog post for a note to my future self.

## It is not Side Effect
Usually, what they meant for "Effect" or "Effectful" is no side effect (sometimes it does). It is _Main Effect_.

## It has something to do with Type Category
A type category is a Math Structure to abstract out representation for all the different fields in Math. When designing a program, we can think in the properties of that program before writing code instead of the other way around. For example, a function `sum` can be empty (identity law), has the property of combined operation and needs to be associative. (1+2 is equal to 2+1). We can characterize them as [_Monoid_](https://en.wikipedia.org/wiki/Monoid), and restrict input function to be a _Monoid_. This way, we can create a solution in a systematic approach that generates fewer bugs.

Within Type Category, there is [_Monad_](https://en.wikipedia.org/wiki/Monad_(functional_programming)), which is a fancy word for a wrapper that produces an "effect" on a given type. I will quote the statement that Alvin Alexander mentioned in [_Functional and Reactive Domain Modeling_](https://www.amazon.com/Functional-Reactive-Domain-Modeling-Debasish/dp/1617292249/ref=as_li_ss_tl?ie=UTF8&linkCode=sl1&tag=devdaily-20&linkId=031fc4db53f522208ba4bbea056be0f8&language=en_US):

1. _Option_ models the effects of optionality
2. _Future_ models latency as an effect
3. _Try_ abstract the consequences of failures

Those statements can be rewritten as:
1. _Option_ is a _monad_ that models the effect of optionality (of being something optional)
2. _Future_ is a _monad_ that models the impact of latency
3. _Try_ is a _monad_ that models the impact of failures (manages exception as an effect)

Similarly:
1. Reader is a _monad_ that models the effect of composting operations based on some input.
2. Writer is a _monad_ that models the impact of logging
3. State is a _monad_ that models the impact of State
4. Sync in Cats-effect is a _monad_ that models the effects of synchronous lazy execution.


## It is an F[A] instead of A
An effect can be said of what the monad handles.

Quoting from Rob Norris in Functional Programming with Effects - an effectual function returns `F[A]` rather than `A`.

For example:
```scala
def division(num1:Int, num2:Int): Either[Exception, Int] = if(num2 == 0) Left(new IllegalStatement("num1 cannot divide by zero")) else Right(num1/num2)
```

By looking at the code above, we know that without diving through the code, the function can have an "effect" of returning some Exception or integers based on the input type. By explicitly declaring what effect your function can return, other tasks that call `division` will need to supply a mechanism to handle those "effect" types, making your program deterministic.

<blockquote>
  <p>
    "I don’t remember where I read it, but someone said that if you want to think about it philosophically, when a function returns an A, that A has already been fully evaluated; but if that function returns F[A] instead, that result has not already been fully evaluated, the A is still inside F[A] waiting to be evaluated. So, rather than writing a function that returns a raw type, an effectful function returns a raw type inside a useful wrapper — where that wrapper is a monad that lets the result be used in a monadic style (i.e., in a Scala for-expression)."
  </p>
  <footer><cite title="What is it mean when you said effectful program?">Alvin Alexander</cite></footer>
</blockquote>


## Summary
- Effect can also mean the "main-effect" of your program.
- Instead of thinking about Side-effect, and effect is what the monad handles.
- An "effectful" function returns a `F[A]` rather than `A`


### Source
[Effect or "Effectful" Type](https://alvinalexander.com/scala/what-effects-effectful-mean-in-functional-programming/#:~:text=As%20mentioned%2C%20rather%20than%20thinking,monad%20that%20you're%20using%3A&text=Reader%20is%20a%20monad%20that%20models%20the%20effect%20of%20composing,models%20logging%20as%20an%20effect) 
