---
layout: post
title: "C++ Brain Teasers by Anders Schau Knatten"
date: 2024-10-16
category: books
tags: [cpp, brainteasers, watercooler, books]
excerpt_separator: <!--more-->
---
I bought [C++ Brain Teasers](https://amzn.to/3Nuj5bG) at [C++ On Sea](https://www.sandordargo.com/blog/2024/07/10/cpponsea2024-trip-report) where the author also gave a presentation and made a few references to his book. I couldn't wait to read it. Now that I finally did it, I'd like to share my thoughts.

Despite the fact that it's a relatively short book (the ebook is 122 pages), it has something to teach to everyone I think. Maybe you'll improve your C++ vocabulary and you'll better use some terms and expressions, maybe you'll get every teaser right at the beginning (kudos if you really do so!) and you'll simply learn about the reasons behind, but I'm sure you'll find something useful in this book.

C++ Brain Teasers consists of 25 puzzles. Each one starts with some code and you have to guess the output. Then you get the answer and a detailed explanation and some further readings. As I mentioned, even if you guess the output right, the discussion part will certainly teach you something.

C++ is a complex language with many details and quirks. It's difficult to guess all the outputs right. After the first few puzzles, I thought I could, but then I quickly realized that it was not the case.

So what topics will you learn from this book? I cannot give you an extensive list, but for sure by the time you finish reading the book you'll have a better understanding of
- when copies and movies are made
- when return value optimization takes place
- when and how types are implicitly converted

For me, the biggest surprise was that if we add up two `char`s, the resulting type will not be a `char` but rather an `int` or an `unsigned int` depending on your platform. For the details, you should check out Puzzle 14 of [C++ Brain Teasers](https://amzn.to/3Nuj5bG).

The most esoteric puzzle was probably guessing the output of this: `std::cout << +!!"";`. Without revealing all the details, it turns out that a string literal can be converted to a `bool` and some compilers offer warnings to catch such implicit conversions.

So I quickly tried Clang's `-Wstring-conversion` on some codebases I work on, and it turns out that I found quite some code to think about.

Sometimes an esoteric example from a puzzle book turns out to be something that can either give you a lot of work or at least strange code to think about.

If you're interested in what is going on under the hood of C++, [C++ Brain Teasers by Anders Schau Knatten](https://amzn.to/3Nuj5bG) is a must-read.