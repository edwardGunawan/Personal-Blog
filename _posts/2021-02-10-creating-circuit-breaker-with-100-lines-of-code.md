---
layout: post
title: Creating Circuit Breaker with 100 lines of Code
date: 2021-02-10 23:52
summary: I Don't Understand Circuit Breaker. Therefore, I learn to create one ... with 100 Lines of Code
categories: scala functional-programming programming
tags: scala functional-programming programming
---

![Photo by Ralph (Ravi) Kayden](https://images.unsplash.com/photo-1601462904263-f2fa0c851cb9?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=584&q=80)


"If you want to avoid your entire service going down because one component is down, you should use a circuit breaker," Kevin told me when I try to design one of our microservices. We talked in front of the ramen shop on a cold winter night in New York City in 2019. 

I asked him, "What is it exactly, circuit breaker?", "Dude! It is a popular microservice pattern. Search it up!" 

I pull out my phone and start typing "Circuit Breaker." In brief, it a mechanism that helps improve the resiliency of your services. I thought to myself, I need to understand more about the tool of this. Therefore, I determined to go back that day, do more research, and implement my own circuit breaker version.

Surprise, surprise - as always, I procrastinated. Fast forward 2 years later, I have learned so much about distributed systems, scala, and functional programming - and I would like to share the what and the how of circuit breaker mechanism. In this article, I would like to share how I created a circuit breaker library using scala and leveraging the concurrent primitive types.


## What is Circuit Breaker
Imagine when you request an HTTP call from one service to the other. It may result in multiple conditions. You may get a successful response. However, you may also get a failure response. In case of failure, we usually will return a failure or will do a retry. If it is a retry, you usually will keep doing it in another thread until success. 

Although the retriable error is prevalent to create resilience in a distributed system, too much retry may actually hurt your service. If you dedicate a thread that keeps retrying the external service, and the service B that is calling your service keeps retrying their calls, it may exhaust all the resources in your services and causes it to go down. This situation usually can escalate to multiple services that depend on the service that is down.

<img src="{{site.baseurl}}/images/creating-circuit-breaker-with-100-lines-of-code/Circuit Breaker Higher Level.png" alt="Circuit Breaker Higher Level"/>


Therefore, one way to mitigate this is to have some sort of proxy that will be smart about telling your service to stop the retry - because we know that another service is down, there is no need to keep retrying. 

This is where the circuit breaker is useful. You can make it as a wrapper on a function or a full service - it has an internal state machine to determine if you need to retry the other service or not. You usually create a proxy for that service to advise if the service needs to send the request or not based on the number of failures. If the current state in the circuit breaker says - "Don't Retry," then the service can simply return an error or default message and not occupy a thread for retry.

## High-Level Implementation of Circuit Breaker
The state machine within the circuit breaker is critical to make that smartness in Circuit Breaker. There is an Open, Closed, and Half Open. 

You start at the Close State. Each time when the call fails, it will increment the error count. 

Once that error count hits a certain threshold, the state machine transforms to an Open State. In the Open State, the circuit breaker will advise the service not to do any calls because it is assumed that the other service is down. Therefore, each incoming call to the service will simply reject it.

The breaker waits a couple times because it transitioned to the Half-Open state. The Half-Open state is where the breaker will tell the service to try on the call again - just to see if the market returns any 5xx. If the call is successful, then it will transition to a Closed State. If the call returns 5xx, then it will reset the timer on the Open state.

<img src="{{site.baseurl}}/images/creating-circuit-breaker-with-100-lines-of-code/Circuit Breaker State Machine.png" alt="Circuit Breaker State Machine"/>

If you still don't understand, Martin Flower explains it well in his [article](https://martinfowler.com/bliki/CircuitBreaker.html). 


## Implementing Circuit Breaker
Sometimes we have to see it in real code to understand what it actually does. The code that I showed was inspired by other circuit breaker articles out there, and I will link them to the reference below. 

I will implement a simple circuit breaker as a wrapper that has 2 states - Open and Closed. You can also use Half Open as a state to do some more optimization in your circuit breaker wrapper, such as testing external service in percentage during the Half-Open condition to see if the state will go back to a Close state or not. However, for simplicity, the Half-Open state in the code below will be a function to see whether or not we will switch from Open to Closed state.


I will use the [Cats effect](https://typelevel.org/cats-effect/), a famous Scala that adopts many functional libraries, with the Cats effect to write the circuit breaker.

First, before we start, we need to define the interface for what this wrapper can do. We also need to determine the three-state and the configuration for creating this Circuit Breaker.

<script src="https://gist.github.com/edwardGunawan/16f2514b32e613f6d9513246cec15d7b.js"></script>

We have a `Status` of `Open` and `Close`. `Open` will carry the timestamp's duration that it remains open and the error message of the reason the breaker set it open in the first place. Then, the Close holds the attempted error counter. 

We also have CircuitBreakerConfig to set the configuration for the maximum number of retry the service can do before it switches to the Open state. When the status turns from Close to Open, it will construct an `openTimestamp` based on the `current time + offsetTimeout`.


Then, we want the client to run any of their code with the `protect` function. Something like this:

<script src="https://gist.github.com/edwardGunawan/964f20ca6a2991e491a90bcbc7fac984.js"></script>

## Keeping State in Concurrency Environment
Another challenge for doing a circuit breaker wrapper is keeping track of the error count and the open timestamp?

Luckily, the Cats effect library concurrent primitive `Ref` is a pure atomic reference that helps you modify the value in a concurrent environment. It is the same as java atomic reference, except `Ref` uses tagless final abstraction and always contains value inside. 

`Ref` is suitable in this kind of environment because the circuit breaker may be shared with multiple application. Something like this:

<script src="https://gist.github.com/edwardGunawan/2bcbb4c0a225bd6e831cd564f30dbf8e.js"></script>


But if you want to use separate circuit breaker on two separate service, just don't let both service shared the `circuitBreaker` together like this:

<script src="https://gist.github.com/edwardGunawan/464b26d7a97d22a5fabf00e98b611563.js"></script>

If you are interested in learning more about `Ref`, check out the Cats Effect library [here](https://typelevel.org/cats-effect/).


## Boilerplate Code
Let's set up the boilerplate for this circuit breaker, and then we can tackle the Closed Status, Open Status, and Half Open Status case.

<script src="https://gist.github.com/edwardGunawan/4f31e0675257985a5c2a73f7d4e26754.js"></script>

We create a boilerplate to implement the `CircuitBreaker` interface that we defined at the beginning of this article. Then, we can start implement the `executeOpenState` and `executeCloseState`.

## Creating Closed Status
This is what `executeCloseState` first executes the function `f: => IO[A]`. If it is a success, we just simply return that value. If it is a failure. We will:
- check if the error count is over the threshold - if it is over the threshold set the state to Open with the `currentTimestamp + offsetTimeout`
- increment the error count
- raised that error and returned it back to the caller

<script src="https://gist.github.com/edwardGunawan/4b845347be36e449c1da1a72efc7eb6d.js"></script>

## The Open State
Each time when a request comes with breaker being in the Open state:
- Check if the current time has passed
- If the `currentTime > openTimestamp`, we go to Half-Open state.
- if the `currentTime < openTimestamp`, we will immediately throw a rejection failure response to the caller.

<script src="https://gist.github.com/edwardGunawan/06c686a4ad1047cb6ea2e1144c311e42.js"></script>


## The Half Open State
The Half Open State is executed within the `executeOpenBreaker`. Since this is a very simple use-case, the Half Open State checks whether we need to switch from Open to Closed. 

Therefore, inside the `executeHalfOpenBreaker`, we simply call the `executeCloseBreaker` again to check if the execution succeeded. If it succeeded, we change the state back to `Closed` with an error count of 0. If it failed, the `executeCloseBreaker` would handle the case of failure.


<script src="https://gist.github.com/edwardGunawan/18968327ae2ede057f1c053907a1a7d2.js"></script>

That's it! I hope you learn something from this article. You can wire any function onto the circuit breaker method to have that smart-ness by creating this. To simply illustrate this, let's create 3 services that run concurrently, changing the resource ref:


<script src="https://gist.github.com/edwardGunawan/a94117f57a7289530759a0700ac684ca.js"></script>

In this example, we spawn 4 [fiber](https://typelevel.org/cats-effect/datatypes/fiber.html), which is the equivalent of thread, and wire them with `circuitBreaker`. We set the config for the `circuitBreaker` to have a max number of retry of 3. Therefore, 3 failures call will set the status of the state to Open. The subsequent call will then failed. However, after waiting for 400 milliseconds, the last market, `circuitBreaker.protect(callSuccess)` will set the state to Half Open. Once the call succeeded, it will assess the condition to Close again. 

I hope you learn something about circuit breaker and their internal implementation. Tose the full code, you can check out the GitHub link [here](https://github.com/edwardGunawan/Blog-Tutorial/blob/master/ScalaTutorial/circuitBreaker/src/main/scala/circuitBreaker/CircuitBreaker.scala). 

Finally, some great resources about circuit breakers and other articles inspired me to implement this circuit breaker. Check out the resources below!
- [Circuit Breaker Pattern with Cats Effect](https://levelup.gitconnected.com/circuit-breaker-pattern-with-cats-effect-25947d0a4cba)
- [CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html)



