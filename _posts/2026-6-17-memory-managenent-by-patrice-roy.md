---
layout: post
title: "Memory Management in C++ by Patrice Roy"
date: 2026-6-17
category: books
tags: [books, cpp, memory, bookreview]
excerpt_separator: <!--more-->
---
Memory management is one of those topics that every C++ developer has to deal with, yet very few of us take the time to study it systematically. [Memory Management in C++ by Patrice Roy](https://amzn.to/441ob7B) is a book that finally gives this fundamental topic the deep and up-to-date treatment it deserves. It covers everything from the basics of how objects live in memory to modern techniques introduced as recently as C++26.

What impressed me right away is how current the book is. It discusses [erroneous behaviour](https://www.sandordargo.com/blog/2025/02/05/cpp26-erroneous-behaviour), which is something that was only introduced in C++26. It also proposes the usage of [deducing this](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23) as a way to reduce boilerplate in `begin()`/`end()` implementations. Finding a book that is both thorough in its foundations and aware of the latest developments is rare.

As Patrice writes, *"C++ is powerful and flexible, but if you program in C++, you're expected to behave responsibly and professionally."* This sentence sets the tone for the whole book. It's not about dumbing things down or hiding complexity behind abstractions. It's about understanding your tools well enough to use them correctly.

Let me highlight three ideas from the book that I found particularly valuable.

## `char*` is not what you think it is

One of the most eye-opening sections deals with the true meaning of `char*`. Most of us think of `char` as a character type. But as Patrice explains, *"char\* means 'pointer to a byte.' Due to the C language roots of C++, a char\* can alias any address in memory (the char type, regardless of its name, which evocates 'character', really means 'byte' in C and, by extension, in C++)."*

This is not just a historical curiosity. It has real consequences for optimizations. *"There is an ongoing effort in C++ to give char the meaning of 'character,' but as of this writing, a char\* can alias pretty much anything in a program. This hampers some compiler optimization opportunities (it is hard to constrain or reason about something that can lead to literally anything in memory)."*

The solution has been available since C++17: `std::byte`. As the author puts it, *"std::byte\* is the new 'pointer to a byte,' at least since C++17. The (long-term) intent of byte\* is to replace char\* in those functions that do byte-per-byte manipulation or addressing, but since there's so much code that uses char\* to that effect, this will take time."*

This distinction between `char*` and `std::byte*` is something I think many C++ developers — myself included — haven't fully internalized yet. The book does a great job explaining both the historical reasons and the practical implications.

## What is an object, really?

There's a section where Patrice digs into what constitutes an object in C++, and the answer might surprise you if you haven't thought about it carefully.

*"Pointers are objects. As such, they occupy storage. References, on the other hand, are not objects and use no storage of their own, even though an implementation could simulate their existence with pointers. Compare std::is_object_v<int\*> with std::is_object_v<int&>: the former is true, and the latter is false."*

This is one of those fundamental distinctions that most C++ developers use every day without thinking about explicitly. Pointers are objects with their own identity and storage. References are aliases — they have no identity of their own. Understanding this distinction deeply matters when you reason about memory layouts, lifetimes, and move semantics.

## Measure before you optimize

The chapter on optimization contains advice that should be tattooed on every developer's forearm - though one would need quite a big one:

> *"Before trying to optimize parts of your program, it's generally wise to measure, ideally with a profiling tool, and identify the parts that might benefit from your efforts. Then, keep a simple (but correct) version of your code close by and use it as a baseline. Whenever you try an optimization, compare the results with the baseline code and run these tests regularly, particularly when changing hardware, library, compiler, or version thereof. Sometimes, something such as a compiler upgrade might induce a new optimization that 'sees through' the simple baseline code and makes it faster than your finely crafted alternative. Be humble, be reasonable, measure early, and measure often."*

I've seen this kind of advice before, but what I appreciate here is the nuance about compiler upgrades potentially making your hand-crafted optimization slower than the simple version. It's a humbling reminder that the compiler is often smarter than we are, and that optimizations have a maintenance cost. Your clever trick might become a liability the moment you upgrade your toolchain.

## Conclusion

[Memory Management in C++ by Patrice Roy](https://amzn.to/441ob7B) is a thorough, well-written book that fills a real gap in the C++ literature. Memory management is one of the defining aspects of C++ — it's what makes the language both powerful and demanding — and having a dedicated, modern treatment of the topic is invaluable.

As the author writes, *"With every step, C++ becomes a richer and more versatile language with which we can do more and express our ideas in more precise ways. C++ is a language that provides ever more significant control over the behavior of our programs."* This book helps you wield that control with confidence. A recommended read for any C++ developer who wants to truly understand what happens beneath the surface.

{% include connect-deeper.html %}
