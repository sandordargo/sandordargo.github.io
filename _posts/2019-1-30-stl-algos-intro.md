---
layout: post
title: "The big STL Algorithms tutorial: Introduction"
date: 2019-1-30
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
I've already written quite a few articles about features introduced by C++ 11 and how much it changed how I look at the language. The feature I liked the most is probably the one of lambda expressions. I don't like them for their sheer existence, it's not _l'art pour l'art_, but it really helps using the different STL algorithms. If you want to get a quick introduction to all the 105, have a look at [this video](https://www.youtube.com/watch?v=2olsGf6JIkU ) by the owner of [fluentcpp.com](https://www.fluentcpp.com/).
<!--more-->

How I write C++ code changed a lot due to the combination of lambdas and STL algorithms. Yet I know that I don't a lot and I want to improve my knowledge on the STL. What's the best way to learn? Either by doing it or by teaching it. I'm already doing it, so is why hereby I'm starting to write a series on the STL algorithms.

I don't know yet, how frequently I'll write about them and how many I'll cover in one article, but every second technical article I'll write will be about the STL algorithms - the order of publication is another question.

Let's get started!

The algorithms we are going to discuss are basically a set of functions that we can use well together with STL containers and another common point is that they all can be found in the <algorithm> header.

According to [cplusplus.com](http://www.cplusplus.com/reference/algorithm/), we can categorize them into 8 groups plus others:
- Non-modifying sequence operations (e.g. all_if, any_of, find)
- Modifying sequence operations (e.g. copy, copy_if, transform)
- Partitions (e.g. partition, is_partition)
- Sorting (e.g.sort, is_sorted)
- Binary search (e.g. binary_search, lower_bound, upper_bound)
- Merge (e.g. merge, set_union)
- Heap (e.g. push_heap, pop_heap)
- Min/max (e.g.min, max...)
- Others

Some groups I'll show you in one post, like min/max, but some other groups which are way bigger, like _Non-modifying sequence operations_ I'll break down into smaller chunks.

Already published articles of this series:
* [all_of, any_of, none_of](/blog/2019/02/20/stl-algorithm-tutorial-part-1-any-all-none)
* [for_each](/blog/2019/04/03/stl-algorithm-tutorial-part-2-for_each)

Stay tuned!
