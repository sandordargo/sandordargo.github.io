---
layout: post
title: "Trip report: Meeting C++ 2025"
date: 2025-11-12
category: dev
tags: [tripreport, meetingcpp, cpp, watercooler]
excerpt_separator: <!--more-->
---
What a year I had! One more conference, one more trip report! I had the chance to go to Meeting C++ and give not just one but two talks!

I remember that [last year](https://www.sandordargo.com/blog/2024/11/20/trip-report-meeting-cpp2024) I said that Berlin in November is the perfect place to have a conference because you want to be inside that four-star hotel and not outside. This year, the conference was held a week earlier, and the weather was so nice that it was actually tempting to go out and explore.

But people resisted the temptation. The lineup and content were very strong — this year there were more than 50 talks across 5 different tracks. Also, Meeting C++ is a fully hybrid conference, so you can join any talk online as well.

It might sound funny, but I must mention that the food is just great at Meeting C++. It's probably the conference with the best catering I've ever been to — from lunch to coffee breaks, everything was top-notch.

This year, there were no evening programs. I'm not complaining; it's both a pity and a blessing, and I'm not sure how I feel about it. For example, when I first attended C++ On Sea, there were no evening events, and I really enjoyed discovering Folkestone in the evenings. Over the years, the schedule there got extended, and sometimes I had no time to visit my favorite places. But at least some socializing was guaranteed. One can say that you can do it on your own, but many of us are introverted, and if we're not *forced* to socialize, we just won't. That’s even easier to avoid in a big city like Berlin. I remember that last year I didn't have time to go out until the end of the conference. It was different this year.

But let’s talk about the talks.

## My three favourite talks

Let me share with you the three talks I liked the most. They are listed in chronological order.

### [Software and Safety by Antony Williams](https://meetingcpp.com/mcpp/schedule/talkview.php?th=kldfkdsafadlkadkfaai7ad9f7adjnkjadfs)

Antony talked about why safety is important — a reminder that even if we're not working in safety-critical fields, there's still a lot at stake. In health care, automotive, or aviation, it can literally be a matter of life and death, but even outside those domains, robust safety practices matter.

One of his main messages was not just to perform input validation but to define *intended behaviour* for every possible input. That behaviour doesn't have to be useful, but it must be intentional.

That naturally led to the topic of C++ contracts, which is goint to be a great addition to the language. But even without language support, we can already implement informal contracts — even a simple comment like "this pointer cannot be null" is far better than nothing and helps the next programmer.

I've never seen so many questions after a talk, and there was plenty of time for them! Among the many insights Antony shared, I want to highlight one: he believes that exceptions make code simpler — at least the hot path. Thanks to exceptions, there's no need to scatter the code with error checks, and when they're not thrown, they have zero runtime cost. And when they *are* thrown, you’re already on the unhappy path, so performance isn't the top concern anymore.

### [To Err is Human: Robust Error Handling in C++26 by Sebastian Theophil](https://meetingcpp.com/mcpp/schedule/talkview.php?th=206125c3d5084002b6e8eb326c46a9645b598418)

C++ has made big steps toward better error handling over the years. Sebastian gave a practical talk showing the various tools we have at hand to deliver stable products. Reliable error handling is essential. Even if we're not working on mission-critical systems, our goal is to ship customer value through dependable software.

Most of us still use C-style error codes or exceptions. Since C++23, [we also have `std::expected` to improve error handling](https://www.sandordargo.com/blog/2022/11/16/cpp23-expected). And even if your compiler doesn't support C++23 yet, it’s easy to backport. I especially appreciated Sebastian’s elegant code examples.

But the evolution doesn’t stop there. C++26 brings two new mechanisms: **contracts** and **library hardening**.

Contracts let us express preconditions, postconditions, and invariants. We’ll look at them on this blog in a few months, but if you want a short intro, check out [this article by Timur Doumler](https://timur.audio/contracts_explained_in_5_mins) referenced in the talk.

Library hardening allows turning certain undefined behaviours into contract violations. It enforces pre- and postconditions and builds on top of contracts. [You can read the proposal here.](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3471r4.html)

## [Unlocking the Value of C++20 Features by Alex Dathskovsky](https://meetingcpp.com/mcpp/schedule/talkview.php?th=f47cea4aeed5de351b6612e851c03a55a99265a9)

You might think a talk reviewing C++20 features has no place at a conference in late 2025 — but you’d be wrong. As Alex showed through shared stats, only about 43% of people can actually use C++20 at work. The majority are still stuck on C++17 or earlier.

This is "horrible", to use Alex's word. We keep complaining about the lack of safety and security that newer versions try to address, yet we fail to adopt them.

C++20 is huge, and it's impossible to cover it all in an hour. To make the most of it, Alex skipped the big four (concepts, coroutines, modules, and ranges) and instead focused on the many smaller language and library features, giving great references to learn more.

You won't learn them in depth from the talk, but you'll walk away with a sense of what exists — and a list of topics to explore. Among others, we learned about [abbreviated function templates](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them#the-4-ways-to-use-concepts), [designated initializers](https://www.sandordargo.com/blog/2024/09/04/structs-and-constructors), and [safe integer comparisons](https://www.sandordargo.com/blog/2023/10/11/cpp20-intcmp-utilities).

As Alex put it: *"C++ makes us lazier programmers, but that’s not a bad thing."* Especially when that laziness also brings safety.

## My three favourite ideas

Now let me share three lovely ideas that stuck with me during the conference.

### Cast are lies (Patrice Roy)

In his talk, [To lie... and hopefully, to lie usefully](https://meetingcpp.com/mcpp/schedule/talkview.php?th=74db8cfe2bfeb79a6f5ac93430811433bf389f65), Patrice Roy reminded us that different kinds of casts are essentially lies. `dynamic_cast` isn't a lie per se, but it's a red flag — a signal that something might be wrong with your design. You use it to validate what you *think* is correct. [We've talked about this before on this blog](https://www.sandordargo.com/blog/2023/04/26/without-rtti-your-code-will-be-cleaner), but this was a great refresher from someone with deep insight.

### Teachers are not allowed to be angry (Frances Buontempo)

[In her keynote](https://meetingcpp.com/mcpp/schedule/talkview.php?th=kdfjadwkfaldfjaldjfadkfadsfakdfajdk786tzdh), Fran talked about learning and teaching. One idea that really stuck with me was: *"Teachers are not allowed to be angry."*

Teaching and learning can both be frustrating — often for both sides at the same time. Yet while students can show frustration, teachers shouldn't — at least not in a negative way. If someone doesn't understand you, it's your job to find another way to explain.

This struck me because I've seen so many people get angry while teaching. It's bad enough in a przofessional setting, but it's worse when it happens between parents and children. That kind of frustration can leave long-lasting scars.

Yes, teaching is frustrating. But don't express your anger. Don't punish your students.

### Handling mutants doesn't always lead to better code (Nico Eichhorn)

[Nico talked about mutation testing](https://meetingcpp.com/mcpp/schedule/talkview.php?th=bdcf6ad04266e86a3ce18f8df89d7343238d7b3a) and gave several ideas I want to explore. At the very least, I want to try [mull-project/mull](https://github.com/mull-project/mull) for mutation testing.

Testing is important, and mutation testing can be a great addition to your toolkit. But as Nico showed, it can also be slow, costly, and not always beneficial — sometimes handling mutants doesn't lead to better code at all.

Still, I’m curious to try it and share my experience later.

## My personal experience

This was my second time at Meeting C++ and my third time in Berlin. This year, I even gave two talks — a technical one about namespaces and another one about software engineering and human dynamics.

I was a bit nervous about both, but for different reasons.

For the technical talk, I hadn't had much time to prepare, but I'd already given it at [C++ On Sea](https://www.sandordargo.com/blog/2025/07/02/cpponsea-trip-report), so I was confident. I still spent some time the night before and the next morning rehearsing, and it went well. I shared thoughts about namespaces — what they are, how they work, and some best practices. I’ve already covered some of these topics [on the blog](https://www.sandordargo.com/tags/namespaces/), and maybe I’ll write more in the coming months — but for now, I have higher priorities.

Something special about this talk was my guest: *Cippi*.

![Me from Folkestone to Dover]({{ site.baseurl }}/assets/img/cippi-me-meeting-cpp2025.JPG)

If you don’t know Cippi, [she's been attending C++ conferences ever since Rainer Grimm couldn't travel](https://www.modernescpp.com/index.php/my-als-journey-24-n-cippis-world-tour/). You probably know that he recently passed away. Having Cippi at my talk meant a lot to me, though I chose not to mention it during the session — I could have easily gotten too emotional.

My second talk, the more "scary" one, was about code reviews — and it didn’t contain a single line of C++ code! I didn't talk about the technical parts; there are plenty of great talks on that. Nor about how code reviews fit into business processes. ([There's a great talk for that too!](https://www.sandordargo.com/blog/2025/09/24/trip-report-cppcon-2025#my-favourite-talks)) Instead, I focused on the human aspects — why we do code reviews, what pitfalls to avoid, and how to make them less frustrating.

Because, yes, engineering isn't only about code. It’s about people.

Did people take it well? I think so. I heard several attendees say how refreshing it was to have more talks focused on the human side of software. This shift may have started with [last year's opening keynote by Titus Winters at Meeting C++](https://www.youtube.com/watch?v=_dLLIjKz9MY). It’s good to see that trend continue.

## Conclusion

Meeting C++ 2025 was once again a reminder of why this community is so great. The talks were inspiring, the speakers were approachable, and the hallway conversations were full of energy. It's one of those conferences where you constantly learn — not just about the language, but about the craft of being a software engineer.

What I loved most this year was the balance: deep technical content mixed with thoughtful discussions about teaching, learning, and teamwork. We're not only debating template syntax, but also how to write better, safer, and more humane software.

Personally, I'm coming home energized - I'm writing this from the airport. I have new tools to try, topics to explore, and stories to tell — and that’s the best sign of a great conference.

Last but not least, a big thanks to [Jens Weller](https://meetingcpp.com/2024/Speaker/items/Jens_Weller.html), the founder and organizer of Meeting C++, and to all the staff for making it such a great experience. Hope to see you next time!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
