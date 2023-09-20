---
layout: post
title: "Once more about clean code"
date: 2023-9-20
category: dev
tags: [cleancode, performance, watercooler, cpponsea]
excerpt_separator: <!--more-->
---
This summer, I gave a talk at C++ On Sea about [Why Clean Code is not the norm](https://cpponsea.uk/2023/sessions/why-clean-code-is-not-the-norm.html) where at the end, Conor Hoekstra - who you might know from [code_report](https://www.youtube.com/c/codereport) or from other podcasts - asked me a question about what I think about the [<< "Clean" Code, horrible performance >>](https://www.youtube.com/watch?v=tD5NrevFtbU) video.

I had not seen it, but I said in the beginning of July that it would be a nice topic for my blog.

And here we are in September... :)

Yes, it's been more than two months.

To be honest, I started to watch the video right in Folkestone once the conference ended and I stopped it a few minutes later. I didn't like the style that I considered completely disrespectful or simply smug. Two months later, I decided to watch it. The style of someone shouldn't prevent me from hearing what that person has to say.

I still don't like the style, which might have had an effect also on how I interpret his message.

Still, I think it's worth [watching the video](https://www.youtube.com/watch?v=tD5NrevFtbU). We waste so much time scrolling social media, watching short videos for instant gratification and reading others' rants on completely insignificant topics. This video has at least something to say. Even if you disagree with a big part of it.

Let me summarize it in a paragraph. Casey Muratori finds "Clean Code" harmful in terms of performance. He attempts to showcase that following "Clean Code" guidelines you throw away decades of hardware improvements and therefore "Clean Code" is one of the reasons behind sluggish code. Muratori seems particularly against OOP and virtual functions.

Below, let me share my answer not to Muratori, but to Conor as he asked me what I think about this video and in particular the performance implications of clean code.   

## Why does this video have nothing to do with my talk?

What would I have answered to Conor if I had known this video? First of all, I hope I made it clear in the very beginning of my presentation, that it's about clean code and not about "Clean Code". I was not talking about [the books](https://www.sandordargo.com/blog/2018/03/14/most-important-books-to-start-with#clean-code-a-handbook-of-agile-software-craftsmanship) and work of Bob Martin. Even though I appreciate a lot his vision and I think the Clean Code series heavily influenced how I code and Uncle Bob pushed me in the good direction.

Yet, in the talk, I didn't refer to his "Clean Code" and the guidelines he shared. I referred to clean code as something more vague. I defined clean code as 

> *"something that is easy to understand and easy to change."*

And with that, let me get back to the video.

Muratori is obviously a very smart person who can write performant and correct code. A smart person, yet I think he **acts** ... well, differently ... and I think his goal is to make fun of "clean coders" and show them as incompetent programmers.

Why do I say so? He said that he wouldn't use a `for` loop with iterators because the book didn't say so. The book used plain old `for` loops. I had a strong urge at that moment - again - to close the tab. The book is not language-specific and was written a long time ago. Plus it's not written for morons who cannot make a single decision on their own and decide to go with a range-based `for` loop. Or was it? 

If iterators or a range-based loop is what makes a code "cleaner" then that is what you should use, it's that simple.

But I never wanted to get into such trouble and talk about specific rules that might depend on the language or even the standard you use, therefore I went with the above definition of clean code.

Is that kind of code less performant than optimal?

Yes! It is! And I know about it, probably most programmers do too.

At each C++ Conference, usually among the lightning talks, you'll find one or two which is about optimizing the last cycles out of some well-known and widely used standard functions. They are usually brilliant. And the people who present them on stage know more about the ins and outs of C++ than most of us.

But even they wouldn't use those optimized versions everywhere because those are usually difficult to understand and difficult to maintain.

Let's get back to the video. Is the performance of clean code really decades worse as Muratori claims? I doubt that, but I'm not going into a measurement war. Especially because we define clean code differently.

