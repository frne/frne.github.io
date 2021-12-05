+++
date = "2015-04-21T13:09:32+02:00"
draft = false
title = "If pragmatism raises technical debt, call it oversimplification (rant)"
tags = [ "agile", "pragmatism" ]
categories = [ "Business", "Engineering Teams" ]
author = "Frank Neff"
+++

The word "pragmatism" or "pragmatic" is, in my personal opinion, the most 
overrated word in agile development. Many people use this as a buzzword without
knowing what it means. I hear people saying "He solved that complex problem in
half an hour, he's so pragmatic!" and think for myself "Yeah, but that 'solution' 
probably causes other devs three times more effort than a sustainable solution
would take."

<!--more-->

Okay, but that's only my anger speaking. Let's read a definition of pragmatism:

> a reasonable and logical way of doing things or of thinking about problems 
> that is based on dealing with specific situations instead of on ideas and 
> theories &ndash; [merriam-webster.com](http://www.merriam-webster.com/dictionary/pragmatism)

## A Reasonable and Logical Way

So even pragmatic solutions have to be reasonable and logical. I would ask so 
many people out there: "Is missing out documentation, tests and software design 
principles really a logical and resonable way of solving problems?" At least I
don't think so. Maybe if it's only for a Executive Board presentation, you can 
rapidly build a prototype without having documentation or tests. But doing that,
one has to take care that this prototype is not going life two weeks later.

## Based on Dealing with Specific Situations

Of course, if there is a disaster case, you cannot think about 
the best way to build a solution for 2 days. What I am talking about are 
estimated, scheduled projects which are barely "fucked up" technically, because
one wants to finish them fast or without thinking about it. In a second, or 
maybe third release, technical dept is appearing but because you have not 
documented nor tested, you cannot refactor. This is not pragmatism!

For specific situations or problems in development, there are often provided
patterns or parts of solutions. Using those "best practices" is probably the 
quintessential of pragmatism. Building an over-simplified custom solution,
because one doesn't understand the best practice, is not.

## Instead of on Ideas and Theories
There are many theoretical approaches in development. Let's take the [Enterprise 
Design/Architecture Patterns](http://martinfowler.com/eaaCatalog/) 
([Gang of Four](http://www.amazon.de/Patterns-Elements-Reusable-Object-Oriented-Software/dp/0201633612)) 
as an example. There were some people telling me, that they don't use them,
because this is theoretic and has nothing to do with pragmatic solutions. But if
a "pragmatic solution" is the best way to deal with a specific situation, then 
I would highly recommend those patterns because the are very pragmatic. 

## Finishing...

Many people use words like "pragmatism" or "best practice" as buzzwords but 
don't understand what they mean. Therefore, please be aware that pragmatism does 
not mean laziness or oversimplifying a complex problem. It means **thinking 
practical, but also logical and resonable**, so don't misuse them.
