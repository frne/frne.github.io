---
title: "Working with Remote Teams - General Challenges"
description: "..."
image: "images/post/2021-12-07-working-with-remote-teams-general-challenges.png"
c: Drawn myself using [draw.io](https://drawio-app.com/)
date: 2021-12-07T07:10:00+01:00
author: "Frank Neff"
tags: ["leadership", "processes", "staffing", "remote", "series"]
categories: ["Leadership", "Engineering Teams", "Working with Remote Teams"]
draft: true
---

I have managed and worked within multiple software engineering teams so far. Many of them had remote resources. Be it 
internal, external, near- or offshore people, agencies or contractors, there are numerous ways of working with remote 
teams. In this blog post, I want to share some of my experiences, best practices and examples of optimizable and also 
well-performing team setups...

<!--more-->

### Why Remote?

Remote work in development teams is part of everyday life in many companies today. Basically, software development is 
not bound to a specific location and is predestined for specialists abroad, not least because of its universal language 
(English). The demand for ICT specialists in Switzerland is extremely high[^1] and the available specialists in Switzerland 
are scarce.[^2] Some have already opted for remote developers in the course of unsuccessful recruiting. We also live in 
a country with a very high income, which is driven up further by the shortage of skilled workers. It also sounds 
plausible to look for employees abroad because of the hourly rates, but this is not entirely consistent with my 
experience.

To sum it up, there are reasons to staff teams partially or completely with remote positions:

* Shortage of skilled workers
* Short-term availability / risk minimization
* Hourly rates
* Know-how acquisition

{{< notice "note" >}}
Also, these days, employee health and related government regulations due to Covid-19 is a topic, but I won't go into
that in this article.
{{< /notice >}}

### Preconditions for Remote Work

There are probably almost endless possibilities how to build remote resources or remote teams. Most requirements for 
remote work setups are very specific and are defined by the nature of the collaboration. I will refer to some of them 
later. However, I believe there are some preconditions which are beneficial in any case when working with remote 
resources:

##### Efficient Communication

The more distributed the team is, the more communication overhead arises. Therefore, make sure you have an efficient and 
transparent communication setup and working tools. For the latter, I've had very good experiences with 
[Slack](https://slack.com/) and [Google Meet](https://meet.google.com/). Both tools have a relatively extensive free 
tier, which works well for small teams. Also, use the communication tools which integrates best with your existing 
ecosystem. If you're on Microsoft, use [Teams](https://www.microsoft.com/de-ch/microsoft-teams).

##### Easily Accessible Information

I like to use two systems as information carriers for my teams. One for general information, a "wiki" and a second one for 
specific information per task, story, bug, etc., a "ticket system" or "issue tracker". For me, this distinction helps to 
address information better, as they follow different workflows and processes that need to be mapped. However, it helps 
tremendously to have tight integration between these systems.

The top dog here is certainly Atlassian. [Jira](https://www.atlassian.com/de/software/jira) and 
[Confluence](https://www.atlassian.com/de/software/confluence) in combination is extremely strong, but relatively 
expensive in comparison. With a smaller budget and somewhat lower requirements, I can recommend 
[YouTrack](https://www.jetbrains.com/youtrack) or [GitLab](https://about.gitlab.com/). GitLab is 
particularly interesting because it offers a ticket system and a wiki and this is also linked to the code version 
control.

##### Well-defined Processes, Roles and Boundaries

Especially when you're scaling, you need to define boundaries for different teams very well. This includes, in
particular, which team works on which product, module or feature, how roles are defined in the teams and by whom they
are staffed, as well as the internal and cross-team processes, contact- and escalation points.

##### Agile Methodology

Working in an agile setup like Scrum, Kanban, etc. has many advantages. For me, some of the most relevant are:

* Short feedback cycles
* Regular adaption to changing circumstances
* Iterative testing and delivery of the product
* "Inspect and adapt" on a regular basis

{{< notice "note" >}}
Some companies still decide to work in waterfall-oriented traditional project setups for products which are developed 
externally. Even in a complete offshore project, I would always opt for an agile method, because traditional 
specifications are simply not designed for changing requirements and I like short, modular feedback cycles much more 
than a large final sign-off.
{{< /notice >}}

Since this is not an article about agile methods, I will keep it short at this point.

##### Team Building

For the majority of people I have worked with, remote communication and collaboration is more challenging and exhausting 
than working together on-site. A lot of non-verbal input such as facial expressions and gestures is lost, "coffee breaks" 
are eliminated, you don't accidentally hear something inspiring from the neighboring table, etc. For this reason, it is 
even more important that the team knows each other well, understands each other and is attuned to each other.

Regular retrospectives and social events help, even if they are remote. Your first after-work beer in front of a webcam 
may feel very strange, but it promotes team cohesion and improves unofficial communication within the team and beyond.

##### Positive Error Culture

People make mistakes, whether they're sitting in the office next to you or on the other side of the world. They should 
be allowed to make mistakes and the company culture should support them in making mistakes often and early to learn from 
them. Even in a team where mistakes are not allowed to be made, they happen. They are only concealed and thus only discovered 
when the damage is at its maximum and the learning effect is gone.

Of course, this applies to all teams. Unfortunately, it makes gossiping easier and thus to foster a dysfunctional blame 
culture, when someone is working remotely.

### Two Cents about "They don't deliver!"

An exclamation I heard many times in different companies, mostly when working with fully remote teams (offshore / 
nearshore). Of course there are team constellations and/or individuals which don't perform well. In the last instance, 
sometimes only an HR decision helps. However, in my experience, these are the rarest cases.

Much more often, preconditions are missing, processes don't work, or people simply blame the remote teams because it's 
so easy. Unfortunately, I have already experienced this as well.

Especially remote or offshore teams need planning, requirements engineering, software architecture and last but not 
least leadership to function. If a team, no matter if internal or external, does not perform over a longer period of 
time, you should look beyond your own nose and question the processes instead of looking for the fault with the others 
first.

{{< notice "note" >}}
Another reason for bad performance can be **complexity**. There is another blogpost about it:<br>[Why complexity in software projects is bad (and you should not advertise it)](/blog/2021-08-10-why-complexity-in-software-is-bad/)
{{< /notice >}}

### Closing Words

As I was writing, I realized that this topic is much bigger than a single post. That's why I decided to keep this one 
pretty general and make it a series. In the next post I will write about specific challenges and solutions with fully 
remote teams (nearshore / offshore).

[^1]: ["Wie gross ist der Fachkräftemangel in der Informatik tatsächlich?" on do-it.swiss](https://www.do-it.swiss/fachkraeftemangel_in_der_informatik/)
[^2]: ["Fachkräftemangel in der Schweiz - Ein Indikatorensystem zur Beurteilung der Fachkräftenachfrage in verschiedenen Berufsfeldern" on seco.admin.ch](https://www.seco.admin.ch/seco/de/home/Publikationen_Dienstleistungen/Publikationen_und_Formulare/Arbeit/Arbeitsmarkt/Informationen_Arbeitsmarktforschung/fachkraeftemangel-in-der-schweiz---ein-indikatorensystem-zur-beu.html)
