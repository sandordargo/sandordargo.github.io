---
layout: post
title: "Trip report: C++ On Sea 2025"
date: 2025-7-2
category: books
tags: [cpp, conference, cpponsea, tripreport]
excerpt_separator: <!--more-->
---
Another year, another [trip report](https://www.sandordargo.com/tags/tripreport/) from [C++ On Sea](https://cpponsea.uk/)!

First, a heartfelt thank-you to the organizers for inviting me to speak, and an equally big thank-you to my wife for taking care of the kids while I spent three days in Folkestone — plus a few more in London to visit the Spotify office and catch up with some bandmates.

> If you have the chance, try to arrive early or stay around Folkestone for an extra day. It's a lovely town and it's worth exploring it. The conference program is very busy even in the evenings, so don't count on the after hours.
>
> This year, I arrived an half a day in advance and I had a wonderful hike from Folkestone to Dover. It was totally worth it.

![Me from Folkestone to Dover]({{ site.baseurl }}/assets/img/cpponsea2025-folkestone-to-dover.jpg)

In this post I'll share:

- Thoughts on the conference experience.
- Highlights from talks and ideas that resonated with me.
- Personal impressions, including reflections on my own sessions — both the main talk and the lightning talk.

*I'll update this article with links to the recordings as soon as they become available.*

## My favourite talks

Before diving into individual sessions, I want to highlight the overall mood at the conference — full of enthusiasm and excitement. C++ On Sea 2025 took place just after the WG21 meeting in Sofia, Bulgaria, where several game-changing proposals were discussed and adopted. Herb Sutter's keynote focused entirely on a few features of C++26, and Timur Doumler's talk spotlighted an exciting upcoming feature: contracts.

With that context in mind, here are my three favourite talks, presented in the order they were scheduled.

### Three Cool Things in C++ (26) by Herb Sutter

Herb is part of that rare breed of presenters. He is extremely knowledgeable, his presentation style is entertaining and if this wouldn't be enough, he is also always enthusiastic.

As I mentioned earlier, C++ On Sea 2025 took place right after a particularly productive committee meeting, which added even more excitement to the atmosphere — both for Herb and for the audience.

Herb's keynote focused on three major features coming in C++26:
- [Erroneous behavior](https://www.sandordargo.com/blog/2025/02/05/cpp26-erroneous-behaviour)  
- Reflection  
- `std::execution`  

Thanks to the new erroneous behavior, uninitialized variables will no longer result in undefined behavior by default. You can still opt in to keep variables uninitialized, but now, as Herb put it, *"the sharp knife is in a drawer by default."* (If you're curious to dive deeper, I wrote more about [erroneous behavior here](https://www.sandordargo.com/blog/2025/02/05/cpp26-erroneous-behaviour).)

Now, reflections — this one is huge. We can't possibly cover it in just a few paragraphs. But according to Herb and pretty much everyone I've talked to, it's set to be a real game changer for C++. We don't yet fully grasp all the possibilities it will unlock.

In a nutshell, reflections will provide a standardized, generalized API to a language-level abstract syntax tree (AST). In C++26, we'll be able to reflect on types, functions, and parameter lists. The rest will follow in C++29. What's especially exciting is that, unlike in many other languages, C++ reflection will be entirely compile-time, meaning there's **no runtime overhead**.

This article isn't the right place to go into too much detail, but reflection is expected to massively simplify things like creating language bindings. Just as I did for [concepts](https://www.sandordargo.com/tags/concepts/), I plan to dedicate an entire blog series to reflections — both to learn it myself and to share my insights with you.

### Software Engineering Completeness Pyramid: Knowing when you are Done and why it matters by Peter Muldoon

Peter is another highly energetic speaker, and his talk focused on the hardcore discipline of software engineering — without touching a single line of code. He opened with a question we all hear from our managers:

> **"Are you done yet?"**

But how can we *really* know? And, once we're "done", how can we be sure the software actually delivers value?

Peter argued that software only brings value when it is **available, usable, and reliable — all at the same time**. Large and slow releases rarely meet that standard; small, incremental changes that satisfy all three criteria do far better.

To clarify what it really takes to deliver valuable software, he introduced the **Software Engineering Completeness Pyramid**, a four-level hierarchy reminiscent of Maslow’s. As with Maslow, you usually need to satisfy each level before you can meaningfully think about the next.

![Software Engineering Completeness Pyramid]({{ site.baseurl }}/assets/img/software-engineering-completeness-pyramid.png)

Where do *you* operate?

- Are you simply shipping features and squashing bugs?  
- Have you moved up a level to caring about codebase health — an essential step toward senior-engineer territory?  
- Do you think like a systems engineer, considering how each change fits the broader architecture and influences system stability?  
- Or have you reached the summit, weighing every decision against business goals and market positioning?

That final tier may seem distant, but we should all keep the business context in mind. After all, there's no point in delivering features the business — and its users — don't actually need.

### Why Software Engineering Interviews Are Broken – and How to Actually Make Them Better (Kristen Shaker)

I was deeply moved by Kristen's talk. More on that at the end.

Most of us would likely agree that the software engineering interview process is fundamentally flawed. We're asked to solve LeetCode-style problems, analyze runtime and memory complexities — the infamous Big O — not because they're part of our daily work, but because that's how the industry has standardized hiring.

As Kristen explained, these kinds of interview questions are bad for both developers and companies. Candidates often need to spend months preparing just to stand a chance. A full interview process can take up to 8 hours — just for a single position! Yet if we don't change jobs frequently, we risk falling behind in compensation.

It's a bad deal for companies too. They don't necessarily end up hiring the engineers who would perform best on the job. And because the interview process is so exhausting, many people stay in roles they're unhappy with — quiet quitting instead of seeking new opportunities. This system also tends to favor hiring the same kinds of people over and over, while diverse skill sets would lead to stronger, more balanced teams.

**So what can be done instead?**

Kristen proposes that we move away from LeetCode-style questions and ask better ones — questions that signal whether someone will actually succeed in the role. Questions that have multiple valid answers. That allow candidates to demonstrate different skill sets. That start simple, but can go deep. That value real-world experience.

Examples of such questions include:
- *What's your favorite feature of C++ (or another language)?*
- *Review this piece of code.*
- *"Yap" about a past project you worked on.*

In my view, we do fairly well at Spotify — but that's clearly not the industry average.

**Why was I so touched by this talk?**

When I saw Kristen was speaking, I immediately remembered her lightning talk at [C++ On Sea 2022 about querying the Clang AST](https://www.youtube.com/watch?v=2LOxsfpCCyI). Being curious, I googled her name and found a real estate agent with the same name. She looked familiar, but I thought, "That can't be her."

[It *is* her](https://www.shaker.nyc/). She was so fed up with the software industry — especially the interview process — that she left engineering and became a real estate agent instead.

Good luck, Kristen.

## My favourite ideas

Now let me share three interesting ideas from three different talks.

### The embedded world needs more C++ (Marcell Juhasz)

Marcell Juhasz gave a talk with the title [*"Balancing Efficiency and Flexibility: Cost of Abstractions in Embedded Systems
"*](https://cpponsea.uk/2025/session/balancing-efficiency-and-flexibility-cost-of-abstractions-in-embedded-systems). He essentially took an embedded project written in good-old C and started to add layers of abstractions in C++, making the code more readable, testable and maintainable. Goals that I deeply care about.

But Marcell didn't only made the code better, but after each step he made some measurements. Mostly about binary size as that's what mattered him the most. If he found any increase, he checked where it comes from and tried to get rid off the increase while keeping the benefits of the new layer of abstraction.

The outcome?

One cannot justify using plain C because of worse performance and bigger binaries. When applied cautiously, modern C++ features are perfect for the embedded world.

### Compile-time debugging (Mateusz Pusz)

Mateusz Pusz gave a talk on features that help us write great C++ libraries — both existing ones and those coming with C++26. While the talk was informative and full of useful insights, I want to focus on one specific feature that really stood out to me and that I'd love to explore further: **compile-time debugging**.

Debugging `constexpr` — let alone `consteval` — functions can be quite a challenge. Traditional debugging tools are mostly useless in this domain, making issues hard to trace and fix.

This is where [P2758](https://wg21.link/p2758) comes into play. It introduces new ways to emit messages at compile time — not just plain output via `std::constexpr_print_str`, but also **compile-time warnings** using `std::constexpr_warning_str` and even **compile-time errors** via `std::constexpr_error_str`.

These additions go far beyond simple "printf-style" debugging at compile time. They allow library authors to:

- Communicate clearly what's going wrong (or right) at compile time.
- Surface warnings proactively before they become runtime issues.
- Provide error messages that are both specific and user-friendly.

I believe these features have the potential to **significantly improve the developer experience in C++**, making compile-time diagnostics clearer and more actionable than ever before. If used well, they could help us build libraries with error messages that are both meaningful and educational — something C++ has long needed.

### Difficiult test? Think about your design! (Björn Fahller)

Björn gave an excellent talk on software testing. It was both insightful and educational, covering a big variety of testing strategies — from different types of tests and their purposes, to comparisons of various unit testing frameworks.

But there's one key takeaway I want to highlight from his presentation:

*If you find that something is extremely difficult to test, and you just can't figure out how to approach it — don't keep banging your head against the wall.*

Instead, **pause and reflect on your API design**.

If testing a component is overly complicated, the problem might not lie in your testing skills or the framework you're using — it could be a sign that your design needs improvement. Clean, testable APIs usually indicate a well-thought-out architecture. On the other hand, if you're struggling to test something, it may be tightly coupled, doing too much, or hiding behavior behind obscure layers of abstraction

## My talks

Finally, let me share my contributions to the conference.

My time came very quickly this year. I had my slot on the *C++ Fundamentals* track right after [Herb Sutter's keynote about *Three Cool Things in C++*](https://cpponsea.uk/2025/session/three-cool-things-in-cpp). That's both terrifying and calming at the same time! 

![C++ On Sea 2025 schedule]({{ site.baseurl }}/assets/img/cpponsea2025-schedule.png)

I was even more surprised — and humbled — to see Jason Turner attending my talk. I had a brief discussion with him the next day, and he mentioned that there was some overlap between our topics and he wanted to refresh his notes on namespaces. What a pleasant and unexpected surprise!

It's no secret that I talked about **namespaces**. What they are, how they work, and what best practices you should follow when using them. I've already covered some of these topics [on the blog](https://www.sandordargo.com/tags/namespaces/), and more may come. Of course, I'll share the video once it becomes available.

While I had my talk on the first morning, a lightning talk awaited me later that evening. I try to grab these opportunities to speak — it's a great way to fight stage fright! I presented a technique for **designing your workweek**, something I've written about on [The Dev Ladder]().

And finally — no clicker issues this time! After [Meeting C++ last year](), I bought a [Logitech Spotlight](https://amzn.to/44pbmUy) and it was one of my best conference decisions. No glitches, just smooth transitions. Same goes for my presentation overall — though next time, I'll aim to highlight key points in code examples more clearly.

## Conclusion

C++ On Sea was a great experience in 2025 as well as [any other year](https://www.sandordargo.com/tags/cpponsea/). Three days packed with inspiring talks about various topics, including not just C++26, but embedded, testing, engineering interviews and many more.

The best we can do is to spread the word - share the videos, tweet your favourite insights, write about what you learned - so that maybe even more people join next year.

I hope to be back to Folkestone in 2026!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
