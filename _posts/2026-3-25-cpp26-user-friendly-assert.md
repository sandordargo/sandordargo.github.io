---
layout: post
title: "C++26: A User-Friednly assert() macro"
date: 2026-3-25
category: dev
tags: [cpp, cpp26, reflection, syntax]
excerpt_separator: <!--more-->
---
C++26 is bringing some long-overdue changes to `assert()`. But why are those changes needed? And when do we actually use `assert`, anyway?

At its core, `assert()` exists to validate runtime conditions. If the given expression evaluates to `false`, the program aborts. I'm almost certain you've used it before — at work, in personal projects, or at the very least in examples and code snippets.

So what's the problem?

## The macro nobody treats like a macro

`assert()` is a macro — and a slightly sneaky one at that. Its name is written in lowercase, so it doesn't follow the usual *SCREAMING_SNAKE_CASE* convention we associate with macros. There's a good chance you've been using it for years without ever thinking about its macro nature.

Macros, of course, aren't particularly popular among modern C++ developers. But the issue here isn't [the usual - but valid - *"macros are evil"* argument](https://arne-mertz.de/2019/03/macro-evil/). The real problem is more specific:

> The preprocessor only understands parentheses for grouping. It does not understand other C++ syntax such as template angle brackets or brace-initialization.

As a result, several otherwise perfectly valid-looking assertions fail to compile:

```cpp
// https://godbolt.org/z/9sqM7PvWh
using Int = int;
int x = 1, y = 2;

assert(std::is_same<int, Int>::value);
assert([x, y]() { return x < y; }() == 1);
assert(std::vector<int>{1, 2, 3}.size() == 3);
```

Each of these breaks for essentially the same reason: the macro argument parsing gets confused by commas or braces that aren't wrapped in an extra pair of parentheses.

You can fix each of them, of course, by adding an extra pair of parentheses. For example the last assertion would become `assert((std::vector<int>{1, 2, 3}.size() == 3));` - you can [play with the examples here](https://godbolt.org/z/9sqM7PvWh).

But let's be honest: this is ugly, easy to forget, and not exactly beginner-friendly. Sure, after hitting the problem once or twice, most people internalize the workaround — just like with the [most vexing parse](https://www.sandordargo.com/blog/2021/12/22/most-vexing-parse). Still, it's unnecessary friction for such a fundamental utility.

## P2264R7: Making `assert` less fragile

This is where Peter Sommerlad’s proposal, [P2264R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2264r7.html), comes in.

The proposal fixes the `assert` macro by redefining it as a variadic macro using `__VA_ARGS__`. Instead of accepting a single parenthesized expression, `assert` now takes `(...)` as its parameter. That small change makes all the previously broken examples just work — no extra parentheses required.

## What about diagnostic messages?

Originally, the proposal allowed users to attach diagnostic text via the comma operator, similar to `static_assert`. That idea didn't survive the review phase.

Instead, there is a mechanism to prevent the use of the comma operator on a top level, so you cannot accidentally create *always true* assertions like this:

```cpp
assert(x > 0 , "x was not greater than zero");
```

Something that you could very easily create from an existing `static_assert`.

So if we want to have some diagnostics, we still have to use the `&&` operator instead:

```cpp
assert(x > 0 && "x was not greater than zero");
```

This keeps the semantics clear and avoids subtle bugs.

## But aren't contracts coming?

One common objection addressed in the proposal is the claim that `assert` is obsolete in a future with contracts.

Contracts will be great. But they won’t make assert disappear overnight.

Just as [concepts]() didn't eliminate SFINAE or older template techniques — they simply gave us better tools — contracts won't erase `assert` either. Assertions will continue to exist in real-world codebases, whether directly or wrapped inside higher-level precondition utilities. Improving `assert` is still valuable, especially when the changes are small, simple, and easy to backport.

If you're curious, the paper discusses several other potential concerns in detail; you can find them in the [section on potential liabilities of the proposed change](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2264r7.html#potential-liabilities-of-the-proposed-change).

## Compatibility and availability

Importantly, this change does not break existing code. All previously valid usage patterns remain valid — the proposal merely enables new, less fragile ones.

At the time of writing, (February 2026), none of the major compilers support this feature yet. As with many C++26 improvements, it will take some time before it becomes widely available.

## Conclusion

The C++26 update to `assert()` is a great example of incremental language evolution done right. It doesn't reinvent assertions, replace them with something flashier, or force new programming models on existing code. Instead, it quietly removes a long-standing sharp edge.

By making `assert` variadic, the language eliminates a class of surprising compilation failures, improves readability, and reduces the cognitive overhead of using a tool that every C++ developer relies on. It's a small change — but one that makes everyday C++ just a little bit nicer to work with.

Sometimes, that's exactly the kind of progress we need.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
