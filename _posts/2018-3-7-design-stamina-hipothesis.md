---
layout: post
title: "Design Stamina Hypothesis"
date: 2018-3-6
category: dev
tags: [code, architecture, meetup]
header: "Attending meetups can be great, you can learn a lot from others, you might hear about concepts you never heard off. You take some notes and there you go, you can dive into them later.."
---
I've learnt about [Design Stamina Hypothesis](https://martinfowler.com/bliki/DesignStaminaHypothesis.html) at one of the latest events of the French Riviera Craftsmanship Community Meetup. This hypothesis is all about architecture or the lack of it including the costs and tradeoffs.

The main question leading this hypothesis is that in a given project is it worth to spend on the architecture (or not) and if so, how much.

From a practical point of view, think about it like this: in case if you spend a lot of time on design, ideally, later on adding new features will be relatively cheap. In the beginning, implementing features will be relatively costly and in addition you may start implementing features later, because as I've just written, before, you spend some time on design, architecture, laying down processes, whatever.

You might have noticed, that I used the term _relatively_ a few times. Why did I do that? Relatively in the aspect of of what? Relatively in the aspect of having no design/architecture/processes at all. Take the other extreme, you just start imlementing the features by jumping into coding. This can produce good results pretty fast in the beginning. But eventually you will end up with an application that is hard to maintain, where things are hard-wired and where adding new features becomes a nightmare.

If you think about and have a look at the below graph, you will see that there is a point until when the second "don't think, but just do it" approach will be more effective, but after that point, inital and constant design and following an architecture pays off and over time it pays off with higher and higher dividends.

![Design Stamina Graph by Martin Fowler]({{ site.baseurl }}/assets/img/designStaminaGraph.gif)

However finding that intersection point is difficult and in fact there is no way to calculate it properly. One will be better and better at by gaining experience and failing a few time.

The key takeaway is that the bigger and longer lasting your application will be or you plan it to be, the better it is to invest in design and processes. Not just once in the beginning but permanently. If you want to read a bit more about this topic, you can start by [this article](https://martinfowler.com/bliki/DesignStaminaHypothesis.html) of [Martin Fowler](https://martinfowler.com/). By the way, to learn more about design I'm reading his book called [Patterns of Enterprise Application Architecture](http://amzn.to/2EFnZyj). You might want to check it out if you want to improve your skills on application design. Soon, I'll write a review on it.
