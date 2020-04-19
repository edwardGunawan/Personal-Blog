---
layout: post
title: All You Want To Know About The Concept Of Immutable Server Is Here
date: 2020-04-19 16:36
summary: It used to not be immutable because the cost are high
categories: devops
tags: devops
---

![Photo by İsmail Enes Ayhan](https://images.unsplash.com/photo-1584169417032-d34e8d805e8b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1470&q=80)

In the traditional mutable server, servers continually modified in place. Engineers can ssh into their server and modify some configuration in their instance. They can also deploy new code directly to that server by server, and changed their code afterward in that server, all through ssh. 

Although it is frowned upon now if we see engineers ssh into the production server and update its configuration, it is understandable in the pre-cloud-computing era.

Before the rise of virtualization, containerization architecture, and cloud computing, deploying an application are time and resource-intensive. You need to buy a physical server, which costs much money. When you want to set up the server, it can take a couple of weeks to let the server run.

Since the cost of changing the server was so high, it is practical to keep the server running as long as it can to avoid downtime, and mutate the codebase, or configuration within that server. However, the cost of mutating the server for ad-hoc fixes may encounter some unexpected bug that is hard to replicate.

With the rise of cloud computing, virtualization, especially now containerization, make a turning point on the server architecture. A virtual server is not as expensive to own as owning a physical server. Server management configuration and fixes can be done programmatically instead of operationally to the hardware. Moreover, virtual servers are more at scale, and they can be easily destroyed and created within hours instead of days or weeks. The advent changes of the virtual server make immutable server deployment possible.

Below are some of the advantages of an Immutable server.

## Known server state and fewer deployment errors
By using a permanent state, you are not modifying the current state of the running server. You modify the base image that deploys a new instance of the server.

Each time when there is new configuration changes or features, you modify the base image. Run all the test on that base image, and only deploy the server once those tests pass. 

With modifying the base image, we can assure the current state of the server through the new configuration code that we deployed. We can keep the server state entirely consistent.

## Rollback and Recovery Process
With immutable server, it introduces the blue-green deployment <a href="https://martinfowler.com/bliki/BlueGreenDeployment.html" target="_blank">strategy</a>. It makes the roll back to the previous changes easier when something goes wrong with the new modified feature. 

Each time when we create a build or release, we create a new version of that release once all the test cases pass. Therefore, we keep all history of the version release and can roll back to the previous version easily.


## Data Handling
Even though we treat the server as disposable, the data that live within that server is not. Therefore, data usually are now persisted or replicated somewhere outside of the server, at a shared file system or a cloud database. 


It sounded very familiar nowadays as a lot of the organization has adopted the immutable server pattern. It uses in the practice of Continuous Integration (CI) and Continuous Delivery (CD), where it deployed a single known artifact once and running and running applications in all environments. An immutable server pattern is widely used in cloud services, such as Amazon Cloudformation, provided a fast way to automate your infrastructure through a template. It automates your build and automatically rolls back to the original state if the build is a failure. The artifact serves as a single source of truth. 

## Conclusion
Immutable servers widely use across the organization, and it has its history on why such a pattern exists. 

With immutable server, we can maintain consistency throughout our infrastructure, and avoid any <a href="https://martinfowler.com/bliki/SnowflakeServer.html" target="_blank">Snowflake server</a>. 

An immutable server moves the configuration management from the server instance to the base image, by creating a new version. With creating a new version, the server can quickly roll back and recovery if it encounters some unexpected issue.



## Source
[ImmutableServer](https://martinfowler.com/bliki/ImmutableServer.html)
[What Is Immutable Infrastructure? \| DigitalOcean](https://www.digitalocean.com/community/tutorials/what-is-immutable-infrastructure)
