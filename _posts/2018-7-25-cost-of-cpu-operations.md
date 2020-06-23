---
layout: post
title: "The cost of CPU operations"
date: 2018-7-25
category: dev
tags: [cpp, performance]
excerpt_separator: <!--more-->
---
What are the most important things to understand before performing source code optimization? Or even better, what factors to consider when you intend to write performant code?

<!--more-->

I think the most important thing is to understand that the optimization you have in mind might be completely irrelevant. Don't optimize the performance of a piece of code that is barely used. Make it readable - that's much more important. And hum the words of Knuth: ["premature optimization is the root of all evil"](/blog/2018/03/09/coders_at_work_donald_knuth).

First, you have to understand what are the bottlenecks of your application, which functions are worth to optimize to get the biggest bang for your buck.

Then - and that's what I focus on here - you have to understand the cost of different CPU operations so that you know what kind of operations you shall get rid off.

Recently I've been to a C++ optimization training and I think the table of Ignatchenko was maybe the most important takeaway. Even if the techniques we learnt were really interesting. Even if this is part of 2nd-year university curriculum.

So let's all have a look at this slide of the cost of CPU operations:

![Operation costs in CPU cycles]({{ site.baseurl }}/assets/img/cpu-costs-ignatchenko.png "Operation costs in CPU cycles")

We must see that the scale is logarithmic, in other terms, growth is exponential and that word is something really dangerous in computer science.

Do you know the story of the wise man who asked for rice from the Indian King Sharim? The man gave the king a chessboard as a gift. In return, he asked for one grain of rice on the first square, two on the second, four on the third, eight on the forth and so on. Soon the king realized that there is not so many rice in the whole world. That's what exponential growth is. Something soon out of control, hence we try to avoid it in algorithms.

So when we think about performant code, we should avoid operations that appear in the bottom part of the previous chart. Given our performance goals, we can go higher and higher, but to be honest I don't think that your problems will lay above C++ virtual function calls.

In fact, to me this chart also shows that having a lot of small - well named - functions is not a performance issue - as some people still advocate for long, unreadable and unmaintainable monstres. Even though in the most inner loops of a performance critical embedded system it can be worth to avoid as many function calls as possible. In other cases, avoid function calls won't help you.

On the other hand, there are some important things to notice:
* While we say that let's avoid disk I/O and keep things in memory, we also have to see that depending on your goals, RAM can be a slow beast. Keep your hottest data the closest you can to your CPU. If an operation is outside the socket or the CPU and it has to reach for the RAM, it's orders of magnitudes slower than reading from the L1 cache. It also means it does matter how your data is organized because with each memory read the costs are accumulating.
* Multi-threading is costly. It is extremely error-prone and context switching is expensive - not just for humans, but for computers too. As we saw during the training, in many conditions, multi-threading might end up slower than a simpler single-thread solution. Avoid multi-threading, if you can. If you must use it, take extreme care.
* Throwing and catching exceptions is still expensive. I won't tell you that you should avoid using them, but keep in mind that in C++ it's something really heavy. While in other languages using them as control structures are okay, in C++ it's not the way to go. If you use them, use them in situations they were designed for exceptional ones.
* Avoid system/kernel calls whenever possible. They are as expensive as context switches.

## Conclusion

The key takeaways from this short post are:
1. Identify the bottlenecks of your software before you'd start heavy optimization works.
2. The most costly operations are related to interacting with the RAM or the disk, multi-threading and exception handling, not forgetting about system calls. You should first try to eliminate such operations from your code.ter