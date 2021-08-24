---
layout: post
title: Why is REST more popular than RPC?
date: 2021-08-24 15:54
summary: REST is much more flexible
categories: programming system-thinking architecture 
tags: programming system-thinking architecture 
---

![Photo by Isaac Smith](https://images.unsplash.com/photo-1561991909-35afb1bd1e7e?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1050&q=80)

The primary goal of a microservice architecture is to deploy independent services so that product can create multiple fast iterations at different times. Thus, each team can deploy services separately without needing to coordinate with other teams. 

With microservice architecture, service and service communicate through network calls, and both services usually agreed on some contract - called API. 

Ask a developer why they need to define an API, and they tell you that API helps application integration with another service. One prioritization for designing such API is the ease of development and integration and the execution efficiency from the client and server sides.  

RPC seems like a good fit for this problem. RPC tries to make remote network calls like function calls or methods in your programming language. Each application will require to contain code generation and automates tooling when sending the network call. 

API Integration with RPC can be seamless by using their favorite programming language - you don't need to learn a new design philosophy by integrating with RPC equivalence. As RPC uses the binary format to send requests between networks, integrating with RPC is more efficient and compact than RESTful API. 

If RPC is such a good idea for creating APIs, why do most API integration use REST? 

## Problems with RPC
Although constructing API with RPC seems easier, maintenance is the main threat. 

RPC tries to make remote network calls like a function call or methods in your programming language. However, this abstracts the location transparency to conveniently encapsulate network calls in their application code. Moreover, network calls are totally different than function calls.

### Predictability
A local function call is predictable. It either succeeds or fails. However, network calls may go stagnant due to network connection loss. The remote machine is slow or unavailable, and network calls may not receive anything. These problems are outside our control, and the only recovery option is to retry a failed request. 

A local function may return a success, an exception, or it may just never return. However, network calls can return all of the above, and another possible outcome may return without any results due to timeout. There is a lot of possibility of a timeout. It can be that the request that you sent goes through, but the response from the remote service encounters a network issue. It can also be the request doesn't go through the remote service at all. 

When running a local function, it normally takes the same time to execute. A network request is much slower than a function call, and its latency is widely a variable. When there is a good network, you can get back in a few milliseconds. However, when the network is congested or remote service is overload, it may take a few seconds to get back a response.

### Not Referential Transparent
If one plus two is three. Then, you can confidently replace three to one plus two without changing the program's behavior for any programs that contain three. We can say that the corresponding value is referentially transparent. 

A local function call (without any side effects) is referentially transparent. Therefore, if you run the function multiple times, you will always get the same result. However, retrying network calls can cause the action to be run multiple times if you don't have idempotence in the protocol.


### Brittle
It looks simple to integrate RPC as network calls are similar to calling a function call. However, it can be complicated when integrating services that use different programming languages. 

The RPC framework must translate data types from one language to another. This can end up being ugly since not all languages have the same data types. For instance, JavaScript has a problem with numbers greater than 2^53. Therefore, we may need to enforce the servers and client uses the same programming language.

When you first start writing software to automate a particular problem domain, most of your development will focus on automating basic features and workflow. As the software matures, a lot of the values of the system come with integrating and augmenting existing systems instead of building a new one. 

Many companies have initiatives to expose API to other systems for better customer interaction. There is also an initiative where we want to expose our API to businesses to see an overall picture of its customer, suppliers, or partners across multiple systems. 

These systems create API to use the data innovatively for mobile application integration or third-party partners integration. Thus, a system needs to be extensible, or integration may take a lot of time and resources.


## How is REST Different

### Flexible
One of the remarkable characteristics of the world-wide-web is that every address on the web exposes exactly the same API - HTTP. That means, navigating through the world-wide-web, you just need a single API - HTTP. 

Usually, RESTful API will send messages through the network using JSON. REST decoupled the payload request with the services that implement encoding and decoding of that request. Therefore, each service can easily integrate with any programming language they want as long as they implement the encoding and decoding on that payload. 

HTTP doesn't mandate a particular data format, but JSON is by far the most popular. JSON was initially designed to be a good match for how JavaScript represents data, but all programming languages support it. 

JSON is an abstract format, and it is free of technology-specific constraints - the service doesn't have to maintain various programming languages during integration. 

REST helps create rules for linking multiple entities together that are in different systems. For example, order service, price service, and payment service are all needed to be linked together to purchase an item or do a refund. To link this entity together, each of them needs to have a clear identity that can be referenced outside of the application. REST helps create a unified concept of URL, an identifier for a resource that is valid everywhere. In contrast to RPC-based APIs, the identity of an entity is expressed in the form of a local application. It burdens other services in the integrations process because they will need to understand the nuances of the local application.

### Transparent
We want functions to be as transparent as possible. A transparent function decreases the number of bugs in the production environment. 

It is useless to try making a remote service look too much like a local object in your programming language because it is fundamentally different. 

HTTP didn't hide the fact that it's a network protocol. If you are getting an identity reference from some third-party entity, you will do a GET call, and you can be sure that multiple GETs will not change the result of the service. 

If you are trying to create something on a third-party entity, you will do a POST call. You know, if there is some error occurred during the process, you can see a network issue or a server error based on the HTTP status code.


### Ease of Experiment
This point relates to the point of flexibility. Flexibility is king when we want to do fast prototypes and experimentation.

You can simply use a web browser or command-line tool, curl, without before setting up code generation or software installation. REST is also supported by all programming languages and has a vast ecosystem of tools available, such as servers, caches, load balancers, monitoring, debugging tools, testing tools, etc.

## Does that mean RPC is dead
Despite the above problems, the RPC framework isn't going away. New frameworks that build on top of the RPC framework address these problems. For example, Thrift and Avro come with RPC support included. gRPC is an RPC implementation using Protocol Buffers. [Finagle](https://twitter.github.io/finagle/) uses thrift. 

This new generation of RPC frameworks is more explicit that a remote request is different from a local function call. For instance, Finagle uses futures to encapsulate asynchronous actions that may fail. Futures creates an abstraction to simplify calling multiple service calls and combine their result afterward (using `traverse` or `sequence`).

Another advantage of RPC is that it is much more compact than REST by using binary encoding format to send requests. Hence, it is more memory efficient than REST. 

## Pick the API style that fits your Goals
Suppose your goal is to create an API that can communicate between two services of the same organization within the same data center. In that case, RPC or gRPC may be an excellent choice for designing and implementing your API. 

RPC helps make sending a request within services in the data center much more efficient than REST.

If your services want to expose public APIs or create isolation, then using HTTP and REST API can provide more extensibility in your system. Anyone who knows HTTP from the standard documents will be able to use your APIs.

## Source
- [REST vs. RPC: what problems are you trying to solve with your APIs? \| Google Cloud Blog](https://cloud.google.com/blog/products/application-development/rest-vs-rpc-what-problems-are-you-trying-to-solve-with-your-apis)
