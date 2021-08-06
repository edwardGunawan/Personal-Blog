---
layout: post
title: As a Software Engineer, This is Why you should Learn To Love Legacy Code
date: 2021-08-05 19:41
summary: Conquered the challenge of legacy code and you will have your job for life
categories: career software-development programming life-lesson
tags: career software-development programming life-lesson
---

![Photo by Johan Mouchet](https://images.unsplash.com/photo-1556010392-36e3577857ce?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=634&q=80)

Today is your first day of work at a tech company. You are very excited and looking forward to the onboarding process. Once you finish the onboarding process, you are very excited to see your first tasks. Your manager has a list of tickets for you. However, as you examine those tickets, you realized that you are maintaining a system running for more than ten years. You may be thinking, "That's okay, this is my first time in this company, and there is a reason why they give me the most undesirable task the first time. Next sprint, I will get a more excited 'greenfield' project." 

As time goes on, you realized that many of the team projects you are currently dealing with are legacy systems. Your heart started to slump. You thought that this new job would be different than your previous job. You are looking forward to working on some cool projects. Yet, there seem to have no new projects in the future roadmap. You have been studying for that interview questions days and nights. However, once you crack the interview, you are working on legacy code.

Despite all these confrontations on yourself, you still have hope. There are two ways to deal with stuff you don't like - change your circumstances, or change your mindset. Which one would you choose?

The first option seems the most straightforward. The average tenure of a software engineer in a job is around two years. Most software engineers switch jobs every 1 - 2 years. Hence, you can change your circumstances by looking for a company that works on 'green field' projects. 

Although this option is more popular than the latter, you soon realized that all companies have legacy code. The more senior you go, the more you spent most of your programming career working in legacy code in one form or another.

As a software engineer, I want to explain why you should change your mindset on avoiding working with legacy code and finding positive in legacy code. 


## You will understand Your Domain Better
What better way to understand your company's architecture and domain works than by getting your hands dirty on legacy codebase? Working on legacy code doesn't mean you need to leave it as legacy code. A good developer will try to leave the project better than the original codebase. 

You often find some useful insights within legacy code. The pieces of code that keep the business running solve all the edge cases required for the business. Legacy systems have gone through the production field, and it has real bugs squashed by a real user.

You'll understand why the previous developer uses an if-else statement instead of some design pattern. You'll realize that your business charges the customer with double-dollar precision. 

Once you know these insights, you will become the domain experts on that system. You can easily create a fast delivery on new feature requests from the product team. When there is a production issue, you can quickly resolve the issue and give a great explanation of what is wrong with the system. This creates trust in your team members and upper management to rely on you and assign you to new projects. 

A lot of developer in my team doesn't want to touch legacy codebase. There is one developer assigned to maintain the legacy systems. However, when it comes to any production issue about the legacy flow, no team knows what happened except for that developer. As time goes by, other teams' members and upper management know him because he has demonstrated that he has the expertise to know all business use-cases and edge cases. 

You will become the go-to person for changes or advice in the team because you conquer the fear of working with the legacy system.

If you look at the version control system, often, you will realize which file changed the most. This can help guide important decisions, such as which part of the system is worth getting under test. If no one has touched the code for ten years, chances are, that code doesn't require much testing as the code that changes every one day to handle another edge case. 

Once you understand your domain, you can suggest new technology or practices make the system better. Thus, it leads to a bigger impact.

## Legacy Code is Robust Code
There is a reason why it is a legacy. Perhaps, the story goes like this - when the team started on the project, the team created a system that is an excellent design that people started to say, "Okay, maybe we can use it for this, and this and this." 

Although as time goes on, it accumulated technical debt, it still works. These systems are amazingly resilient - it is the code serving the business and has served the business for so many years. Otherwise, it won't survive.

When we need to maintain or fix the legacy system, there are two choices - duct tape it or rewrite it from scratch. 

While duct tape way of solving small bugs may works in the short-term, sooner or later, your future self will hate you for copy-paste that ugly piece of code. Just like an abandoned building, it may stay undamaged for a long time, but as soon as there is a single faucet broken, the entire restroom just crumbled. While thinking about this, you may choose option number two - forget about the legacy system and rewrite it from scratch.

Can you imagine what will happen if you forget the legacy system and rewrite it from scratch? 

The rewrite will often end up not working, or the developer will never finish it. We often look at legacy code with a squint in our eyes and thought, "If whoever wrote this horrible code can make it work, it must be quite easy." 

But it's not. 

I remembered spending a good amount of my time trying to rewrite a completely unreadable function. However, as soon as I rewrite the function, other things break. I kept the function as it is - as there is some magic within that keeps the system running functionally.

You may think that the code is horrible, but it's code that has survived countless battles. When you write from scratch, you often don't know about the battle story, and you often lose some key insights and points about the domain.

Digging your hands dirty on a legacy system helps you understand all the patches and scares behind the battles and learn about the system domain without having a cost of a production issue.

## Master the Knowledge of Problem Solving
We were often taught various design patterns and the use of SRP or DRY while writing our code. While the example that we saw on the web may be obvious, it isn't easy to know why. Why should we abstract out a series of repeated codes into its method? Why do we need to use that design pattern instead of creating a new class object?

Dealing with legacy code forces you to use these design patterns and confront the design pattern that you learn early on. You will discover new patterns, learn new techniques in refactoring and unit testing. You will discover that creating a new database instance inside a function instead of passing it in as an argument may be hard to unit test. You will discover that writing to an interface is much better than writing to an object. You will quickly realize the danger of certain code structures makes things hard to test. Without the safety blanket of unit tests, you will quickly learn which code can be confidently refactored without a test. Yes, there is some code that you DO need to refactor without test, and I'll leave that for the discussion.

This skill set is transferable. These skills are what differentiate an entry-level engineer from a mid-level engineer. You will be able to quickly adapt to the new code base, especially a codebase that has been used for decades. These skill sets are not learning some library that will be obsolete in next year. This skill is essentially the bread and butter of problem-solving.


## Closing
If you are working for any software company, chances are you must encounter legacy code. Legacy code may strike fear and disgust in most developers. However, legacy code is the best way for you to learn the domain of the system. Legacy code is also robust in its design because legacy code used to be the best way to make the software running smoothly. Lastly, you obtained the skills of problem-solving and refactoring that you can use in your next software engineer role. Mastering the challenges of legacy code, and you'll have a job for life.

## Reference
- [Learning to Love Legacy Code. Some time ago I gave a presentation for…](https://medium.com/ingeniouslysimple/learning-to-love-legacy-code-e5e5459dd2eb)
- [Learn to Love Your Legacy Code. Legacy code is survival code](https://medium.com/oreillymedia/learn-to-love-your-legacy-code-4296b93d8408)
