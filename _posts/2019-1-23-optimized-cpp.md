---
layout: post
title: "Optimized C++ by Kurt Gunteroth"
date: 2019-1-23
category: books
tags: [books, cpp, optimization]
excerpt_separator: <!--more-->
---
After I attended a training on the subject of optimizing C++, I felt I'd be interested in going a bit deeper. At least to read a bit more about this topic. So I asked the trainer for some books he'd recommend about optimization. One was [Kurt Guntheroth's Optimized C++](https://amzn.to/2vGyHRT). As soon as I finished reading [Essential Skills for the Agile Developer](/blog/2018/06/27/essential-skills-for-the-agile-developer), I started to read this one.
<!--more-->

I liked the book, but to be completely honest, by the end I felt a bit lost. This just means that the book starts with simple ideas and heads toward the complex ones. Apparently, I didn't dedicate enough time to understand well the last two chapters which are about concurrency and memory management. I'm not working in an environment where I would need the benefits offered by the techniques described there, that's my excuse.

On the other hand, in the rest of the book, I found many pieces of advice that can be useful for me right now, or in the near future. Guntheroth explains why optimization matters, when you should start optimizing and how you should do it. He goes into details about the costs of different sorting and search algorithms, dynamic variable allocation, data structures to name a few. He goes from the most common towards the rarer solutions.

It makes complete sense. Most of the time you don't need anything fancy, just to review your algorithm. I remember at the very beginning of my programming career when I did something in O(n**4) instead of O(logn*n). I was called out for it when the app turned out to be extremely slow. I did some measurements and turned out that we spent less than 1% of the time there - just as I expected - and more than 95% in a third party library which was not so well documented and we realized after weeks that we didn't clean up properly after it - in fact, we cleaned up too frequently.

And here is a very important point. Don't optimize in vain and when you do optimize, measure the effects. One thing I liked a lot about [the book](https://amzn.to/2vGyHRT) is that the author tells us about his assumptions and failures. He explains that he expected one data structure to be better than the other by orders of magnitudes but it ended up being just a bit faster. Or in other cases, even slower. This gives him credibility and emphasizes the importance of experiments.

I think I'll keep [Optimized C++](https://amzn.to/2vGyHRT) on my (virtual) bookshelf and whenever I'll encounter hot code parts that need to be more performant, given the clear structure of the book, I'll know where to open it for some good pieces of advice.