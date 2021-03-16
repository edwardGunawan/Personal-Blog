---
layout: post
title: Three Stories that Good Programmers Told
date: 2021-03-15 21:55
summary: You tell the story to computers, stories you tell to humans, and a story you tell to the end-user.
categories: ideas
tags: ideas
---
_![Photo by S O C I A L . C U T](https://images.unsplash.com/photo-1543726969-a1da85a6d334?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=2607&q=80)_


Humans absorb better knowledge through storytelling. People often said that data didn't mean anything if you don't know how to tell a good story about them. I said that good programs tell a good story to machines which operate on them and to the human who understand them.

Let's take an example of why I think programming is storytelling. 

Sometimes we get confused about why a certain codebase behaves in that way. For instance, when you look through an old codebase, some parts of the codebase get you to think, "why does the function return a generic type instead of a simple integer. Why doesn't the previous person know that they can just return a function that is an integer?" Then, you keep going through and try to refactor them - only realized that that function is being shared as a trait across all process flow, and doing so will decrease the amount of code redundancy in the codebase.

You can see where storytelling comes to mind when you conduct a pair programming session. When you are "leading" on the pair programming session, you often need to explain and tell the story of what your intention is on the codebase. The entire pair programming sessions will simply tell stories about what happens to any parts of the projects and explain it so that the other engineers understand. We often have pair programming sessions with engineers who first come to the team or take on the projects. 

In all sense, we want to write code to not just work but also extensible. We are writing code for people to read. If we are only writing code to make it work - we could all just do assembly language or byte code transformation. We have all these programming languages because these languages help us communicate a story with other programmers.

There are two hard parts of programming - knowing how to tell the computer to perform some action and combining all these actions together.

The first one can be solved by how to search your problem on Stack overflow. For instance, you can find the exact code-based and answer how to iterate a deck of array cards. However, knowing these individual operations is not helpful if you don't know how to combine them to do useful tasks. 

The second one is harder and more subtle than the first one. You cannot really search online on how the codebase in your company's project work. You will have to rely on the story that those engineers ahead of you told. 

These stories usually consist of three different abstraction levels - a story they tell to the computer, a story they tell to the fellow programmers, and the story they tell to the end-user.

## Story that they tell to the computer
Like words, a certain kind of word gives a different reaction to the reader who reads it. A description in your algorithm will make the computer execute the task in different reactions.

You are telling the computer to understand the sequence of activities that you want them to do. 

This is the lowest level of components to construct a higher level of your story. 

You want the computer to perform a certain action with either bit manipulation or a loop. Usually, it exists in the core library, and you need to explain what it really does as a comment.

When you are writing programs with a huge instruction line, you are only writing for yourself and the computer. 

For instance, you may write a for loop - follow with a put method, like this:
<script src="https://gist.github.com/edwardGunawan/a18ea1f5f2c9274f2764f203cb7e38a1.js"></script>


However, the only entity that understands the codebase above is you and the computer. The computer can execute all those lines that you construct, and you know what that intent is. 

If you ask someone else to look at the code base above and ask what the program owner is trying to achieve, they will have no clue.

To put this in the analogy of writing, writing many words on each new line may only make sense to you.

At this stage, the only hurdle you have is constructing the syntax and core functions of the language you are writing. This usually happens when you're new to a library or a programming language, and typically, a simple search on Google or Stackoverflow will solve the issue.

To transfer your program's intent to other human beings, you will need to go up a level - by abstract your level of storytelling to the fellow programmers.

## Story to fellow programmers
Class's naming, naming value of the functions, and the design pattern are created to tell the story of the program's intention to fellow programmers.

This abstraction level defines the class's intent you are defining and what sort of state and method exist in that class. 

This level of abstraction is closer to English. Sometimes, we create a function that resembles English words.

For instance, when you write a method that puts the value in a map or dictionary, it doesn't tell other programmers what the method's intent is. If you further describe that the method is `putValueToContainer`, it changes the method's intent - writes the value to a container. 

To illustrate further, let's take the example from the lower abstraction and encapsulate with some functions:

<script src="https://gist.github.com/edwardGunawan/ce707a566c6c47aaecd0a0bb2568f26f.js"></script>

Humans can better understand your thought process - trying to find some value and put those values into some cache. 

At this stage, an intentional code and method names will tell a better story. Therefore, you will need to proofread your codebase often so that your intentions are clear. 

Proofreading is equivalent to code refactoring. The first time you construct your thoughts in any IDE, you will most likely need refactoring. You might do a lot of code refactoring to give your program an exact method name to give fellow humans how the computer happens to be doing.

Many programmers struggle on this storytelling level because they tend to write their function to target all audiences. And we know, when you want to target all audiences, you are targeting no audience. Thus, writing over-generalized methods can sometimes be a double-edged sword. A simple tip to change this is to write code that gives clear intention.

A story that is understandable for the other programmers but not for computers is useless because the purpose of the code is to make the computer perform an action. Therefore, the higher level cannot be achieved by the lower level of abstraction. 

If you reach this stage of storytelling, you also implicitly accomplish the lowest abstraction level.

## Story to the end-user
The final level of storytelling is telling the story of what can the end-user expects.

This is the story for humans to write software - to accomplish our goals and make life more efficient. 

This kind of storytelling will tell fellow programmers and our end-user what the application will do.

This kind of story helps other engineers and higher-ups, and managers understand what the programmers are trying to achieve.

For instance, if you do behavior-driven testing or end-user story testing, you will see a fine description of our end-users expectations. You will write test cases like, "when you do [this] it will do [this]."

If we want to put a higher-level abstraction on the code from the previous example, we can write a unit test for our application like this:

<script src="https://gist.github.com/edwardGunawan/1d87561df61a19686cfe093ed6ee80a5.js"></script>

Now, we get more insights into what the intent of the entire application is trying to do. We now know that it is an array of catalog prices instead of any random integers. Then, we know that the `putValueToContainer` is actually a cache. Lastly, we also know this program's purpose is to put the catalog's price in the cache.

Behavior-driven test cases are very important because it is explanatory and simple to follow—a simple-to-follow technical specs resemblance a great storyline. 

The best part of understanding a new project is looking at their unit test and following through and seeing why things are created differently.

This end user story cannot be achieved if you didn't describe your story in the lower abstraction level. Like how you cannot write a paragraph without constructing a sentence, you cannot write a sentence without constructing words. 

## Key tips for creating a better software
The key to writing good software is to balance and separate these three stories as clear as possible.

Like writing articles, there are grammars and word of choice that can make a sentence more concise. There are also ways to write code so that it can make it easier to understand and maintain.

Fuzzy sentences usually cause by the cluttered organization of thoughts. There may be two different logic in one sentence - and that confuses the hell out of your reader. However, writing software is the same. If we put too much logic in the method or our application, it can be hard to read. 

To put this analogy to code - I saw software that combines lower-level actions with the end-user business logic. On one end, the software is trying to fetch the item from a database. On the other hand, the software is trying to toggle the UI button and make it true. 

It makes it hard to write code in terms of all three levels of stories - therefore, usually, we tend to break it down into principles, design patterns, and laws to make the story that we want to tell much more succinct. 

There have been decades of research and books on how to construct a readable and maintainable codebase. Separation of concerns, DRY, or SRP (Single Responsibility Principle) is just technical terms and principles to tell programmers how to separate the three-level of abstraction and story in their programs.

## Closing
Just like writing is to organize your thoughts in the right way that is understandable. Writing software is also organizing what you want the program to do in an understandable and maintainable - to the human eye.

Therefore, one way to tell a better story on your code is to separate the boundaries on each of these levels of stories you tell. Start from brainstorming. Group smaller instructions into a small method. Make each of those method intentions as clear as possible. Then, write your whole application's overall intentions with these smaller methods to create the story to empower the user to accomplish an incredible task.
