---
layout: post
title: "Your Code as a Crime Scene and Software Design X-Rays by Adam Tornhill"
date: 2020-1-22
category: books
tags: [watercooler, books, cleancode, refactoring]
excerpt_separator: <!--more-->
---
Usually, I write about one single book in a given article, but this is a peculiar occasion. Last months I read both books written by [Adam Tornhill](https://twitter.com/adamtornhill?lang=en): [Your Code as a Crime Scene (_YCCS_)](https://amzn.to/2QKIlhD) and [Software Design X-Rays (_SDXR_)](https://amzn.to/30e2S18).
<!--more-->
Despite that three years passed between publishing the two books (_YCCS_ in 2015 and _SDXR_ in 2018), I read them one after the other, which was probably not the wisest decision, even though I enjoyed both.

I have heard a lot about _Your Code as a Crime Scene_ in the craftsmanship community, but mostly from people who also heard about it. You know it's like _I have a friend whose friend heard about someone having read it._ Finally, it ended up on the top of my reading backlog and I really enjoyed studying the book and experimenting along the way.

_YCCS_ introduces the concepts from forensic psychology to analyzing legacy code. It uses solely version control data (for most us it's git) that normally should be available to you for any project nowadays. Clearly, if it's not the case, you have bigger problems than this book can address. Anyway, extracting data from git and analyzing it with the help of various scripts you can help you for example
- find the hotspots of your code
- analyze the coupling between different parts of the code
- build a knowledge map

While I found _Your Code as a Crime Scene_ a totally innovative book, _Sofware Design X-Rays builds_ on the already introduced ideas of hotspots and change coupling metrics and goes much deeper. 

It's not a repetition of the previous book and it does introduce new directions that were not present in _YCCS_. Still, obviously it's not so much a novelty that I would advise you to read it right after the first. Experiment with the ideas introduced in Adam's first book, let it sink, ask yourself questions and then in a couple of months take the second book. You'll value it more.

Before writing the next paragraph, I must tell that I have no financial incentives. In the books, Tornhill uses open source scripts available on [his github](https://github.com/adamtornhill/code-maat) and you're free to reuse them. Besides, he also built a more elaborate version with more metrics, more configurability and a nice UI called [Code Scene](https://codescene.io/). Of course, it's not for free. Soon we are going to experiment with it on the application I'm working on.

Once we have some measurable results, I'll write some follow-up posts.

Until then, I recommend you to read first [Your Code as a Crime Scene](https://amzn.to/2QKIlhD) that is interesting even for its references between software development and psychology and a few months later [Software Design X-Rays](https://amzn.to/30e2S18), they are really worth it.

Happy reading!