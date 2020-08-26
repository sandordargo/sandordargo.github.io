---
layout: post
title: "Effective STL: 50 Specific Ways to Improve Your Use of the Standard Template Library by Scott Meyers"
date: 2020-8-26
category: books
tags: [books, cpp, stl, watercooler]
excerpt_separator: <!--more-->
---
I have learned, [written](http://sandordargo.com/tags/#stl) and [spoken](https://www.youtube.com/watch?v=BEmAo6Fdg-Q) a lot about the Standard Template Library during the course of the last years. My sources have been mostly websites such as [cppreference.com](https://en.cppreference.com/w/), [cplusplus.com](http://cplusplus.com/), blogs, youtube videos, but not so many books.
<!--more-->

Last year I read [The C++ Standard Library: A Tutorial and Reference by Nicolai Josuttis](http://sandordargo.com/blog/2019/07/10/cpp-standard-library-a-tutorial-and-reference) - who by the way gave [a very interesting keynote about std::jthread at C++ On Sea](https://www.youtube.com/watch?v=elFil2VhlH8).

Recently, I decided to pick up another promising book, [The Effective STL](https://amzn.to/32c62Ub) by the great [Scott Meyers](https://www.aristeia.com/). 

Did the book meet my expectation?

It did!

## Who should read it?

I recommend you to read [The Effective STL](https://amzn.to/32c62Ub) if you are not totally new to the Standard Template Library. Why don't I recommend it to complete beginners?

Not because you need some prior knowledge. You need to understand C++ on a basic level of course, but that's not my concern.

In the preface of [My Early Life](https://amzn.to/2YlY21N), Winston Churchill wrote that society, politics, war, youth, values have changed since the events of the book happened and the points of view he wrote down were appropriate to his age and the era even if they are not gerenally accepted anymore.

While the majority of this book is still valid, some pieces of advice bacome obsolete by the almost 20 years that passed by since the release of [The Effective STL](https://amzn.to/32c62Ub) in 2001. Accept the rest as representing the state of the art of the pre C++11 era.

Anyway, if you are a complete beginner and you decide to pick up this book, you'll improve a lot, the only thing is that your code will not be very modern and in some cases, you lose some of the efficiency and expressive power that C++ and the STL gained with its modern versions (starting from C++11).

## How is it organized?

The 50 items of the book are organized around 7 chapters: Containers, Iterators, Algorithms, Functors. No surprise, after all, they are the key elements of the STL.

Wait, this is only 4! The last one is "Programing with the STL" and there are two more, right after the _Containers_: chapter 2 is about vectors and strings and chapter 3 is about associative containers.

To me, this organization is a little bit odd, though I understand that the author wanted to avoid having some huge chapters and decided to break down some.

## What will you learn?

There are some real "tricks" like using the so-called swap trick to remove excess capacity from a vector (Item 17), there are items that I also talked about at [C++ On Sea](https://www.youtube.com/watch?v=BEmAo6Fdg-Q), such as algorithms that are expecting sorted containers (Item 34), but there are at least 2 items that can have a much bigger impact on how you write.

### Item 43: Prefer algorithm calls to hand-written loops

This is something that went viral since then. [In his famous talk, Sean Parent advocated for it](https://channel9.msdn.com/Events/GoingNative/2013/Cpp-Seasoning), it was recently a topic in the Italian C++ Conference disguised as the [Initialize Then Modify anti-pattern](https://www.youtube.com/watch?v=CjHgL5EQdcY) presented by Conor Hoekstra, and I also [wrote about it](http://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms) earlier this year.

I'd suggest you to check out any of these resources, probably in the order I listed them.

Still, let me summarized the main reasons.
Algorithms are:
- more expressive than handwritten loops
- well tested, so less error-prone than raw loops
- and for most us - mortal human beings -, algorithms will be more performant

### Item 47: Avoid producing write-only code

When I read the term write-only, I didn't understand what it is. Of course, you shouldn't use variables that are never read, you shouldn't have unused variables, but they generate compiler warnings anyway. And hopefully, we all handle warnings as errors in our projects.

But this item is not about unused variables. It's about code that you write once and then nobody dears to touch it. That's what Meyers meant by write-only code.

We all know this type of code. A four thousand long shell script where you have functions of several hundreds of lines long and each time your team has to extend it, you fight not to be the next unlucky one who must take it this time, but time is never given to actually understand it and make it more readable.

We all have something like that and item 47 is not about that kind of write-only code!

What else can be?

Have you read the [Software Craftsman by Sandro Mancuso](https://amzn.to/2QdYzOT)? There is a story about his younger self who managed to got into his dream team at his workplace and he wanted to impress his new boss with some brilliant code.

His boss walked by and deleted all of it.

Remember, you don't write code to impress people. You write code to deliver solutions, solutions that can be maintained. Keep it stupid simple. Don't use techniques that no one else will understand. Don't use obscure libraries. 

Write easy to understand, easy to maintain, yet correct code. That's your job.

And what is easy to understand will depend of course on your team. It won't be the same in some niche company mostly with really experienced profile devs and in a huge corporation with a high turn over rate and a big bunch of [expert beginners](https://daedtech.com/how-developers-stop-learning-rise-of-the-expert-beginner/).

You have to assess, you have to find the balance keeping the one goal in mind.

## Conlusion

Despite its age, I still recommend reading [The Effective STL](https://amzn.to/32c62Ub) if you want to boost (no pun intended) your knowledge on the Standard Template Library. You will understand what is going on under the hood when you use certain techniques you knew about, you'll learn new tricks and in general, you'll understand better how the STL is designed, how each element should work together.

Happy reading!