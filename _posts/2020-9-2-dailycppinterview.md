---
layout: post
title: "Let me introduce Daily Cpp Interview"
date: 2020-9-2
category: dev
tags: [cpp, showdev, career, tutorial]
excerpt_separator: <!--more-->
---
I've got something to share. Something I've been building for the last few weeks has gone live today: [Daily Cpp Interview](https://www.dailycppinterview.com/).
<!--more-->

## What is Daily Cpp Interview about?

It's extremely simple. You subscribe and you'll receive a question or an exercise every day strongly related to C++. These questions will help you keep your skills sharp, keep your knowledge up to date, and not fading away.

You'll either get a more theoretical question about C++ such as what vtables are for, or you will get a piece of code that you will have to reason about, what it does exactly and why, or there is a third option: you have to write a short piece of code.

## But how do I know if my answer is right?

That's an important question to ask! The questions themselves help you inspiring your learning process, help you grow, or actually keep your knowledge from vanishing - repetition is the mother of all learning. With Daily Cpp Interview, you prepare for your C++ interviews.

So do you get the answers?

You'll find a link in each daily mail to a page where you can subscribe to the Pro edition. If you are a Pro subscriber, with each question you'll receive the solution as well. After your subscription is confirmed, you'll receive the previous answers as well.

I think the price is fairly reasonable, a little bit less than 10 Euros a month, and if you subscribe for the whole year, you get two months for free.

## Come on, I don't care about C++, but how did you build it?

I'm sure that are many of you interested in this part.

While I won't go into the very details of the code and some parts are still evolving, I'd love to share the major parts.

The site itself is not very interesting, a simple static Github page built with Jekyll.

What is more interesting is what is behind.

For the time being, I try to use as many free or cheap services as possible.

I use [Sendinblue](https://www.sendinblue.com/) to build the subscription forms and to have a mailing list, but I send the daily emails with [AWS SES](https://aws.amazon.com/ses/) which is simply cheaper. 

With Sendinblue's free tier I can store as many addresses as I can and AWS SES's free tier should be enough for my needs. If not, even better. 

On AWS side, I also use [Dynamo DB](https://aws.amazon.com/dynamodb/) to keep track of my users and I also store the questions and answers there. Again, the free tier should suffice.

The data transfer between Sendinblue and AWS services is managed with [Zapier](https://zapier.com/). Just like the data sharing between Stripe and AWS. So yes, I take payments with [Stripe](https://stripe.com/) client only integration.

Possibly my free Zapier resources will run out, but it will be a good sign and I'll be happy to pay.

As you can see, it's fairly simple. For the "backend part" I mostly use free or cheap services and python code, the ultimate glue language.

## Conclusion

I built [DailyCppInterview](https://www.dailycppinterview.com/) in about a month during my mornings and evenings with great enthusiasm.

There are still things to better on the page, in the integrations, but I think that the most important thing is to go out and deliver. Deliver iteratively, otherwise, I'd just stuck in a "still not good enough" state and would never publish.

I'm sure that some people will find it helpful and some will maybe even subscribe which will be great. If not, I already learned a lot about AWS, different tools, and integrations that was already worth it.

If you are a C++ developer, please go ahead and [subscribe for the free daily newsletter](https://www.dailycppinterview.com/).
