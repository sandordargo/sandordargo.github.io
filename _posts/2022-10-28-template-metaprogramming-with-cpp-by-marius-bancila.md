---
layout: post
title: "Template Metaprogramming with C++ by Marius Bancila"
date: 2022-10-28
category: books
tags: [books, watercooler, templates, concepts]
excerpt_separator: <!--more-->
---
Are you familiar with templates? Are you also comfortable with them? Do you know how to use them in an idiomatic way compiling against a modern standard?

If you answered no to any of these questions, [Template Metaprogramming with C++](https://www.amazon.com/dp/1803243457/?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=3b52fe7dec703403826e4dab46d22da9&camp=1789&creative=9325) by [Marius Bancila](https://mariusbancila.ro/blog/) is the book you should read next!

I was contacted by Packt to see whether I could recommend this book. As I explained on my blog several times, I don't write negative reviews on books. The author always works hard on a book, and I see no point in bashing them and their products. So the fact that I receive a book for free does not result in an automatic (positive) review. Sometimes I just share my feedback in private.

But this book from Marius Bancila is worth all the positive reviews.

So what makes this book good?

First, it has a clear structure and builds from the simple to the complex. It starts with the basics of templates, not forgetting about the terminology, history, and why you'd use templates in the first place. It goes into detail about the different types of instantiations, specializations. Even if you are familiar with templates, I think it will help you clarify some tids and bits. This book, helped me understand parts I didn't understand before. In particular variadic templates, parameter packs and fold expressions.

Then, in the second part, it moves to advanced template concepts. It details how overload resolution works with templates, how to use forwarding/universal references, what is *CTAD*, and how to use `decltype` and `std::declval`.

Still in the second part, it discusses type traits, and it gives a clear explanation of SFINAE - finally someone! Once you understand this dreaded technique you can learn about how it can be replaced with *constexpr if* or with concepts. But it doesn't only mention concepts, as they are the completion of templates that Bjarne Stroustrup was dreaming about since templates were introduced they fit perfectly the book. The author dedicated a whole chapter to discussing concepts and constraints. It's quite thorough and that chapter is almost as long as my book dedicated to concepts.

In the third and final part, [Marius Bancila](https://mariusbancila.ro/blog/) details several design and implementation patterns that are using templates. The [Curiously Recurring Template Pattern](https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP), mixins, and type erasure are well explained with lots of code examples. The book also gives a nice introduction to the ranges library.

You might have observed that I'm not following my usual pattern of book reviews where I detail 3 points of the author. One reason is that I already wrote about some of the topics and the other reason is that I plan to dig deeper into some topics and they deserve a whole post. I'll always refer back to [Template Metaprogramming with C++](https://www.amazon.com/dp/1803243457/?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=3b52fe7dec703403826e4dab46d22da9&camp=1789&creative=9325). This is a highly recommended read if you want to level up your knowledge of templates!

