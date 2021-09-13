---
layout: post
title: 6 Hard Truth that Engineer needs to Get Over when Working on Side Projects
date: 2021-09-13 11:16
summary: No.2 Obsession with the Engineering Best Practice Will Slow You Down
categories: tech ideas life-lesson
tags: tech ideas life-lesson
---

Every side project comes with an idea. For instance, I came up with creating a Medium publication index to better search for publication. Therefore, I want to do this side project. 

An interesting fact about ideas and thoughts is that it comes and go. According to studies, we have 6200 thoughts come across our minds each day. However, little do these ideas came to fruition. 

<blockquote>
    <p>"Ideas are cheap. Ideas are easy. Ideas are common. Everybody has ideas. Ideas are highly overvalued. Execution is all that matters."</p>
    <footer><cite title="Jim Shore"> - Casey Neistat.</cite></footer>
</blockquote>

 

When we have an idea, we are often very pumped and excited. We want to quickly see that idea come into real life and see how people react to it. The very exciting thing about starting something new is seeing the outcome of that vision. For instance, creating a product that can help engineers debug production issues easily is very exciting because I can imagine how easy the life of a software engineer will be if this product exists. Further, I am also excited about how the engineers will react when they are using the product.

We can say that idea creates the initial motivation for us to do something. To keep us motivated from executing our imagination, we have to deliberately think about our initial vision.

A lot of the side-projects didn't finish. Because it is a project that you do on the 'side,' you may not have the time to work on the product in a short time. We get caught up on other ideas, look back on the old ones, and think that the old ones just don't turn out to be that great.

Gary Vaynerchuck often said that he has a lot of ideas and gets excited at night. However, those ideas usually don't feel excited anymore the next morning.  He will pursue and take action on those ideas that will still make him excited in the morning because he is still motivated with those ideas.

Most of the ideas don't get excited anymore because we don't have a clear vision of the problem that we are trying to solve in the first place, or we don't constantly think about the vision of the end product.

In this article, I want to share a couple of hard truths about working on side projects. These are lessons that I learn from creating 'smee,' a Medium index publication. 

Disclaimer: I ended up stopping this project because of various reasons that I will share later.

## Think about the Problem instead of the Solution
Many side projects start out wanting to build the "uber for x" or "automation tool for Y." This helps create a fixation mindset that if this solution doesn't work, the side project is finished. 

The mindset of thinking about the problem instead of the solution is more like a flexible mindset. You don't get discouraged if the solution didn't turn out to be the right solution because you focus on fixing the problem.

You can find the problem through data. 

Data doesn't have to be something tangible that we input in excels and create a beautiful graph. Data can be a simple interview with other customers and the end-users. Our brains gather countless patterns and data each day.

You will need at least 3 end users to really understand the underlying problem. People often say that the best way to find the problem is through the first-person point of view - you. For instance, I have a problem trying to find a Medium publication. Therefore, I create a solution for myself. 

In this scenario, I would have to have a conversation with other writers to understand how they submit their articles to publication and their problems. This can further ensure that the problem exists within myself and society before even brainstorming the solution.

The problem that you are solving should be something that they often encounter and want to solve. We will know by talking with the customers.

## Obsession with the Engineering Best Practice Will Slow You Down
Engineers love beautiful code. We love ownership. We believe that programming is a craft instead of a command line to the machine. We also have been taught to write code the right way - with some beautiful design patterns to make our code more extensible. Moreover, we are also excited about the new tools that are currently hot in the market. 

As we start our side project, we tend to deviate our attention to the technology stack - what new shiny framework that I can learn, and often we get deep into the product's scalability. Sometimes, we over-engineer our system.

"Oooh! There is a new technology X that can help you create a better infrastructure as code than using Terraform." 

We want to try that on our next new 'side project.'

In the beginning phase of validating your hypothesis, getting lost in creating the perfect architecture and code is not optimal.

That is normal. As engineers, we have always been taught about designing the system the right way or making the system more scalable. However, creating MVP will require a completely different mindset than scaling a product.

If you are doing a side project to explore new technology, you should take as much time to get to know that new framework. Building a side project because you want to validate product-market fit, being too deeply in love with the code will waste your precious time and feelings.

In creating the medium index publication, I started building the architectural system backend to crawl through Medium websites. 

Although some open-source API can help me do that, I didn't do so because this feature is paramount to moving the product forward. I ended up taking months instead of weeks to build the minimum viable product. Once I developed the features, Medium had a new policy to promote individual writers instead of publication, and it has completely changed the game and the purpose of this project. 

Getting your features out as fast as possible to validate product-market fit before spending time on features that may be thrown away.

Once you validate a product-market fit, you can expand your architecture by building that key feature from the ground up to avoid any dependencies. 

Minimize the amount of shiny technology you use and use your most comfortable technology and infrastructure. If you have been building your infrastructure with AWS, for instance, and your most comfortable programming language is Clojure, use that language and infrastructure to build the product. The point here is to move fast as possible and get the minimum viable product out for validation. 

