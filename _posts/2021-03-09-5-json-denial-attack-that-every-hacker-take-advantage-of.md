---
layout: post
title: 5 JSON Denial Attack that Every Hacker Take Advantage Of
date: 2021-03-09 22:52
summary: Do you check all these scenarios when you parse JSON input in your Application?
categories: programming
tags: programming
---

![Photo by Ferenc Almasi](https://images.unsplash.com/photo-1610986602538-431d65df4385?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1382&q=80)

Designing application based on security is as important as designing for scalability.

In 2017, a group of attackers sent a spoofed packet to 180,000 servers in [Google Cloud](https://www.zdnet.com/article/google-says-it-mitigated-a-2-54-tbps-ddos-attack-in-2017-largest-known-to-date/) and reached a size of 2.54 Tbps. This sort of attack is an attack not to get the password or personal information but to simply take down the service.

A Denial Of Service attack is an attack to disrupt the normal service of the targeted server. Just like how a huge amount of payload can take down a server, an attacker can use the same method to deliberately take down an application server. The key to preventing this usually is the rate-limiting service on top of your application to limit the incoming rate. It throttles the number of requests coming into your system to protect your server. One example is when you try to enter the wrong password more than a number amount of time, the application will temporarily lock you out - tell you to wait for a couple minutes and try again. 

Another example is if you try to scrap a website, it will tell you to verify yourself as a human. These prevent the attacker from using brute force methods or automation to disrupt your server. 

This sort of attack is quite easy to mitigate. We have various checks within our frontend and a rate-limiting service to prevent malicious users from disrupting the service. However, I want to share another way of dos attack that we, as a developer, often oversee that hackers often used to take down servers. 

These are denial of service attacks through JSON payloads. We often didn't put these guards on our APIs and making our server work harder than it should.

In this article, I will be talking about these interesting JSON denial attack and how we can be aware of and mitigate it in the future.

Let's take a look at one JSON payload from Medium:
<script src="https://gist.github.com/edwardGunawan/a2effc866c5a44c75bce9c3f12d50271.js"></script>

This is an API that Medium provided for writers to create a POST article.

Imagine if the backend server in Medium will also receive the exact same JSON message. There are a couple of things that a hacker can do:

## Huge Message Body
Adding a huge message body will require the server to parse all those JSON payloads.

JSON allows continuous whitespace. Therefore, we can add 1GB worth of whitespace at the `content` to make it less obvious. 

The communication channel is compressed between the server and the client - someone who sends this payload doesn't need to pay for a 1GB usage. However, the server will get that huge payload and try to parse it. The 1GB worth of whitespace is now loaded in the memory somewhere in the server.

Essentially, we are attacking the server heap consumption. 

### How to mitigate this
To mitigate this, you can try to do limit the number of messages in the JSON body. Let's say the application limit the JSON body to 1MB. This sounds like an easy one. 

However, there are always ways to attack your server with a 1MB JSON body.

## Add Redundant Data
Hackers can find an object with a valid JSON and add the key and a huge value or find an optional field value. The domain usually does not do any business processing logic and adds as much redundant data as possible. Put [Unicode](https://en.wikipedia.org/wiki/List_of_Unicode_characters), which makes the server parse harder, and make as much as the limit of the JSON message - 1MB.
<script src="https://gist.github.com/edwardGunawan/4ef8f70cd0c93d1f16f31148a3bfd63b.js"></script>

The server will not only need to parse this. It has to deal with the fact that there is Unicode in there, will have to allocate memory. It ended up allocating more memory than it needed and takes up a lot of processing power. 

Another way to attack the CPU power for creating a huge message body is through making multiple redundant messages. Usually, the JSON library will need to parse the string into an AST to be easily processed, and it will need to process these attacks no matter what.


They will take advantage of the fact that the server will need to parse each JSON value in memory. 


### How to Mitigation This
One way to mitigate this sort of attack is to put a size limit on your JSON payload. If the JSON payload reaches a certain threshold, give some sort of warning to the client.

## HashCode Collisions
When these APIs are processing JSON body content, they must create a hash table of all JSON nodes. During this process, the hashmap will compare if there are any hash key collisions.

Hackers can leverage this by making as many keys inside the JSON payload, with some `null` value to cause as much collision as possible. This makes the server do a lot of work.

<script src="https://gist.github.com/edwardGunawan/45311ad44e0a86d0dde0ebcc5166494d.js"></script>

Why does HashCode collision cost so much work on the server-side?

We need to look at how HashMap was created. A Map is just a List of the tuple (key and value). We usually consider HashMap fast on lookup is when we assume that the hash function is good at distributing value in the array that causes the minimal collision. 

When we do a `putItem` on a hash map, the data structure will do some hash algorithm to check if the key exists in the location. If the key exists in the location, it will be a collision, and there are a couple of ways to resolve the collision - open addressing and separate chaining. From there on, the lookup function will be linear time.

Each hash map has some sort of load factor. This load factor is a factor that determines whether the data structure needs to allocate more memory for receiving a bigger load. If the amount of item in the array reaches the load factor threshold, the hash map will need to recompute the hash again to grow the map. This operation will take some time and more power to process than other results.

### How to Mitigate This
Setting the maximum number of JSON keys per message can help mitigate hash collisions in your payload.


## Leverage Cache Misses
In the JVM, every object has a hashcode function. To know if two objects are the same, we compute their hashcode to see if they are the same. HashMap invokes this function to determine the object index. 

To perform O(1) lookup in HashMap or HashSet, it will need to compute the hashcode for a specific object to determine which index of the array it will be placed to. When you want to look up or delete the object, you will also compute the hash to get the index's value. The only overhead of HashSet is calculating the hash of the object. 


Now, in the old JDK, the hashCode function implementation didn't memoized values on hashCode that is equivalent to 0. The function is something like this:

<script src="https://gist.github.com/edwardGunawan/e1f293e7b65dd166cf0c22c027c1b9ce.js"></script>

If the hashcode results in 0, it will compute the value in line 4 - 6.

Therefore, each time we have a string value with the resulting computed hashCode to 0, it takes a longer time to compute the hashcode.

One way to create a DOS attack on JSON is by having multiple key values that cause hash collisions and hashCode values of 0 so that the server run an expensive computation. Just create a bunch of key strings that will be computed as a 0 hashCode.


You can search [online](https://stackoverflow.com/questions/18746394/can-a-non-empty-string-have-a-hashcode-of-zero) on what hashCode causes the computed result hash value to 0, and you will see a list of string and valid English characters that makes hashCode to 0.

<script src="https://gist.github.com/edwardGunawan/45311ad44e0a86d0dde0ebcc5166494d.js"></script>

The above json example `hashCodeCollision.json` on `zipped daydreams thunderflashestitle` resulted in hashcode value of 0. 


### How To Mitigate This
The way to mitigate this is similar to the Hash Collision case. Set a max amount of keys that you can compute so that the server will not parse a million keys that resulted in 0 hashCode values.

## Range Attack
Netflix security team gave a great presentation on [DefCon 2017](https://www.youtube.com/watch?v=a29ILArKjGo) that they describe one of the frequent DDoS attacks that they experience in their microservice as the Range Attack. 

Having a range based API attribute allow for quick access to data. However, this capability can also be abused for the bulk extraction of data. What happens when we try to web scrap data for a search query and return many ranges. 

Let's take an example where a user can submit a `searchTerm` to the server with a variable range.

<script src="https://gist.github.com/edwardGunawan/13b86d6eab2f4fe766c5efc6c96ee4ce.js"></script>

If your server didn't check for the maximum range number of values, the attacker could make the range to `[1, 10000000]` to let the server retrieve a huge amount of data. 

This makes some microservices in your application work really hard - so hard that it takes down your entire application.

### How To Mitigate This
Having a limit on the number of ranges that users can retrieve the data will decrease the amount of load on some of the microservices doing lots of processing.


## Closing
The method of attacks is now more sophisticated than before. The attackers on the Google Cloud service use a cumulative six-month campaign that utilized multiple attacks to hammer the server infrastructure. The Netflix security team at Defcon 2017 mentioned that it took them the cost of 1.71 dollars to run the five-minute test on one of the evacuated data center regions in Netflix to make a 5-minute shortage. 

These use cases seem quite serious, but they help us raise awareness of attacks that look subtle but can cause huge server outages.

Although we cannot prevent 100% of attacks, I hope these four ways of denial of service attack through JSON payload can increase your awareness when you create your next application.

## Resources
- [The Dark Side of APIs: Denial of Service Attacks - Akamai Security Intelligence and Threat Research Blog](https://blogs.akamai.com/sitr/2018/08/the-dark-side-of-apis-denial-of-service-attacks.html)
- [The curious case of Java String HashCode \| by Animesh Gaitonde | Medium](https://animeshgaitonde.medium.com/the-curious-case-of-java-string-hashcode-6d98c734a313)
- [Google says it mitigated a 2.54 Tbps DDoS attack in 2017, largest known to date \| ZDNet](https://www.zdnet.com/article/google-says-it-mitigated-a-2-54-tbps-ddos-attack-in-2017-largest-known-to-date/)
- [Denial Of JSON in Scala](https://www.youtube.com/watch?v=3Cz6D8JLSSA&t=340s)
