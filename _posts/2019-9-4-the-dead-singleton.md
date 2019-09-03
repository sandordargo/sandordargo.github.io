---
layout: post
title: "The dead singleton and three ways to overcome it"
date: 2019-9-4
category: dev
tags: [clean code, cpp, craftsmanship, technical debt]
excerpt_separator: <!--more-->
---
Oh, singletons... We can't live with them, we can't live without them.

I remember that in my first team there was a guy with a very bright mind, but he was not yet mature enough just after the university and in all the questions he was way too much opinionated and a bit too smug. Typical to most of us at the beginning of our career, isn't it?
<!--more-->

He was always there to express how he hates singletons and how the worst thing they are in the realm of software development.

No, I'm not going to tell you that they are the best thing on Earth since sliced bread. Not at all. I was not so good in programming as he was - probably I am still not. But I had more life experience and I already understood that most of the things are not black or white.

I didn't like this very extreme view, so I read a couple of articles and watched a long conference talk and I came up with a different opinion than his.

Why do I tell this? I'd like you to understand that even though I try to avoid using singletons, sometimes they are a valid alternative and we have to know how to use them.

The easiest thing is to open up [the book from the Gang of Four](https://amzn.to/2KWCkLN) at the singleton chapter, read it and implement it. Easy-peasy.

## The problem of the dead reference

Recently I suffered through [Modern C++ Design: Generic Programming and Design Patterns Applied](https://amzn.to/33W81fy) by [Andrei Alexandrescu](https://twitter.com/incomputable). It's not a bad book, not at all. It is me, the problem. I'm not so great at templates, to say the least. This book has a chapter on singletons. I furrowed my eyebrows. Templates? Singletons? On the same page? What? Why?

You might be able to pimp up singletons with templates and you can address problems that are already there but you maybe never thought of.

I don't want to walk you through the whole chapter and how [Alexandrescu](https://twitter.com/incomputable) implements singletons using templates, but I want to highlight one problem that I had not thought of before and that is probably specific to C++. I'm a bit worried that I don't think that any of my colleagues thought about this. Or at least, they haven't shared their concerns with the rest of the tea.

I'll use Alexandrescu's example here. Let's assume that in our system we have the concepts of `Keyboard`, `Display` and `Log`. As in that system, you can have only one of each, they are implemented as singletons. How do they work? Each singleton object has only one instance and it's usually initialized when it's first called.

How are they destroyed? Randomly? It would be bad, and luckily it's not the case. They are destroyed in reverse order of their creation.

If we suppose that we log only in case of errors and we envision the next scenario, we can encounter a big problem:

* `Keyboard` is created successfully
* `Display` has a problem while it's created
* But it managed to create `Log`
* Some code is getting executed, probably error handling
* Log is destroyed
* Keyboard destruction has a problem, it wants to log... oh oh..

This issue can arise in any application which uses multiple interacting singletons. There is no automated way to control their lifetime.

One solution, of course, is to eliminate interacting singletons from the code. If this is not a viable possibility, the function returning the singleton instance has to check if it has been already destroyed. This is something that you can track with a boolean. Now we at least can know if our singleton had been already destroyed, but our issue has not been solved. Yet.

Alexandrescu proposes three ways to solve the issue.

### The Phoneix singleton

The concept of it is rather simple. If you try to get the reference of an already destroyed singleton, it just gets recreated.

Its downsides are that it might be confusing as it breaks the normal lifecycle of a singleton, plus if it had a state, it would lose it in the destruction-resurrection cycle.

### Singletons with longevity

The idea of this solution works well when you know in what order you want to destroy your singletons. You assign longevity, in other words, an order, a priority for them and you register the singletons with their longevity to a dependency manager that will take care of their destruction in the good order at the end of the lifecycle.

This is a fine solution, but it introduces extra complexity.

### Infinite singleton

In this case, we have no destruction at all. The [GoF book](https://amzn.to/2KWCkLN) implicitly uses this solution. This is very simple, but inelegant. Only the operating system will take care of the cleanup. Depending on your use-case, this can be acceptable, but you also have to consider that it might introduce some important memory leaks. There is no magic bullet, just best practices and case-by-case analysis.

## Conclusion

In this article, we learnt about the problem of the dead reference/ dead singleton which might occur when you have multiple singleton objects interacting with each other. 

We saw three main ideas to tackle it shared by [Alexandrescu](https://twitter.com/incomputable) in his book. If you don't want to rely on the simple C++ rules (last created, first destroyed), you have the options to create a resurrecting "Phoneix" singleton, to set the longevity for each of them or to simply never destroy them and leave it to the operating system.

If you want to learn more about this problem, including the implementation details of the listed options, I encourage you to read the corresponding chapter of [Modern C++ Design: Generic Programming and Design Patterns Applied](https://amzn.to/33W81fy).