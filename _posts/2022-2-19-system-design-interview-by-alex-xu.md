---
layout: post
title: "System Design Interview: An insider's guide by Alex Xu"
date: 2022-2-19
category: books
tags: [books, career, interview, systemdesign]
excerpt_separator: <!--more-->
---
Being a software engineer is special compared to many other professions in several ways. One aspect of this speciality is that you don't just go to a job interview after polishing a bit your CV, thinking about your career and maybe reading a few interesting and professionally relevant articles.

Getting your next job often requires extensive learning and practice no matter what your current level is.

To get into good companies, often you'll have to go through at least half a dozen interviews proving that you're capable of doing things that you have never done since university and you'll never do on the job.

![Useless questions at tech interviews]({{ site.baseurl }}/assets/img/tech-interview.jpeg)

Even if many disagree with it, that's the process to get into certain companies.

We have to prepare.

One kind of interview is covering system design skills. In my opinion, even if you're not preparing to become a software architect, the knowledge tested in such interviews is way more relevant than crafting sorting algorithms on a whiteboard. They are about problems that we have to be solved in most applications. It's really useful to understand as a developer how our systems are composed.

[System Design Interview: An Insider's Guide](https://www.amazon.com/gp/product/B08CMF2CQF/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=sandordargo-20&creative=9325&linkCode=as2&creativeASIN=B08CMF2CQF&linkId=6bd98a7b06a1165368754af945bae169) by [Alex Xu](https://twitter.com/alexxubyte) is a great tool to prepare for such interviews.

Even if you're experienced with system design I think it's useful to read the book as 
- it will help you refresh your knowledge
- it talks about the problems from the perspective of an interviewer, focusing on what your potential future employers are looking for
- it'll give you some ideas that you might figure out on your own when designing a system, but for sure you'll not just make them up during interviews.

## What does it include

Let me briefly share what kind of system design challenges are shared in the book.

- Scale From Zero To Millions Of Users
- Back-of-the-envelope Estimation
- A Framework For System Design Interviews
- Design A Rate Limiter
- Design Consistent Hashing
- Design A Key-value Store
- Design A Unique Id Generator In Distributed Systems
- Design A Url Shortener
- Design A Web Crawler
- Design A Notification System
- Design A News Feed System
- Design A Chat System
- Design A Search Autocomplete System
- Design Youtube
- Design Google Drive

It's worth noting that often one chapter builds on another, such as a unique ID generator is referred to in later chapters. Still, you can read the chapters in any order as the references are explicit and if you are interested, you can easily jump to other topics.

The list of references and further readings at the end of each chapter are also very valuable.

## Some ideas to take away

Let me share 3 ideas from the book that I especially liked.

### The 4-step process to tackling system design questions

Throughout all the chapters, the author uses the same 4-step process to explain how to design the particular system. He also recommends using the same approach in actual interviews. Here are the 4 steps:

- Step 1 - Understand the problem and establish design scope
- Step 2 - Propose high-level design and get buy-in
- Step 3 - Design deep dive
- Step 4 - Wrap up

First, you have to **make sure that you understand the problem**, you understand what the interviewer is interested in. Ask questions!
- Make sure you know what are the specific features to build. 
- You need to know approximately how many users you're going to have
- How fast will the company scale

This will ensure that you don't start designing something completely different from the expected and it will show the interviewer that you don't just start building software based on premature or possibly false assumptions.

As a next step, **come up with a high-level design on which you can agree with the interviewer**. Don't just work silently, but collaborate, draw diagrams, ask for feedback.

Treat the interviewers as teammates. After all, it's possible that you'll actually become teammates and will have to similarly collaborate.

Go through a couple of use-cases that will help you find shortcomings of the design and some edge-cases you wouldn't have thought about otherwise.

By the end of the second step, you should have already agreed on the goals and the scope of the design and you should have a high-level blueprint of the overall design reviewed by the interviewer.

In an interview, you won't have the time to work out everything in detail. You should get feedback from the interviewer to know what are the parts she's mostly interested in. It might be some components of the architectures, maybe the high-level design will be enough, maybe you'll have to focus on some non-functional requirements, such as system performance. **Make sure that you have the buy-in** from the interviewer on what to focus on and start going into details.

Finally, wrap up. You **recap what you built, what are the parts that could be or should be improved**. What are the potential bottlenecks, error-cases that should be considered. Share how would you continue if you had more time.

### Vertical vs horizontal scaling

For many of you, it might seem basic, but it's still important to emphasize some basic vocabulary when it comes to system design interviews.

When you design a system, you must think about what you'd do in case the volume of traffic significantly increased. You have to think about scaling your application.

But scaling where and how?

In brief, there are two kinds of scaling.

- Vertical scaling is often referred to as scaling up your application. It means adding more physical resources such as CPU, RAM, etc to your servers.
- Horizontal scaling is also called scaling out your system. Instead of having more powerful servers, it's about having more servers.

The larger applications you design, the more you have to think about horizontal scaling as vertical scaling has some hard limits. It's limited to how many CPUs, how much RAM you can add to a server.

Probably these are the most basic fundamentals to learn when you start preparing for your interviews.

### Back-of-the-envelope estimations 

In system design interviews you'll often have to estimate either performance requirements or capacity limits. Many of us, developers, had to learn some complex math at the university. These interviews will not be the place to use that precious knowledge.

To come up with the numbers, feel free to approximate, simplify and scribble some numbers on a piece of paper. In other words, you'll only have to perform some estimations, some back-of-the-envelope calculations.

While it seems a simplification and indeed it is, it doesn't mean that it's that simple.

Effectively performing such estimations requires practice and a "good sense of scalability basics". I remember that we had to practice it in university. We were specifically told in certain cases not to perform precise calculations, but to estimate instead.

If you are not that familiar with such calculations, pay special attention to the second chapter of each design challenge where such calculations are carried out.

## Conclusion

[System Design Interview: An Insider's Guide](https://www.amazon.com/gp/product/B08CMF2CQF/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=sandordargo-20&creative=9325&linkCode=as2&creativeASIN=B08CMF2CQF&linkId=6bd98a7b06a1165368754af945bae169) is a very useful resource if you plan to apply for some jobs. It will help you think about technical problems you might have not met so far in your career. It would be quite difficult to come up with good designs at an interview without systematically thinking them through. And here comes the other major value of this book. It gives you a systematic approach to preparing for and tackling such interview questions.

A highly recommended read.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
