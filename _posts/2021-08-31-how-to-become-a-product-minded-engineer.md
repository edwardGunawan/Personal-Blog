---
layout: post
title: How to become a Product-Minded Engineer
date: 2021-08-31 14:56
summary: Engineers who love to ask the 'why.'
categories: best-practice jobs management life-lesson
tags: best-practice jobs management life-lesson
---

Most engineers are taught how to build things. We learn all the engineer skillsets, understand various tradeoffs in system design, and implement the most efficient algorithm. At a lot of the companies, engineers come during product implementation. Engineers will discuss how to minimize these engineers' efforts in building this product. Engineers think deeply about scaling, performance, and how to build such a product. Software Engineer interview often asks about algorithm efficiency, system design, and scalability problems, but less often talks about the product; because those seem like the job of a product manager. We never ask about _why_ to build such a product. 

Product-minded engineers are engineers that developed a lot of strong interest in the product itself. They want to understand how the product brings value to the business and love making product decisions. 

These engineers are not only good at identifying engineer challenges but also product challenges. This kind of engineer usually is not seniority tenure. Still, it was how they work—having deep curiosity about the _why_ is what differentiates a regular engineer from a product-minded engineer. 

This kind of engineer can have a huge impact on creating a great product.

"Once you have the product foundation, you need devs who engage with the 'why' actively. Engineers who have a thirst for using technologies to leapfrog human/user problems. Those with empathy reach for magical experiences. That is what defined a product engineer in my books. Bad ones cut too many corners. Great product engineers know that minimum lovable products need the right depth to be considered during the build phase." - Jean - Michel Lemieux

## Traits of a Product Minded Engineer

## Keen Interest in the Business and the Product
The product-minded engineers don't just settle with the product manager's current specification and jumping to implementation. They think about the various approach that can create a better product impact and minimizing engineer effort. Often, they will suggest alternative product approaches that may work better.

## Curiosity of the "Why"
Engineers with product-minded often ask the questions of "why." Why do we need to iterate on feature X instead of feature Y? What is the ROI of having feature X? How will this feature be measured? 

## Practical Handling of Edge Cases
Most engineers will consider edge cases in the codebase and cover all those edge cases to create more robust systems. Therefore, they may ask something like, "What if the user didn't enter anything and it is a null value?", "What happens if the call to the third party is timing out?". 

A product-minded engineer will take a higher level of dealing with these edge cases. They will think about each use case and scenario of the customer journey through the feature. 

They often handle edge cases from the business point of view - that don't require any engineering work. They will map out most of the things that can go wrong and bring the suggestions to the edge case early on in the product discovery phase. 

For instance, if one in a hundred calls to the third party may encounter an error, they may consider the tradeoff of fixing it versus other solutions of not fixing it. If we didn't do anything, what happens to our customers? Can they retry until it succeeds? Can customer support help during this incident? Can the product be slightly modified so this edge case won't occur?

## Strong Sense of Product Ownership
Most engineers will work a certain product end to end - from specification, implementation deployment, and maintenance. Once the feature is up and running, product managers may pull them to another project. Usually, after rolling out to production, an engineer's job is to maintain the system or add a new feature into it. However, a product-minded engineer takes a step further. 

Once they finish rolling out the feature, they are curious about its result by looking at the user behavior and business metrics. They actively engage with product managers, data scientists, and customer support channels to learn how users use the feature in the real world. It doesn't take too much engineering effort. However, through this understanding, they can bring new product suggestions to the table if the product they shipped is mismatched.

They have a strong sense of ownership with the product they build and persistence for wanting to know: how is my work doing?


It may be hard to develop a product-minded engineer since it encourages engineers to be as generalist as possible. Knowing the foundation of the engineer is much more important than knowing product intuition. So how can you work on becoming more product-minded? 

In this article, I want to give four tips for any engineer to become more product-minded.

## Understand how your team fits into the Bigger Vision of the Company
A company working on the web consumer space will have a different approach to building the software than an enterprise business software.

We often learn about the "How" of a feature, but not the "Why." Learn more about the "why" helps us understand the impact of rolling out the features and define metrics to understand if the feature performs as expected. 

For instance, if you are in the commerce space, understanding the primary mission of having a good payment system in your company. Why does your team want to integrate every payment provider? What is the level of effort and ROI between delegating the task to a payment provider such as Stripe or integrating these payment providers in-house? How many resources do we have at the moment? What will be the value we give to our end-user, such as customers using the product, when we do one vs. the other? How many resources and money can we save in the future if we choose one vs. the other?

