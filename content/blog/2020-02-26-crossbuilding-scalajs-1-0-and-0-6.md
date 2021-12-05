---
title: "Crossbuilding Scala.js 1.0 and 0.6"
image: "images/post/2020-02-26-crossbuilding-scalajs-1-0-and-0-6.jpg"
date: 2020-02-26T17:05:18+01:00
author: "Frank Neff"
tags: ["sbt", "scala", "scala.js", "library"]
categories: ["Programming", "Functional Programming"]
draft: false
---

As you may know, Scala.js `1.0.0` [just went final](https://www.scala-js.org/news/2020/02/25/announcing-scalajs-1.0.0/).
For end-products, the upgrade is rather simple, just bump the plugin version and fix potential compile issues. If
you're maintaining a library, cross-building for `0.6.x` and `1.0.x` is still a bit tricky, but possible...

<!--more-->

### How to parametrize the plugin version?

First of all, one needs to find a way to locad different versions of the SBT plugin. This is possible using the 
following snippet:

*project/plugins.sbt*
```scala
val scalaJSVersion = Option(System.getenv("SCALAJS_VERSION")).getOrElse("1.0.1")

addSbtPlugin("org.scala-js" % "sbt-scalajs" % scalaJSVersion)
```

That's basically it. Now one can release for different Scala.js versions using shell commands like the following:

```shell
SCALAJS_VERSION=0.6.32 sbt +publish
SCALAJS_VERSION=1.0.1 sbt +publish
sbt +publish # picks default version
```

As usual, the above will publish for all `crossScalaVersions`, but keep in mind, that Scala.js `1.0.x` only supports 
Scala `>= 2.12`.

### Enabling the JSDom

Scala.js `1.0.x` has externalized the JSDom support to its own package, which in turn is not compatible with sjs 
`0.6.x`. One approach to solve this, is adding the following to your `project/plugins.sbt`:

```scala
libraryDependencies ++= {
  if(scalaJSVersion.startsWith("1.")) {
    Seq("org.scala-js" %% "scalajs-env-jsdom-nodejs" % "1.0.0" )
  } else {
    Nil
  }
}
```

Now, the JSDom from Node.js can be used with both versions:

*build.sbt*
```scala
jsEnv := new org.scalajs.jsenv.jsdomnodejs.JSDOMNodeJSEnv
```

### Version-based dependencies

There are some libs, which are completely different between the versions. One example is `scalajs-tools` which was 
splitted into different ones (e.g. `scalajs-logging`) for `1.0.x`. The following SBT code snippet enables version-based 
dependencies:

*build.sbt*
```scala
libraryDependencies ++= Seq(
    "org.scala-js" %%% "scalajs-dom" % "1.0.0",
    "co.fs2" %%% "fs2-core" % "2.3.0"
  ) ++ {
    if(scalaJSVersion.startsWith("1.")) Seq(
      "org.scala-js" %%% "scalajs-logging" % scalaJSVersion
    ) else Seq(
      "org.scala-js" %%% "scalajs-tools" % scalaJSVersion
    )
  }
```