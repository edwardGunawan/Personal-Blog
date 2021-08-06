---
layout: post
title: Functional Programming has made My Job Easier as a Software Engineer. Here's Why.
date: 2021-08-05 19:41
summary: Type level system able to let me sleep well at night
categories: functional-programming scala software-development programming
tags: functional-programming scala software-development programming
---

<img src="{{site.baseurl}}/images/functional-programming-has-made-my-job-easier-as-a-software-engineer-here-s-why-/why-you-should-learn-functional-programming.png" alt="Profile Pic"/>

"We have 10 minutes left. Do you have any questions from us?" At the end of our technical interview session, I asked one of the interviewers. "yes, I do," he replied. "I saw your company's blog post and see that you guys write code in Scala instead of other languages. What is the reason for that?" 

I thought about it and said, "We use Scala because Scala has the Functional programming features that we need to scale our business." 

He waited for a while, processing his thought. At that moment, I thought I didn't clearly explain that functional programming is a huge practice in our team. Then, he asks, "What is good about functional programming?"

I waited before answering this time, thinking through in my head, and give a generic explanation that Function Programming (FP) feature such as immutability and no-side effects helps us reason our code better in a concurrent environment. Then, I explained the basic terminology of FP. However, during that time, which is a year ago, I didn't understand what it means to write code in a functional style and how functional writing code helps my work be more productive.

More developers use languages with an explicit bias towards FP, such as Scala and Haskell. In contrast, OOP languages and their communities adapt the features and practices (Think about React and Typescript in the Javascript community). However, when I first wrote functional code, limiting myself to immutability in the code neither helps with my productivity in producing quality code nor creating a readable codebase (think about needing to do a recursive function to loop through a list of elements). Besides, developers have a hard time learning all the FP abstract features while having a project deadline.

Immutability and side effects are the first thing that I exposed when learning about FP. However, that is not the reason real reason why the team adopted FP. 

When I started down the FP route, this was my view, but now I understand why they enforce these features to developers - for local reasoning and composition. Ironically, writing FP code makes me more productive and makes my job easier, and the language features and programming style are just in service of these goals. In this post, I attempt to explain why functional writing code makes your job easier.

## Addressing all your Exceptions Explicitly
Although it is a debate that static typing increases productivity, having type safety is one of the best things to mitigate errors and surprises during run time. I didn't understand how helpful type safety was until recently when I tried to debug one of my colleague's codebase. He asked me how he keeps getting runtime exceptions such as null pointer in one of the microservices. Our services are written in Scala. Therefore, getting a runtime exception is not as common as writing code in Java. 

It has various error types that the developer needs to handle. I keep scanning through the entire codebase multiple times, running the same failed test cases, and trace through the bug. Three hours later, I realized that he didn't handle any exception when getting one of the AttributeValue in DynamoDB. 

That one single case causes three hours of resources in the team. The bug is often encountered when we transform Java code into Scala. You cannot see that the return value of function definition may return null, and a developer will need to constantly remind themselves to check for null cases. A type-safe way of writing is to return an `Option`. Declaring an `Option` type in your return function indicates to the caller that the returned value may be empty. Therefore, the caller needs to handle any empty case during compile-time, avoiding any surprises such as NullPointerException in the run time.

Let's take an example of getting items from a database and then parse the underlying item in the database. 

<script src="https://gist.github.com/edwardGunawan/fb1b33b2a7c1bb8d9e43f7089da8b5fb.js"></script>

An experienced developer will know right away that there are at least two types of errors, network errors, and parsing errors. Network errors may cause when the database is down, and the parse exception may cause by the model mismatch. However, these error is hidden to the caller - we need to look into its implementation and "guess" what sort of error it has. 

Error handling in this scenario is usually a `try` and `catch` error, and we need to think about what sort of error is possible in the `catch` block.

It is not required to handle all errors. Thus, you cannot see how complex the function above is and may miss important errors that will crash your program at night.

An aha moment for this is that exceptions are a form of hidden complexity. We can to explicitly tell these two exceptions to the caller as an "effect." Let's see what it will look like in the below code snippet:

<script src="https://gist.github.com/edwardGunawan/2329d731eb4d6e31b6c97153f998d037.js"></script>

I won't go into the implementation since we can determine what error will occur in this function by looking at the function definition itself. The first outer value is `IO`, which means that this function will have some side-effect. Having `IO` gives a cautious to the caller that the function may return an error, which is a network error. Secondly, we have an `Either` of `ParsingError` or the `Json` value that we want. `Either` is a data type in Scala or other functional languages representing that the result can either be a `Left` or a `Right`. In convention, a `Left` is the error case, and the `Right` is the caller's result. 

You see, right now, the caller is "forced" to handle these scenarios. They cannot retrieve `Json` without handling the `Left` case. They will also know what types of errors occur during the "compile" time to handle those errors based on their application logic.

More importantly, type safety helps you forced to handle corner cases. It makes a simple function more complex because it made everything explicit rather than hiding it.


## Reason our Program Flow logic in Compile Time
Have you ever wished to write your program all at once without compiling, and everything works when you compile your program? Writing functional code makes you do that.

