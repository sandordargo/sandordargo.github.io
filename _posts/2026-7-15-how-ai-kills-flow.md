---
layout: post
title: "The Prompt-Wait-Evaluate Loop: How AI Kills Flow Without You Noticing"
date: 2026-7-15
category: dev
tags: [ai, productivity, career, watercooler]
excerpt_separator: <!--more-->
---
A few months ago, I wrote about [finding joy in programming in the age of AI](https://devladder.substack.com/p/can-we-still-find-joy-in-programming). In the personal discussions I've had with fellow developers — both before and after that article — one thing keeps striking me. Most people don't argue. They just say: *yes, that's exactly how it feels*.

But a question kept coming up in those conversations: *why* does it feel this way? Not in the philosophical sense — I covered that. In the mechanical sense. What is actually happening to our attention when we work with AI coding assistants?

I want to explore that topic more deeply, because I think understanding the mechanism matters. It's the difference between vaguely sensing that something is off and being able to do something about it.

<!--more-->

## What flow actually is

In 1990, the psychologist Mihaly Csikszentmihalyi published [*Flow: The Psychology of Optimal Experience*](https://amzn.to/453XAaw), the book that gave a name to what probably all of us had already experienced. Flow is the state of full absorption where action and awareness merge. You lose track of time. Self-consciousness fades. The work just happens.

Csikszentmihalyi identified three preconditions:

- **Clear goals.** You know what you're trying to accomplish at each moment, not just at a high level, but at the resolution of the next few keystrokes.
- **Immediate feedback.** You can tell right away whether what you just did worked. The compiler, the test, the output — something tells you where you stand.
- **Challenge that matches your skill.** The task is hard enough to require your full engagement but not so hard that you freeze. You're stretched, not stuck.

Programming, at its best, is one of the purest flow states there is. That's half of why we loved it. Think about a good debugging session. You have a clear goal — find the bug. You get immediate feedback — the test passes or it doesn't, the log says this or that. And the challenge scales with you. As you eliminate hypotheses, the search narrows. You're fully in it.

The same goes for writing a new component from scratch. You hold the architecture in your head, you lay down the skeleton, you iterate. Each compilation tells you whether you're on track. The difficulty is real but manageable. Hours vanish.

I think most experienced developers can point to moments like this. These are the sessions you remember fondly, even years later. You were not just productive. You were alive in the work.

## The old thieves

Flow has always had enemies. We already knew them well before AI entered the picture.

Meetings that break your morning in half. Slack messages that make you look away for *just a second*. Email notifications. Code review requests that pull you out of your current task and into [someone else's context](https://www.sandordargo.com/blog/2026/06/03/making-reviews-actually-fast). Somebody popping up at your table in the open office just to have you for a minute. The daily standup scheduled at 11 AM, right when you were about to get into it.

Gloria Mark, a professor at UC Irvine who has spent over two decades studying how knowledge workers actually spend their time, [found](https://dl.acm.org/doi/10.1145/1357054.1357072) that after a single interruption it takes an average of 23 minutes and 15 seconds to return to the same level of focus. Not to start the same task again — to reach the same *depth* of engagement. Her follow-up research, published in her 2023 book [*Attention Span*](https://amzn.to/4ymXurW), showed that the average time people spend on a single screen before switching has dropped from 2.5 minutes in 2004 to just 47 seconds.

These numbers are not new. We have known about them for a long time. But they matter now more than ever.

Because AI added a new thief. And it's sneakier than all the old ones.

## The new interruption you invited in

Here is what a typical AI-assisted coding session looks like.

You have a task. You think about it for a few moments. You write a prompt. You submit it. You wait. The model generates something. You read it. You evaluate it. Maybe it's close, maybe it's off. You adjust the prompt. You submit again. You wait again. You read again. The cycle repeats.

Prompt. Wait. Evaluate. Re-prompt.

This is not flow. This is the *opposite* of flow. Let me walk through why.

**The goals are no longer clear at each moment.** When you type code yourself, you know what the next line should do. You hold a mental model and you execute against it. When you write a prompt, you're describing what you want at a high level — but you have no moment-to-moment clarity about what the machine will produce. You are waiting for a surprise.

**The feedback is no longer immediate.** Instead of the tight loop of type-compile-see, you get a chunk of output after a delay. With a chat-based assistant, that might be a few seconds or thirty seconds. But if you work with an agentic coding tool — and more of us do every day — the wait stretches to minutes. Sometimes tens of minutes. The agent is running, editing files, calling tools, and you're sitting there watching a progress log scroll by.

And what do you do while you wait? This is where the real damage happens. Some of us try to stay close to the problem — re-reading the code, thinking about edge cases, deepening our understanding. But be honest: how often does that actually happen? More likely, you check Slack. You glance at email. You open another browser tab. You doomscroll. Or you start a second AI session on a different task, which means you're now context-switching between two problems you're not fully inside of.

By the time the agent finishes, you're somewhere else entirely. And now you have to come back, re-read what it did, and figure out whether the result matches what you had in mind five or fifteen minutes ago.

**The challenge no longer matches your skill in the right way.** Writing the prompt is often too easy. Evaluating the output is often too hard — because now you're reviewing code you didn't write, built on assumptions you didn't make, with patterns you might not recognize. The difficulty is in the wrong place.

All three pillars of flow collapse at the same time.

And here's what makes it insidious: it *looks exactly like work*. You're in your IDE. You're typing. Code is appearing on screen. Your manager sees commits going into the repository. Your metrics might even look better than before.

But inside your head, you're not building. You're context-switching. Every prompt is a handoff. Every response is an interruption. And every evaluation is a cold start.

## The cost nobody sees

Chris Parnin and Spencer Rugaber studied what actually happens when a programmer gets interrupted. Their [2011 paper](https://link.springer.com/article/10.1007/s11219-010-9104-9), based on 10,000 recorded sessions of 86 programmers and a survey of 414 more, contains a finding I keep coming back to:

> Only 1 in 10 interruptions let the programmer resume coding within a minute.

Think about that. Nine out of ten times, when your programming gets interrupted, you need more than a minute to get back. And that minute is just when you start typing again — it doesn't mean you've recovered your full mental model.

Because resuming a task is not about finding the right file and putting your cursor back in place. It's about reconstructing the mental context: the goals, the plan, the invariants you were holding in your head, the constraints you had already considered and dismissed. Parnin and Rugaber found that 93% of the sessions they observed involved significant navigation — opening files, scrolling, searching — before the programmer could start editing again. They weren't looking for where they left off. They were *rebuilding* where they left off.

This is exactly what happens between each prompt-response cycle. You write a prompt. You get a response. To evaluate it, you have to load the generated code into your head, compare it against the mental model of what you wanted, and assess whether the two match. That's not resuming. That's rebuilding from scratch — every single time.

And as Gloria Mark [put it](https://news.gallup.com/businessjournal/23146/too-many-interruptions-work.aspx) when describing her research: *We don't have work days. We have work minutes that last all day.*

When I think about my own experience, that description fits disturbingly well. In the old days, I might have had two or three deep sessions during a work day. Some were an hour, some longer. Now I have dozens of short bursts, each of them interrupted by a prompt-response cycle that resets my mental state. The total productive output might even be similar. But the *feeling* is completely different. A day of shallow work that produces the same output as a day of deep work is not the same day. Not for your skills, not for your morale, not for your growth.

## Why we don't notice

If the prompt-wait-evaluate loop broke flow visibly — if every time we submitted a prompt we felt the same jolt as a Slack ping — we'd protect ourselves. We'd batch our AI interactions. We'd set boundaries.

But we don't. Because AI-assisted work doesn't feel like an interruption. It feels like the task itself. The loop is woven so tightly into the workflow that we can't tell where the work ends and the interruption begins.

This is different from all the old flow-killers. A meeting has clear boundaries. You leave your desk, you come back. A Slack message is an external event. You notice it arriving. But when you type a prompt and wait for the response, it doesn't *feel* like you stopped working. It feels like you're still inside the task.

You're not. You're in a holding pattern, waiting for someone else's output, and when that output arrives, you're not continuing — you're starting over at a different altitude.

About 44% of all interruptions are self-generated, according to [Gloria Mark's research](https://amzn.to/4ymXurW). We do it to ourselves. The AI prompt cycle is the latest and most productive-looking form of self-interruption we have ever invented.

## So what do we do?

I don't think the answer is to stop using AI tools. That ship has sailed for most of us, and besides, they are genuinely useful for a whole class of tasks.

But I think we need to be honest about the cost, and deliberate about when we pay it.

One approach is to separate your day into modes. There are tasks where the prompt-wait-evaluate loop makes sense: boilerplate, configuration, exploratory prototyping, writing tests for well-understood behaviour. These are tasks where you wouldn't reach flow anyway. Let the AI take them.

But there are also tasks where flow is both possible and valuable: designing an architecture, debugging a subtle issue, implementing something that requires holding a complex model in your head. For these, the AI assistant might hurt more than it helps — not because it gives bad code, but because the interaction pattern itself prevents you from doing your best thinking.

I've started recognizing the difference in my own work. When I notice that I'm prompting and re-prompting and the quality of my thinking is getting worse with each cycle, I stop. I close the assistant. I start typing myself. Not because I'm faster — I'm not. But because the engagement is different. I'm inside the problem instead of orbiting it.

The other thing I've started doing is batching my AI interactions. Instead of keeping a chat window open all day, I collect the tasks that are suitable for AI help and handle them in a dedicated block. That way, the rest of my day stays available for the kind of deep, uninterrupted work that produces flow — and growth.

## The quiet trade

This is not really about productivity. The metrics might be fine. The velocity chart might even look better than before.

It's about something harder to measure. It's about the texture of your working day. The difference between a day where you were fully present in the work and a day where you supervised machines. Both produce output. Only one produces satisfaction.

Understanding the mechanism — knowing that it's the loop itself that breaks the state, not some vague unease about AI — makes it easier to act. You're not fighting a feeling. You're managing an interruption pattern.

And once you see it as an interruption, you can start treating it like one.

{% include connect-deeper.html %}