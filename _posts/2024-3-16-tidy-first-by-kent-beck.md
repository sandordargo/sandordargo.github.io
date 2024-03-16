---
layout: post
title: "Tidy First? by Kent Beck"
date: 2024-3-16
category: books
tags: [watercooler, refactoring, softwaredesign, architecture]
excerpt_separator: <!--more-->
---
Let's start by explaining what tidying means when it comes to software development. Maybe a decade ago, this book would have been called "Refactoring first?", but the term "refactoring" got inflated when people started to refer to long pauses in feature development as such. Even worse, the most essential part of refactoring - it shouldn't change the system's behaviour - is not always respected. No new feautres, possible damage. Many stakeholders dislike refactoring.

*"Tidyings are the cute, fuzzy little refactorings that nobody could possibly hate on."* They usually don't go beyond a few hours of work.

When you first hold the book in your hand, you might be surprised as it's barely a hundred pages long. It's very dense, without any clutter, yet it's extremely clear. It's the first part of a series on software design. This first book covers small tidyings scaling from a few minutes to hours and focuses on the individual.

The second book will target not only individual contributors but also a team of programmers and will cover refactorings that can go from a few days to a couple of weeks. Finally, the third book will be addressed to all stakeholders and will discuss architectural evolutions which often span over several years.

## How to tidy?

[Tidy first?](https://www.amazon.com/Tidy-First-Personal-Exercise-Empirical/dp/1098151240?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=a8e969e45c753f932fbb0e7a09e53832&camp=1789&creative=9325) is partitioned into three parts. The first part covers tidying techniques. If you are familiar with refactoring techniques and "clean code", it won't give you many surprises, but it's a nice summary of what tools you have when you want to perform small-scale structural changes. Let me just name a few of them: use guard clauses, use helper variables and functions, and reorder code so that relevant parts are close to each other. 

The first part also has some valuable thoughts on comments. The author both advocates for and against comments, and this is perfectly fine! First of all, you shouldn't refrain from writing explaining comments if you run into some complicated piece of code that you just managed to understand. Maybe you don't have the time or the confidence to rewrite it. Yet, it would be good to avoid the next person (maybe you in a month) spending so much time once again understanding the code. Just write a comment that explains it. 

At the same time, *"when you see a comment that says exactly what the code says, remove it. The purpose of code is to explain to other programmers what you want the computer to do."*. Comments accuracy is somewhat questionable even when they are written and later they will rot. If a piece of comment doesn't bring value, remove it.

## How to present tidyings?

While the first part is about how to tidy code, the second part is about its presentation. Should you post it along with behaviour changes or separately? Should you perform tidyings before or after the behaviour changes? Or maybe just later or even never? For the latter questions, the answer is that it depends on different factors. Cost is only one factor and it's great if it can drive the decision, but you cannot always quantify how much you will gain by performing a tidying and how much it would cost if you did it later or never.

Regardless of the cost, let's not forget that programmers are human beings and we defined tidyings as short activities. Sometimes you'll just "not have the energy to tackle a new feature, but [you] want to work". Tidying might keep you happy and being happy will make you a better programmer.

On the other hand, it's easier to answer the first question. It's better to present a tidying separately from behavioural changes. Code will be easier to reason about, to review and eventually to revert if you overlooked something.

## Economics and software design

We, programmers, are human creatures who put a big emphasis on logic. It's important to understand the theory behind certain - best - practices so that we accept it and can reason about it. The third and final part of [Tidy first?](https://www.amazon.com/Tidy-First-Personal-Exercise-Empirical/dp/1098151240?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=a8e969e45c753f932fbb0e7a09e53832&camp=1789&creative=9325) attempts to uncover the theory behind software design and more specifically behind tidying.

The author shares that until his mid-30s he didn't understand the physics of moving money at all. Later, thanks to a "series of finance-related projects", he slowly started to get it. The lessons soaked in so much that even the way he sees development changed.

Two surprising properties defined his view:

> - A dollar today is worth more than a dollar tomorrow, so earn sooner and spend later.
> - In a chaotic situation, options are better than things, so create options in the face of uncertainty.

In real life, that's not so easy. To make future options, you have to spend money now and the value of these future options is not always possible to calculate. Still, the less a behaviour change will cost, the more options you have. The more tidying you perform, the less behaviour changes will cost. But a dollar today is still worth more than a dollar tomorrow. It's not easy to find the right balance.

One kind of change that is definitely not narrowing your list of options is reversible structural changes. Just as a bad haircut is more reversible than a bad tattoo, a structural change is more reversible than a behavioural change that might damage your reputation if it goes bad. As we know reputational damage can be permanent. In general, software design changes are reversible but behavioural changes are less so.

Let's get back to the cost of software. It's not about the money you spend on building it. It's more about how much it costs to change your software. The Pareto distribution applies here as well, the overall cost will be dominated by a few big changes. Those few outliers are the results of coupling. In other words, according to *Constantine's Equivalence*, the cost of software can be derived from the level of coupling. Therefore, if you want to reduce how much a piece of software costs, you must reduce coupling.

At the same time, *"decoupling isn't free and it is subject to trade-offs"*. The more you reduce coupling in one place, the more it appears somewhere else. You cannot completely eliminate coupling. Decoupling does create options, but it also has diminishing returns, it's not worth pushing it too far.

## Conclusion

[Tidy first?](https://www.amazon.com/Tidy-First-Personal-Exercise-Empirical/dp/1098151240?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=a8e969e45c753f932fbb0e7a09e53832&camp=1789&creative=9325) is a short, but impactful group about small scale software design changes. It explains what steps to take to clean up your code. Steps that don't take more than a few hours at most. It also advocates for presenting these changes separately from behavioural changes so that it's easier to reason about them and it's also easier to back out just in case.

In the last part of the book, the author explains the theory behind tidying and how tidying should decrease the cost of software while widening the list of available options.

But in the end, we shouldn't forget that programming is a human activity. Besides the financial aspects, we must consider that tidying brings peace, satisfaction and your to our programming. At least some. That's important because if we are our best selves, we are better programmers. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
