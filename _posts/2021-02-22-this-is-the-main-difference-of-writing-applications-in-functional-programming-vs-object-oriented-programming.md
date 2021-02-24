---
layout: post
title: This is the Main Difference of Writing Applications in Functional Programming vs. Object-Oriented Programming
date: 2021-02-22 22:48
summary: It is not immutability or inheritance, but more on the structure of the application if you use functional programming vs object-oriented
categories: functional-programming programming 
tags: functional-programming programming 
---


<img src="{{site.baseurl}}/images/this-is-the-main-difference-of-writing-applications-in-functional-programming-vs-object-oriented-programming/The Main Difference of Functional Programming and Object Oriented.png" alt="Profle Pic"/>

Algebraic Data Types have their pros and cons. It enables you to write your application in a type-safe way - when you do pattern matching, it will exhaust all the data types for you. However, there is also a disadvantage of using ADT if your data types keep changing.

I'll cut through the chase and give you the answer on the difference - when you write an application with a functional programming paradigm, everything is a group by operation. When we write the program in an object-oriented way, everything is a group based on the objects. 

For me, this is a huge discovery! It explains why ADT sometimes can be a pain.

I'll illustrate what that means - and this can give you a rough idea of what I think the real difference between writing programs in functional programming vs. object-oriented programming. 

Let's make it a Tree data type. On our first try of creating a tree, it will be something like this:

<script src="https://gist.github.com/edwardGunawan/2ba4c9ab196395191a1eb862532070da.js"></script>

Then, we want to have a function that computes the height of the tree. Let's make it here:
<script src="https://gist.github.com/edwardGunawan/1925448288aa32732a2b4f53fa867dc7.js"></script>

Then, we want to have a function that computes the sum of all the branches:
<script src="https://gist.github.com/edwardGunawan/c3f77f2c4a496c04e82984bf3c261f16.js"></script>


From the two functions above, we noticed that once we created our ADT, we will need to implement the cases on each ADT - `height` and `sum` needs to implement operation based on two data types - `Branch` and `Leaf`.

If we want to implement this in a object oriented way - we will put the method inside the `Tree` interface:
<script src="https://gist.github.com/edwardGunawan/65918370340a3275d4826d4c9a7dd671.js"></script>

In an object-oriented way, we are grouping our operation based on per class. Therefore, `Branch` will have its own function of `height` and `sum`, and `Leaf` will have its own function of `height` and `sum`.


These are two ways of structuring your application can have a different impact on how you structure them. Let's see each of the pros and cons of structuring this way.

## Adding a New Data Types
We want to extend `Tree` to have another data type, `Empty, ' representing an empty child in the application. 

If we initially implement our Tree Type with ADT and group them in operations, we will need to go through each operation and add an `Empty` in the pattern matching case.

<script src="https://gist.github.com/edwardGunawan/a0c3ed41c3fbd36d65ac77bb25460dfd.js"></script>

If we have many functions in our application based on the `Tree` data types, we have to go through each of them are adding a new `Empty` case.

However, if we did it in an object-oriented way, we just need to implement a new class, `Empty`, that extends the `Tree` trait.

<script src="https://gist.github.com/edwardGunawan/b72b1eed126f481680ecf45b4ca94299.js"></script>

In this scenario, adding new data types will be cumbersome if you group your application based on data types - because we will need to change all operation in the application to include another case. However, in an object-oriented way, since each function is grouped based on the object, adding a new data type needs to implement the `Tree` interface.

## Adding a new operation
We want to add a new operation in the `Tree` called `isEmpty`.

If you group your application by separating functions and data types, creating an `isEmpty` function is relatively straightforward.

<script src="https://gist.github.com/edwardGunawan/ad7adb5984ebd166c1de61fdadb80d45.js"></script>

On the other hand, if we implement our application in an object-oriented way, we need to go through all the files in our application and implement the `isEmpty` function because we changed our interface.
<script src="https://gist.github.com/edwardGunawan/311e2bb7aa72f2a8d188ba06de5966dd.js"></script>

This has been a problem in Java. If we have multiple nested inheritances, we need to change all the class that implements the interface.

In this case, the change to add a new operation in a useful way of structuring your application code is local. However, in an object-oriented way of structuring your application, adding a new process will profoundly change the application structure.


## What do we learn here?
Structure your data model wisely! We never know the future of the application that we are building on. Therefore, flexibility is critical. 

Designing software is partly art and partly technical. Therefore, each developer will have its own style of structuring their application.

There has been a topic all over the web about what is the best practice in structuring your code. However, I think the best way is based on your current application and business logic.

If you know that your data types will keep changing in the future,  perhaps creating the value in an object-oriented way will be more flexible. If you know that your data types will most likely stay the same, functionally structuring your data model can benefit the future because new functionality will be added more than new data types.


