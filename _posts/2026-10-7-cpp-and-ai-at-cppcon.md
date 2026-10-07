---
layout: post
title: "C++ and AI at CppCon: We have to use agents - more or less?!"
date: 2026-10-7
category: dev
tags: [cpp, ai, cppcon]
excerpt_separator: <!--more-->
---
A few weeks ago, [I went to CppCon 2026](https://www.sandordargo.com/blog/2026/09/23/trip-report-cppcon-2026). Unsurprisingly at a C++ conference this year, one of the most prevalent topics was AI and how we should use it — better — in our industry.

There were ten talks mentioning AI or LLMs in their titles - excluding pre/post conference workshops:
<!--more-->

- *AI is UB with Better PR* — Matt Kulukundis, Andy Soffer
- *C++ in an AI World* — Jody Hagins
- *C++ in the Age of AI: How Visual Studio Is Evolving* — David Li, Augustin Popa
- *Ensuring Code Quality in the Age of AI: More Code, Less Engineering!* — Peter Muldoon
- *Leveraging LLM to Generate Unittests for Notifiers in Taskflow* — Snikitha Siddavatam
- *One C++26 App Three Ways: By Hand, By AI, and Together* — Mike Spertus
- *The Death of Flow? How AI Changed Programming — and How to Get Joy Back* — Sandor Dargo
- *What's New for C++ in Visual Studio Code: C++ and AI Tooling for Your Build, Your Code, Your Workflow* — Sinem Akinci, Ben McMorran
- *When Air Gaps Learn: AI, Data, and the New C++ Attack Surface* — Matthew Butler
- *You're absolutely wrong! How to get agents to solve complex problems with complex code.* — Greg Law, Mark Williamson

Most probably AI was mentioned in even more talks, in one way or another.

Did I attend all of them?

Obviously not.

Will I watch all of them once they are released?

Probably not.

But I've seen enough that I can share some ideas about the general trend and, of course, my feelings are taking over a bit.

## General mood

If you talked with top management at almost any company about AI a few months ago, you could slowly notice tears of joy forming in their eyes as they started fantasizing about 10x developers hanging from the Christmas tree.

That optimism has probably faded a bit by now - and you could never really feel it among developers in the first place. At CppCon, I certainly didn't see general happiness or even excitement about AI in the C++ community.

On the contrary, most people seemed bothered by the fact that our job has changed completely. We're trying to adapt to it and to the new expectations from our management - while still trying to figure out what those expectations actually are. At the same time, we know and acknowledge that AI is not going away.

Most of the people I talked to barely write code with their hands anymore. Some more experienced folks were shaking their heads in disbelief and discontentment.

All in all, we've all accepted that AI is here to stay and will only become more relevant. The question is not whether to use it, but how to use it without losing our minds - or our craft.

## What is the problem with AI?

But what is the problem with AI?

First of all, does it even make sense to say anything like that? AI is just a tool, there is nothing wrong with it. It's not a human being. It's like there is nothing wrong with a gun - right, I've just spent a week in the States -, the problem is with people who (mis)use it. 

AI can be used well and also misused. Just like a gun. But okay, I stop here with the analogy.

So there is nothing wrong with AI, the problem is with expectations and usage. Let's face it, many of us - including me - have been misusing and probably overusing it. And there were clear and loud expectations to overuse it. But let's not blame anyone. We're all in this together, still learning.

What are the problems? Let's enumerate a few of them:
- Their results are nondeterministic, even if you turn the temperature down. (For the uninitiated: LLMs have a parameter called *temperature* that controls how random the output is. Even at the lowest setting, the output is not fully deterministic.)
- It strengthens cognitive surrender as Peter Muldoon said. We're increasingly ready to accept faulty AI reasoning - and we become more confident while doing it.
- It's not as smart as we often think it is.
- AI is leading to more coupled, copy-paste code - in other words, quality erosion.
- We didn't observe a significant performance boost in task completion over the last year.
- We are drowning in code reviews of pull requests growing in size, shrinking in quality.
- Churn is going up. We don't just create more PRs, but we have to touch the same files over and over again.
- That's because the number of incidents tripled since we started to rely on AI for generating code.

Does AI have any positive effects?

I've seen different interpretations - even within the same talk by Peter Muldoon, depending on the angle. AI-powered tools have a positive impact on QAs, DevOps, EMs and above all experienced software engineers. They say alcohol reinforces personality traits. AI does the same with experience and productivity.

But it's not all pink clouds and rainbow dust for experienced engineers either. As Peter said, _"junior engineers feel productive from day one. Senior engineers go from whiteboard sketch to working prototype before lunch. Yet, **somebody still has to catch the security gaps, compliance risks, and architectural conflicts buried in years of tribal knowledge.**_"

In other words, mid and senior engineers are buried under that load, burning out.

## How to use AI differently

But what comes of this?

First of all, we should not rely on it as much. But probably we should also use it - even - more.

Does it sound contradictory?

Maybe. Let's clear that up.

### Don't surrender your judgment

Generating code has become easy. It's very easy to become lazy and just accept it. That's cognitive surrender in action.

But we cannot give up on the task of judgment, which has quietly become a more important part of our job than writing code.

At the same time, as I said in my talk, accepting the machine's diff feels like signing someone else's work. And in our hurry to deliver more, fixing more bugs than ever, we lose our touch with the codebases we work with. Using AI like this works on codebases we got familiar with before the age of AI, but it's much harder with new codebases.

We cannot exercise judgment properly if we cannot build the same thing by ourselves. This doesn't mean that we have to build everything by hand, but it does mean that we have to keep some manual zones in order to *"invest experience in the strength of your opinion,"* as Alexandrescu ended his closing keynote.

### Use agents smarter

While we shouldn't rely on AI to judge, we should also use it more. But that usage might look a bit different compared to what we're used to.

Yes, in the beginning of a bigger piece of work, we should use agents for brainstorming, rapid prototyping and experimentation.

But even when we do so, it's worth limiting the agents' contexts and using them for microtasks - both for getting better answers and to keep the costs lower and the responses faster.

As Jody Hagins suggested, you might even want to experiment with breaking down tasks into microtasks and calling your agent in headless mode with small tasks in loops. That will help keep the contexts small and the agents focused.

>A small detour. If you want to build such a harness as Jody suggested, you'll probably use an AI. And this points in the direction that Peter Muldoon suggested: use nondeterministic agents to generate scripts that will provide you with deterministic behaviour.

Alexandrescu got some strong applause for stating on a slide that we should *"stop using an agent for coding."* But to the big surprise of people, the emphasis was on the singular form - he continued by saying that we should use more than one agent.

Let that sink in.

We should use more than one agent in our daily developer workflow: a planner-reviewer and an executor, from competing vendors. After all, you are not supposed to approve your own code — for checking generated code, you should use an agent from another vendor. And since implementation is cheaper than planning or verification, we can use cheaper models for the coding part.

### Our job has changed

If you really think about all this, it's evident that our job has completely changed. It didn't lose the software engineering part - we have to stay competent software engineers - but we have to earn that competency in much less time, while we also become good managers. Managers of soulless agents.

But that's not the only new trait we have to pick up. Though agents are soulless, you might feel sometimes that they act like human beings. They are often lazy but try to hide their laziness, and they act very confidently. That's why I think it's worth questioning the agents' "thoughts" as if you were a detective.

I often ask the agent to defend its position and tell me why its fix will work. Where the explanation is weak, that is where I dig deeper.

Once I'm happy with the outcome, I ask it to do the inverse: tell me why the solution wouldn't work. But the agent wants to satisfy me, so it makes things up that would actually not be a problem. There, my job is to filter out the false claims.

Engineers with management and detective traits. That's what we've become.

## Closing thoughts

Even when you follow a new working model with several agents, you still have to be extra cautious in certain areas. When you touch production-critical code. When it's safety- or security-critical. When regulatory compliance is involved. When the consequences are irreversible. When correctness is difficult to verify.

In other words, more frequently than not.

That's why the most critical skills going forward will be critical thinking, architecture design and domain expertise. Not prompt engineering. Not knowing which model is cheapest. The same skills that mattered before AI — they just matter more now, because the cost of not having them is higher when code arrives faster than judgment.

{% include connect-deeper.html %}
