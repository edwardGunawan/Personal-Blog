---
layout: post
title: How to Implement Paxos Algorithm in Pure Functions
date: 2021-03-18 08:36
summary: Implementing state machine replication in a stateless manner
categories: algorithm
tags: algorithm
---

<img src="{{site.baseurl}}/images/how-to-implement-paxos-algorithm-in-pure-functions/Paxos Made Functional Background.png" alt="Profle Pic"/>

Imagine that you are on a football team. After practice, the team loves to go out together and eat. Let's say that the team usually wants to eat pizza or burgers. However, you want the whole team to go out to the same place after practice because it is more fun that way. Therefore, you need to get the team to agree on the football field to where everyone is going - getting burgers or pizza.

But there is one problem - the coach went home early. Everyone is exhausted and hungry after the practice, so they got easily distracted and want to come to a decision fast. Moreover, you cannot yells at the football field cause the entire team tends to talk really loud, and your suggestion can be easily overwritten by the other person.


You must be thinking, "There is only one way that I can achieve this agreement, is through person-to-person communication, and want everyone to achieve a consensus."

There is a similar problem in the distributed system, but you are dealing with many servers this time. There are many ways you want a bunch of servers to reach a ceratin agreement, but today we will talk about Paxos. Paxos is one of the earliest published papers about this distributed consensus algorithm that runs rounds and rounds of times to help many servers agree on a value proposed by a group member.

