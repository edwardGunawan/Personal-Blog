---
layout: post
title: Parsing Json with Circe - Beyond the Basics
date: 2020-07-21 22:55
summary: Encode and Decode ADT with Circe
categories: circe
tags: Circe tech scala functional-programming
---

![Photo by Patrick Fore](https://images.unsplash.com/photo-1505678137082-8a4376f90c86?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1500&q=80)

Circe has been the go-to parsing Json Library in Scala. The power of Circe lies in the fact that it can polymorphically derive Json String to an ADT. However, I experienced frustration the first time using Circe - partly because I was new Scala as a Programming Language and touched into the world of Functional Programming. Sometimes, the error messaged is not transparent, or there is some specific configuration that needs to go through the source code to achieve particular goals.  

This article is an on-going continuation of the article [7 quick Tips for Parsing Json with Circe](https://medium.com/@edwardgunawan880/6-quick-tips-to-parse-json-with-circe-9bbe51ce5778). 

As I developed more and more applications with Scala, and gain more understanding in Functional Programming Paradigm, I want to share all the gotchas that I encountered while parsing Json with Circe. These are the use cases that I met in my workplace and how I solved them. 

## Encoding/Decoding Coproduct (Sum) Type in an ADT
Sometimes we want to decode a coproduct in ADT with a different string representation than the coproduct type. 

For example, we have 4 Houses in the [Hogwarts houses](https://en.wikipedia.org/wiki/Hogwarts) in Harry Potter. 

We want to model the four houses as a coproduct type as an ADT, and be able to encode/decode the respective `case object` polymorphically:
```scala
case class House(`type` : HouseType)

sealed trait HouseType
case object GodricGryffindor extends Houses
case object SalazarSlyntherin extends Houses
case object RowenaRavenclaw extends Houses
case object HelgaHufflepuff extends Houses
```

However, after talking with other teams about the contract, they decide to send the `Houses` types as a Snake_Case. 

```
"Godric_Gryffindor" => GodricGryffindor
```

If we use `circe.generic.semiauto.{deriveEncoder,deriveDecoder}`, the result of the JSON type will be `GodricGryffindor`.

```scala
{
  "type" : "GodricGryffindor"
}
```
First, define the encoder and decoder instances for `House`.
```scala
import io.circe.generic.semiauto.{deriveEncoder, deriveDecoder}

implicit val houseEncoder:encoder[Houses] = deriveEncoder
implicit val houseDecoder:encoder[Houses] = deriveDecoder
```

Doing a quick transformation on the incoming Json and convert them to one of the `case object`.

```scala
 implicit val housesEncoder: Encoder[HouseType] = (obj: HouseType) => obj match {
    case HelgaHufflepuff => Json.fromString("Helga_Hufflepuff")
    case RowenaRavenclaw => Json.fromString("Rowena_Ravenclaw")
    case GodricGryffindor => Json.fromString("Godric_Gryffindor")
    case SalazarSlyntherin => Json.fromString("Salazar_Slyntherin")
  }

  implicit val housesDecoder: Decoder[HouseType] = (hcursor:HCursor) => for {
    value <- hcursor.as[String]
    result <- value match {
      case "Helga_Hufflepuff" => HelgaHufflepuff.asRight
      case "Rowena_Ravenclaw" => RowenaRavenclaw.asRight
      case "Godric_Gryffindor" => GodricGryffindor.asRight
      case "Salazar_Slyntherin" => SalazarSlyntherin.asRight
      case s => DecodingFailure(s"Invalid house type ${s}", hcursor.history).asLeft
    }
  } yield result

```

We can achieve a difference in the' type' field member with a little work on defining the encoder and decoder the Coproduct.

```scala
val gryffindor = (Houses(`type` = GodricGryffindor)).asJson
println(gryffindor.spaces2)

// {
//  "type" : "Godric_Gryffindor"
// }

```

## Transforming EpochMillis to Instant in a Product Type ADT

You want to create a `Currency` class that has a field of `createdDate`. 

Sample `Currency` class:
```scala
case class Currency(id: Int, name:String, description:String, isoCodeAlphabetic:String, createdDate:Instant)
```

The JSON String pass `createdDate` as an EpochMillis. However, you want to convert it to an Instant to make it easier to do any operation on the `createdDate`.

Sample `Currency` JSON String: 

```scala
{
  "id" : 1,
  "name" : "US Dollars",
  "description" : "United States Dollar",
  "isoCodeAlphabetic" : "USD",
  "createdDate" : 1595270691417
}
```

Create another encode/decode instance if you want to transform specific members of the `case class` in Circe.

You just need to create another instance in the implicit scope for converting from `Long`, EpochMillis to `Instant`.

```scala
 implicit val encoder:Encoder[Instant] = Encoder.instance(time => Json.fromLong(time.toEpochMilli))
 implicit val decoder:Decoder[Instant] = Decoder.decodeLong.emap(l => Either.catchNonFatal(Instant.ofEpochMilli(l)).leftMap(t => "Instant"))
```

Then, create an encoder/decoder for `Currency`:
```scala
implicit val encoderCurrency: Encoder[Currency] = deriveEncoder
implicit val deoderCurrency: Decoder[Currency] = deriveDecoder
```

Circe will look at the implicit scope to check if there is an encoder/decoder instance from one value to another. With implicit resolution, Circe can derive from one type to another if you provide the encoder/decoder instance of that type.


## Encode/Decode Polymorphic ADT

Let's define the JSON String that you want to derive in this use case:
```scala
{
  "houseType" : {
    "type" : "Rowena_Ravenclaw",
    "characteristics" : [
      "Loyal"
    ],
    "animalRepresentation" : "eagle"
  },
  "number" : 12
}
```

And we want to convert it to:
```scala
House(RowenaRavenclaw(List(Loyal),eagle),12)
```

Noticed that the `type` indicating what constructor names you want the JSON string to transformed it to (in this case, it is RowenaRavenclaw).

Decoding with a regular CirceDecoder will return the case class below.
```scala
House(houseType(`type`: "Rowena_Ravenclaw", List(Loyal),eagle),12)
```

How would you polymorphically decode a JSON String by matching the member of its field to the constructor name?


There are two ways. The first will be regular encode and decode, and the second will use `Circe.extras`.

### Regular Encoding and Decoding
The way that you structure the ADT makes a huge difference. To explain the workaround of the use case above, I will be using `@JsonCodec` to derive encoder and decoder for a regular case class automatically.


Defining the model type for `House` and `HouseType`:
```scala
@JsonCodec
case class House(houseType: HousesTypes, number:Int)

trait House
object House {
    @JsonCodec
    case class GodricGryffindor(characteristics:List[String]) extends HousesTypes

    object GodricGryffindor{
      val typeId: String = "Godric_Gryffindor"
    }
    
    case object SalazarSlyntherin extends HousesTypes {
      val typeId: String = "Salazar_Slyntherin"
    }

    @JsonCodec
    case class RowenaRavenclaw(characteristics:List[String], animalRepresentation:String) extends HousesTypes

    object RowenaRavenclaw{
       val typeId: String = "Rowena_Ravenclaw"
    }

    @JsonCodec
    case class HelgaHufflepuff(animalRepresentation:String, colours:String) extends HousesTypes

    object HelgaHufflepuff{
      val typeId: String = "Helga_Hufflepuff"
    }
  
}
```

In the above ADT definition, we want to have an implicit encoder and decoder for the `HouseType`. 

During encoding specific `HouseType`, we want to append a `type` field to the JSON String. 
```scala
{
  "houseType" : {
    "type" : "Rowena_Ravenclaw", << - We want to append this based on the specific HouseType
    "characteristics" : [
      "Loyal"
    ],
    "animalRepresentation" : "eagle"
  },
  "number" : 12
}
```

Circe encoding instance: 
```scala
  implicit val encoder:Encoder[HousesTypes] =  {
    // deepMerge - insert the encoded Json with another field `type`
    // Basically overriding the current encoder with the `type`
    case obj: GodricGryffindor => obj.asJson deepMerge(Json.obj("type" -> Json.fromString(GodricGryffindor.typeId)))
    case obj: RowenaRavenclaw => obj.asJson deepMerge(Json.obj("type" -> Json.fromString(RowenaRavenclaw.typeId)))

    case obj: HelgaHufflepuff => obj.asJson deepMerge(Json.obj("type" -> Json.fromString(HelgaHufflepuff.typeId)))
    case obj: HousesTypes => Json.obj("type" -> Json.fromString(SalazarSlyntherin.typeId))
  }
```

We want to retrieve the `type` field in the `houseType` JSON string and decode the entire Json String based on that type during decoding. For instance, `Rowena_Ravenclaw` will point to `RowenaRavenclaw` case class.

```scala
  implicit val decoder:Decoder[HousesTypes] = (cursor:HCursor) => for {
    tpe <- cursor.get[String]("type")
    result <- tpe match {
      case GodricGryffindor.typeId => cursor.as[GodricGryffindor]
      case RowenaRavenclaw.typeId => cursor.as[RowenaRavenclaw]
      case HelgaHufflepuff.typeId => cursor.as[HelgaHufflepuff]
      case SalazarSlyntherin.typeId => SalazarSlyntherin.asRight
      case s => DecodingFailure(s"Invalid house type ${s}", cursor.history).asLeft
    }
  } yield result
```

### Using Circe.extras
Circe has a specific library, `circe.extras`, that solved encoding/decoding polymorphic ADT. 

First, let's re-write our model by changing the HouseType to a  `sealed trait`:
```scala
sealed trait HouseType {
    def `type`: String
  }

  object HouseType {
    case class GodricGryffindor(characteristics:List[String]) extends HouseType {
      override def `type`: String = "Godric_Gryffindor"
    }
    case object SalazarSlyntherin extends HouseType {
      override def `type`: String = "Salazar_Slyntherin"
    }
    case class RowenaRavenclaw(characteristics:List[String], animalRepresentation:String) extends HouseType {
      override def `type`: String = "Rowena_Ravenclaw"
    }
    case class HelgaHufflepuff(animalRepresentation:String, colours:String) extends HouseType {
      override def `type`: String = "Helga_Hufflepuff"
    }
  }
```

You can set the `type` in the JSON String as a discriminator, indicates the constructor, in a configuration, and declare the configuration implicitly. 

```scala
implicit val houseTypeConfig = Configuration.default.withDiscriminator("type").copy(
      transformConstructorNames = {
        case "GodricGryffindor" => "Godric_Gryffindor" // from `type` on the right transform the case changes to the left
        case "SalazarSlyntherin" => "Salazar_Slyntherin"
        case "RowenaRavenclaw" => "Rowena_Ravenclaw"
        case "HelgaHufflepuff" => "Helga_Hufflepuff"
      }
    )
```

I am setting the configuration to transform the constructor names. I want to convert the entire JsonString based on one of the field `type`. The JSON field `type` contains the value on the right side of the case statement ("Godric_Gryffindor", "Salazar_Slyntherin", ...). I want to transform the JSON string to the case class on the left side of the case statement ("GodricGryffindor", "SalazarSlyntherin"). The left side of the case statement will match the model that we defined above.
 

Then, during the encoding/decoding process, use `deriveConfiguredEncoder` and `deriveConfiguredDecoder` in `circe.generic.extras` to polymorphically enode/decode the Json string:
```scala
 implicit val house2Encoder = {
    implicit val config = houseTypeConfig
    deriveConfiguredEncoder[HouseType]
}

implicit val house2Decoder = {
  implicit val config = houseTypeConfig
  deriveConfiguredDecoder[HouseType]
}
```

Testing and running the above command:

```scala
val ravenClaw: HouseType = RowenaRavenclaw(characteristics = List("Loyal"), animalRepresentation = "eagle")
val ravenClawJson = ravenClaw.asJson
val ravenClawStr = ravenClawJson.noSpaces
println(ravenClaw.asJson.spaces2)
println(decode[HouseType](ravenClawStr).right.get)
```

## Takeaway
- You can encode and decode specific Json field by providing an encoder/decoder instance of those types. Circe leverage the compiler implicit resolution to transform the JsonString to the desired ADT.
- Use CirceExtra to encode/decode Coproduct type ADT by using `circe.generic.extras.Configuration`.
- Use `deepMerge` to merge one JSON object to another and inject any specific fields that you desired.

That's it for encoding and decoding an ADT in Circe! 

I hope this article can help you get started on your next projects to encode/decode ADT type with Circe. 

All the source codes are [here](https://github.com/edwardGunawan/Blog-Tutorial/blob/master/ScalaTutorial/parsejsonwithcircetutorial/src/main/scala/DecodingComplexCoproduct.scala).
