+++
date = "2016-05-02T23:11:01+02:00"
draft = false
title = "Play Framework Actor Pooling with Guice (Java)"
tags = ["java", "play", "guice", "akka"]
categories = [ "Programming" ]
author = "Frank Neff"
+++

Working with the Play! Framework means working with Akka, intentionally or not. But working with Akka Actors can be 
tricky, especially when it comes to dependency injection. Play! 2.4 uses Google's 
[Guice](https://www.playframework.com/documentation/2.4.x/JavaDependencyInjection) for DI and of course it has the 
ability to also bind Actors so an `ActorRef` can be injected anywhere.

<!--more-->

Single Actor DI
---------------

Biding and injecting one single Actor is simple and well 
[documented](https://www.playframework.com/documentation/2.4.x/JavaAkka#Dependency-injecting-actors) . Just bind it in 
a Module:

```java
package modules;

import actors.MyExampleActor;
import com.google.inject.AbstractModule;
import play.libs.akka.AkkaGuiceSupport;

public class ActorModule extends AbstractModule implements AkkaGuiceSupport {

    public static final String EXAMPLE_ACTOR_DI_NAME = "my-example-actor";

    @Override
    protected void configure() {
        bindActor(MyExampleActor.class, EXAMPLE_ACTOR_DI_NAME);
    }
}
```

Enable the module in the app configuration:

```scala
play.modules.enabled = ${play.modules.enabled} [
  modules.ActorModule
]
```

Now you can inject the bound actor into every class using the `@Inject` and `@Named` annotation, e.g. in a Controller:

**Keep in mind:** Actors should always be injected as `ActorRef` and not as the concrete class which has been 
implemented.

```java
package controllers;

import akka.actor.ActorRef;
import akka.util.Timeout;
import modules.ActorModule;
import play.mvc.Controller;
import play.mvc.Result;
import scala.concurrent.Await;
import scala.concurrent.duration.Duration;
import scala.concurrent.duration.FiniteDuration;

import javax.inject.Inject;
import javax.inject.Named;
import java.util.concurrent.TimeUnit;

public class AssetController extends Controller {

    @Inject
    @Named(ActorModule.EXAMPLE_ACTOR_DI_NAME)
    ActorRef exampleActor;

    public Result publish(String assetType, String assetUuid) {
        FiniteDuration duration = Duration.create(5, TimeUnit.SECONDS);
        String response = Await.result(
                ask(exampleActor, "ask something", new Timeout(duration)).mapTo(classTag(Strijg.class)),
                duration
        );
        return ok(response);
    }
}
```

*// disclaimer: Stuff like error handling and message protocols intentionally left out in favor of clarity*

Actor Pools
-----------

As you maybe know, there is the possibility to create pools of Actors instead of only one single instance. A pool
behaves like a single actor, using a [router](http://doc.akka.io/docs/akka/current/java/routing.html) to distribute 
messages to different child actors. The configuration options are huge, but most of the time, I personally only need an option
to bind an actor pool using [round robin distribution](http://doc.akka.io/docs/akka/current/java/routing.html#Pool). No
problems to achive this using the Akka configuration, but I could not find any documentation how to combine it using 
Play / Guice DI (in Java).

It turned out, that it is simple once you know it. Just pass the props of a `RoundRobinPool` to the `bindActor` method 
as a third argument:

```java
bindActor(MyExampleActor.class, EXAMPLE_ACTOR_DI_NAME, p -> new RoundRobinPool(5).props(p));
```

Now, instead of a single instance of `MyExampleActor` there is a pool of 5 of them waiting for messages.

The example works for Play! Framework 2.4 using Java 8 and will maybe break in future versions.
