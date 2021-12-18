---
layout: post
title: "Trip Report: Meeting C++ 2021"
date: 2021-11-17
category: dev
tags: [cpp, conference, meetingcpp, tripreport]
excerpt_separator: <!--more-->
---
I feel very lucky that I could attend so many C++ conferences during the last 2 years. It feels a bit strange, but without Covid, this would not have been possible.

My last conference was [Meeting C++](https://meetingcpp.com/2021/), between 10th and 12th November. Even if it was fully virtual this year, I had a great time.
<!--more-->

At a point it was a bit stressful, it was definitely tiring, but it was totally worth it. I'm really grateful to my management for granting me the necessary time to attend it.

Why was it stressful, you might ask?

I gave a presentation about the [basics of C++ concepts](https://meetingcpp.com/2021/Speaker/items/Sandor_Dargo.html) and while I'm generally not someone who stresses a lot, definitely not in advance, but during the presentation, I had some distractions and they made me nervous.

A few minutes after I started I was told that the slides are not moving forward. So we had to restart the stream and it messed up my timing a bit and as a consequence, at the end, I couldn't take questions. The battery in my mouse died in the middle of the presentation and for some seconds I thought that my laptop froze. That's not all, but I don't want to bore you with the petty details.

One needs practice to handle this. I've barely given a dozen talks during the last years and I felt that's already a lot. Then Phil Nash said at his talk that since 2015 he delivered about 115 talks. Oh la la!

I don't like all positive reports, to me they feel not honest. I think we should not forget about the less great parts, so that we can improve. The one thing I didn't find great was the software used for the conference. As talks and Ask Me Anything sessions were categorized differently, it was hard to get one simple overview on the schedule. Talks could be exported to your calendar, other events couldn't be. Besides, based on my experience with other conference software, there were a bit too many technical difficulties.

Anyway, these didn't overshadow the quality of the sessions. The organizers, with Jens in the lead, worked incredibly lot to make everything as smooth as possible and they made a great job. The more than 320 attendees made a good ambience, the comments, the questions were gentle and relevant at the same time. I didn't hear or read anything inappropriate.

Once again, thanks for this great event.

## My 3 favourite talks

Let me share my 3 favourite talks from the conference.

### [Zen and the art of Code Lifecycle maintenance by Phil Nash](https://www.youtube.com/watch?v=tjnFXS10jU0)

It might be surprising, but one of my favourite talks were not about C++. Probably it's less surprising if I tell you that it was about software quality and it was delivered by the main organizer of [C++ On Sea](https://cpponsea.uk/), [Phil Nash](https://twitter.com/phil_nash).

Software quality is something difficult to measure, it's even difficult to put it into words. Some even say that it's a meaningless marketing term. It's meaningless because everyone means something different while talking about it. 

Yet, people know what good quality software looks like when they see one. Still, that's something difficult to define upfront. One cannot not think about [Justice Potter Stewart trying to explain hard-core pornography](https://corporate.findlaw.com/litigation-disputes/movie-day-at-the-supreme-court-or-i-know-it-when-i-see-it-a.html).

Phil cited the criteria of the [Consortium for Information & Software Quality](https://www.it-cisq.org/), where they already tried to define software quality and they came up with 4 pillars:
- security
- reliability
- performance efficient
- maintainability

Phil rephrased some and added two more ending up with the following 6 elements:
- **m**alleability / evolability
- **r**eliability
- **c**orrectness
- **r**easonability
- **a**pplicability
- **p**erformance / efficiency

Connecting the initials made him realize that this list in this form is not really compelling - though I personally think it's related to quality...

Anyways, he rephrased and reordered the elements and came up with *career*:

- **c**orrectness
- **a**pplicability
- **r**eliability
- **e**volability
- **e**efficiency
- **r**easonability

From that point, the talk could have been a bit boring if these elements had been covered one by one, but Phil was examining the intersections of the elements which was definitely interesting and I'd definitely recommend you to watch it. I particularly liked how he was putting fuzz testing into the intersection of correctness and reliability.

He absolutely convinced me to read [Zen and the Art of Motorcycle Maintenance](https://www.amazon.com/gp/product/B0026772N8/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=sandordargo-20&creative=9325&linkCode=as2&creativeASIN=B0026772N8&linkId=b6e3038308f349e9e00809ff90e2f1ed) which I already started and I find it fascinating after the first few pages.

### [How to rangify your code by Tina Ulbrich](https://www.youtube.com/watch?v=d9ToM7sIvq0)

There were some slots where I wanted to watch multiple talks at the same time. Luckily the uncut recordings were quickly available, so in the evenings and during the next days' lunchtimes I could watch some more talks.

By the time I got to watch [Tina](https://twitter.com/_Yulivee_)'s talk, I already heard many people recommending it. They were right, Tina gave a very interesting presentation with many real-life examples on how to use ranges in your code.

I found it a great idea that she explained what qualifies her to talk about ranges and that she didn't just share youtube links of other videos at the end, but she took the time to explain whose videos and why she'd recommend watching them if you wanted to learn more about ranges.

I don't want to share her examples, soon you can watch the talk, I'd rather share a few words on my impressions.

If you are a regular reader of my blog, you know that I'm a fan of [using standard algorithms over raw loops](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms). I believe that they make your code not just more correct, but also easier to understand.

For ranges, I don't see the same yet. In the simpler examples, I found the rangified code more readable for sure, but as we move forward with the presentation, the rangified versions were shorter for sure but more and more obscure - to me. 

I have no problem with the pipe syntax, I have no issues with functional programming concepts either, recently I even started to learn about Clojure that I truly enjoy.

So what can be the issue?

The issue is simply that I don't know enough about ranges. 

We have a lot of new *verbs* introduced by the ranges library that were not available before and we have to learn them. We have to learn a lot of new vocabulary in order to be able to use ranges effectively.

Tina's presentation is a great starting point to learn more. Watch it, stop it, try the code yourself, read the documentation and then continue watching the video.

[This is one of the videos](https://www.youtube.com/watch?v=d9ToM7sIvq0) that I don't recommend watching once, or watching all at once, but take more time to fully benefit from it.

### [Breaking Dependencies: Type Erasure - A Design Analysis by Klaus Iglberger](https://www.youtube.com/watch?v=jKt6A3wnDyI)

At [C++ On Sea](https://cpponsea.uk/) I attended a few hours of Klaus' workshop on modern C++ design patterns, but due to work matters, I couldn't stay the whole day. Yet, I was impressed by the calmity and professionality of his way of presenting. I knew I wanted to attend his presentation.

As its schedule was conflicting with the evening routine of my kids, I watched it the other day. The only thing I lost was the availability to ask live. A fair deal to be able to tell some bedtime stories.

In his talk, Klaus was sharing his thoughts on software design in general, inheritance and the strategy pattern as well. He covered much more than type erasure - which would have been already worth it.

The most important challenge of software design is to welcome changes. Software will have to change, no matter what you think or do. It is meant to change by definition, that's why it's called _**soft**ware_.

With good design, you have to ease changeability and limit the number and the strength of dependencies.

With the help of the good old shapes examples Klaus showed why inheritance on its own is not a - good - solution, how we can and how the STL uses the [strategy pattern](https://en.wikipedia.org/wiki/Strategy_pattern).

The bigger half of the presentation was dedicated to Type Erasure which lets us creating something that is still dynamic polymorphism but without the burden of any virtual functions. 

Type Erasure is a mixture of three design patterns:
- External Polymorphism
- Bridge
- Prototype

The big strengths of this talk are the detailed example with tons of code and Klaus' great explanations. I don't even try to detail Type Erasure for you here in a couple of lines and I also don't want to claim that now I have a deep understanding of it.

I'll keep revisiting [this video](https://www.youtube.com/watch?v=jKt6A3wnDyI) and implement Type Erasure myself on some code katas and post my experience so that I can confirm Klaus' summary on the extremely interesting design pattern that reduces dependencies and improves the performance while also improving readability and comprehension. That sounds like an ideal combination.

## My 3 favourite thoughts

Besides my 3 favourite presentations, I'd also like to highlight 3 engaging thoughts that I heard during the conference.

### On the alignment trap by Phil Nash

I'd like to mention one thought from [Phil Nash](https://twitter.com/phil_nash)'s presentation on [*Zen and the art of Code Lifecycle maintenance*](https://www.youtube.com/watch?v=tjnFXS10jU0).

He mentioned the alignment trap that was introduced by [Allan Kelly](https://twitter.com/allankellynet). He examined many teams and put categorized them along two axes. What makes a team more successful? Doing the right thing or doing the things right?

![Alignment Trap]({{ site.baseurl }}/assets/img/alignment_trap.png)

No surprise that the most successful teams are doing the right things the right way. But it might be surprising that doing the things right is more important than doing the right thing.

The reason is that fixing what you do is much easier than fixing how you do things. It's easier to reach the ideal quadrant from the wrong thing/right way combination than from the right thing/wrong way combo.

That's definitely a message that I'll share with my teammates.

### On tools by Daniela Engert

One thing that Daniela said during her Ask Me Anything session really resonated with me. As AMAs cannot be rewatched, I cannot quote her properly, but she said something like *each and every developer is a snowflake, we are really sensitive when it comes to our tooling. Therefore tools should adapt to the developers and not the other way around.*

It's a painful truth. Painful because so often we are left with poor tooling and we just try to find our ways around because we don't invest the time and money to find and/or develop tools fitting our needs.

This thought is far from a novel idea, but it's a very important reminder for us to improve both our productivity and our satisfaction.

### On ~~forwarding~~ universal references by Nico Josuttis

[Nico was mentioning](https://www.youtube.com/watch?v=ey4pTOfdi9k) certain ranges that cannot be passed by `const&`. When you are unsure what kind of ranges should be accepted by a function, you should rather be prepared for everything.

It's not that difficult in this case as there is a type, a reference that can refer to everything. While a normal non-`const` reference cannot refer to temporary objects, a universal reference can. It can universally refer to anything by keeping all its attributes. That was the original usecase for unirversal references, and it's an old term. 

As time passed by, universal references (`T&&`) were more and more frequently used for perfect forwarding and Nico too started to use the term *forwarding references*.

But time continued to pass by and we use them more and more not for forwarding, but to accept any kind of references and so people - including Nico - are using more and more the old term, *universal references*.

From a technical point of view this is not a particularly interesting story, but from a higher perspective I think it is fascinating. You cannot know how things are going to change, what kind of direction (technical) evolution or history itself takes.

Old fads, habits and patterns that went out of fashion can reappear anytime and they might become more actual than ever.

## Conclusion

I would like to thank once more the organizers of [Meeting C++](https://meetingcpp.com/2021/) for making this great event happen. It's another great place to be if you want to learn about trends and great techniques of modern C++.

I shared here only a couple of talks and thoughts but I can assure you that there were many others that could have made it to this report. The talks are available at the [Youtube page of Meeting C++](https://www.youtube.com/c/MeetingCPP).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!