## Before we start
In this article, you will not get to see the "why" and steps of reaching the algorithm consensus - if you are interested in "why" Paxos Algorithm works, you can look at this great brief introduction on the Google [Tech Talk](https://www.youtube.com/watch?v=d7nAGI_NZPk). Secondly, I assumed that the proof and the theory works, and this article will be mainly about the implementation.

## A Brief Paxos Introduction
There are 3 roles in the Paxos algorithm - the proposer, the acceptors, and the learner. 

The proposer will propose a value by sending messages to another member of the group.

Acceptors will accept the proposed value.

The learner learns whether the group has reached consensus in a particular round of the algorithm.

<img src="{{site.baseurl}}/images/how-to-implement-paxos-algorithm-in-pure-functions/Paxos Role Simple Diagram.png" alt="Paxos Made Simple Picture"/>

Paxos Algorithm takes in two phases (prepare phase and accept phase) as follows:

### The Prepare Phase
Each group's proposers select a proposal number and send a prepared request to the system's acceptor. The message doesn't need to be received by all of them. It just needs to be the majority (a quorum) for the algorithm to proceed.

The acceptor who receives this message will make a comparison with the current highest proposal number. If the incoming request is higher than the proposal number, it will accept and sends a hopeful message back saying to the proposal, "okay, your proposal is higher than what I currently have; I will choose you." 

If the acceptor already accepts a message, it will send the same thing, except it will say, "okay, your proposal is higher than what I currently have. However, I have already accepted a message proposal. I'm going to attach that proposal number and value in the message too." 

If the acceptor receives the message with a lower proposal number, it will just simply ignore it.

<img src="{{site.baseurl}}/images/how-to-implement-paxos-algorithm-in-pure-functions/Prepare Phase.png" alt="Prepare Phase"/>

### The Accept Phase
If the proposer receives a majority's promise response, it will check if any promised messages have an accepted message. If it has an accept message, the proposal will accept the message to send the accept request to the processor again.

Suppose the proposer doesn't receive responses from most acceptors that it broadcasts its message. In that case, it will simply assume that "my proposal number is not high enough, I'm going to create a higher proposal number and broadcast it again to the acceptor."

If the proposer receives responses from the majority of the acceptor, it will inform the learner that it has reached a consensus.

Suppose an acceptor receives an accept request with the proposal number equal to its promises. In that case, it will send back a confirmation to that proposer that the proposal value is accepted. 

If an acceptor receives an accept request with the proposal number less than the prepared request, it simply ignores it.

On the learner side, once it receives the majority (quorum) of value coming from the proposal, it will mark itself as it has reached a consensus.

<img src="{{site.baseurl}}/images/how-to-implement-paxos-algorithm-in-pure-functions/Accept Case.png" alt="Accept Phase"/>

In theory, it sounded simple. However, in practice, it has lots of cases that needed to account for. A couple of [papers](https://static.googleusercontent.com/media/research.google.com/en//archive/paxos_made_live.pdf) talks about their experience in implementing Paxos in the production system. I saw many sources implement Paxos based on an object-oriented language, such as the java [implementation](https://github.com/cocagne/paxos#cocagnepaxosessential) or implementation with [actor system](https://github.com/ahanwadi/paxos). So I thought, "why not try to implement one in functional programming terms?" 

Please note that this implementation is based on the Paxos-made simple algorithm paper. It doesn't have any fancy invariant such as an account for machine failure, leader election...etc. This implementation also implements a single-round basic Paxos algorithm.

## The Challenge

### Model Construction
The essence of the Paxos algorithm is communication between nodes. Therefore, it will be the most intuitive to model the role as objects. Since it mentioned how proposers, acceptors, and learners interact, an actor system is the easiest way to create the algorithm. We can have 3 actors - encapsulate the logic and state within the actor. 

### Immutable State Management 
This can be a challenge if we want to implement something immutable by nature because the algorithm requires a constant change in the internal state. Object-oriented programming can encapsulate the state management for proposers, acceptors, and learners. On each of these phases, we can mutate the state inside each instance. For instance, an acceptors instance can have a max_id as a mutable internal state, and it can change the max_id if it receives a prepared message that contains a higher id number. 

## Implementation
Let's discuss the implementation and design of implementing the Paxos algorithm functionally. I will use scala and leverage the cats' data library to not reinvent the wheel of creating its own monad instances and its law.


### Domain Models
Let's start with each of the domain models. The simplest way to do this is to visualize what the proposer, acceptor, and learner needs.

#### Proposer Model
<script src="https://gist.github.com/edwardGunawan/95528a2d8ef56c091d884de24479c4d0.js"></script>

The proposer will consist of a value, a proposal number, and a quorum size. The proposal number needs to be unique and in increasing number. The common way to implement this proposal number is an id with machine-id to ensure uniqueness. 
<script src="https://gist.github.com/edwardGunawan/9bae19520c83d074937eb986f5a734d4.js"></script>

The compare statement is equivalent to a Java comparator where you can check if one is equal to another.

The quorum size is the number of nodes in the system. 

#### Acceptor Model
The acceptor consists of a promise proposal number and the proposal number and value it accepts. Since those two values might not exist, let's make them optional.
<script src="https://gist.github.com/edwardGunawan/f6749b1bd04ea6080356f19788da03e1.js"></script>

#### Learner Model
The learner needs to keep track of all the accepted responses that it receives and check if the incoming value is over the majority. Therefore, it needs to have a key-value mapping to keep track of those counts. The key will be the accepted value, and the value will be the number of counts. To know the majority, it needs to have the quorum size. Also, there needs to be a way to chose a value when it is above the majority.  Therefore, the learner will have a quorum size, a mapping for accepted value so far, and the final value that is chosen.

<script src="https://gist.github.com/edwardGunawan/b10f028898956e3a3a8b7572f747d0aa.js"></script>


Usually, in the OOP way, we will put the functions inside the model, and its function will mutate the state of the model. Functionally, I decided to separate the functions into their own object, `Ops` that can interact with the proposer. These separate the model and the operation so that we separate the concern on state mutation. 

### Messages
Next, we will think about the message type to communicate for each two-phase algorithm. There are 4 message type total - messagePreapre, messagePromise, messageAccept, and messageAccepted. Let's combine these messages as a sealed trait.

<script src="https://gist.github.com/edwardGunawan/74f270702d7c1fdf5e912d18e5410412.js"></script>


Since an acceptor possibly has not accepted any proposal when responding to a promise request, we will use an Option type to embed the `AcceptedValue` in the `Accept` message.

### Action
One of the benefits of implementing Paxos or any other consensus algorithm in pure functions is that we start thinking of putting all the side effect IO in one component. There are many ways to push IO's boundary to the [end of the world](https://stackoverflow.com/questions/13340458/what-does-the-world-mean-in-functional-programming-world). However, one easy way to do this is to change the statement into a value. Therefore, we will define an `Action` type in the implementation. This `Action` type will be a value type that describes any impure side effects required by the protocol. For instance, in Paxos, the protocol's side effect sends messages to other machines and processes. Therefore, this action type contains `Broadcast` and `Send`, which will broadcast the proposer, either the acceptor group or the learner group. Therefore the Action coproduct will have something like this:

<script src="https://gist.github.com/edwardGunawan/8d7ce27c8a85407c940abf225bb82e76.js"></script>

## Creating a Pure Message Handling Logic
Once we separate the algorithm's impure effects, we can focus on the algorithm's core state management.  

How do we manage to have state management without mutation?

One way to do this is that each function will be wrap in a `State` monad. A state monad is a monad that takes in the previous state, and it returns a new state with some results. If you are not familiar with it, check out my previous blog post about State monad [here](https://edward-huang.com/scala/functional-programming/monad/programming/2020/12/21/must-know-patterns-for-constructing-stateful-programs-without-any-mutation/).

Therefore, we can have a function that has a wrapper of `State[Proposer, Action]`, which translates to `Proposer => (Proposer,Action)`.

To simplify the design, I created three `Ops` objects - ProposerOps, AcceptorOps, and LearnerOps - to simplify the implementation's design. This kind of design is influenced by how the actor system or object-oriented way of implementing the algorithm separates each of the operations inside the role's object. The code is more modular and clean this way.

<script src="https://gist.github.com/edwardGunawan/4db7adb40e149ed19d10d09227545ed4.js"></script>


Each of the function operations inside the `Ops` object class will do the action to mutate the role. For instance, let's take a look at the prepare phase on the acceptor. The acceptor receives the `PrepareMessage` and evaluates if the `proposalId` is the current max `proposalId` seen so far. If the `proposalId` is the max `proposalId` seen so far, it will reply back with a `PromiseMessage`. If the value is less than the current max id, it will ignore it.

<script src="https://gist.github.com/edwardGunawan/0d56c182f0c8c2a129e59761c85919a1.js"></script>


Therefore, most of the function will have function definition like this:

<script src="https://gist.github.com/edwardGunawan/ee2e032f31283ce66a8b248f9357e112.js"></script>

With this, the algorithm's core logic becomes stateless - it doesn't really care about each machine's internal state and process. We have created pure state management. This makes testing and debugging the Paxos algorithm easier.

The important part here is that the `Action` method can be carried out by some other functions outside of the Paxos Algorithm to call those side effect calls.



## Closing
We just implemented a Paxos algorithm in Pure Functional style. This can be quite hard to implement because the algorithm inside the paper is a stateful algorithm. You need to keep track of each state to reach a consensus. 

Nevertheless, we take the approach to separate all algorithms into two logic - the impure functionality and core state management.

Separating the IO and the core state management logic in the algorithm is the biggest advantage to make the algorithm testable even in the concurrent environment. Further, we also make each message handling logic as stateless as possible - each call can finish without consulting any internal states of each machine or process. Lastly, constructing all your algebra and all the messages that it needs at the beginning really helps shape implementing the algorithm itself. 

The rest of the code implementation is in this GitHub [here](https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/paxos/src/main/scala/paxos).

If you are interested to learn more about the Paxos algorithm and implementation, you can check out these resources:
- [scala-composable-paxos/Learner.scala at master · cocagne/scala-composable-paxos · GitHub](https://github.com/cocagne/scala-composable-paxos/blob/master/src/main/scala/com/github/cocagne/composable_paxos/Learner.scala)
- [Understanding Paxos](https://understandingpaxos.wordpress.com/)
- [Paxos in Haskell](https://www.scs.stanford.edu/14sp-cs240h/projects/ma.pdf)
