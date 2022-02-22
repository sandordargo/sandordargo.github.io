---
layout: post
title: "The big STL Algorithms tutorial: wrapping up"
date: 2022-2-23
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
With the last article on algorithms about [dynamic memory management](https://www.sandordargo.com/blog/2022/02/02/stl-alogorithms-tutorial-part-30-memory-header), we reached the end of a 3-year-long journey that we started at the beginning of 2019.

Since then, in about 30 different posts, we learned about the algorithms that the STL offers us. We are not going to have a crash course on them, if you are looking for something like that watch Jonathan Boccara's video from CppCon2018, [105 STL Algorithms in Less Than an Hour](https://www.youtube.com/watch?v=2olsGf6JIkU)

Instead, let's remind us of a couple of key concepts and oddities that we learned along the way.

## You don't pay for what you don't need

Standard algorithms showcase perfectly that in C++ you don't pay for what you don't need.

One example like that is bound checks.

Most of the algorithms that need more than one range take only the first range via two iterators (`begin` and `end`), the rest is taken only by one iterator, by one that denotes the beginning of the range.

It's up to the caller to guarantee that the additional input containers have enough elements or that an output container has enough space to accommodate the results. There are no checks of the sizes, no extra cost for ensuring something that is up to the caller to guarantee.

While this means potentially undefined behaviour, it also makes the algorithms faster and as the expectations are clearly documented we have nothing to complain about. 

## Lack of consistency sometimes

We've also seen that sometimes the STL lacks consistency quite a bit. Even though it is something standardized, it's been under development for almost 3 decades, so I think it's normal to end up with some inconsistencies.

As C++ and the standard library is widely used, it's almost impossible to alter the existing API, so we have to live with these oddities.

But do I have in mind?

- `std::find` will look for an element by value, `std::find_if` takes a predicate. At the same time, `std::find_end` can either take a value or a predicate. There is no `std::find_end_if`. While it's true that `std::find_end_if` would be a strange name, it would also be more consistent.
- Wile `exclusive_scan` can optionally take an initial value and a binary operation in this order, `inclusive_scan` takes these optional values in the different order, first the binary operation and then the initial value. Maybe it's just a guarantee that you don't mix them up accidentally?
- I found it strange that `transform_reduce` takes first you pass the reduction algorithm and then the transformation. I think the name is good because first the transformation is applied, then the reduction, but perhaps it should take the two operations in a reversed order.

## Algos are better than raw loops!

No more raw loops as Sean Parent suggested in his talk [C++ Seasoning](https://www.youtube.com/watch?v=W2tWOdzgXHA) at GoingNative 2013. But why?

STL algorithms are less error-prone than raw loops as they were already written and tested - a lot. Thousands if not millions of developers are using them, if there were bugs in these algorithms, they have been already discovered and fixed.

Unless you are going for the last drops of performance, algorithms will provide good enough efficiency for you and often they will not just match but outperform simple loops.

The most important point is that they are more expressive. It’s straightforward to pick the good among many, but with education and practice, you’ll be able to easily find an algorithm that can replace a for loop in most cases.

For more details read [this article](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms)!

## Conclusion

Thank you for following through this series on STL algorithms where we discussed functions from the `<algorithm>`, `<numeric` and `<memory>` headers.

After about 30 parts, today we wrapped up by mentioning once again some important concepts and inconsistencies of algorithms. We discussed how algorithms follow one of the main principles of C++: you don't pay for what you don't need.

We saw three inconsistencies within the STL, such as sometimes you have to postpend an algorithm with `_if` to be able to use a unary predicate instead of a value, but sometimes it's just a different overload.

Finally, we reiterated the main reasons why STL algorithms are better than raw loops.

Use STL algorithms in your code, no matter if it's a personal project or at work. They'll make your code better!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
