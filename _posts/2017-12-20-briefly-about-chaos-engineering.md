---
layout: post
title: "Briefly about Chaos Engineering"
date: 2017-12-20
category: dev
tags: [testing, chaos engineering]
header: "Since a while ago I've been attending the events of French Riviera Software Craftsmanship Community. Sometimes we do a coding dojo and practice - among others - our pair programming skills, sometimes we just discuss various topics."
---
Recently discussed different testing strategies and came across chaos engineering and also mutation testing came up. As I didn't really know about these concepts, I think it can be useful to others as well if I briefly present what chaos engineering and mutation testing are and how they relate to each other. Today I'll briefly discuss Chaos Engineering.

Chaos Engineering was developed by Netflix. The engineers wanted to test the resiliency and recoverability of their Amazon Web Services. Hence they developed a software tool called the Chaos Monkey that simulates failures of instances of services running within Auto Scaling Groups (ASG) by shutting down one or more of the virtual machines.

Let's have a look at one of the definitions: 

"Chaos Engineering is the discipline of experimenting on a distributed system in order to build confidence in the system’s capability to withstand turbulent conditions in production."

What does it mean in practice?

At least in our imagination, there is an ideal state of our system which represents normal behaviour. In chaos engineering, this is called the steady state.

We make two groups, just like in other experiments we set up a control and an experimental group.

Now we try to introduce the chaos. Think about real-world events, like crashing servers, a mal-functioning network switch, a spike in traffic, etc. The bigger potential impacts and chance of occurrence, the better.

Check your system. Is it still working as expected? Have you managed to tilt it from its steady state? If no, your confidence in your system under test just grew. If yes, try to fix it, try to make your system more robust/resilient.

There are several tools outside you can use for practicing the chaos. Not surprisingly one of the most widely used is [Chaos Monkey by Netflix](https://github.com/Netflix/chaosmonkey). Yes, it is open source.

I found this interesting [GitHub repository](https://github.com/dastergon/awesome-chaos-engineering) which gathers many sources of Chaos Engineering. If you'd like to explore it deeper, I'd advise you to check it out.

If this short introduction is enough for you, your key takeaway is that Chaos Engineering is about changing the external world - from a system point of view. Its purpose is about raising your confidence in your system's robustness and resiliency. If you practice chaos from the beginning, you can have an extreme confidence in your system.