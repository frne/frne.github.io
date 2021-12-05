---
title: "Why complexity in software projects is bad (and you should not advertise it)"
description: "There are (still) many people telling proudly, how complex their product is. This creates a long-tail problem, not only for engineers..."
image: "images/post/2021-08-10-why-complexity-in-software-is-bad.jpg"
date: 2021-08-10T18:30:00+01:00
author: "Frank Neff"
tags: ["complexity", "software-design", "product-management"]
categories: ["Business", "Leadership", "Engineering Teams"]
draft: false
---

There are (still) many people telling proudly, how complex their product is. There are job ads, explaining that you will 
"create highly complex software". These false signals, which I personally saw in many positions and projects, create a 
potentially huge long-tail problem, not primarily for engineers, but for the product management and engineering team 
lead. In this blogpost, I want to point out some of those impacts of advertising complexity[^1]...

<!--more-->

### Complexity of Business Process

When building digital products, it turned out as a good practice to categorize complexity in categories. The first, I 
call it "complexity of business process" describes how complex the process behind your product is. So for example, a 
simple camera app (press button, take photo, show photo) is not very complex, but a full-blown CRM probably is. If the 
requirements or even simpler, the process you want to digitalize is complex, the software you're building will always 
be[^2].

Of course it's becoming worse, if the business process is broken or not well defined. Then the software made of it will 
probably be broken or at least buggy.

***Keep in mind:** If you digitalize a not well-defined or not well working process, you just move its complexity and 
problems into code, where you can't observe it anymore.*

### Intrinsic Complexity

My second category covers "self-made" complexity during an engineering process. There are various reasons for intrinsic 
complexity, e.g.:

* Changing requirements
* Technical debt (historic technical problems, which lead to problems in current development)
* Lack of Knowledge (an engineer didn't knew a simpler solution)
* "Superheroism" (an engineer wanted to make it more complex then needed for personal/egoistic reasons)

Intrinsic complexity can be prevented to a certain extent by actively promoting knowledge and establishing a working 
development process with quality assurance mechanisms like static code analysis and peer reviews[^3].

### Impacts on the Product

When it comes to the practical impact, the kind of complexity is not relevant anymore. It doesn not make a difference, 
if the business process is badly designed, the requirements are unclear, or the quality of the product was not assured 
from the beginning. All those different kinds of complexity (or "problems" they become), have the same impacts on the 
product:

##### Bad User Experience (UX)

To the user, the application feels unintuitive, difficult to handle, or even not usable. These problems often arise due 
to poorly designed processes in the application or insufficient UX design[^4]. Of course, there is no standard solution 
for these problems. However, it often helps enormously to staff the roles in the project (especially business analyst, 
UX engineer and UI / frontend designer) strongly. Especially in projects with smaller budgets these roles often overlap, 
but it is crucial that they are defined and staffed.

{{< notice "tip" >}}
In most of the projects I've seen fail so far, these roles were not or insufficiently staffed. Don't skip proper 
business analysis or UX engineering because of low budget.
{{< /notice >}}

##### Bad Performance

Since performance is a separate discipline in software development, it cannot be generalized in any case. Also, the 
reasons for poor performance can be almost anything. However, one reason for this, which I myself have observed many 
times, is the complexity of the application itself.

In theoretical computer science, complexity always means time. Simple things are fast, complicated things take a long 
time. Since process complexity cannot always be prevented, performance becomes a constant companion. 

{{< notice "tip" >}}
You can significantly reduce the impact of this problem category by introducing and further developing quality assurance 
measures (static code analysis, automated tests, load tests, etc.). If performance is a critical factor, and it often 
is, use technical tools to continuously measure and analyze performance.
{{< /notice >}}

##### High Ongoing Costs

This is a purely logical conclusion, because complexity always causes additional costs[^5], but also simply an effect 
of the previous problems. Poor performance causes higher hosting expenses, confused users more support cases.

{{< notice "tip" >}}
Even in the rare case that you don't care if your software is slow and barely unusable, it will cause high
costs in hosting and support, which will increase with time and further development.
{{< /notice >}}

### Conclusion

Complexity, also from a business perspective, is always bad. Try to avoid or minimize it, not only because it fosters a 
poor development culture and morale, but also because it can result in very high, ongoing costs. Last but not least, 
complexity always means effort and frustration, which you can lower for everyone involved, by taking business analysis, 
development processes, and quality assurance seriously.

[^1]: [Picture from Christina Morillo on Pexels](https://www.pexels.com/de-de/@divinetechygirl?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)
[^2]: [N. Fenton, P. Krause and M. Neil, "Software measurement: uncertainty and causal modeling," in IEEE Software, vol. 19, no. 4, pp. 116-122, July-Aug. 2002, doi: 10.1109/MS.2002.1020298.](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=1020298&isnumber=21951)
[^3]: [S. F. Ochoa and F. J. Gutierrez, "Architecting E-Coaching Systems: A First Step for Dealing with Their Intrinsic Design Complexity," in Computer, vol. 51, no. 3, pp. 16-23, March 2018, doi: 10.1109/MC.2018.1731079.](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8320209&isnumber=8320205)
[^4]: [Francisco Inchauste, "The Dirtiest Word in UX: Complexity" on uxmag.com, 10.08.2021](https://uxmag.com/articles/the-dirtiest-word-in-ux-complexity)
[^5]: ["Complexity Costs" on wilsonperumal.com, 10.08.2021](https://www.wilsonperumal.com/blog/blog/complexity-costs)