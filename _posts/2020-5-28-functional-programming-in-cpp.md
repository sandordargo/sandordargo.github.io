---
layout: post
title: "Functional Programming in C++ by Ivan Cukic"
date: 2020-5-28
category: books
tags: [books, cpp, functional, watercooler]
excerpt_separator: <!--more-->
---
C++ is an Object-Oriented language, right?

Well, it'd be better to say among others. It can be used as such, but in reality, it's a multiparadigm language, suitable to use as a procedural, object-oriented, generic, and functional programming language.
<!--more-->

Today what I'd like to present to you is [Ivan Cukic](https://twitter.com/ivan_cukic)'s book called [Functional Programming in C++](https://amzn.to/2LeUiZ3). Obviously it mostly covers the functional parts of C++.
Why do I write "mostly"? There are 2 main reasons:
- Functional and generic many times go hand in hand
- You can use functional elements even in a procedural or object-oriented style. Who doesn't use the STL after all? While it's based on functional and generic concepts, it's a very embedded part of our OO C++ code.

## What will you learn from this book?

If you are someone who grew up eating OO paradigms for breakfast, notably in C++, this is an ideal book to learn about FP concepts. It starts at a very high level and then little by little goes into details. You might not even read it from cover to cover because you are not that much interested in template metaprogramming and functional design of a whole system, but still, I'd recommend reading it for curiosity.
Besides universal FP concepts, you'll also learn a lot about the main ideas behind the STL implementation. In particular, you'll understand why you have to pass an input range by two iterators and why you have run into a concrete wall if you wanted to compose multiple STL algorithms.

With C++20 we have something in the standard library that overcomes this issue of the STL and that was already available since C++14 through an external library: [`ranges`](https://en.cppreference.com/w/cpp/ranges). I don't say that [this book](https://amzn.to/2LeUiZ3) is a step-by-step tutorial for ranges and it shouldn't be. But it clearly expresses concepts behind and gives you enough examples so that you understand the basics and you want to discover more.

In fact, by the time you reach the chapter on ranges, you clearly wish something like that existed in the language. Is this a value of the structuring of this book or the library itself? I leave that question open.

No serious book on C++ can be written without discussing data structures and [Cukic's book]() is no exception to that rule. While detailing data structures that are ideal for functional programming is interesting, I found even more important the part where he details how you should design your data, your data classes in order to seriously limit the possibility of bugs. Algebraic data types sound fancy and maybe even alienating for some, but in practice, it's really handy "to minimize the number of states your program can be in and removes the possibility of having invalid states". Basically it advocates for using strongly typed states instead of a couple of booleans where some combinations don't make any sense. A practice that can be really important to practice.

The last third of the book contains more advanced ideas, like the aforementioned system design in a functional way, monads and template metaprogramming, I think it's worth to read it. Before I didn't even think that I understood some of the ideas at all. Now, I still know that I'm far away from a deep understanding, but at least I didn't leave the book despaired. Instead, I felt I learned something and have an idea about Monads, SFINAE. In fact, while reading the chapter on template metaprogramming I wrote more templates than ever before - we still don't speak about a huge number. This was clearly not a book that [I suffered through](https://amzn.to/2WE9N1Z).

Based on the above, I cannot do anything else but heartily recommend you [Functional Programming in C++ by Ivan Cukic](https://amzn.to/2LeUiZ3) if you are a C++ developer and interested in functional concepts. But even if you are not, the parts on STL, ranges, templates, and algebraic data types are worth the days/week you'll spend reading on it and for sure will help you to become a better C++ programmer.

Happy reading! 
