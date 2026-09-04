---
layout: post
title: "C++ Concepts is now a Leanpub course"
date: 2026-9-4
category: books
tags: [cpp, concepts, books]
excerpt_separator: <!--more-->
---
When I published *C++ Concepts* as a book back in 2022, I was happy with how it turned out. It covered the right material at the right depth, and the feedback from readers was positive. But a book is a passive experience — you read, you nod, you move on. I recently learned about Leanpub courses and I thought that it's time to try something new.

So I turned *C++ Concepts* it [into a Leanpub course](https://leanpub.com/c/cpp20-concepts).

<!--more-->

## What the course covers

The material follows the same progression as the book: start with why concepts exist, learn the four syntax forms for functions, apply them to classes, explore the standard library's fifty-odd built-in concepts, then write your own from scratch — combining concepts, constraining operations and return types, verifying nested types, and using nested requirements. The final lessons show slightly more complex case studies: a complete `number` concept that actually excludes `bool` and character types, self-documenting utility functions, and multiple destructors with constrained member functions.

If you read the book, you already know the content. What the course adds is structure around it.

## What changed from the book

The most visible change is that the eight chapters became fifteen shorter lessons. I split the longer chapters — especially *"How to write your own C++ concepts"** which was way too long — into focused lessons that each teach one idea. For a course where you're expected to stop and practice, shorter units work better in my opinion.

Every lesson now ends with a quiz and most include hands-on exercises where you write concepts yourself. The quizzes are graded — you get immediate feedback on whether you understood the material. The exercises are ungraded practice with model answers you can compare against.

I also took the opportunity to fix things some issues I found in the original book:

- The `number` concept now correctly excludes `signed char` (a distinct type from `char` in C++, which the original missed)
- The multiple destructors section is updated — the Clang bug that prevented this feature from working was fixed in Clang 15, so all major compilers now support it. The standard term *prospective destructors* is included.
- Various code fixes: a missing semicolon in a requires expression, a reserved keyword used as a concept name, incorrect error message references, and a handful of prose errors

The book content is still there if you prefer reading straight through, but the quizzes and exercises only appear in course mode.

## Who this is for

C++ developers who want to start using concepts in their codebase. You need basic template knowledge — knowing what `template <typename T>` means and having written a simple function template. No SFINAE or `enable_if` experience required; in fact, concepts are the replacement for those.

You'll need a C++20 compiler (GCC 10+, Clang 10+, or MSVC 19.28+) if you want to follow along, though [Compiler Explorer](https://godbolt.org/) works fine too.

## Where to get it

The course is available on [Leanpub](https://leanpub.com/c/cpp20-concepts).

If you already bought the book, you should have received a coupon for an upgrade — I priced the cheapest possible on the platform. If you haven't, the course is the better starting point. You get the same material plus quizzes and exercises to make sure you actually retain it.
