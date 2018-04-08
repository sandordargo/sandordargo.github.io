---
layout: post
title: "Coders at work: Ken Thompson"
date: 2017-12-29
category: books
tags: [books, coders at work]
header: "The 12th guy interviewed in <a href=\"http://amzn.to/2wKEeVt\">Coders at Work: Reflections on the Craft of Programming</a> is <a href=\"https://en.wikipedia.org/wiki/Ken_Thompson\">Ken Thompson</a>, the creator of Unix and B programming language."
---
He has many more important contributions to computer science including the first special purpose chess computer or UTF-8, but I let you read about it on the internet.

To me, Thompson seems to be a very pragmatic guy, not someone who is convinced that the same rules apply everywhere and those rules are written in stone.

To mention one thing, let's see what he tells about the garbage collector. He thinks that we can do much faster, much better things than that. And because of its limitations, he couldn't imagine writing an operating system using a language with automatic garbage collection. On the other hand, he thinks it's still very useful because most of the programs don't need that performance gain what we can achieve manually taking care of freeing the memory up.

From time to time he was teaching some courses at universities but he never liked to do so on the long run, because a teacher repeats the same things over and over he doesn't find it satisfying.

Thompson is a bit scared of modern computing. This long bearded Unix hacker thinks that in modern programs there are just too many layers. Too many to understand what a program does. The recurring pattern appears again: you just cannot keep modern programs in your mind.

An idea I really liked and where I totally agree with him is that many people treat their code as they are written in stone. Once it's written, it's written, it should and it will never change. We should not be in love with existing code. We should refactor all the time because code rots inherently.

He also speaks against premature optimization. He thinks first we should write things as simple as possible. He considers writing a complex algorithm which is almost never used a really stupid thing, a "bug generator". Most of the time simple brute-force will just work fine.

There is one thing I found really surprising. I learnt that he didn't mean to write an operating system. All he wanted was a file system, he needed something to test that file system. That's how the Unix OS was written. After a while, he realized he wrote a real time-sharing system.

Let's finish with a very important thought of his. Don't work long hours to meet deadlines. Sometimes you just enjoy a project and you'll work day and night, but it's different. It's called excitement programming. When you work long hours driven by external deadlines it will just give you stress and you will burn out. These external short deadlines are never short term. There is always another one and eventually, you will burn out. Don't do that, work sane hours, keep your code and mind clean!