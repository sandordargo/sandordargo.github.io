---
layout: post
title: "Trip report: ACCU On Sea 2026"
date: 2026-6-24
category: books
tags: [cpp, conference, accuonsea, tripreport, cpponsea]
excerpt_separator: <!--more-->
---
Once again, I got the chance to come to Folkestone, UK. I think this was my fifth time, but the first one at ACCU On Sea. Yes, there is no more [C++ On Sea](https://www.sandordargo.com/tags/cpponsea/), there is no more ACCU in Bristol — they got merged into ACCU On Sea.

<!--more-->

The venue is the same as C++ On Sea, many organisers are the same, but of course there are also changes. The conference chair is no longer Phil Nash but Guy Davidson. The schedule is more packed — this year, the conference ran over four days (ignoring the two workshop days), including Saturday. Probably due to the fact that these two major C++ conferences were merged, I think there were more people than ever — around 400 C++ enthusiasts. It was sold out.

And yet, in general, familiar faces. This place holds a special place in my heart as it was the first in-person international C++ conference which I attended as a speaker.

In this post I'll share:

- Highlights from talks and ideas that resonated with me.
- Personal impressions, including reflections on my own sessions — both the main talk and the lightning talk.

*I'll update this article with links to the recordings as soon as they become available.*

## My favourite talks

Let me share with you my four favourite talks — one per day, in chronological order.

### The Next 20 Weeks of Systems Engineering by Andrei Alexandrescu

I think the first time I checked the schedule of [ACCU On Sea](https://accuonsea.uk/2026/schedule/index.html), the title of this talk was different. It was about the next 20 years. It turns out, I was not hallucinating as agents do. That's what Andrei originally had in mind. Then he went for 20 months, then 20 weeks, and stopped there.

Frankly, I think this was planned by Andrei, but it clearly demonstrates how rapidly our craft is changing — thanks to, or maybe better to say, because of AI.

Andrei is one of the most entertaining technical speakers and this occasion was no different. But to be fair, about 20 minutes into his keynote, while I was having a great time, I didn't understand where he was going. But hey, he still had 70 minutes left to connect the sometimes biblical, sometimes historical stories with the technical message he wanted to convey.

I must say he did it. By the end, his talk made sense as a whole.

He even got into making predictions — about the future. That's quite a difficult thing to do, not only because, well, it's about the future and therefore unknown, but also because we always try to adapt. That's not a new idea, the book [The Rational Optimist](https://www.sandordargo.com/blog/2020/02/12/the-rational-optimist) also discussed it.

It's worth mentioning that Andrei clearly admitted that he is wrong in 90% of his predictions. Partly because of that and due to the lack of space, I won't go through all of them, but I want to highlight his ideas about abstractions.

Andrei expressed his disagreement with Elon Musk who claims that the need for abstractions is going away — in the sense that AI will write machine code directly. Andrei thinks Musk is wrong. AI needs abstractions, and moreover, thanks to metacognition, it's going to develop its own without our intervention. Sure, it could write machine code directly, but that's not efficient enough.

> *What is metacognition? In short, it's the ability to think about one's own thinking. When applied to AI, it means the system can reflect on its own reasoning process, evaluate its approach, and adjust its strategy on the fly. Instead of blindly following instructions, an AI with metacognitive capabilities can recognise when an abstraction would help and create one — much like a human developer would when noticing repetitive patterns in their code. This is what makes it possible for AI to go beyond simply executing prompts. It can reason about the problem at a higher level and build the right tools for the job.*

On a slightly related note, Andrei predicts that prompt engineering is also going to go away. The AI will be able to refine the prompts we feed it on its own.

Where is this all leading? If you ask me, to the haven of project managers. Andrei predicts that projects are going to grow both in size and complexity thanks to better abstractions and because we'll need to hold fewer details about the systems in our heads. Yes, we'll have to keep focusing on the higher level abstractions.

How? We don't exactly see yet, as we're only in the *"bubblesort era of AI assisted coding"*. Maybe in 20 weeks, we'll see more...

### Modernizing Legacy Codebases without Stopping the World by Peter Muldoon

Peter's talks are regularly featured in my trip reports. He is an excellent speaker and our areas of interest often overlap. Code modernization has always been a pet peeve for me, so I was keen to see his brand new talk.

He started with a crucial question when it comes to modernizing legacy code. What is legacy code? Probably the most prevalent answer is code that has no tests, but Peter went further. Legacy code is code that is in production. As simple as that. Understandable.

When it comes to new features — features that we often learn about at conferences — the problem is that it takes years to get them in the compilers we use at work. But let's say you have them. Then what? Will the business pay for the modernization? Or would you take the risk of mass-updating the code? Both answers are probably no.

Basically you have two paths ahead. A risky, high-impact, difficult-to-reason-about refactor or even rewrite, or the path of small, mechanical fixes with localised reasoning, lower risk and effort.

In the vast majority of cases you'd go with the latter.

But how to do it? Peter walked us through two approaches. Either you use deterministic static analysis tools or non-deterministic AI-based ones. The problem is that based on his experience, neither works out of the box — and on top of that, people don't even trust AI. Who can blame them?

Even a seemingly simple change like replacing `typedef` with `using` doesn't work at scale.

But something that has worked out quite well so far for Peter is combining the two approaches. Fix the shortcomings of static analysis tools through AI-generated scripts, or even add new checks altogether. With that you solve the non-deterministic nature of AI and also the lack of trust in it, while at the same time you can capitalise on its productivity.

The takeaway is Muldoon's Law of Automated Refactoring: *"The more complicated/risky the change, the smaller the scale & scope of the refactoring must be."*

### Opening the Black Box: Legally Testing Private Members in C++ by Hubert Liberacki

One of the talks I enjoyed the most was a short one. While in certain rooms 90-minute talks were going on, in two of the rooms three 20-minute talks were stacked back-to-back.

One of them was about how to test the private parts of your classes.

Sounds like a horrible idea. Yet, I absolutely enjoyed this talk.

Hubert approached the question very pragmatically. He said — as one might expect — not to do this, and in fact he's not a heavy user of his own library either.

But if you absolutely need to, because you must test a legacy codebase without clear interfaces or you have to deal with third-party libraries that cannot be modified, there is a way beyond invoking undefined behaviour via the usual tricks that I don't even want to mention.

So how?

There is actually a loophole in the language:

> "Explicit instantiation definitions ignore member access specifiers: parameter types and return types may be private." ([CppReference](https://en.cppreference.com/w/cpp/language/class_template))

This means that we can explicitly instantiate a template that references private data — and the compiler won't complain about access. We can exploit this property to store a pointer-to-member for the private data during explicit template instantiation, making it accessible from the outside. There are a couple more issues to solve, but this is the essence of the solution.

You can check the example code [here](https://godbolt.org/z/qrcPnbz1W) and the corresponding library [here](https://github.com/hliberacki/cpp-member-accessor).

### Contract assertions on virtual functions by Timur Doumler

I don't want to rank my favourite talks, but if I had to pick a personal favourite from the conference, it would probably be Timur's — delivered right before lunch on a sunny Saturday morning.

As I'm preparing a series on contracts, the topic simplified my choice. When Timur Doumler, Jason Turner, Matt Godbolt, Roth Michaels and Francis Glassborow all give a talk at the same time, it's not an easy decision. But the subject matter made it easy.

Timur is definitely one of the most knowledgeable people to give a talk on this topic. And it's not the first time he's spoken about contracts either.

As he explained, the previous year at ACCU he talked about the proposal that was accepted into C++26, but he also covered how different languages such as D or Ada support assertions.

Later at C++ On Sea 2025, he gave a less technical talk on how to make stubborn engineers agree on something.

And now he came back to share how he actually made stubborn engineers agree on how C++ contracts should support pre- and postconditions on virtual functions.

In the first part of his talk, he focused on his methodology for finding consensus on technical design questions:

> 1. Establish shared concepts, definitions, terminology
> 2. Define and motivate the problem
> 3. List the requirements
> 4. List the known solutions
> 5. Arrange the known solutions in an evolution graph
> 6. Construct conformance table
> 7. Refine conformance table
> 8. Re-evaluate solution space
> 9. Arrange conflicting requirements into a decision graph
> 10. Approve the consensus solution

> ![Timur Doumler presenting P3097]({{ site.baseurl }}/assets/img/timur-doumler-p3807.png "Timur Doumler presenting P3097")
> _Timur Doumler's methodology for finding consensus_

Next he presented what contracts are in C++ and what ashtrays on airplanes have to do with contract violation handlers. I'll let you watch the recording to find the answer to that question.

Then the focus shifted to pre- and postconditions, which are already supported by a few languages — like the already mentioned D and Ada — and we saw why their approaches aren't quite good enough for C++.

There were a couple of deep questions that had to be answered in the design process, such as what should happen if only `Base::f` defines conditions but `Derived::f` does not, or the other way around. After reviewing all the requirements and the different existing solutions and proposals, we must say that P3097 provides the best option even with the trade-offs that had to be addressed.

Two particularly interesting requirements concern multiple inheritance: what happens if there are two mutually incompatible preconditions, or mutually exclusive postconditions? The solution designed by Timur Doumler, Joshua Berne, and Gašper Ažman handles even this.

And the best news came at the very end: just a week before Timur's presentation, P3097 was approved for C++29 — a moment the audience celebrated with enthusiastic applause.

## My favourite ideas

And now my four favourite ideas — again one per day.

### The difference between decorators and adapters (Klaus Iglberger)

*"The Design Patterns Guy"* gave another excellent talk on design patterns in which he covered 2.5 patterns: adapters, decorators and expression templates. Why don't they add up to 3? For that you'll have to watch his talk ;)

Now, let's just focus on the difference between decorators and adapters. Both are part of structural patterns — as categorised in [the GoF book](https://amzn.to/4abJXt1) — both are non-intrusive and widely used. But often confused. They are so often confused that even the standard library calls certain decorators adapters.

You can recognise the difference when you think about what they do. A decorator doesn't alter the interface, but it adds functionality or responsibility to the decorated class.

An adapter, on the other hand, doesn't add any functionality. It just alters — adapts — the interface to match the expected one.

When you think about it, it's rather simple. After this talk, I hope I won't mix them up again.

### Attack through keys (Kevin Carpenter)

I've wanted to go to one of Kevin Carpenter's talks for a long time and I finally made it to *O(1) or O(no-no-no): Mastering the unordered_map*. It was an excellent talk on when and why you should choose `unordered_map` over other data structures.

One idea that surprised me and I wanted to share is how the choice of your keys can lead to a security vulnerability. Imagine your server stores user-controlled input — form fields, JSON keys, HTTP headers — in an `unordered_map`.

If an attacker knows how you hash your keys (which is likely if your code is open-source), they can craft keys that are designed to collide into a single bucket. Once all your data ends up in one bucket, every insert and lookup degrades from O(1) to O(n). A few hundred kilobytes of crafted keys can pin a CPU core for minutes. This is known as a HashDoS attack and it's not theoretical — back in 2011, this class of vulnerability hit PHP, Python, Ruby, Java, Node.js, and ASP.NET all at once, resulting in emergency patches across the ecosystem.

Kevin demonstrated the effect live: with 8,000 crafted keys, a strong hash handled them in about 14 milliseconds, while a weak hash under attack took over 500 — roughly 37 times slower.

So what can you do about it? Use randomised, keyed hashing — something like SipHash with a per-process random salt — so that attackers can't precompute collisions. And as Kevin put it: *"Don't hand-roll crypto. Use a vetted keyed hash."*

### The Dusikova / Fahller trick to make `[[nodiscard]]` transitive

Björn Fahller gave a talk about the composability of callables. I won't go into all the details, but it was an excellent talk — the room was full and I think you need no bigger appreciation than having both Jason Turner and Matt Godbolt in the audience.

Among Björn's design goals, one stood out to me: making `[[nodiscard]]` transitive in his pipe-syntax-based library. The trick he used to achieve this is quite elegant.

The idea starts with a tag-like wrapper for functions that carry the `[[nodiscard]]` attribute:

```cpp
template <typename F>
struct nodiscard : F {} 
```

Next, you create a corresponding concept that can detect whether a function type inherits from this tag:

```cpp
template <typename F>
concept nodiscard_function = requires {
	[]<typename T>(nodiscard<T>*) {} (
		std::declval<std::remove_cvref_t<F>*>()
	);	
};
```

Once you have a tag with the right semantics and a concept to match against, you can use compile-time conditionals to branch on it. The full implementation details are beyond the scope of this article, but the key insight is that you can decide with an `if constexpr` how to handle a given type:

```cpp
if constexpr (nodiscard_function<F>) {
	// return a function wrapped into nodiscard
} else {
	// return a function without nodiscard
}
```

And why do I call this the Dusikova or the Fahller trick? As Björn explained, he saw this technique in Hana Dusikova's CTRE library. But as we discussed during the *hallway track*, techniques and tricks are often not named after the first person to use them, but after someone who rediscovers and popularises them — so this might just end up being the Fahller technique.

### The paradox of effort (Yvonne Rogers)

Yvonne Rogers, a professor of Interaction Design at UCL, gave a keynote on what AI is doing to us and for us. Her talk touched on something I've been thinking about a lot lately — something she called the paradox of effort.

The idea is so simple, you probably thought about it: when we make the effort ourselves to perform a task, it can be more rewarding than delegating it. Think of cooking a meal from scratch versus ordering takeaway. Or assembling that IKEA shelf — yes, the result might be slightly crooked, but you feel a certain pride in it. That's sometimes called the IKEA effect, and it applies far beyond furniture.

Now apply this to AI-assisted work. Generative AI is remarkably good at amplifying our cognition — writing, analysing, coding. But by reducing the cognitive effort needed, it may also reduce the satisfaction we get from the work. We no longer tell the computer what to do step by step; we tell it what output we want. That's powerful, but something gets lost in the process.

So should we design AI tools to reduce human effort or to increase it? Rogers argues for the latter. Increasing cognitive effort — involving deeper and more challenging thinking — leads to more engagement, more fulfilment, and ultimately more pride in our work. The goal shouldn't be to eliminate effort, but to redirect it toward systematic reasoning, reflection on goals, and intentional decision-making. Though the results of research are a bit more nuanced.

## My talks

And there are more opportunities than before to give lightning talks. As you can imagine, I proposed more than one. But luckily there were so many people interested in giving lightning talks that each person could only go once. That's a great thing!

I talked about clocks and testing. I recently wrote a series on clocks and I wanted to share one concept that fits five minutes. So I talked about why calling `system_clock::now` directly in your production code is not the best idea when it comes to testability and what to do instead. (Read more on it [here](https://www.sandordargo.com/blog/2026/01/28/clocks-part-9-once-more-about-testing).) This lightning talk was important to me, because in the last two years I didn't give any new talks on C++ — I was focusing more on higher-level principles or soft skills. Though for this year I also proposed a full technical talk (on clocks), but that's not what was accepted.

> ![Me at ACCU On Sea 2026]({{ site.baseurl }}/assets/img/sandor-lt-accu-on-sea-2026.png "Me at ACCU On Sea 2026")
> _Me at ACCU On Sea 2026_

My main talk was on code reviews. It was a polished, slightly reworked and updated version of the talk I gave last year at Meeting C++. I talked about how code reviews can help build better code and stronger teams at the same time. No code, no processes — just how to write code review comments that won't make people defensive and that won't get ignored.

## Personal impressions

It's always great to come here. As I shared last year, I always enjoy Berlin, not to mention Denver. But even though I was born in a big city, deep in my heart, I'm not a big city guy. I enjoy the calm of a small town in the south of France and I have the least intention to swap it for the hustle and bustle of a big city. Even though from time to time I do enjoy that.

So in this coastal Kentish town of Folkestone I almost feel at home, even though I'm not a local in any sense. But I already have my favourite places, including a second-hand toy and bookstore — to the biggest delight of my kids.

I'm really happy that the merged conferences were kept in Folkestone instead of Bristol.

When it comes to organisation, perfection is not what I'm looking for. I mean, that's a great thing to achieve, but I don't think it's expected. What I'm looking for in any event that runs for longer than a day is reactivity and improvements. Do they recognise issues and provide improvements from day to day? They did. I was a bit puzzled on the first day that there was enough coffee but not enough cups, and water was running low. But by day two, that was better.

And even though lunch was served at fewer places than in earlier years due to the higher number of parallel tracks, somehow it was smoother and I think I queued less than in earlier years.

I cannot express the gratitude I feel towards the people who made it possible for me to be here. That includes my family, the organisers, and also my bosses who let me come.

And even though sometimes a conference is very demanding — especially for introverted people — and I often feel impostor syndrome, it's so nice to get gentle and live feedback from people who follow my work. And when some highly respected people whose shoelaces I couldn't even tie call me by name, that's just humbling and really makes you feel part of the community.

## Conclusion

What more to say? As always, the On Sea conference was great — ACCU just as much as C++ before it. Four days packed with inspiring talks about topics ranging from AI's future to legacy code modernisation, from design patterns to contract assertions.

I walked away with new insights, fresh ideas, and a stronger sense of belonging in this community. Thanks for having me there. I hope to be back in 2027.

{% include connect-deeper.html %}
