---
layout: post
title: "Hands-On Design Patterns with C++ by Fedor Pikus"
date: 2022-7-23
category: books
tags: [books, cpp, design patterns, architecture]
excerpt_separator: <!--more-->
---
In [Hands-On Design Patterns with C++](https://www.amazon.com/Hands-Design-Patterns-reusable-maintainable/dp/1788832566?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=30b0f28f6d2b14f4979cee40a2d91f43&camp=1789&creative=9325), the author [Fedor Pikus](http://www.pikus.net/~pikus/physres.html) shares the most common design patterns used in modern C++. In fact, not only design patterns but also some best practices and idioms. He doesn't end the stories simply where the [Gang of Four Design Patterns book](https://amzn.to/36VKyO2) ends, but he looks deeply into the specialities of C++ implementations.

But what does the book cover?

It starts with discussing inheritance and polymorphism in general. Then it goes to detailing templates and how to do generic programming in C++. While it seems a bit strange to discuss templates right after inheritance, if you think about it a bit more, it's not odd at all. Polymorphism can be achieved dynamically with the good old virtual functions or statically, using templates heavily. The author reaches the [Curiously Recurring Template Pattern](https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP) a bit before the half of the book and that pattern accompanies us in many subsequent chapters as a building block for more complex solutions.

I particularly enjoyed the chapter that presented *Type Erasure* which has growing popularity in the  C++ community. The author taught me something in this subject that I haven't really considered before. Notably that `shared_ptr` is not slower than `unique_ptr` only because shared pointers do reference counting. No. Shared pointers also erase the type of the optional deleter and that has a cost!

I won't enumerate all the topics, the author discusses across the 18 chapters of this book, you can have a look at this sample on Amazon, but I have to mention that Chapter 10 greatly improved my understanding of *local buffer* and *small strings* optimization, what they are and how they are implemented.

In chapter 14, the template method pattern (which has nothing to do with C++ templates) is on the table. You might simply know the Non-Virtual Idiom. Combine them and you get a very C++ centric idiom.

I already used it our code base since to refactor a big piece of code and remove some hundreds of copied lines. The basic idea behind is that the public API of your classes only provide a non-virtual interface, keeping it fairly simple.

No need to think about what implementation is called in the end. On the other hand, the non-virtual base class function has branching points where it calls usually pure virtual private functions and the differences are concantrated at those places. I'll write about this idiom in more lengths in the coming months.

I also enjoyed the chapter on policy-based design, which is just a fancy name of the good old strategy pattern from the [GOF](https://amzn.to/36VKyO2). I liked the way the author explained it and how he applied the pattern at compile time using generic programming. In fact, I think this whole book helped me a lot in understanding better programming with templates.

If you are looking for a book that will give you practical knowledge on how to design a modern C++ application, [this is the one you should read](https://www.amazon.com/Hands-Design-Patterns-reusable-maintainable/dp/1788832566?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=30b0f28f6d2b14f4979cee40a2d91f43&camp=1789&creative=9325)! A highly recommended read!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
