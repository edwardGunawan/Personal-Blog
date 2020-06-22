---
layout: post
title: Algebraic Data Types and Its Generic Presentation In Scala
date: 2020-06-21 23:28
summary: 2 Ways to represent product and coproduct in Scala
categories: scala functional-programming shapeless
tags: scala functional-programming shapeless
---

_![Photo by Diomari Madulara](https://images.unsplash.com/photo-1468404166635-56e2d75ee491?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)_

The essence of programming is to encode, decode, and to manipulate data in a useful way. Therefore, there are ways to represent data based on different programming philosophies. In Functional Programming, data representation is usually in Algebraic Data Type.

Today, I would like to explain about two ways of representing ADT in Scala briefly. The first is `sealed trait` and `case class`. The other is not often discussed in describing data, but it is an alternative way of generalized data representation if you want to do more generalized programming. 

## A Visit to ADT (Algebraic Data Type)

Algebraic data types (ADT) is functional programming concepts with a fancy name for data representations using "ands" and "ors".

In the ADT terms, "ands" are usually named as products, and "ors" are coproducts. We can also say coproducts as Sum Types. If you are interested in knowing more about ADT, check out my [previous](https://edward-huang.com/functional-programming/2019/12/30/what-is-an-adt-algebraic-data-types/) post about ADT.

Let's look at an example of how you can represent data of the description below:

1. A Payment Method can be a CreditCard __or__ Paypal
2. A CreditCard has a first four __and__ last six
3. A Paypal has a username __and__ password

## Representing with Case Class Sealed Trait

In Scala, we will represent the product as `case class` and coproduct it as `sealed trait`, which means "ands" with `case class` and "ors" with `sealed trait`.

```scala
sealed trait PaymentMethod
case class CreditCard(name:String, expiryDate:Instant) extends PaymentMethod
case class Paypal(username:String, password:String) extends PaymentMethod
```

The beauty of ADT is type safe. That means, the compiler has a complete knowledge on what the data type is, enabling us to write complete, correctly typed method, involving our types (basically easier to pattern match):

```scala
def derivePaymentMethod(paymentMethod:Payment): String =  paymentMethod match {
  case CreditCard(name, expiryDate) => s"Creditcard name ${name}"
  case Paypal(username, password) => s"Paypal user $username"
}
```

## Alternative Data Encoding
`sealed trait` and `case class` are undoubtedly the most convenient ways of representing data. One being it is easier to reason and understand, and it makes encoding data type-safe. However, there are other ways to encode data into ADT in Scala. In Scala standard library, we can represent the product as _Tuples_ and coproduct as _Either_.

Let's take an example from above:
```scala
type PaymentMethod2 = Either[CreditCard, Paypal]

type CreditCard = (String, Instant)

type Paypal = (String, String)

def derivePaymentMethod(paymentMethod: PaymentMethod): String = paymentMethod match {
  case Right((username,password)) => s"Paypal user $username"
  case Left((name, expiryDate)) => s"Creditcard name ${name}"
}
```

Using _Tuples_ and _Either_ is less readable than _case class_ and _sealed trait_ from the first example, but both have the same desirable properties.

However, _PaymentMethod2_ is more general than _PaymentMethod_. Any code that operates with a pair of `String` and `Instant` can also work with _CreditCard_ and vice versa. 



## Conclusion
As a scala developer, we prefer to represent data in a semantics with _case class_ and _sealed trait_ instead of the generic ones _Tuples_ and _Either_. However, in some cases, generic is desirable. For instance, if we want to serialize data into HTML components, we don't care about a String or Paypal pair. We write the two numbers into an HTML format and be done with it. Generalized programming also helps to solve problems with a wide variety of types with little code, avoiding repetition between various types.


To learn more about [Generalized programming with Shapeless](https://books.underscore.io/shapeless-guide/shapeless-guide.html).

