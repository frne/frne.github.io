---
title: "Java Libs in Scala - A bit more Functional"
image: "images/post/2017-12-18-using-java-libs-in-scala.jpg"
date: "2017-12-18T22:04:38+01:00"
author: "Frank Neff"
tags: ["scala", "java", "implicits", "libs", "solrj"]
categories: [ "Programming", "Functional Programming"]
draft: false
---

Every Java library can be used in Scala, which is, for me, one of the good parts of the JVM world. But Java libs are 
mostly object-oriented and not functional, therefore full  of side effects and somtimes "ugly" to use in Scala. But 
there are some approaches how to make Java libs (or their interfaces) more functional, so they can almost be used like 
a Scala lib.

<!--more-->

## Java 8 Type Conversion

Many Java types like `Map` or `List`, but also functional types (Java 8) like `Optional<T>` have Scala pendents. As this
is one of the really log hanging fruits, convert those types to Scala. All collection types of Java can be converted by 
using Scalas own `scala.collection.JavaConverters._`. After importing the implicits, one can call `.asScala` on every 
Java collection:

```scala
import java.util
import scala.collection.JavaConverters._
import scala.collection.mutable

val jList: util.ArrayList[String] = new java.util.ArrayList[String]()
val sBuf: mutable.Buffer[String] = jList.asScala
val sList: List[String] = sBuf.toList
``` 

The `Optional<T>` of Java 8 is, in my humble opinion, a huge benefit, but a bit disfunctional compared to Scalas 
`Option[T]`. When using Java interfaces that return optionals, it can also be converted:

```scala
import java.util.Optional

implicit class Java8Optional[T](in: Optional[T]){
  def asScala: Option[T] = if(in.isPresent) Option(in.get()) else None
}

val jOpt: Optional[String] = Optional.of("foo")
val sOpt: Option[String] = jOpt.asScala
```

Also, Java 8 functional interfaces, like `java.util.Function<A,B>` can be converted:

```scala
implicit class Java8Function[A, B](in: java.util.function.Function[A, B]) {
  def asScala: A => B = (a: A) => in.apply(a)
}

val jFunc: java.util.function.Function[String, String] = ???

val sFunc: String => String = jFunc.asScala
```

_**Note:** Despite those conversions work, they are only meant for demonstration purposes. Scala has a compatibility library
[scala-java8-compat](https://github.com/scala/scala-java8-compat) to convert Java to Scala types. Use it._

## Implicit Classes as Decorators

As you could see, implicit classes are very handy to decorate Java objects, which you cannot change. This becomes 
especially handy when using Java libraries, like, for example 
[SolrJ](https://lucene.apache.org/solr/guide/6_6/using-solrj.html).

To simplify object construction (good old Java setters in that case) and minify observable side effects, functional 
object builders can be implemented that way. I used this approach, to design a Scala interface for the 
`org.apache.solr.common.SolrDocument`:

```scala
/** Implicit ops for solr documents */
private[search] implicit class SolrDocumentOps(in: SolrDocument) {
  def asScala = new ScalaSolrDocument(in)
}

/** Typesafe representation of a solr document
  *
  * @param underlying The original (wrapped) solr document
  */
private[search] class ScalaSolrDocument(val underlying: SolrDocument) 

  /** Helper method to add fields to existing result documents **/
  def withField(fieldName: String, value: Any): ScalaSolrDocument = {
    val doc = new SolrDocument()
    underlying.entrySet().asScala.foreach(f => doc.addField(f.getKey, f.getValue))
    doc.addField(fieldName, value)
    doc.asScala
  }
  
  /** Transform function to maintain fluid interface, because document is not monadic */
  def transform[T](f: SolrDocument => T) = f(underlying)
}
```

To prevent confusion of the newly added Scala methods with the existing interface, the only method on the implicit 
decorator class is `asScala` which returns the custom `ScalaSolrDocument` which wraps the original object. An example 
to get rid of side effects is the method `withField` which constructs a new underlying `SolrDocument` every time a field
is modified.

The usage is pretty simple:

```scala
val doc = new SolrDocument
val sDoc: ScalaSolrDocument = doc.asScala.
  withField("name", "Frank"),
  withField("profession", "programmer")
val res: SolrDocument = sDoc.underlying
```

## Implicit Conversion

Another approach to convert between the original `SolrDocument` and the custom `ScalaSolrDocument` is Scalas 
[implicit conversions](https://docs.scala-lang.org/tour/implicit-conversions.html) feature.

By providing two implicit values containing the converter functions, the Scala compiler will implicitely convert between 
the two types where needed:

```scala
import scala.language.implicitConversions

implicit val solrDocToScala = (in: SolrDocument) => new ScalaSolrDocument(in)
implicit val solrDocFromScala = (in: ScalaSolrDocument) => in.underlying

// could be a method of an interface which needs the original document
def save(doc: SolrDocument) = ???

val jDoc = new SolrDocument() // original doc

// implicit conversion to ScalaSolrDocument
val sDoc = jDoc.withField("name", "Frank") 

// implicit conversion back to SolrDocument
save(sDoc)
```

_**Note:** Implicit conversions are considered an **advanced language feature**, because things can get confusing fast. 
Don't overuse them._

