---
layout: post
title: "Coders at work: Bernie Cosell"
date: 2018-1-31
category: books
tags: [books, coders at work]
header: "The 14th and last but one person interviewed in <a href=\"http://amzn.to/2wKEeVt\">Coders at Work: Reflections on the Craft of Programming</a> is <a href=\"http://www.codersatwork.com/bernie-cosell.html\">Bernie Cosell</a>, one the trio who wrote the software the first Interface Message Processors (IMPs) of ARPANET"
---
It was inspiring to read about some his thoughts which appear in many modern programming books.

* He says that you have to think about testing before you actually start writing the first lines of your program. Hello, Test Driven Development.

* A few years after he had started his career, he realized that code is for humans, not for machines. The most important thing is to write readable code. Hence he doesn't put a lot of comments in his code. The code should be already readable, it should express what it is supposed to do.

It is partly related to this, how he was considered a great bugfixer. He tried to understand where the bug was, what that component was doing, but if it was not evident he didn't spend too much time on it. After identifying the problematic part, he rewrote it.

He speaks a lot about cleaning up the code. When you fix a bug, you shouldn't just fix the bug, but you should reorganize the component, you should make things the right way. He thinks that you should never ask approval for a new design in your software. Just sneak it in little by little. In fact he speaks about refactoring, a term he didn't know.

He always overestimates compared to the easiest solution, so in the end he always has some time to improve the code.

He also speaks about premature optiomization in two contexts. He thinks that programmers are aweful in optimization. In fact they are the worst optimizers in the world. We always optimize the part of oue code that's the most interesting to optimize, but we almost never optimize that part that actually needs optimization. We end up having unreadable, unmaintainable code without any reason.

He returns to similar thoughts when speaks about students. We already did the same as students. We see opportunities to implement "an AB unbalanced 2-3 double reverse backward pointer cube thing", we don't see opportunities to make the code better.

He emphasizes the importance of practicing outside of work. Writing a lot of programs is the most important things, and if we are lucky we can do it at work, but it's not enough. The ultimate goals of practicing and delivering production are differnt.

He read from cover to cover [the Big Book](http://amzn.to/2CtDGdy) of D. Knuth, but he wouldn't recommend it as tutorial to programming.

This LISPer who sonsiders himself a combination of an artist and craftsman also thinks that C is actually a problem. It's not a problem itself, but it's too complex and used at too many places. Few people understands it well enough. Perl is more secure and lets him to do the same stuff in many ways. He can decide how he does it, that's why it's artistic.

He thinks that Java is autoriatian because it takes away a lot of things. But maybe that's a good thing. I found this particularly interesting because [Uncle Bob](http://blog.cleancoder.com/) in his [Clean Architecture](http://amzn.to/2DL4hj7) also writes about what different paradigms take away from you.

In the header the author mentions that Cosell moved to a farm and barely programs. I was a bit said that this aspect of his life was not elaboreated in the book because he was not at all so negative about IT as [L Peter Deutsch was](/blog/2017/12/22/coders-at-work-l-peter-deutsch). 