## Flexibility is King
One of the hardest aspects of building a product in the initial phase is to account for the dynamic changes in customer needs. 

However, if you are building this feature as you iterate with your customer, you realize that additional improvements and features need to be. Your architecture will need to be flexible enough to adopt that.

We have to lift our artisanship hat, put it in our product manager's hat, and focus instead on the product. The code refactoring and scaling come later once we validate our hypothesis. 

The one thing that we wanted to do was to always be flexible. I would say flexibility is king in all aspects of software craftsmanship. 

In building my side project that never comes to fruition, I adopt all kinds of best software practices in the beginning phase. I put a lot of emphasis on the CICD pipeline, the monitoring tools, the architectural design. I would argue that if you want to build a fast prototype in weeks and you don't have that many resources, you can forget about the CICD pipeline if that may take up a couple weeks. You can build the prototype and ship it off; you can quickly fix it if it breaks. The CICD pipeline can come later when you get validation that your hypothesis is correct. 

## For MVP, Shipping Fast Trumps everything
It is okay if your code breaks things because bugs are inevitable anyways. 

We want to get the product out there because we don't know if the idea will solve the problem. 

For instance, in March, I wanted to build a Medium publication index, Smee, to expand my reach to other publications. During those times, Medium still support Medium-owned publications. Therefore, the best to gain further audience reach is to put your article in the right publication. If you are writing about love, you don't want to put your publication in Better-Programming. Even if you do, your article will get rejected. 

As new writers, we don't know which publication to publish. Since Better-Programming has a low acceptance rate, many new writers may get rejected at first when they write to major publications. Further, many publications have stopped publishing a new article, meaning it is not maintained anymore. Having an 'active' Medium publication index may help the new writer explore smaller publications and slowly build their credibility. 

In the middle of building Smee, I got a big project in my full-time job and could not develop the prototype fast enough for iteration. In June 2021, Medium changed its policy.

It changed its focus from publication-focused to writers-focused, meaning they want to support the indie writer and new writer to write articles on the Medium platform. 

Since then, a lot has changed. 

You don't need to be curated in major publications to reach a Medium front page. Although the publication is still very useful to submit your articles, the appeal to writing to publication is not as appealing as before. Again, I am not saying that publications are useless, as I tend to submit my articles for publication. However, I am re-thinking about the product-market fit, the initial problem that I am trying to solve, and learned that things have changed drastically for a short amount of time. 



## The truth about Minimum Viable Product
When we consider the minimum viable product, we consider the minimum number of features to validate our hypothesis. This terminology is not new. We often hear this all the time when creating some new product in our day-to-day job. 

The problem with the state of minimum viable products that we often see at product hunt is very different from the actual meaning of the minimum viable product - they are more like the minimum marketable product. This product didn't create a minimum feature. The minimum viable product should create the minimum feature that needs to solve the problem. 

If you want to create a feature about automation on a CICD pipeline, you don't need to create a user sign-in page and a payment page. UI may not need either. Since you are solving the CICD process, giving the developer the ability to execute the script or an API will do the job.

Airbnb's first minimum viable product is a landing page with a search on the listing. Because that is the main pain point that they have with finding houses. However, there are no two important key features that you would consider Airbnb in their minimum viable product - a map view and a payments page. You would first think that these two features are the minimum viable product because these features are paramount to conversion. Nonetheless, Airbnb's minimum viable product is to validate its hypothesis that such a problem exists in the market. 

When you want to test if the hypothesis works, you will first solve your customers' most critical problem. 


## Testing Will help Speed your Development in the Long run
Unit testing is in the grey area on MVP.

At some point, writing test cases can slow you down in the short term. However, writing test cases can also speed you up for multiple iterations. When building Smee, I wrote unit tests on my code. It takes a lot of time at first. However, it speeds up the time I need to instantly change some aspects of the feature as I don't have to worry about some other parts break. 

Testing is important even at the initial phase of building a product because it can speed up your development process in the long run.


## Closing
Every software engineer should do side projects in their free time. While some engineers are completely burnt out from their day-to-day job, a side project can create a different development point of view. You will learn a lot about creating a product and the big picture in creating side projects. 

Building an MVP is different than building a mature, scalable product. The mindset of thinking about finding the product-market fit needs to come from understanding what your end-user needs. 

Being in love with the code may hurt you in the long run because your project will often always change. Thus, the system and the codebase will change as well. The key over here is to create a system and to write code flexibly. 

Forget about the new technology that you wanted to try out when creating your MVP. Leave those behind when you just want to play around with the cool framework and focus on shipping your MVP as fast as possible. 

No matter how slow your project goes, writing test cases helps speed up your development process in the long run. When things change in the future, you no longer need to worry if the new changes in your code will break the old code. Think about writing test cases as saving yourself more time in the future.

Thanks for reading. These are the mistakes and hard truths that I learn building products. Do you have any hard lessons/truth when it comes to building side projects? Comment them below!
