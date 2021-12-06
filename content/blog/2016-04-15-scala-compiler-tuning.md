---
date: "2016-04-15T22:15:28+02:00"
image: "images/post/legacy-blog-article-without-images.jpg"
title: "Scala Compiler Tuning"
tags: ["scala", "compiler", "sbt"]
categories: ["Functional Programming"]
author: "Frank Neff"
draft: false
---

As my Scala projects go on, I want to share some compiler configuration and tricks with you, which I use on many 
projects. Some tiny configuration options can greatly improve your code and warn you about things, you would probably 
never discover.

<!--more-->

Basically, you can pass compiler options to `scalac` using console arguments:

```bash
$ scalac -deprecation -unchecked -Xlint something.scala
```

If you are using SBT, it's even simpler... You can just use the following configurations snippet in your `build.sbt` 
file to add scala options:

```scala
scalacOptions ++= Seq("-deprecation", "-unchecked", "-Xlint")
```

## Common Scalac Options

There are some common options which I'd like to share with you. Most of them make it harder to compile "not that good" 
code or warn about certain conditions:

| Option                    | Description                                                                               |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| `-deprecation`            | Emit warning and location for usages of deprecated APIs.                                  |
| `-feature`                | Emit warning and location for usages of features that should be imported explicitly.      |
| `-unchecked`              | Enable additional warnings where generated code depends on assumptions.                   |
| `-Xfatal-warnings`        | Fail the compilation if there are any warnings.                                           |
| `-Xlint`                  | Enable recommended additional warnings.                                                   |
| `-Ywarn-adapted-arg`      | Warn if an argument list is modified to match the receiver.                               |
| `-Ywarn-dead-code`        | Warn when dead code is identified.                                                        |
| `-Ywarn-inaccessible`     | Warn about inaccessible types in method signatures.                                       |
| `-Ywarn-nullary-override` | Warn when non-nullary overrides nullary, e.g. def foo() over def foo.                     |
| `-Ywarn-numeric-widen`    | Warn when numerics are widened.                                                           |
| `-Ywarn-value-discard`    | Warn when non-Unit expression results are unused.                                         |
| `-Ywarn-unused`           | arn when local and private vals, vars, defs, and types are unused.                        |

This is just a very small subset of available options for the Scala compiler. If you want to see all options available, 
just ask your scalac for it. To show all the options, just run `scalac -X` or `scalac -Y`. You can also just display 
the compiler help using `scalac -help`.

Finally, my personal Scala options (as SBT settings) for most of the Scala projects I currently work on:

```scala
// compiler tuning for the win - makes it harder to build schwurbl
scalacOptions ++= Seq(
  "-deprecation",
  "-feature",
  "-unchecked",
  "-Xfatal-warnings",
  "-Xlint",
  "-Ywarn-adapted-args",
  "-Ywarn-dead-code",
  "-Ywarn-inaccessible",
  "-Ywarn-nullary-override",
  "-Ywarn-numeric-widen",
  "-Ywarn-value-discard",
  "-Ywarn-unused",
)
```

Of course there are also such options for javac, the Java compiler. Just run `javac -help` or `javac -X`...

As always, I really apreciate feedback of all kinds, or just send me your compiler configuration...
