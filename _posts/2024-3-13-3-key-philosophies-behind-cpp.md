---
layout: post
title: "Three key philosophies behind C++"
date: 2024-3-13
category: dev
tags: [cpp, meta, philosophy, design]
excerpt_separator: <!--more-->
---
Recently, I had to refresh some training material for software engineers who are not new to programming but are new to C++. It's a short introduction by all means and as participants are expected to know how to program in a C-like language (usually Java, Kotlin or Python), I don't have to focus on syntax basis.

I decided to follow this agenda for this first part of the course:
- A little bit of C++ history
- The characteristics of C++
- Three key ideas driving C++
- Why to still use C++

In this article, I'd like to focus on the third part, the three key ideas driving C++.

## "What you don’t use, you don’t pay for"

In short, this philosophy is explained in the FAQ section of [isocpp.org](https://isocpp.org/wiki/faq/big-picture#zero-overhead-principle)

> *The zero-overhead principle is a guiding principle for the design of C++. It states that: What you don’t use, you don’t pay for (in time or space) and further: What you do use, you couldn’t hand code any better.*
>
> *In other words, no feature should be added to C++ which would make any existing code (not using the new feature) larger or slower, nor should any feature be added for which the compiler would generate code that is not as good as a programmer would create without using the feature.*

So on the one hand, no new feature should make existing code larger or slower. Fair enough. But there is more to this philosophy, I think.

C++ is a language that gives you great control over what exactly the program should do and with that great power it also gives great responsibility. In other words, C++ is not babysitting you, it assumes that you know what you do and why you make certain choices.

Think about `operator[]`. It doesn't do any bounds checks, it's your responsibility as a programmer to ensure that `operator[]` will not look behind the bounds of a container. If you fail to program up to that assumption, your "reward" is undefined behaviour. On the other hand, you made sure that you don't pay for bounds checks. If you need those checks and you want the compiler to warn you - with an exception - when you are beyond bounds, you can use `at()` - or implement the checks by hand. You only pay for the bounds checks when you communicate your needs.

That's far from being the only example in the standard library. Just think about smart pointers. Raw pointers require manual memory management which is error-prone and dangerous. `std::unique_ptr` just gives you the bare minimum to make sure that allocated memory is freed up properly. If you need to share the resource, you can use `std::shared_ptr`. I love this granularity of C++. On the one hand, it might mean complexity for you, but on the other hand, it means fine-tuned control and optimal costs.

Speaking about costs, still part of this are zero-cost abstractions. Abstractions that increase the expressibility of C++ without incurring further runtime costs. Just think about how templates help you avoid repeating (almost) the same code over and over again without any extra runtime costs. Yes, [templates increase the binary size](https://www.sandordargo.com/blog/2023/04/05/binary-size-and-templates) and compile-time, but not more than actually repeating the code.

Another example of zero-cost abstractions is exceptions. They increase the expressibility of the language and your code, yet you don't pay any runtime costs for any unthrown exceptions. ([You do pay a certain toll in binary size and compile-time though.](https://www.sandordargo.com/blog/2023/03/29/binary-size-and-exceptions))

## [Backward compatibility is the superpower of C++](https://www.youtube.com/watch?v=0_UttFDnV3k)

Another key driving idea behind C++ is backward compatibility. If you browse the list of changes of any new standard, you will notice 4 things.

1) There are very few changes that break existing APIs and require code changes from the user.

2) There are sometimes behaviour changes, but they are either by accident and are corrected later or more frequently, they are for the better. Just think about move semantics or the growing support for `constexpr`.

If you upgrade your compiler you might find that certain copy operations became moves and function calls that used to happen in run-time, might happen in compile-time. These kinds of changes are usually not bothering. 

3) There are very few deprecations or removals in C++. The reason behind this is that nobody wants to break existing code. There are systems that should work for decades without needing a major rework. Yet, they should benefit from recent compilers having active support and security fixes. Moreover, you should be able to maintain and extend old systems without having to rework all the existing code.

The biggest one I remember was the deprecation and removal of `std::auto_ptr`, but the transition to `std::unique_ptr` (or `std::shared_ptr`) was quite straightforward. Other examples include `std::bind1st` and `std::bind2nd`, but frankly I had to look them up.

4) As a direct consequence, if you take an old C++ project and want to compile it with a modern compiler, there is a fair chance that it will work right away or with very few modifications. It doesn't mean that even if it was great code back in time, we would consider it great nowadays, but at least it will be just as correct as it was when it was written and you can improve it gradually. Both Matt Godbolt and Jason Turner have such videos online, but I'm sure you can find others as well.

## Portability, interoperability, and X-platform development

The third key idea behind C++ that I want to discuss today is about where C++ is used and how it is written. No matter how many new features have been introduced and how much more expressible the language has become, these topics still have the utmost importance.

From the very beginnings of the language, C++ is designed to be portable across architectures and operating systems. Did you know that even the number of bits in a byte is implementation specified? The standard doesn't mandate 8 bits, it's simply the most prevailing value. Indeed, C++ is everywhere starting from microcontrollers, through your browsers, through games to supercomputers.

To make it not only possible but also comfortable it offers platform-independent abstractions in the standard library. While it's certainly possible to provide different implementations in your code for different platforms with preprocessor conditionals, the need is relatively rare and if you need to do so too frequently, your code is probably flawed.

But C++ code isn't only mostly portable between different platforms, it can also work well with, it and can interoperate with code from other languages. You can easily call any *C* function and [you can directly embed assembly code](https://en.cppreference.com/w/cpp/language/asm) within a C++ file.

Beyond that, C++ can communicate with other languages through language bindings. Often, critical libraries for Python applications or JVM-based ones are implemented in C++ to benefit from its raw performance.

C++ is ideal for cross-platform development. Usually, code runs with minimal modifications on several platforms and popular libraries are also designed with that in mind. Mostly not only the code has this characteristic, but other parts of the ecosystem as well, such as build systems, package managers, and sometimes even compilers.

## Conclusion

In this post, we discussed 3 ideas behind C++ driving the evolution of the language. As one of C++'s key offerings is raw performance, it's essential that you don't pay for what you don't use. The most well-known example is probably about the missing bounds checks in the subscript operator. If you know what you do, you can live dangerously.

The second idea we discussed is backward compatibility which is both a superpower of C++ and the main cause behind the ever-growing complexity of the language. Last, but not least, we listed another key selling point of C++ which is portability, interoperability, and cross-platform development.

What other key ideas do you see behind C++?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!