Read [C++ Software Design by Klaus Iglberger](https://www.sandordargo.com/blog/2022/12/17/cpp-software-design-by-klaus-iglberger). Most of the patterns he shares have several versions. The ones using value semantics are usually clean and easy to maintain. In my opinion, even the author of Clean Code would consider them "clean" (I might be wrong), but I think Muratori would not as they are not based on runtime polymorphism.

But would the lower performance matter that much?

Not always.

## When isn't "clean code" a bottleneck for performance?

Muratori comes from a world where performance is king. Game development. I think he has to rely more on performant and optimized code than many others.

I spent 10 years in the enterprise world, and nowadays I'm working on code behind mobile apps.

Performance is not the only thing that matters and it might not even be among the most critical aspects.

Let me share two stories with you.

### Horrible barely used code didn't matter

When I started my development journey in 2013, I was thrown into the deep water quickly. I wrote almost alone a new service for our backends. I wrote some horrible code and I received no meaningful code review on time. Such a badly managed project... Anyway, I shipped the code. It "worked". The returned results were correct, but there were two issues:
- a huge memory footprint
- extremely slow code

A talented but at the time not very experienced developer came by and he saw that something that I should have done in probably O(n) time was done in O(n^4). Yes, it was that bad, I was so clueless.

I might have been new to development, but I was not completely without common sense. (Despite my horrible algorithm!) I was convinced that that little piece of code could not be responsible for the overall slowness of the service. I didn't know how to measure performance correctly, but I had time, I had a spreadsheet and I had logs with timestamps.

The service spent a fraction of the time in those two horrible functions. They were not responsible for our problems. I'm not saying that they should not have been corrected. They shouldn't have been merged in the first place! What I'm saying is that they didn't matter for the performance of the system overall. It turned out that the 3rd party library was performing initialization and cleanup at every invocation of the service instead of the startup/destruction of the backend itself.

Once that was fixed, our performance and memory footprint issues were gone.

### The key to 10x traffic was not through removing virtual calls

A few years ago I was working on a service which faced a 10x traffic growth in a few months. The service was already quite old, but we secured a contract with one of the biggest players in online travel.

We shared our prediction with our business unit about how big of a growth our service could support, and we were right. Sadly, project managers didn't really care until the service actually crashed. The way out of the situation was not through optimizing our code in terms of performance or through removing class hierarchies to save a few cycles here and there, preferably on the hot path.

The way ahead was through decreasing the number of database connections we used and through using read-only connections wherever we could.

And guess what, we even made the code clearer. Some of the excess database usage came due to some horrible, difficult-to-maintain spaghetti code.

---

What I'm trying to say here is that you might work in fields where the runtime performance of your code is the most important aspect. Maybe in game development that's the case. Maybe with microcontrollers as well. But there are many other fields where it's different and even if you will have performance issues, that won't be solved by writing code that mimics how the CPU thinks. Managing database connections and network usage is usually more important.

On the other hand, if you write code that is easy to understand and easy to change, there is a fair chance that you won't even introduce such problems, but if you do, it'll be easier to recognize the root cause and eliminate the issue.

## "Clean Code" is more than the 5 rules it suggests

I already expressed that I found the video irrelevant in regard to what I had to say. But I found other issues too. If you watch the video, you might have the impression that "Clean Code" is about 5 rules (and Muratori dislikes 4 of them). But even if you just open the book at its table of contents, you'll see that it is about so many other things that are - seemingly - timeless.

Clean Code discusses how to pass function arguments in a way that won't lead to accidental swaps and often unnoticed bugs. Is it not important? I think it is and there are a few talks dealing with such problems at almost every C++ conference.

Another topic in the book is the usage of meaningful, descriptive, easy-to-read names. You might think it's obvious, but if you ever tried to understand a bigger corporate code base, you know that it's not a piece of knowledge each developer was born with. At my second C++ On Sea conference, in 2020, [Kate Gregory gave an awesome keynote on this topic](https://www.youtube.com/watch?v=MqugiGzricQ).

The Boy Scout rule applied to software development is another topic. Leave the code cleaner than you found it. Or if you want to avoid the term clean code, you can say leave the code in a better shape than you found it. I find that an essential rule to keep the codebase healthy.

But that's just 3 additional topics from the book, I haven't even mentioned its guidance on how and when to write comments, the importance of consistent formatting, or [the problem with returning nulls](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/).

Of course, there are some parts of the book that might be outdated or that would translate differently to other languages as we discussed the emergence of value-semantics-based patterns. But identifying "Clean Code" with the 5 rules he took and suggesting that "Clean Code" is the root of all evil when it comes to sluggish software is a bit of a far-fetched thought to my taste. I'm happy that at least the video doesn't suggest that those who write maintainable code are responsible for severe issues in our society as we use more cycles than necessary...  

## Conclusion

Overall, if I had to answer Conor's question in a few sentences, I would say that << "Clean Code" horrible performance >> has nothing to do with my talk as he picked on the book and I used a different definition for clean code. Still, I also think that the "Clean Code" he refers to is much more than the 5 rules he took.

At the same, I think that the performance implications of "Clean Code" or clean code applied to real-life problems rarely mean a bottleneck to software. Fields where this might be a problem are the niche and exception, not the rule.

Software designed for decades to serve us usually has to deal with other problems, such as maintainability and managing dependencies. Even in terms of performance, network, DB, and I/O, in general, are more problematic than virtual function calls and the costs of OOP in general.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!