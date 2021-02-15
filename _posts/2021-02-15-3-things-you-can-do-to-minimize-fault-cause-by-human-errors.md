---
layout: post
title: 3 Things You can Do to Minimize Fault Cause by Human Errors
date: 2021-02-15 16:19
summary: Better UI Design, Separate Faults, Quick Recovery
categories: tech life-lesson software-development
tags: tech life-lesson software-development
---

![Photo by Michael Dziedzic](https://images.unsplash.com/photo-1610337673044-720471f83677?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=966&q=80)

We build, design, and monitor software systems. Even when we have a good intention to make things work, humans are not reliable.

Faults usually happen when something is not working as the specs. Most defects cannot be prevented but avoided. The key to dealing with faults in the system is what operations you take to help avoid and quickly resolve those situations.

There are numerous articles about minimizing hardware error and software error. However, a lot of the [outages](https://variety.com/2019/digital/news/facebook-apologizes-outages-server-configuration-error-1203163429/) in an application are a cause of configuration error. 

How do we make our system reliable despite its unreliable operators? 

In this article, I want to share 3 items that you can adopt in your team to minimize faults caused by human errors in your system.

## 1. Design System that minimizes opportunities for Errors
A lot of the errors lies in different assumption between expectation and reality. A good design API and UI interface can help prevent users from stepping into the danger zone. For instance, if your team builds an internal tool that didn't do some sort of confirmation when doing operations that is not reversible, your team will end up losing some important data. The developers who created that internal tools will need to work with how the admin usually operates the systems. In general, you want to design an abstraction that discourages doing the wrong thing. For instance, a system that wants to minimize misconfiguration can do various system operations to ensure that the users didn't enter the wrong thing. If the user wrote the configuration of us-west-2 in an us-east-1 environment, the system would first ask for a confirmation message if the operation is correct. Second, the system can give warning signals if the procedure is not reversible. Lastly, the system can also give a confirmation page before proceeding to ensure that what the user enters is right.


A lot of the design usually will restrict developers from doing certain kinds of operations to prevent certain unchangeable procedures. However, with restrictive design, the user will try to work around them and disregard their benefit. Therefore, be aware of how you want your system to be restrictive. For instance, Commercesox compliance on the CICD pipeline helps prevent developers from creating errors in the production environment. However, suppose you restrict developers to manually push and build during the dev environment. In that case, they will have to try to work around by creating a script to automatically publish code in that restrictive environment.

Besides, minimizing your application system also requires a significant amount of documentation. Documentation is a great way to provide detailed steps to steps runbook on preventing any kinds of human errors. However, developers often don't look at the documentation or runbook when operating the systems. Therefore, it comes down to how your team can design a system that makes reading documentation boring to read through the documentation to ensure they didn't do anything that cascade failure.


## 2. Separate the process where humans will make the most errors to where the faults usually occur

Various nightmare stories about some developers in the development team misunderstood the UI admin tool in the database system causes the developer to do regular testing in the production environment. 

Developers will usually try to do functional testing and integration testing to ensure the code they developed correctly. In this scenario, provide a fully-featured non-production sandbox environment for your team or your client to safely test their features and experimentation without affecting the real users.

 Separating the critical irreversible action such as deleting data or deploying QA application to prod to another separate system than the regular operations. This way, the developers in your team can avoid any accidental mistakes of doing one of these operations.

## 3. Quick Recovery
We cannot prevent faults in the system. Therefore, it is essential to recover from the errors quickly to not to cause a further cascading failure. 

For example, we can create a backup database in the production environment to ensure that there will be a backup number if one accidentally deletes the database. If there are particular version mismatches in the prod environment, the system should roll back quickly and provide tools to recompute the data.

How you recover from the error is critical to minimize the impact in case of failure. Therefore, redundancy and replication are heavily discussed in system design because it is one way of handling error. 

Besides, a runbook for any production issue can help any person in your team who is on-call to be on the same page on resolving the problem quickly. Usually, it doesn't matter what the error is, but it matters how you handle it.


## Closing
These three lists are the list that you can adapt to your team. You need to account for other errors or know before it becomes prominent, including server monitoring or setting up essential metrics in your systems. Creating a fault-tolerant system is not based on how little the error will occur but on how the system can handle that errors.

I hope you learn something from this article and can implement these quick tips to your team.
