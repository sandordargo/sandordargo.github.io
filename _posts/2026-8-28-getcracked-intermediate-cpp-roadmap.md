---
layout: post
title: "I helped to build GetCracked's Intermediate C++ Roadmap"
date: 2026-8-28
category: dev
tags: [cpp, career, learning]
excerpt_separator: <!--more-->
---
A few months ago, [Coding Jesus](https://www.youtube.com/@CodingJesus) reached out to me about collaborating on content for [GetCracked](https://www.getcracked.io?via=sandor), his C++ learning platform. The pitch was simple: he was building an interactive roadmap for intermediate C++ developers and wanted someone to help with building the technical content — quiz questions, curated resources, explanations. That sounded like exactly the kind of work I enjoy, so I said yes.

<!--more-->

## What GetCracked is

GetCracked is a progress-tree-based learning platform for C++ developers at every level — from beginners to advanced. Think of it as a structured roadmap where each node covers one specific concept — `std::span`, fold expressions, `if constexpr`, `std::pmr`, whatever — and tests your understanding with multiple-choice questions built around real code snippets.

It's not a video course and it's not a textbook. Each question gives you a C++ code snippet and asks you what happens. You pick an answer, and if you're wrong, you get a detailed explanation of *why* — not just a correction but a walkthrough of the mechanism you missed. The questions are designed so that the wrong answers are plausible. No throwaway distractors; every option maps to a real misconception.

## The Intermediate C++ Roadmap

The intermediate roadmap is the one I collaborated on. It covers a wide range of modern C++ topics that I think a lot of developers either skip or only half-learn:

- **Formatting** — `std::format`, `std::print`, custom formatters
- **Object creation** — structured bindings, designated initializers, guaranteed copy elision, the copy-swap idiom
- **Modern containers** — `std::span`, `std::mdspan`, `std::inplace_vector`, `std::hive`, the flat associative containers
- **Type erasure** — `std::any`, `std::variant`, `std::visit`, the Pimpl idiom
- **Templates and concepts** — template template parameters, constrained auto, requires expressions, subsumption
- **Ranges** — algorithms, pipe operator, views, sentinels, projections, lazy evaluation
- **Time** — `std::chrono` in depth: clocks, durations, time zones, calendar types
- **Memory and alignment** — `std::byte`, `alignof`/`alignas`, placement new, object layout, `std::launder`
- **Callable utilities** — `std::apply`, `std::invoke`, `std::function`, `std::move_only_function`, `std::reference_wrapper`

...and more. The full roadmap is live on the platform.

My contribution was authoring quiz questions and curating the resources for each node. Writing good multiple-choice questions for C++ is surprisingly hard. The code must compile (or intentionally fail to compile), the distractors must be grounded in real misunderstandings, and the question name must never hint at the answer. I caught myself violating that last rule more often than I'd like to admit.

## What's next: the Allocators and Memory Resources tree

We're currently working on a new progress tree focused on **C++ allocators and memory resources**. This one goes deep: why custom allocators exist in the first place, the cost of general-purpose allocation, `std::allocator` and `allocator_traits`, the full `std::pmr` story (polymorphic allocators, `monotonic_buffer_resource`, pool resources, chaining), writing your own `memory_resource`, and when to justify the complexity over the default allocator.

Allocators are one of those corners of C++ that most developers know they *should* understand but never quite get around to studying. The quiz format works particularly well here because the gotchas are real — propagation traits, the allocator-is-part-of-the-type problem, `pmr` container pitfalls — and they're much easier to internalize through concrete code questions than through reading a reference page.

## Why I think it's worth your time

I've written a [book on C++ interview preparation](https://leanpub.com/cppinterviewbook), I write this blog every week, and I still found that authoring questions for GetCracked sharpened my own understanding of topics I thought I knew well. There's something about formulating a *wrong* answer that's plausible enough to trick someone — it forces you to think about the exact boundary between "almost right" and "actually right."

If you're a C++ developer who wants to fill gaps in your knowledge — or if you're preparing for a C++ role and want something more active than reading — [give it a look](https://www.getcracked.io?via=sandor). Using that link gets you 10% off.

{% include connect-deeper.html %}
