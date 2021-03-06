# MongoLess
[![Build Status](https://travis-ci.org/a14e/MongoLess.svg?branch=master)](https://travis-ci.org/a14e/MongoLess)
[![codecov.io](https://codecov.io/gh/a14e/MongoLess/coverage.svg?branch=master)](https://codecov.io/gh/MongoLess?branch=master)

Shapeless based BSON serialization for from Mongo Java and Scala Drivers


MongoLess is a simple lib for encoding scala case classes for [Mongo Java Driver](https://github.com/mongodb/mongo-java-driver).
You can also use it with [Scala mongo driver](https://github.com/mongodb/mongo-scala-driver)

#Installation

MonadLess is currently available for Scala 2.12 and Java 8.
To install MongoLess just add following line to your sbt file
```scala
libraryDependencies += "com.github.a14e" %% "mongoless" % "0.2.4"
```


# Case class to/from bson encoding

Encoding to Bson is quiet simple: just ```import a14e.bson.auto._``` and call ```asBson```.
For decoding ```import a14e.bson.auto._``` and call ```.as[...]```.
If you want to replace field name with ```_id``` use ```ID``` wrapper.

## Simple example
```scala
import a14e.bson._
import a14e.bson.auto._

case class User(id: ID[Int],
                name: String,
                age: Int)
val exampleUser = User(
  id = 1,
  name = "some name",
  age = 25
)

val bson = exampleUser.asBson
// { "age" : 25, "name" : "some name", "_id" : 1 }
bson.as[User] == exampleUser
// true

```

Nested and recursive case classes are also supported

## Bigger example 
```scala
import a14e.bson._
import a14e.bson.auto._

case class SampleUser(id: ID[Int],
                      name: String,
                      job: Option[Job],
                      children: Seq[SampleUser])

case class Job(company: String,
               salary: Long)

val user = SampleUser(
  id = 213,
  name = "name",
  job = Some(
    Job(
      company = "some company",
      salary = 123
    )
  ),
  children = Seq(
    SampleUser(
      id = 456,
      name = "name1",
      job = None,
      children = Seq.empty
    )
  )
)

val bson = user.asBson

// { "children" : [{ "children" : [], "name" : "name1", "_id" : 456 }], "job" : { "salary" : { "$numberLong" : "123" }, "company" : "some company" }, "name" : "name", "_id" : 213 }

bson.as[SampleUser] == user
// true
```

## Bson builder

MongoLess also offers PlayJson like builder for bson:

```scala
import a14e.bson.Bson
Bson.obj(
  "_id" -> 213,
  "name" ->  "name",
  "children" -> Bson.arr(
    Bson.obj(
      "_id" -> 456,
      "name" ->  "name1",
      "children" -> Bson.arr()
    )
  )
)
// { "_id" : 213, "name" : "name", "children" : [{ "_id" : 456, "name" : "name1", "children" : [] }] }
```


## Search and Recursive search
MongoLess also supports helpers for search and recursive search. 
Use `\` to search in root and `\\` for recursive search of nearest node with expected key
```scala
import a14e.bson._
val bson = Bson.obj(
  "_id" -> 213,
  "name" ->  "name",
  "children" -> Bson.arr(
    Bson.obj(
      "_id" -> 456,
      "name" ->  "name1",
      "children" -> Bson.arr(),
      "someKey" -> 123
    )
  )
)
bson \\ "_id"
// Success(BsonInt32{value=213})

(bson \\ "_id").as[Int]
// 213

(bson \ "children" \\ "name").as[String]
// name1

(bson \\ "someKey").as[Int]
// 123

```
