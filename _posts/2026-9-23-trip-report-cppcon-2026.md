---
layout: post
title: "Trip report: CppCon 2026"
date: 2026-9-23
category: books
tags: [cpp, conference, cppcon, tripreport]
excerpt_separator: <!--more-->
---
[Last year](https://www.sandordargo.com/blog/2025/09/24/trip-report-cppcon-2025) I wrote that a dream came true by attending and speaking at CppCon. And here I was again! Someone is living the dream, apparently.

21 LinkedIn posts shared, 24 sessions attended, 3 talks given.

Those are just numbers, though. The human connections are what make a conference special, and CppCon being the largest C++ conference, there is no shortage. If you just want to wind down with a hot cup of coffee, good luck — someone you've been meaning to talk to will inevitably pop up next to you.

I didn't even have to reach the venue to start catching up with old friends. I planned to work during my layover at the Munich airport, but as I approached a table I bumped into a fellow presenter, [Alex Dathskovsky](https://cppcon2026.sched.com/speaker/calebxyz). We ended up taking the same flight to Denver — so much for my productivity plans.

> ![Meeting Friends At The Airport And Having A Beer]({{ site.baseurl }}/assets/img/cppcon-2026-muc-airport-alex-and-me.jpg "Meeting Friends At The Airport And Having A Beer")

Enough gossip, let's talk about the conference!

I know that some people find the venue a big, luxurious cage, but I still like it. It is spacious, comfortable, and with food delivery services you have plenty of options to eat. To be fair, I didn't experience endless queues even in the hotel restaurants this year.

Sure, you don't have the proximity of a big city or the sea, but you do have a wonderful view of the Rocky Mountains instead.

>Speaking of which. In my opinion, if you get the chance to visit a place at the other end of the world, you should take advantage of it.
>
>Last year, I visited the Rockies for a day. This year I planned well ahead and then did completely different things. I visited the Forney Museum of Transportation to see the finest cars from earlier decades, and the Clyfford Still Museum. I'm not particularly fond of modern art, but I loved it.
>
> ![Forney Museum of Transportation, Denver]({{ site.baseurl }}/assets/img/cppcon-2026-denver-forney.jpg "Forney Museum of Transportation, Denver")

## Favourite talks

Here are my favourite talks, in chronological order.

### [Towards a complete contract-assertion facility for C++ by Timur Doumler](https://cppcon2026.sched.com/event/2RT7w/towards-a-complete-contract-assertion-facility-for-c++)

Earlier this year at [ACCU On Sea 2026](https://www.sandordargo.com/blog/2026/06/24/trip-report-accu-on-sea-2026), I attended Timur's 90-minute talk focusing solely on contract assertions for virtual functions — a feature that didn't make it into C++26. I consider Timur one of the best technical speakers nowadays, and I'm particularly interested in contracts, so I was eager to attend his one-hour talk focusing on the other five extensions that will hopefully ship with C++29.

Obviously, a trip report is not the place to get into long technical details — I'm preparing a long series on contracts — but I do want to at least list them here.
- *Pre and post assertions on virtual functions (P3097)*
- *User-defined error messages (P3099)*
- *Explicitly triggered contract violations (P3290)* — integrate existing contract-checking facilities by being able to directly invoke the contract-violation handler
- *Postcondition captures (P3098)* — compare the old values of a parameter with the new values in a postcondition
- *Assertion-control objects, a.k.a. labels (P3400)* — fine-grained, composable, in-source control over contract-assertion properties like allowed semantics, violation handlers, and mutual exclusivity
- *Implicit contract assertions (P3100)* — classifies undefined behaviour in the language by checkability and cost, and proposes turning them into compiler-generated "implicit contract assertions"

### [Ensuring Code Quality in the Age of AI: More Code, Less Engineering! by Peter Muldoon](https://cppcon2026.sched.com/event/2RT4q/ensuring-code-quality-in-the-age-of-ai-more-code-less-engineering)

Peter has already given some excellent talks over the last few years on engineering practices. In this talk he was on a quest to find what happens to code quality in the era of AI.

According to several different studies, it's not simply that quality is going down, but even our productivity is not as high as we tend to perceive it. True, we can have the AI generate code for us en masse, but what actually gets merged to master is a different question.

> ![Peter Muldoon at CppCon 2026]({{ site.baseurl }}/assets/img/cppcon-2026-peter-titianc.jpg "Peter Muldoon at CppCon 2026")
> 
The worst news is that AI is causing cognitive surrender. We are ready to accept faulty AI reasoning and while we do it we also become more confident.

While juniors are shipping faster than ever and architects are architecting with less effort, the people in between are drowning. Those who have to catch security gaps, compliance risks, architectural conflicts are burning out.

We have to stay in control and own the final decisions. A way to do that is by not just handing over thinking to AI, but quizzing and challenging it on the code it generated. Ask it to point out tradeoffs and challenges. Request alternatives. Also challenge your colleagues to explain what they are delivering.


### [When Zero-Cost Abstractions Aren’t Zero-Cost by Steve Sorkin](https://cppcon2026.sched.com/event/2RT6r/when-zero-cost-abstractions-arent-zero-cost)

The goal of abstractions is to increase readability and maintainability. But at what cost?

Ideally, an abstraction is zero cost, in the sense that the compiler can see through it and remove all the runtime overhead. Yet, it should allow you to write highly expressive code, expressing what you want to achieve instead of focusing on the how.

And that often works, but not every time. Often the abstractions are not transparent, the concrete types are hidden and the control flow is difficult to analyze.

In his talk, Steve showed a couple of examples where code with a higher level of abstraction led to slower code. `std::function` was slower than a template, manual loops faster than the ranges version, or a raw loop faster than the one built up from independent reusable stages.

These were not surprises for experienced engineers. But the goal of the talk was not to surprise people but to make a point.

Steve's point was not to avoid higher-level abstractions. They can improve clarity and reusability. But beware that certain techniques will always come with real runtime overhead. That's not necessarily a problem — those costs might be acceptable on a non-hot path where your main goal is maintainable code. On the hot path, though, you probably want to optimize for performance. Understand costs and benefits and reason about them.


### [The Biggest Misconception of Computer Science by Alex Dathskovsky](https://cppcon2026.sched.com/event/2UEbC/the-biggest-misconception-of-computer-science)

What can it be?

If I could only share one slide from the talk, it would be this:

> ![Theory vs practice]({{ site.baseurl }}/assets/img/cppcon2026-alex-theory-practice.jpg "Theory vs practice")

Yes, practice and theory differ in CS as well.

At the university, we learn about algorithms and spend a lot of time understanding algorithm complexity and the big O notation. We spend so much effort on this that we even build coding interviews around it...

The problem is... practice is different...

The big O notation of an algorithm doesn't take into account the physical reality of computers, or more specifically processors. Thanks to pipelines, parallelization, branch prediction, cache locality, you might end up with a faster solution even if there is another that is better in terms of big O.

As Alex put it, the world is dirty, not clean. You need something that works for you. Use CPU and memory to your advantage and be a better programmer.

### [Your Next 20 Days of Systems Engineering by Andrei Alexandrescu](https://cppcon2026.sched.com/event/2RT7V/your-next-20-days-of-systems-engineering)

When Andrei jumped in to give the final keynote due to Herb Sutter's illness, I told myself that even though his *Your Next 20 Weeks of Systems Engineering* was an excellent talk at [ACCU On Sea](https://www.sandordargo.com/blog/2026/06/24/trip-report-accu-on-sea-2026), he cannot make it to this list with the same talk.

Well, some people didn't really like his talk...

> ![Andrei Alexandrescu at CppCon 2026]({{ site.baseurl }}/assets/img/cppcon2026-andrei-diarrhea.jpg "Andrei Alexandrescu at CppCon 2026")

But anyway, he changed the title to *Your Next 20 ~~Weeks~~ Days of Systems Engineering* and earned his place with a new talk. And no, he didn't only earn his place by quoting me twice(!!!), but by being entertaining and also relevant.

It's not easy to highlight points of this 90-minute talk, but if I absolutely must — and yes I must — here are two:

- Stop using an agent for coding. Period. Use at least two, ideally from different providers. Nobody should review their own code.
- Invest experience in the strength of your opinion. Don't make a strong opinion when you have no experience, in time you'll regret it.

## Favourite ideas

Here are my favourite ideas, again in chronological order.

### Profiles are coming (Bjarne Stroustrup)

Profiles are coming with C++29. Profiles can restrict what you can do in a program at compile-time through static analysis.

You might be able to finally restrict the usage of certain features in a codebase without removing them from the language.

Bjarne presented the first two profiles, *["allowing people to eliminate initialization errors and dangling pointer errors through simple static analysis"](https://www.linkedin.com/feed/update/urn:li:activity:7506077633705840640/)*.

But more importantly, profiles are also a framework, so we can expect many extensions.

### Have One Goal (Jody Hagins)

If there is one thing — though there were many others — to retain from Jody Hagins' talk on [AI in a C++ world](https://cppcon2026.sched.com/event/2RT5Q/c++-in-an-ai-world), it's that you should break down into small steps what you want to achieve and always give one small, very specific goal to your AI agent.

If you ask them to do many different things, they mess up. Make sure that the contexts are kept clear and small. If there are five steps to perform, that should mean five calls to the AI with clear contexts. Something to try.

### AI and undefined behaviour have identical failure modes (Andy Soffer)

They might do bad things, we can't control those things, and they're hard to detect.

That doesn't mean we can't rely on these tools, we just have to be cautious. Both with undefined behaviour and with AI.

Some very interesting thoughts and very nice slides by Andy Soffer.

> ![AI and UB]({{ site.baseurl }}/assets/img/cppcon-2026-brontosource-nasal.JPG "AI and UB")
> 
### VTables generated with reflection (Ryan Keane)

Ryan Keane brought the best surprise of this conference! His first appearance at CppCon — or at any C++ conference if I'm not mistaken — and he was honoured with a standing ovation. Something you rarely see.

He was invited to the conference after he shared [his library `rjk::duck`](https://github.com/RyanJK5/rjk-duck), a header-only library based on C++26 reflection that helps use unrelated types with the same interface as if they were inheriting from the same base class.

A great idea on how to use reflection and a fantastic implementation.

### The Library Author's Credo (Jon Kalb)

Jon Kalb started off Friday morning with a talk on how to write better C++ application code. His point is that application code doesn't have to be written using the "just get it done" mentality, but it can and should benefit from how library authors write code.

He went through different techniques and practices, but what I want to share is what he calls the library author's credo:

- I will design clean interfaces and hide my implementations
- I will prefer value types and composition over inheritance
- I will build code from small, composable, and testable parts
- I will strive for zero-cost abstractions
- I will create a valuable asset, not a liability

### Inlining might optimize more than you'd think (Matt Godbolt)

Inlining is not just about removing call overhead. The real payoff is what happens next: the compiler can propagate constants, eliminate dead branches, and sometimes throw away entire function bodies.

Matt wrote a [great blog post](https://xania.org/202512/17-inlining-the-ultimate-optimisation) on this — well worth a read.

## Five talks I wanted to go to...

... but didn't manage to due to scheduling conflicts. Can't wait for the recordings.

[Are You Smarter Than A Branch Predictor](https://cppcon2026.sched.com/event/2RT7J/are-you-smarter-than-a-branch-predictor) by Michele D'Souza. Last year I went to Michele's talk and she was such an energetic and engaging presenter that I really wanted to make it. But I decided to go to another talk that helped me prepare for one of my own. Yes, I'm selfish.

[Code You Didn't Write: Understanding Large and Unfamiliar Codebases](https://cppcon2026.sched.com/event/2RT88/code-you-didnt-write-understanding-large-and-unfamiliar-codebases). I wanted to attend because of both presenters, but it would have been very rude to my audience as I presented at the same time.

[Type Punning: the joke is on you, pun intended. Fixing UB of reinterpret_cast!](https://cppcon2026.sched.com/event/2RT6x/type-punning-the-joke-is-on-you-pun-intended-fixing-ub-of-reinterpretcast). I liked both the presenter and the topic, but I checked and had no practical current use for it — and I really wanted to be at Steve's talk instead.

[Ranges Without Compromises: Designing for Simplicity, Performance, and Composability](https://cppcon2026.sched.com/event/2RT5u/ranges-without-compromises-designing-for-simplicity-performance-and-composability) by Oleksandr Bacherikov. A topic I want to learn about more, but I really wanted to be at another talk. Wouldn't it be great if Oleksandr started blogging about this?

[Rule of 0,1,2,5,6,7,8,9,10?](https://cppcon2026.sched.com/event/2RT5D/rule-of-0125678910). Jason is a great speaker — it feels like a sin to skip his talk. But this time I decided to go for a speaker I didn't know instead, and it was not a bad choice.

## My talks

Well, this section will be longer than usual. I might have overcommitted this year by giving three talks, but I think it was worth it and I enjoyed every one of them.

But how three? Two of my talks got accepted. Later I realized that at CppCon there are open content sessions that are not recorded. I thought that it would be the perfect opportunity to give a dry run of my *Death of Flow* talk that I'll present in November at Meeting C++.

### [The Death of Flow? How AI Changed Programming — and How to Get Joy Back](https://cppcon2026.sched.com/event/2WOfl/open-content-the-death-of-flow-how-ai-changed-programming-and-how-to-get-joy-back)

I was very honest and clear. It was an alpha version and I was looking for feedback. I think the talk went well, though I found some parts where the transitions were not smooth enough and I still need more practice.

But the feedback was awesome. I had never had such a level of engagement — so many questions and a long discussion after that we almost missed the beginning of the next talks. I definitely missed my lunch — but hey, it was worth it.

I was more than surprised to see that Andrei Alexandrescu quoted from this talk — even twice — in his closing keynote. At the same time, remember what someone wrote about his earlier keynote?

But the best indirect feedback was Peter Muldoon's talk later that day that I already mentioned earlier. Our conclusions were very similar. Keep manual zones, interrogate your AI, *"don't just be a meat proxy"* as Peter said. The main difference between our talks is that his was about keeping workloads sane and quality high, backed up with data. My talk was more about feelings, based on impressions from people I talked to and my own.

Apparently where joy and code quality live are very close places. And that's good news.

### [The Clocks of C++: Knowing When (and Why) to Use Each One](https://cppcon2026.sched.com/event/2RT6W/the-clocks-of-c++-knowing-when-and-why-to-use-each-one)

My second talk is probably my least favourite one. I find the topic very useful, though. It's on clocks and time handling. We rarely think about how many different clocks we have at our disposal — until we pick the wrong one or try to match timestamps across systems.

The best part of this talk was the aftermath. A high proportion of the people stopped me in the hallways over the next days to say thank you and how useful they found it.

Apparently, I was not alone with my problems.

### [The Real Story of C++26: Beyond the Headline Features](https://cppcon2026.sched.com/event/2RT3x/the-real-story-of-c++26-beyond-the-headline-features)

This was the first time I gave this talk and I think it went quite well, despite two things.

I got a bit confused with my thoughts in the middle when I talked about function wrappers, but I managed to overcome it.

The other thing was that I started very fast. I was a bit more than excited, especially given the level of my audience — far above me! But seeing people you admire nodding and taking photos is quite reassuring, and I found my pace.

I decided not to talk about the big three of C++26 (so no contracts, no reflection, no sender/receiver model), but about a relatively small subset of the new features telling a story. How C++26 makes correct C++ easier to write and incorrect C++ easier to catch — both at compile and runtime.

Given my early fast pace, there was plenty of time for discussion, which this time I genuinely enjoyed - even though the C++ convenor asked me a really cheeky question!

When I hear kind words, I might think that people are just being nice because they are standing next to you. But when you see [such posts](https://lnkd.in/p/eQFx9qcY), you suddenly realize that they meant it. I cannot even describe the feeling.

## Conclusion

If last year was a dream come true, this year felt like home. Three talks, countless hallway conversations, and the realization that this community keeps pulling me back — not just for the C++, but for the people.

Over the last two years, I've had some difficult times both at home and at work. Sometimes I had to question myself, or others did. From this conference, I can definitely come away with strengthened confidence.

But I don't want to close with me. The conference had a couple of clear themes.

First and foremost, AI. It is changing how we write code, and the C++ community is trying to figure out what that means. Peter, Jody, Andy, Andrei, and I all circled the same questions from different angles. The answers are still forming, but the fact that joy and code quality seem to live in the same neighbourhood gives me hope.

On the language side, C++26 is at the centre of interest, but we already started to talk about C++29. There's a lot to look forward to.

Thank you to the organizers, the speakers, and everyone who stopped me in the hallway to chat. See you next year, hopefully!

{% include connect-deeper.html %}