These are hard questions to answer, and we need to understand the long-term in which the company is heading and the current product development stage. 

If the focus on the current product development is to build more features on the main business and have limited resources, delegating the commerce to the third payment for a short-term can help iterate on the main business product quickly. However, will this short-term solution cause a long-term impact on the product for years to come? These are all good questions that you can ask as an engineer to build your product muscle.

Understanding the domain takes time. If you are in the e-commerce business, you will need to know the e-commerce domain. If you are in the payment business, you will need to know about the payment domain. Knowing the domain makes you see your project and your task from the product perspective. You will also be able to communicate better with other non-engineer teams because you know how to explain the tradeoffs of their feature request.

## Understand how the end-user uses the product that you built
Have empathy for how your user uses the product and understand how a product works or should work as a complex system.

The end-user over here doesn't have to be the end-user of your company. If you are building internal tools and your stakeholder is a data scientist, understand their pain points.

One of the hardest things to think about a product-minded engineer is that many engineers in a traditional company have the hierarchical top-down approach to work. 

Engineers are often needed to think about implementation. We need to know how much effort we need to implement such a feature and focus on optimization. Sometimes, we get caught up in the new shiny tools or engineer best-practice and forget about the ROI of different efforts, especially widely-used products. 

For instance, a unit test is important for engineer best practice. When there is a backlog on user feature implementation vs. finishing up on a unit test for an application running in production smoothly for a couple of years, the engineering sense of us will want to decrease tech debt before taking on that new feature. However, the product sense will be to understand the ROI on both efforts and bring the most value to our end-user.

Engage with user research or customer support, where you can learn more about how the product works. Pair with other non-engineer teams who have to interact frequently with users. 

For instance, the best time to learn about the product is when there is any production issue. When there is a production issue, teams usually gather together to identify the underlying problem. During that time, there are many "why" questions ask on any part of the implementation. During those moments, I can see how the customer reacts from the customer support team and what other problems we can solve in our system to increase value to our end-user.

## Take initiative on Product Suggestions
Try to bring small suggestions to your current project. When you suggest a product suggestion, be sure to think about the underlying "why" and explain the benefit through a business lens. Then, explain the engineering and product effort that makes it easy to prioritize in the backlog.

"When scoping the effort to build the product, the engineering effort to build a key feature might be significant. Many engineers would start to look for ways to reduce the effort and figure out what the impact of the reduced effort would mean for the feature itself. A product-minded engineer thinks from both angles - engineer tradeoffs and product impact. They often go back to the product manager, suggesting a completely different feature to be built, given the product impact would be similar, but the engineering effort vastly smaller." - The Pragmatic Engineer.

When examining the projects and their tradeoffs, think about engineering tradeoffs and product tradeoffs that result in less engineering effort. 

## Constantly Ask for Feedback

To know if you are doing a great job recommending product features on the table, giving tradeoffs suggestions is by asking for feedback from your product manager. 

A product-minded engineer not only needs to be good at engineer skillset but also have good product skills. Therefore, reach out for feedback on how valuable they see your product suggestions and thoughts on improving.

## Closing
Having product-minded engineers in your team can help you build better features and iterate on your product more efficiently. They usually are keen to ask the "why" on the product discovery phase. They will first try to understand the business and its ROI. Then, they will give you suggestions and product tradeoffs based on the product impact and engineering effort. Lastly, they have a strong sense of product ownership and are constantly curious about how their product performs.

As an engineer, building your product muscle requires deliberately mapping the system you build to the overall big picture of the company. Moreover, you can challenge and asks questions regarding the reason behind rolling out the features. 

Just like new technology comes and goes, we need to keep up with the newest technology currently in the industry. There will always be a new trend in the product space. If you want to be more product-minded, seek new product knowledge, and understand the current trend. It can help you build domain expertise on your product, and you can bring a lot of insights into the product to the table. Last but not least, constantly ask for feedback from the product manager and any suggestions for areas to improve.

## Source
[Who is Product Engineer \| Hacker Noon](https://hackernoon.com/who-is-product-engineer-cl4l3yiu)
[Product engineers. We had just finished a customer… \| by Sherif Mansour | Medium](https://sherifmansour.medium.com/product-engineers-f424da766871)
[The Product-Minded Software Engineer - The Pragmatic Engineer](https://blog.pragmaticengineer.com/the-product-minded-engineer/)
[How Is A Product Engineer Different From A Full-Stack Engineer?](https://www.forbes.com/sites/quora/2017/09/06/how-is-a-product-engineer-different-from-a-full-stack-engineer/?sh=1436bd2426e7)

