---
layout: post
title: How to Turn Domain Model into DynamoDB AttributeValue
date: 2021-08-26 00:50
summary: A brief introduction about Dynosaur
categories: scala functional-programming AWS
tags: scala functional-programming AWS
---


If you are working with DynamoDB with Scala, chances are you will spend at least one-day implementing transformation on DynamoDB attribute value to a Scala model. Creating a function that transforms Java HashMap to Scala is not the problem - the problem is transforming the domain model to attribute value. 

Imagine if you have a case class like this:

<script src="https://gist.github.com/edwardGunawan/99ebfacbda81102ac65af77faeae396c.js"></script>

You want to transform `Foo` into an attribute value map type.

<script src="https://gist.github.com/edwardGunawan/127640f936d0603ea636b5a329b4ae14.js"></script>

Now you see that this structure is a little bit hairy. This is a lucky day where you only have 3 attributes in your algebra. How about if you have 12 attributes?

Doesn't look at this make you frustrated? I know I did. 

I know what you think, "why is simple DynamoDB model transformation in Scala so hard to achieve?"

There are a wide variety of libraries out there that helps transform DynamoDB attribute value into Scala model. [Scanamo](https://www.scanamo.org/) is one popular one, which helps you do an automated derivation transformation from attribute value to your domain model. However, great power also comes with great responsibility.

## Why not use Scanamo?
Automatic derivation requires `Encoder/Decoder` type classes. This approach can cause inflexibility since your database model schema may often change over time. It can easily break the format of the derivations. Like Circe, you will need to duplicate the same line of code of encoder and decoder, so using automatic derivation in your encoder/decoder logic increases redundancy in your codebase.

"As a result, people abuse automatic derivation, and end up with either ugly serialized data and nice code model, or nice serialized data and an ugly code model." - Dynosaur documentation.

With Dynosaur, you have the flexibility to define your data based on the schema in a declarative way (you'll see what I mean by that) rather than the traversal of a low-level representation.

We will use [Dynosaur](https://systemfw.org/dynosaur/#/) use a schema definition approach to transform the Scala domain model into DynamoDB AttributeValue. 

I haven't found an article which can read through (for example, while I'm commuting), especially those that can give me a good flow in learning Dynosaur step by step in various scenarios. That's why I'm writing this blog.

 I hope that this article can ease your scala application's integration with DynamoDB.



## Basics of start writing your Scala Domain model with Dynosaur schema
Dynosaur centered around schema. Once we create our schema, we can expect to transform that schema into a `DynamoValue`. `DynamoValue` is just a thin wrapper for DynamoDB Attribute. The primary goal of this is to create easy interoperability with the AWS SDK.

Let's say we want to define a domain model like the below:
<script src="https://gist.github.com/edwardGunawan/85197a561e9937d4d55ef78cfc8fa337.js"></script>


First, create your schema:
<script src="https://gist.github.com/edwardGunawan/4287e9038bb5423528bc0021ca8f180c.js"></script>


Then, once you create your schema you can try to transform this schema to AttributeValue:
<script src="https://gist.github.com/edwardGunawan/80100c87fd485ced46b0f93b30f07010.js"></script>


Lastly, reading the `AttributeValue` into the `Invoice` schema by using `read`:
<script src="https://gist.github.com/edwardGunawan/ea14bc688f33d22a131a9fc1b515bb3a.js"></script>


Sounds simple, eh? Let's check out some gotchas and tips for working with this library.

## Tips on Working with Dynosaur

Most of the tips on working with Dynosaur revolve around schema creation. Before we start, let's create a domain model. Imagine that we want to encode and decode invoices in our system. 

Let's say we have an `invoiceId`, an `orderCode`, and a `metadata` field:
<script src="https://gist.github.com/edwardGunawan/bc34e8062135195125683d28d6ec9287.js"></script>


In the metadata field, there is `isTest`, `region` , and `sourceOfRequest`:
<script src="https://gist.github.com/edwardGunawan/15ea98da6e99795938d322b13cbc8046.js"></script>



Now, let's make it more complicated where there is the status of the invoice.
<script src="https://gist.github.com/edwardGunawan/fb062ca6f420dd53c62412a75b313714.js"></script>


Then, let's assume that you can update the value of the database if the invoice has been received by the other services. We will make it an ADT of `Received`:

<script src="https://gist.github.com/edwardGunawan/56962d4d90b224adb37feebf81f971f9.js"></script>


This is what the model overall looks like:
<script src="https://gist.github.com/edwardGunawan/f41c3920d96d1963b5623ecd80483ef8.js"></script>


## Creating Schema with Regular Case Class
Creating a simple case class schema will use `records`:

<script src="https://gist.github.com/edwardGunawan/fd186f08840e842fe7e2492401cd51f1.js"></script>


Realize there is an error that Dynosaur cannot identify the schema of `region`? 

It means Dynosaur didn't provide that schema definition out of the box. 

Since we use the AWS region type, we need to create the `Region` schema, and we want the `Region` value in our scala case class to transform into a `String` value in DynamoDB. 

<script src="https://gist.github.com/edwardGunawan/573f9b865ea47cae969f333ef78fb17e.js"></script>

`imap` represents the definition of bi-directional transformation between `Region` and `String`. 

We want to keep a specialized representation of `Region` in our code and represent `region` as a `String` in DynamoDB.

Now, we can provide the `regionSchema` in our `metadataSchema` definition:

<script src="https://gist.github.com/edwardGunawan/168486e774b9371cecf43fc718641652.js"></script>


Now, let's define the `Received` ADT schema.

## Define Enum Fields
Let's define the `Received` Coproduct. In this case, we will use `imapErr`. The difference between `imapErr` and `imap` is that `imapErr` provides a case where encoding cannot fail, but decoding can. 

For example, transforming between `Received` ADT to string will not fail, but decoding from a string to `Received` ADT may cause a parsing error. 

First, we need to create a transformation for `Received`:
<script src="https://gist.github.com/edwardGunawan/f232ad045d0dda5d241913b7f7a56a1a.js"></script>


`imapErr` takes in two-parameter, one for decoding and the other for encoding.

<script src="https://gist.github.com/edwardGunawan/6932fbbbb10aa3b2caca98bd3f9715a3.js"></script>

Using the `String` schema to decode the `String` type to the `Received`  coproduct may cause errors. `toRight` will transform the `Option` type into an `Either` type. 

The second argument is to encode `Received` coproduct into a `String`.

## Define Coproduct Schema
This is where things get very interesting and sometimes tricky. The documentation mentioned using `oneOf` to define the coproduct schema. We use the `Status` model from the example above as a use case. 

Dynosaur uses [`Prism`](https://www.scala-exercises.org/monocle/prism) internally to choose which model for encoding or decoding. 

For encoding, it pattern match to transform the model into the DynamoValue. Decoding checks whether the coproduct actually matches a given case, and if there is one successful one, it will choose that one to decode. If none of them matches, it will simply throw an error.

 Let's look at how we can do define `Status`.

<script src="https://gist.github.com/edwardGunawan/00c80bdf9e0bbd5a881219aa1f2a6007.js"></script>



It looks like it is okay, but there is a possibility of ambiguity. As the document mentioned, use discriminator keys and fields for solving ambiguity.

However, one of the gotchas is to add a safeguard by adding a discriminator key and field on the coproduct to differentiate between two case classes. 

Since the schema definition uses `|+|`, which serves as a combination for monoid but an `orElse`. It will decode whatever matches that schema definition. In the case of `Status` coproduct, if we want all changes to be written to DynamoDB and have an attribute that determines which status the current value, having another constant field, such as `type` that updates the current status. We need to create a constant in our schema to differentiate the model.
<script src="https://gist.github.com/edwardGunawan/0932ec896772676afd5a9b817857a336.js"></script>

Another scenario is defining all optional fields in a product and wanting to have that value as a coproduct.
<script src="https://gist.github.com/edwardGunawan/d006431867fefe6935f1b47d9ea6c18b.js"></script>

Decoding all optional fields in the coproduct will require discriminator keys to solve the ambiguity. Either by using a `tag` or a `const`.

<script src="https://gist.github.com/edwardGunawan/ac337a6dfc4a3f7314a4fc8b6f6b9ae6.js"></script>


Let's finish up with this model creation. if you want to create a schema for `Invoice`, you can just create a `record` of schema:
<script src="https://gist.github.com/edwardGunawan/0197c303df49e466b7a1ebe161d120ac.js"></script>


## Conclusion
That's the end of this blog post. If there's a TLDR from this, they are
- Define discriminator keys to solve ambiguity as a safeguard in coproduct
Imap is a transformation that won't cause any error, and ImapErr is a transformation that there is a possible error in the decoding portion.
- Remember to set the most specific value first on the `|+|` on coproduct because it serves like an `orElse` or pattern matching. If you catch all the schema at the beginning of `|+|`, you won't get the desired schema.

I hope that you enjoy this post, and any feedback is welcome. How do you usually decode the domain model to DynamoDB Attribute? Comment down below!

## Extra Reading
- [Dynosaur](https://systemfw.org/dynosaur/#/schema)
