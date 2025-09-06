---
title: "The opportunity costs of unnecessary scalability and how to avoid them"
description: "TODO"
image: "images/post/2023-10-14-how-to-effectively-visualize-application-landscape-enterprise-architecture/header.png"
imgcaption: "Example Application Landscape, created with Google Sheets"
date: 2023-11-18T12:25:00+02:00
author: "Frank Neff"
tags: ["performance", "scaling", "architecture", "cloud"]
categories: ["Architecture"]
draft: true
---

I bet almost everyone in IT has heard the sentence: "It must be scalable!" Of course, scalability is something every 
modern application should be capable of, and we should think about being able to scale the latest during solution 
architecture. However, most of the time I heard that sentence, people thought scalability came for free and was just 
something to keep in mind while building the solution. In this post, I'll give some insights about the actual cost of 
scalability, why it's often done to a useless extent, and how to avoid it in the next project.

<!--more-->

## Definition & Dimensions

{{< image
  src="/images/post/2023-10-14-how-to-effectively-visualize-application-landscape-enterprise-architecture/template.png"
  alt="Screenshot of GitHubs device code auth screen"
  title="Screenshot of GitHubs device code auth screen"
  caption="Template file with highlighted dimensions"
  command="fill" class="img-fluid" >}}

