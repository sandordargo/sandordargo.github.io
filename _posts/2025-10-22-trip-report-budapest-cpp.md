---
layout: post
title: "Trip report: Budapest C++ - Breaking & Building C++"
date: 2025-10-22
category: books
tags: [cpp, conference, budapestcpp, tripreport]
excerpt_separator: <!--more-->
---
We often spend the French autumn school holidays back in our hometown, Budapest, Hungary — and this year, the timing worked out perfectly for me to attend an event of the Budapest C++ Meetup. I'd like to thank the organizers for putting together such a great evening by sharing a short report from the event.

More than a hundred people registered, and the room quickly filled up with local developers eager to hear three technical talks. The atmosphere was lively and welcoming — it showed the strength of the C++ community in Budapest. **In 2027, [even WG21 might come to Hungary](https://isocpp.org/std/meetings-and-participation/upcoming-meetings)!**

The evening began with Jonathan Müller's talk, **Cache-Friendly C++**, followed by my own session on **Strongly Typed Containers**. Finally, Marcell Juhász closed the event with an insightful and hands-on presentation on **Hacking and Securing C++**.

![Budapest C++ Group (photo by Balázs Bondici)]({{ site.baseurl }}/assets/img/c++meetup_1022.jpeg)
*Budapest C++ Group (photo by Balázs Bondici)*

Let’s dive into some more details.

## Cache-Friendly C++ by Jonathan Müller

This was one of those deep dives that reminds you how much of software performance is bound by the physical reality of hardware. He began by walking us through the fundamentals of memory access — how the CPU's speed advantage over main memory has made caching essential, and why understanding this hierarchy is key for writing fast code. It's one thing to know that "cache is fast, RAM is slow", but Jonathan illustrated this vividly: object size, access patterns, and even the order of members inside a struct can make a measurable difference.

From there, he contrasted data structures that play nicely with caches against those that don't. Linked lists, for instance, are almost a textbook example of cache-unfriendly design: every node jump likely triggers a cache miss. In contrast, `std::vector` shines precisely because its elements live in contiguous memory. That’s the real reason it's so often the best choice — not just because of its API convenience, but because it aligns with how modern CPUs want to consume data.

Jonathan then shifted to the philosophy of data-oriented design, where you shape your data for efficient access rather than just elegant abstraction. This often leads to transforming *"arrays of structs"* into *"structs of arrays"*, which can drastically improve cache performance in tight loops. But he was quick to temper the enthusiasm: this transformation can also harm readability and maintainability. It’s a tool for when performance really matters, not a default approach.

The talk closed on a pragmatic note that summed up the spirit of his message: "*But benchmark to make sure you’re actually optimizing"*. Don't end up situations where you spend a lot of time optimizing, but you don't end up with faster, just less maintainable code.

## Strongly Typed Containers by Sándor Dargó

I guess it's always a special feeling to speak in your hometown. For me it was the first time, and it felt great. In my talk, we explored one of my favorite topics: how far we can take type safety in C++ before the language — or our sanity — starts to push back. Strong typing is easy to advocate for in theory: it reduces ambiguity, improves readability, and catches a whole class of bugs at compile time. But in practice, it raises important design questions — especially when our goal is to make containers not just generic, but also semantically strong.

![Me at Budapest C++ (photo by Balázs Bondici)]({{ site.baseurl }}/assets/img/me-at-budapest-cpp-2025.jpg)
*Me at Budapest C++ (photo by Balázs Bondici)*

I began by examining why we'd even want strongly typed containers in the first place. It's not just about avoiding accidental mix-ups between units or IDs — it's about making code self-documenting and harder to misuse. From there, I walked through different implementation strategies: inheritance (and the pitfalls of both `public` and `private` forms), composition as a safer and clearer alternative, and how to use C++'s type system to wrap `std::vector` or other STL containers without losing their familiar interface. We discussed when it's acceptable to *"bend"* STL design rules, and when it's better to keep them intact.

Toward the end, I mentioned a few open source tools for implementing strong types and went into some details about one. I did, admittedly, miscalculate my timing and had to rush through the last couple of slides, but the audience in Budapest was engaged and forgiving.

Presenting this talk at home made it all the more enjoyable. Sharing ideas about stronger types and safer abstractions in C++ —  and seeing how others connect with those challenges — was a reminder of why I love speaking at meetups like this one.

## Hacking and Securing C++ by Marcell Juhász

Marcell talk was one of the most vivid demonstrations of how dangerous *"undefined behavior"* can become when code meets hardware. He built his entire presentation around a small embedded device — simple enough to understand in detail, yet realistic enough to expose the kind of vulnerabilities that still plague production systems today. We began with the basics: memory layouts and address spaces on microcontrollers. We learned about how de/allocation happens on the stack. When a function exits, the next one might unknowingly read leftovers from its predecessor - maybe someone's password.

From there, Marcell guided us through increasingly severe exploits, showing how seemingly harmless coding oversights can leak sensitive information or even allow arbitrary code execution. The slides on the stack vulnerability were particularly striking — how an uninitialized local buffer can serve as a window into the stack, exposing old data byte by byte. Later, he shifted focus to the heap, illustrating how use-after-free and dangling pointers can emerge from innocent-looking delete operations. Once again, the lesson was clear: C and C++ give you enormous power over memory, but with that power comes the responsibility to initialize, sanitize, and properly manage every allocation.

Marcell didn't stop at the horrors. He also offered a pragmatic roadmap toward safety. Initialize local and dynamic variables. Wipe sensitive data in destructors. Prefer RAII and smart pointers to raw ownership. Use bounded containers like `std::array` and `std::vector`, and never rely on client-side validation for anything security-related. His closing slide said it all, with large C++ shield in the middle:

![Use modern C++]({{ site.baseurl }}/assets/img/use-modern-cpp.png)

In the end, what made this talk memorable wasn't just the exploits — it was how Marcell connected them back to everyday C++ coding. Even if you'll never write firmware for a microcontroller, the principles carry over: memory is fragile, trust is dangerous, and the best security often comes from disciplined simplicity and the most basic features of C++.

## Conclusion

The Budapest C++ Meetup was a great reminder of how strong and curious our local community is. Each talk approached the language from a different angle — Jonathan Müller from the perspective of performance, mine from design and type safety, and Marcell Juhász from security — yet all shared the same core message: understand what C++ gives you and use it wisely. Whether it’s memory access patterns, type abstractions, or resource management, the difference between elegant code and fragile software often comes down to awareness and discipline.

It was inspiring to see so many people gathered to learn, discuss, and share ideas about modern C++. Budapest clearly has a thriving developer scene, and I'm already looking forward my next visit to Budapest - hopefully at the same time with another meetup.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
