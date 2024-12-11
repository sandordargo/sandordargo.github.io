---
layout: post
title: "Let's start exploring C++26"
date: 2024-12-11
category: dev
tags: [cpp, cpp26, meta]
excerpt_separator: <!--more-->
---
During the last 2 years, we spent a lot of time exploring C++23 resulting in [almost 40 blog posts](https://www.sandordargo.com/tags/cpp23/). I'm not saying that we covered [every single new language or library feature](https://en.cppreference.com/w/cpp/23), but we covered most of them. Finally, [C++23 became the official standard](https://twitter.com/Andreas__Fertig/status/1855901059291447577).

We're going to move our focus and over the next months or even years, we'll start exploring C++26, the new version in bake. The scope of C++26 is not set in stone yet, [there is still some time](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p1000r6.pdf), before it becomes feature-complete and it can still happen that certain features will have to be removed if the committee is not happy with the wording.

![C++ 26 standardization schedule]({{ site.baseurl }}/assets/img/cpp26-schedule.png "C++ 26 standardization schedule")

Still, we have more than enough [approved or even implemented features](https://en.cppreference.com/w/cpp/26) that we can learn or play around.

We are going to start our journey with features that are implemented at least on one of the main compilers. At the moment of writing, only Clang of GCC supports quite some of the language features, but on the library front, MSVC also has some items to offer.

Does C++26 offer some big topics as C++20 did with the big four (concepts, coroutines, modules and ranges)? The answer is that it's expected to have a few big features! It would be reflection, contracts and senders/receivers, whose new name nowadays is `std::execution`. But for the moment - if my understanding is correct -, the first haven't made it to the standard and maybe contracts won't even make it this time and `std::execution` is not yet implemented by any compiler.

Therefore, while waiting for them to be finalized and supported, we'll start our journey with different smaller features.

Stay tuned!