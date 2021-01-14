---
layout: post
title: To build a Robust System - You need to Design your System Pessimistically
date: 2021-01-13 21:15
summary: Pessimistic way of thinking is good for the system
categories: architecture programming system-thinking tech
tags: architecture programming system-thinking tech
---

![Photo by Sarah Kilian](https://images.unsplash.com/photo-1557261727-b5475257c850?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1435&q=80)

When it comes to how we view the world, most of us fall into one of the two types: Optimistic or Pessimistic. Whether you are one or the other, scientists said it has to do with our upbringing and culture. There are both pros and cons in viewing the world one or another.

An optimistic view of things makes one willing to try new things. Usually, optimistic thinking equals positive thinking. If you are confident and hopeful that the project you initiate will have a brighter future, you will be more likely to start that project. For instance, if entrepreneurs think that building a blockchain-based social network yields a positive outcome for overall society, he/she will more likely start the projects.

Optimism also helps you cope with the hardship. If a project fails, and you have a positive mindset. You are more likely to be able to cope with adversity.

However, optimism is not the best trait when you are designing your system.

While it is good to be optimistic in viewing through failures, designing a software system is not the case.

If you are optimistic in looking at your system, you might not account for all the pitfalls you will encounter. It is because you might make an assumption and thinking positively that your system will works. You imagine that the user flow of your system will follow a specific criterion that is unimaginable.

However, as we all know, there is no zero fault system. There is going to be an edge case somewhere that we will encounter after we shipped our product. During those moments, we have to be flexible in dealing with patch fixes, writing postmortem and learn our failures.

If you initially view your system that it is not perfect and that the design you do may have some bugs that will eventually be discovered. Essentially, designing your system in a pessimistic view may help you be more skeptical in all kinds of weird user flow and anything that needs to be accountable for.

For instance, designing a server: A GET HTTP call to server B to get some data can have multiple possible failures.

What happened if server A is down?

What happened if server B is down?

What happened if, during the middle of the calling, the data got tampered with?

What happened if calling server B and it didn't get any response?

Is server A a single point of failure? Do you need multiple concurrent calls to server B? Is server B a critical server?

The list goes on.

When you think about all the things that can go wrong or assume that the design can go wrong, you will be more likely to think through so many edge cases that can go wrong with your system. If anything went wrong with the design, you are more likely to be flexible to fix that system and make it more robust and secure.

In the case of a server, A is down. We might need to create replication on server A so that it is fault-tolerant.

If server B is down, we need to think of a way to transition that error to server A that server B is down, or create a default value or cache (if the value is often retrieved) depending on our business logic.

In the case of data tampered with, there are multiple solutions to avoid a security breach. It also depends on the application cases, sending a message in a private network, and doing various checksum to identify the difference.

There are many great applications and libraries created to solve many server issues, and all those solutions and creative and help improve the experience of their customers.

The pessimistic view of designing a system helps you build a robust system. You start to think about all kinds of possible ways that the system can go wrong. Then, you try to do multiple iterations if your system's hypothesis is right before sending your tech proposal to the team.
