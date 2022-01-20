---
layout: post
title: "3 piece of advice for junior developers"
date: 2021-x-x  
category: dev
tags: [beginner, interview, career]
excerpt_separator: <!--more-->
---
A few months ago, my management asked me if I'd like to facilitate some technical interviews. My answer was a hell yeah! I think it's great that management involves senior developers in the selection process. On the one hand, it's good for the team if they know that someone who is hands on with the code saw the new colleague, talked to him/her and asked - hopefully - relevant questions. On the other hand, it's also reassuring for the candidates that they can talk to someone who is doing the same job as he would. 

During these few months, I conducted a couple of interviews, both with junior and senior people and I'd like to share a some pieces of advice - mostly for developers with little experience.

## Your CV vs reality

My first job was a non-developer job at a small company dealing with municipalities. I was called a project manager, that was my official title. I was thinking to put that on my CV.

But it felt better to simply just write put project coordinator on it. It's less fancy and it creates less expectations. Later, with more experience in the bag, I changed the title back to project manager.

Why do I tell this?

Even though the role of your resumé is to let you pass the screening and get you an interview, it's not as simple as that. 

Your CV shouldn't harm you during the interviews!

Tailor your CV for each position. In small startups, sometimes the person with the most experience gets the catchy title of a CTO. Even if that "most experience" is just an extra couple of months. Now, if you communicate that you are a CTO - even at a small startup - and you apply for a (junior) developer job, you'll meet high(er) expectatations if you would have just written that you are a developer at that startup.

Another bad habit people are prone to during the beginning of their career is skill stacking on the CV. If you list 10 different technologies you're proficient with after 1-2 years of work, it's hard to assume that it's realistic.

I wouldn't realistically expect it even from someone with 10 years of experience.

I had a look at my CV, and I also have 10 skills listed as advanced after a being on the market for 11 years. Yet, almost half of them are about the same technology:
- C++ with a version number meaning that I'm up-to-date with up until that version's features,
- the C++ STL meaning that I'm comfortable with the [standard template library](https://www.sandordargo.com/blog/2019/01/30/stl-algos-intro),
- and 2 more C++ test frameworks, [gTest](https://www.sandordargo.com/blog/2019/04/24/parameterized-testing-with-gtest) and gMock.

In case you mention that you worked as an instructor, know at least the basics of what you taught. You don't have to know all the advanced things of that technology, but you must know the basics pretty well.

If you already forgot what you taught and you wouldn't be able to explain those concepts again, maybe it's better not to list yourself an instructor. Or only mention it if the technology is not relevant for the job you applied for. But even that has its own dangers. Assume that you'll meet someone who will ask about those topics that you thought.

## Admit that you don't know

Speaking about not knowing things... It's perfectly fine not to know answers to all the questions. 

Just admit that you don't the asnwer to a given question. 

We all have to look things up, we all - should - learn every day. I learnt C++ syntax a couple of months ago from one of our fresh grads. And I was more than happy about it!

But don't just say something hoping that it will be the right answer. An interview is not a university test where you should throw a dice hoping that you might get it right and get the extra point.

In any case, most interviewers will catch it if you just try your chance.

The more complex the question, the more true this is. Throwing in whatever technology as na answer for an architectural question will never work.

Interviewers don't care whether you'd use docker or another technology for scaling up a service. They, we are interested in how you think and what kind of issues, contradictions you expect to encounter during the design and implementation of a scalable system.

Let me give you an example. While we were scaling an application, we started to add new application servers, but at a point this meant that more and more database connections were opened to the single database server and it couldn't cope with it.

Describing such a situation doesn't require any technology.

## Some concepts you just have to learn

So far we discussed a couple of things to pay attention to when you write your CV and also to admit when you don't know the answer to a question.

There is one last thing, I want to share in this article.

Many questions, many topics are technology dependent.

You don't really care or know about gMock or gTest if you are a React developer.

You don't care about Angular or React when all what you do is touching backend services serving hundreds of transactions (or more) per second.

But you should care about concepts that are useful everywhere in IT.

Some evergreen concepts you should learn about:

- Unit testing
- Automated testing in general
- How to write clean code
- Refactoring
- SOLID design principles
- Data structures
- A bit of algorithms

I don't say that you should know everything nor that this is an exhaustive list or that data structures and algorithms have the same importance at the frontend and at the backend world. Though knowing some basics won't hurt.

Obviously you cannot learn everything at once and you don't have to. The best thing you can do - in my opinion - is to read a few pages every day and practice. [Here are a couple of books](https://dev.to/sandordargo/8-books-every-junior-developer-should-read--4p5h) that helped me a lot.

## Conclusion

In this article, I shared a couple of things, mostly inexperienced developers do when they apply for a job.

I think you can seriously increase your chances if you don't try to suggest something in your CV that is not true.

Definitely you should admit when you don't the answer to a question!

But I think the most important is to keep learning every day and start with concepts that will be useful for you wherever you go.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
