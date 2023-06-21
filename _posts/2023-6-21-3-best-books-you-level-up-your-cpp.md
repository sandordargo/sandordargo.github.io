---
layout: post
title: "The 3 best books to level up your C++"
date: 2023-6-21
category: books
tags: [cpp, books, selfimprovement, bestpractices]
excerpt_separator: <!--more-->
---
In this article, I'll present you 3 books that can level up your C++. For this article, I'm assuming that you have little or medium experience with C++ and you're looking for ways to reach the next level.

Reading blogs is also a good way to enhance your knowledge and keep up with the news. [Here is my curated list of C++ blogs.](https://github.com/sandordargo/cpp-resources/blob/master/blogs.md) There are many ways and you should not stick only to one type of source.

In this article, I'm focusing on books. On books that will help you reach the next level. With the level of knowledge I assume you have, you should be familiar with the language and library basics, maybe a little bit more. You should be able to use them with confidence and probably understand how they work, and some of their internals.

With the below 3 books, you'd learn the ins and outs of several C++ best practices, you'd learn a lot about templates to make them less dreadful and of course, one must get comfortable with architecture if he/she wants to make some career progression.

I'm not going to share full reviews of these 3 books, but you'll get the main idea behind them and a link to my full reviews on them.

## Beautiful C++ by J. Guy Davidson and Kate Gregory

[Beautiful C++: 30 Core Guidelines for Writing Clean, Safe and Fast Code by J. Guy Davidson and Kate Gregory](https://www.sandordargo.com/blog/2022/04/16/beautiful-cpp-by-kate-gregory-and-guy-davidson) is the book to read if you want to learn how to write more readable and therefore more maintainable C++ code. In it, you’ll find 30 handpicked guidelines from the [Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines). 

The authors explain each of those guidelines in detail. By reading the book, you will understand how and why you should apply them. While it doesn't cover all the guidelines (that would be a really long book!), it introduces the [Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines) to you and probably you'll open it up from time to time to see what is in there.

Among others, by reading [this book](https://www.sandordargo.com/blog/2022/04/16/beautiful-cpp-by-kate-gregory-and-guy-davidson) you'll learn why it's sane to avoid trivial getter and setters, why you should specify concepts or why you should opt for immutable data.

You'll also learn how to write more beautiful and more maintainable code than most of your fellow developers.

## C++ Software Design by Klaus Iglberger

[C++ Software Design by Klaus Iglberger](https://www.sandordargo.com/blog/2022/12/17/cpp-software-design-by-klaus-iglberger) is a great book that every non-beginner C++ developer should read in order to level-up their knowledge. Reading this book, you'll learn about the most useful design patterns in C++. More than that, you'll not only learn about their classical implementation, which is usually based on dynamic polymorphism, but you'll also learn about modern implementation approaches. The latter ones are usually value-based and let the compiler do most of the work, instead of the run-time. You also get comparisons, pros and cons so that you can choose in your real-life projects, which implementations you should consider.

If you ever wanted to understand (better) the strategy/policy or the visitor patterns, if you ever wondered what type erasure is or if you wanted to understand why there are so many debates about the singleton pattern, you should read this great book from [Klaus Iglberger](https://www.sandordargo.com/blog/2022/12/17/cpp-software-design-by-klaus-iglberger).

## Template Metaprogramming with C++ by Marius Bancila

Templates can be intimidating. Even though every C++ developer is using them at least through standard library types, more elaborated usages can be seen difficult. In fact, they are. [SFINAE is not easy to grasp](https://www.sandordargo.com/blog/2021/06/02/different-ways-to-achieve-SFINAE) and if you talk to developers who haven't kept up that much with the evolution of C++, they will strengthen this feeling in you.

But if you understand the basics well and use modern language features, such as [`if constexpr`](https://www.sandordargo.com/blog/2022/06/15/cpp23-narrowing-contextual-conversions-to-bool) or [concepts](https://leanpub.com/cppconcepts), templates become much easier. [Template Metaprogramming with C++ by Marius Bancila](https://www.sandordargo.com/blog/2022/10/28/template-metaprogramming-with-cpp-by-marius-bancila) provides you with exactly that. 

It has a clear structure and builds from the simple to the complex. It starts with the basics of templates, not forgetting about the terminology, history, and why you’d use templates in the first place. In the second part, it moves to advanced template concepts. It details how overload resolution works with templates, how to use *forwarding/universal references*, what is *CTAD*, and how to use `decltype` and `std::declval`. It also covers the already mentioned *SFINAE* and how to replace them with modern techniques.

In the third and final part, Marius Bancila details several design and implementation patterns that are using templates. The [Curiously Recurring Template Pattern](https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP), mixins, and type erasure are well explained with lots of code examples. The book also gives a nice introduction to the ranges library.

## Conclusion

In this article, you could learn about 3 great books that will help you reach the next levels in C++. With [Beautiful C++](https://www.sandordargo.com/blog/2022/04/16/beautiful-cpp-by-kate-gregory-and-guy-davidson), you'll learn how to write more beautiful and more maintainable code than most of your fellow developers. [C++ Software Design by Klaus Iglberger](https://www.sandordargo.com/blog/2022/12/17/cpp-software-design-by-klaus-iglberger) will teach you about the most important design patterns in C++ and how their pointer and value-based implementations compare against each other. Last, but not least, by reading [Template Metaprogramming by Marius Bancila](https://www.amazon.com/dp/1803243457/?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=3b52fe7dec703403826e4dab46d22da9&camp=1789&creative=9325), you'll learn the most important details of how to deal with templates, including modern features such as [concepts](https://leanpub.com/cppconcepts/). 


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!