One of the sins of writing functional code is to have a side effect. The two mantras that I have to remember when I learn functional programming are that side-effects and mutability are a sin. If functional programmers spot a mutable reference in a Scala code base, their palm starts to sweat. They started to have shortness of breath, and they start scanning through all the files multiple times to figure out where the surprises of the code may be. 

However, after two years of learning functional programming, I realized that it is not that functional programming discouraging side effects or mutability. Still, it is to declare the intent of your program as much as possible in the function definition. There are various effects in functional programs. (If you don't understand what I mean, I wrote an article about what effects mean in functional programming [here](https://edward-huang.com/functional-programming/scala/monad/2020/06/21/what-is-effect-or-effectful-mean-in-functional-programming/). 

Returning an `IO` in your function tells the developer that the function is doing an IO call. Thus, some side effects will occur during the run time - a high possibility of "unsafe" code.

Every functional code you wrote is a description, and you will need to initiate that call at your main function. We called it the end of the world. This notion of "description" tells the developer what the program expects to behave in the compile-time and guarantees the developers that it will behave as intended.

One example is `IO` vs. `Future`. `Future` is eager evaluated. Therefore, instantiating a `Future` will automatically start an execution context and fire another thread. On the other hand, `IO`  is lazily evaluated. Thus, creating an `IO` will not automatically run the function inside until you call `unsafeRun`. A function with different behavior may have the same result with Future, but not with IO. Let's see the code snippet below:

<script src="https://gist.github.com/edwardGunawan/cc2f5376824f938bb82c7c9142de1168.js"></script>

What does the above code do? We want to fetch the database twice. However, when we evaluate with `Future`, it only resulted in fetching the database once. On the other hand, instantiating with `IO` only describe what the program _will_ behave. Then, the `unsafeRun` will execute that program.

<script src="https://gist.github.com/edwardGunawan/8c6c9d14c4f2bc4fb02b44c137c5933f.js"></script>

The above code will fetch the DB twice. Writing a program as a description instead of a function execution helps us figure out how the program runs in compile time. 

## Decreasing the number of lines of code in The Program with Abstraction
We did this by thinking about the "laws" of what we want our type to behave. Usually, this is where all the type category systems such as Monad or Functor came. 

You will not need to implement a different variation of your function repeatedly and use the most minimal primitive for all your program logic. You start writing function in terms of what kind of "laws" or behavior you want that function to be. For instance, you can make a function that enforces the passing argument to be associative. It means that we can be sure when running `combine(a,b)` will be equal to running `combine(b,a)`. 

With constructing your function based on laws, you don't have to be worried that the caller of that function will supply types that are not associative. Thus, you can implement the function in any order you want, and it is very helpful when running any concurrent programs or map-reduce kind of job. 

This associative law in category theory is called Semigroup. Therefore, you can have a function that the arguments will need to abide by the Semigroup law. 

<script src="https://gist.github.com/edwardGunawan/361d29450fa9170d101813c43851af54.js"></script>

The more you write functional code, the more you will look at your code in building blocks. You often see how functional programmers are obsessed with creating the most primitive possible. What I mean by primitive is the most generalized form of a function. For instance, if you want to write a program that checks if a certain boolean value is true, you must do a certain action. In general, this can be done with an "if-else" statement, like this:

<script src="https://gist.github.com/edwardGunawan/6bc81128b5e587fefcebccb67a42d0aa.js"></script>

However, this function is pretty limited. What if I want to have multiple branches of action based on certain conditions instead of two ways? If we have this function, it will be much powerful than the function with a boolean statement. Therefore, thinking about it, we can use a List where the index will replace the `checkStatement`. Then, based on that index, we can execute some values:

<script src="https://gist.github.com/edwardGunawan/0a7cf06d34d72c81d8fdf18faf030764.js"></script>


Now, we can use `chooseN` to implement `choose`:

<script src="https://gist.github.com/edwardGunawan/ebca0363835f0a83c9eb7adabd4a2461.js"></script>

`chooseN` is not the most primitive possible function. You can make it more generalized and realized that you could use `map` or `flatMap` to implement these functions. However, I'll leave that portion to you to derive `chooseN` into `flatMap`.

## Conclusion
Writing functional code requires a steep learning curve. Therefore, a high-growth company is hesitant to enforce functional programming languages because it takes time for developers to learn a new way of thinking. 

You may need a longer development time because you need to account for all these errors. However, you will also sleep well at night because you can be sure that the code behaves as you intend. Once you get over the hump, you will realize how productive and confident you feel when you push your code.  You'll notice all the errors that may happen in the program before the program runs. You don't need to hit the `run` function simultaneously to see how your function will behave. 

You will write single responsible functions and decrease the amount of repeated logic in your program. Once you learn to write functional programming code for a while and get back into writing imperative code, you become a much stronger programmer that can see various runtime exceptions and bugs within the program. 

Most importantly, you are confident that the feature you build is robust and will create a great user experience for your users.


## Reference
- [What Functional Programming Is, What it Isn't, and Why it Matters](https://www.inner-product.com/posts/fp-what-and-why/)

