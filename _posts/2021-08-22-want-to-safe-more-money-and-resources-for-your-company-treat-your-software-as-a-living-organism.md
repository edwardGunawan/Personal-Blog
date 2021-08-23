---
layout: post
title: Want to Safe more money and Resources for your Company? Treat Your Software as a Living Organism
date: 2021-08-22 21:12
summary: Codebase are not machines but a living organism
categories: ideas programming tech team-work software-development
tags: ideas programming tech team-work software-development
---



![Photo by Michael Schiffer](https://images.unsplash.com/photo-1535127022272-dbe7ee35cf33?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1050&q=80)

Here's what product or business units didn't get when we explained to them about technical debt: if the piece of software works already, why do we need to even "touch" them? 

The best way to explain this is to talk in realistic terms - "if we don't do this now, you may need to hire more resources and bigger outages that will cause the business to crumble in the future." Some understand the reasons for this sort of causality. However, others hear about more money that may require spending in the foreseeable future, or big catastrophe to their business, the fight or flight mechanism starts kicking in. They start to be defensive or think that the engineering team is not capable of doing their job. Thus, they think that hiring an engineering team is not worth the money. Hiring a consultant from the start may be a better option.

One key aspect that all businesses that deal with software need to understand is that a computer is a machine, but the codebase is an organism.

This won't make sense if you never worked on a large or even medium-sized software project - but it is often a surprise for the business unit and new grad. Why? Because they don't tell you that you are dealing with an organism.

## What they Don't tell you about software
Everything that we learn about in Computer Science is how to tell the machine to do what you want. We stress the time complexity of what the machine can do and how we can design to handle scalability. However, building software - or being a part of a team - is different from controlling an obedient machine - you also have to control a large, unruly codebase.

This turns out that you require a few 'softer' skills. One reason is that a machine does not write a software system but a human being. Thus, you cannot be the boss of the codebase and tell what to do and expect it to be obeyed. Instead, we usually maximize our influence to nurture the codebase and system as it grows over months and years. This is why, when telling that we need to change something that is currently working can be hard to understand. 

When you submit CS homework assignments, you don't have to touch them anymore. It is fixed and static. Either your algorithm is correct and efficient, or it doesn't. However, pushing the same algorithm into a software project may mean very different things. When you push your software into production, you are releasing it to the wild. The way I think about this is, when you developed the best aircraft that you believe won't break in the middle of the flight, there is still that possibility of a catastrophe. One of the reasons is that your algorithm will now have to defend itself once it is out. 

Your code will be tossed and torn, battered and bruised by other developers- including yourself at a later point in time. That algorithm you think is the most efficient now may not be that efficient at all. It may suffer from other external elements and factors such as bug fixes, library upgrades, drive-by refactorings. You can realize this by looking at the commit history of one of your project's repositories, and you will be surprised by how your codebase evolves and mutate like an organism.

Treating software like an organism requires a whole different mindset from treating the machine as a tool for the business. It requires knowing how to deal with unpredictability. 

## Managing Decay
Each organism will grow and decay. Some organisms decay faster than others - it is just a matter of time. It is like entropy. You never 'win' against entropy; you try to last as long as you can. 

If you don't take care of the organisms, they will decay sooner. One way to decrease entropy decay is by bug fixes, test fixes, small refactor, migrating off a deprecated API ...etc. These are standard operations that all developers will need to undertake. 

When we are not doing these maintenances often enough, software rot sets in, and decay happens. Just like a bike, if you don't take care of your bike by regularly cleaning and condition the bike frame with rust-proofing lubricant, your bike parts will soon become rusty. The rust cause metals to degrade over time, and ultimately the bike frame crack or a chain to break during a ride.

The older the module, the more likely it is to be suffering from code rot. However, code rot happens when you don't touch it.  If there is a major refactor, you decrease the amount of degradation on the software. 

The more dependency the software has, and the more of that dependency have recently changed, the more likely the module is to have decayed. Just like the older we get, we need to go to the doctor's visit. 

As software engineers, our job is to maintain this system to grow in the right direction and decrease the amount of time of [code rot](https://en.wikipedia.org/wiki/Software_rot). That is why testing is important. The only way to access the system's health is through constantly testing and have someone paying attention to the results. Without testing, no one will notice that the system has a tumor. 

After months or two, the entire module can easily rotten that it is impossible to resurrect. Just like a yearly doctor visit. We will not detect an early tumor or cancer cells if we don't regularly check our body until too late.



## The Problem with Growth
The other parts of the organisms are growing. Organisms can grow in various directions, and it is our job as software engineers to make sure that the code is growing in the right direction. 

The project needs to grow to become valuable. However, a lot of the growth within the software is short-sighted - perhaps premature growth optimization. 

This kind of growth is usually the norm. A software company that is in a high growth stage encounters tight deadlines. Because of the unforeseeable growth that it has the experience, software engineers are often opportunistic and short-sighted. They may try to do a quick optimization or development to establish the needs of business units. However, they may fall short of making globally optimal decisions to grow the system in a healthy direction.

Any experienced engineer with a huge amount of growth soon realized that the more a system grows, the more volumes it has to maintain against the forces of entropy. A project, thus, can devolve into a tangled mess. 

For these reasons, like software engineers, we can be paranoid of the code growth. 

An experienced software engineer knows how to push back features from product and business and re-pivot their focus on long-term growth instead of short-term gains. 

We know that adding more code and services to the system can cause bugs and errors. A module that grows by itself without any intervention may soon be dependent on each other, and it will be harder to separate them in the future. Thus, we need to detect those signals and do surgery to separate two modules that had become inappropriately tangled with each other. 

This kind of surgery is what we usually call solving technical debt.

If a plant grows sideways, where you want it to grow upward direction, you will do a little surgery to make it grow upward. However, you cannot just do a full surgery on the plant because too much surgery can cause the plant to grow weirdly upward. 

The key to dealing with this is a balance between nurture and discipline. 

Engineers cannot be too oppressive not to give any freedom to the system to grow. There needs to be a balance between letting the system grow naturally and making little fixes to decrease the entropy from avoiding it to decay.

One of the discrepancies between what product managers said about projects and software projects is that they treat the software as a machine. 

A project has a start date and an end date. However, the system that engineers build has a start date but no end date. The software that engineers build always evolves as time goes on.  Hiring a consultant is a short-term solution for your software company, but you will need to treat your system as a living organism instead of a machine in the long term. 

This also helps product and business teams understand that things don't always turn outright, no matter how precise we plan and develop. 

The market is dynamic. Changes in user behavior and the moves of the competitions make you realize and need to do some fixes in your software. You will always detect some errors; you may have a new developer working on the software system. 

## Conclusion 
With all these reasons, engineers can never be certain if the software will always behave the same as the initial development of MVP. 

Businesses and products will also need to understand and that nothing is everlasting. Once you understand the organic nature, many tech initiative's projects become clear and easier to understand. 

It very quickly becomes obvious, for example, when engineers have the initiative to increase test coverage in some part of the system, as testing is paramount for the health of your code. How else do you prevent code from rotting? How else are you able to catch bugs as soon as they pollute?

When engineers want to fix the codebase with a bad "code smell," they mean that this piece of code currently won't cause any problem during execution on the machine. However, it's going to cause problems during the evolution of the codebase. It might invite entropy to accumulate somewhere, for example, or put unnecessary constraints to the growth or allow it to grow somewhere it shouldn't. 

In this way, building software isn't at all like assembling a car. In terms of managing growth, it's more like raising a child or maintaining a garden.

## Reference
- [Software is a living organism. For most people, the work is done when…](https://osmancelik.medium.com/software-is-a-living-organism-acb840747bd4)
- [5 Reasons that makes you believe that Objects are living organism](https://www.linkedin.com/pulse/5-reasons-makes-you-believe-object-living-organism-naina-dhiman/)
- [A software system is a living organism.: @noahsussman: Infinite Undo](https://infiniteundo.com/post/129745180883/a-software-system-is-a-living-organism)
- [A Codebase is an Organism](https://meltingasphalt.com/a-codebase-is-an-organism/)
