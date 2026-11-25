---
layout: post
title: "Managers we are, but interrogators we must be"
date: 2026-11-25
category: dev
tags: [ai, career, craftsmanship, watercooler]
excerpt_separator: <!--more-->
---
I think we can all agree that our craft has changed a lot over the last year. For a while, AI was mostly a smarter autocomplete and a faster Stack Overflow. Then, somewhere around the end of last year, many of us simply stopped writing code by hand.

Hence the popular line: *we are not coders anymore, we are managers of agents.*

I think that's only half of the answer. The manager analogy is good, but it misses the part of the job that matters most.

<!--more-->

## Why the manager analogy works

Let's start with what's right about it.

A good manager tells you what to do and sets the boundaries of the task. When you work with agents, you play two managerial roles at once - often without noticing.

As a **product manager**, you define the desired behaviour and the expected outcomes. What should this feature do? What does "done" look like? The agent can't guess that for you, at least not reliably.

As an **engineering manager**, you set the technical boundaries. In a human team, an EM rarely spells those out task by task. They emerge from long, often collaborative processes: architecture decisions, coding guidelines, the accumulated taste of the team. But the EM is still accountable for the technical outcome. And so are you. Whatever the agent writes, it ships under your name.

What you can skip is the human part. No one-on-ones, no career conversations, no worrying about morale. Although I know people who say *please* and *thank you* to their agents. Just to be on the safe side when the machines take over...

A manager also gives feedback to correct course. Hopefully more often than once per performance cycle, but that doesn't mean evaluating every single task either. When a manager feels compelled to intervene in every user story, it's often a sign of micromanagement.

Why don't good managers do that? Because they trust their people. They give them room to make decisions, make mistakes and learn from them. They help you get better precisely because they trust you.

And when that trust is lost, a good manager will work to restore it. If it cannot be restored, they'll eventually make a difficult decision rather than compensate for the lack of trust through constant micromanagement — regulations permitting...

## Where the analogy breaks

And that's exactly where the analogy ends. It's not a short analogy, but it's not a complete one either.

**Your manager trusts you. You must not trust your agents.**

Agents have a better short-term memory than we do. They are also far more eager to satisfy your demands than any human colleague. That eagerness has a price. They make things up. They skim over details that a good engineer would stop at. And while doing so, they are so verbose and so confident that it's easy to believe they know what they are talking about.

So you need a second persona. One that would be extremely annoying in a manager.

![Columbo]({{ site.baseurl }}/assets/img/columbo.jpg)

That's right. You have to be a detective. An interrogator. You don't have to beat up your machine, but you do have to be relentless.

In my experience, a good interrogation goes in two directions.

### Direction one: why does it work?

First, ask your agent to justify its solution. Why does it work?

It will happily go to great lengths explaining its decisions. Let it. Those long explanations are not noise, they are the raw material of your job. As an engineer, you are still responsible for understanding every decision behind the code you ship. Not necessarily every line - reading lines is the easy part - but every decision. Why that container? Why that algorithm? Why that compromise and not another one?

Then you have to actually think about the answers. Read them like a detective reads a suspect's story. Sooner or later you'll feel where the reasoning gets thin. That's where the agent is lying, or to put it more charitably, where it's most likely wrong.

That's where you push. Where the reasoning is weak is exactly where you can make your solution stronger.

### Direction two: why doesn't it work?

The other direction is the opposite question: *why will this solution fail?*

This is where agents shine compared to us. We humans are notoriously bad at finding problems in our own work. Agents don't have an ego to protect, so they find flaws in what they just proposed with remarkable ease.

In fact, they are so good at it - and so keen to please - that they will invent problems when there aren't enough real ones. Ask for ten reasons why the code is broken, and you'll get ten, whether or not ten exist.

That's the interrogator's second job: separating the wheat from the chaff. Discard the made-up problems, keep the real ones. And once you have the real ones, you know exactly what to fix.

## Conclusion

So yes, you are a manager of your agents. Define the outcomes, set the technical boundaries, give feedback. And, just to be safe, be as nice to them as you can.

But a manager alone isn't enough, because a manager's job is built on trust, and trust is precisely what you can't give your agents. Next to the manager, you also need the interrogator. Ask them why they are right. Then ask them why they are wrong. The truth - and the stronger solution - is usually somewhere in the gap between the two answers.

{% include connect-deeper.html %}