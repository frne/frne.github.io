---
title: "Functional Java 1 - Options"
image: "images/post/legacy-blog-article-without-images.jpg"
date: "2016-11-22T08:22:49+01:00"
author: "Frank Neff"
tags: ["java", "library"]
categories: ["Programming", "Functional Programming"]
draft: false
---

This is the first post of my series about functional programming in Java. There's a lot of functional stuff one can do. Everyone knows the Java 8 Lambda expression, but with a little library support, there is way more... In this series, I'll coder som libraries which provide functional paradigms and constructs for Java:

<!--more-->

- [FunctionalJava](http://www.functionaljava.org/)
- [JavaSlang](http://www.javaslang.io/)

## Java 8 Optional

There is a native optional type since Java 8, called `Optional<T>`. It's handy and covers the basic need: A typesafe alternative to `Null`.

## Functional Java Option

The [Functional Java library](http://www.functionaljava.org/) has also an `Option<T>` type. It is not something completely different and has the same basic functionality as the Java Optional type. It's just kind of personal preference which one you use, but I'll recommend to use only one of them per project. The basic operations are heavily inspired by Scala:

```java
Option<String> opt1 = Option.some(new String("foo"));
Option<String> opt2 = Option.none();
Option<String> opt3 = Option.fromNull(new String("foo"));
Option<String> opt4 = Option.fromNull(null);

opt1.isSome() // true
opt2.isSome() // false
opt3.isSome() // true
opt4.isSome() // false
```

### Mapping

On options, there is also a method `isNone()`, but in general, you'll need those two methods rarely, because in most of all cases, you'll map options. In the following example, `fj.data.Option.*` is statically imported, so we don't need the object-prefix:

```java
final Option<Integer> opt1 = some(10);
final Option<Integer> opt2 = none();

opt1.map(n -> n*2) // some(20)
opt2.map(n -> n*2) // none()
opt2.map(n -> n*2).orSome(0) // some(0)
```

The `map(f)` function only applies, if the Option is of type `some()`. Otherwise, nothing happens. The last statement shows how to apply a default value using `orSome(T)`. Of course it is also possible to provide a Lambda to `orSome(final F0<A> a)`:

```java
opt2.map(n -> n*2).orSome(() -> 2) // some(2)
```

