---
layout: post
title: "Briefly about Mutation Testing"
date: 2018-2-14
category: dev
tags: [testing, mutation testing, chaos engineering]
header: "Recently I wrote to small posts about <a href=\"/blog/2017/12/20/briefly-about-chaos-engineering\">chaos engineering</a> and <a href=\"/blog/2018/01/11/mutation-testing\">mutation testing</a>. This will be a similarly short post to recap how they relate to each other."
---
First, why do I even compare them? A few months ago at the [Riviera Craftsmanship Meetup group](https://www.meetup.com/Riviera-SCC) we discussed about different testing strategies and first Chaos Engineering and then Mutation testin came up. I knew nothing about them, hence these posts.

So let's recap.

Both strategies are based on more-or-less random changes related to the software under test.

Chaos engineering is about changing the external world. You simulate traffic peaks, crashing servers, failing network hardwares. Then you check how your software behaves in such conditions.

Mutation testing on the other hand is about changing the code itself. The code is changed to a small extent and then if your tests fails, the mutant can be rejected, a.k.a. killed. The more mutants your tests can kill, the better your test suite is.

While chaos engineering is about raising your confindence in your system's robustness and resiliency, mutation testing is about raising your confidence in your test suite, by locating weaknesses in your test data or in directly in your code.

So both strategies are important and serve different roles. Using them can highly increase the quality of your tests, in the end the quality of your software.