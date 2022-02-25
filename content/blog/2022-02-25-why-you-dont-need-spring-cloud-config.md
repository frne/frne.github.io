---
title: "Why you probably don't need Spring Cloud Config"
description: "..."
image: "images/post/2022-02-25-why-you-dont-need-spring-cloud-config.png"
imgcaption: "Photo by [Code by B](https://www.youtube.com/watch?v=yhAsgwhSGXs) from [YouTube](https://www.youtube.com/)"
date: 2022-02-25T07:10:00+01:00
author: "Frank Neff"
tags: ["spring", "config", "complexity"]
categories: ["Software Architecture", "Programming"]
draft: false
---

Spring cloud config is a configuration management solution and part of the Spring ecosystem. It's easy to integrate and
use, but it has severe downsides from an architectural perspective. Also, I have seen many projects where
it was added just because "it is a thing" and even more often, it was used to solve or work around problems in the
software architecture or deployment for which it doesn't provide a real solution. In this blog post, I'll point out some
conceptional issues and downsides of that project I have seen so far.

<!--more-->

{{< notice "note" >}}
Of course, there are circumstances where it makes sense to use Spring Cloud Config, or more generalized, a configuration
management and distribution solution. Anyway, from my personal experience, it is more often misused or not needed.
{{< /notice >}}

### Self-Contained Applications

I would, in general, not consider an application that needs to fetch its basic configuration over an HTTP connection
as "self-contained". This indirection often adds complexity and virtual hurdles to an application bootstrap. The
config server must be reachable and working. The HTTP connection between all applications and the config server must be
stable and protected (see [Security](#Security)). Also, this adds a dependency for local development, where you
potentially need to start a config server on your local machine just to be able to work on another service. In general,
there is much complexity added by a config management server.

### Security

At least on production, security concerns should be taken into account. An application's configuration contains  
sensitive information like database access credentials, secret keys, certificates, etc. Exchanging this information via
HTTP, even if you're inside a protected perimeter, needs really strong security mechanisms, or should probably just be
avoided in favor of that. Those strong security mechanisms, for example, to provide keys to an application's 
configuration, exist, but they are nowadays mostly baked into your cloud infrastructure. See secrets management of
[Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/) or
[Docker Swarm](https://docs.docker.com/engine/swarm/secrets/) for examples.

### Often-Changing Configurations

A famous pro-argument of some users is that you can reload your Spring configuration without restarting or
redeploying. For me, there are two things that should be thought about when it comes to this significant use-case:

#### Improve your Deployment

One thing that potentially causes you to use this "hot config change" mechanism is that your deployment is not stable
or fast enough. In a modern cloud environment, deploying an application is not only very simple but should also be quick
and "zero-downtime". So, if you need this feature, just because you don't want to deploy your application for a config
change, you're maybe just working around existing problems with your build and deployment pipelines.

#### Don't keep everything in Application Configuration

If you still need fast and often changing configs, also ask yourself which kinds of configs... An application
configuration should be one very stable and semi-static container of information required by the application itself to
connect other services, databases, etc., and configure itself on startup. If you have often changing endpoints or IPs of
services/databases, which is a very usual case in a cloud-based microservice architecture, use a Service-Discovery
mechanism or load balancer instead of changing the configuration all the time. Also, some configuration values which are
often changing by nature should be stored in a database and made accessible/changeable by a management GUI instead.

### Configs for different Profiles

Another argument for a cloud config server is having different profiles, e.g. for different environments or multiple 
tenants. Spring is already capable of managing configuration for different profiles very well, and it has excellent 
documentation...

### Better Alternatives

Management of your application configuration is a concern that should be handled by or near your infrastructure and
deployment. Most cloud and hosting infrastructures provide excellent configuration management by using environment
variables or injecting configuration files. This not only keeps your credentials secret but also allows you to provide
a working local configuration for developers, which is then overridden by your hosting infrastructure in production or test
systems.

### Final Words

I personally only see very rare use-cases where Spring Cloud Config adds real value today. Especially when you're
hosting in a private or public cloud, you should consider using its onboard configuration management mechanisms.
They are often way more powerful than a config-server, without adding dependencies.

> Don't use technology just because it's a thing. Always ask yourself if you really need it.

I recently have removed Spring Cloud Config from a medium-sized project (12 Microservices, 3 SPA) and in-sourced 
configuration using YAML files, which improved its availability, lowered the hosting costs and removed complexity for 
developers. Still, I see many devs out there using and advertising it for use-cases it is not built for.

