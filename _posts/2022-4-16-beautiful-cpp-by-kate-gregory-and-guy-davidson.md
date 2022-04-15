---
layout: post
title: "Beautiful C++: 30 Core Guidelines for Writing Clean, Safe and Fast Code by J. Guy Davidson and Kate Gregory"
date: 2022-4-16
category: books
tags: [books, watercooler, cpp, bestpractices]
excerpt_separator: <!--more-->
---
If you are familiar with the [Pluralsight courses of Kate Gregory](https://www.pluralsight.com/authors/kate-gregory), you won't be surprised by the name of this book. While many consider C++ a complex language that always results in difficult to read and to maintain code, it can be beautiful. It's probably true that with all the coming features, the language is still getting more complex. At the same time, idiomatic modern C++ code is getting easier to write and read thanks to the new language and library features.

But how to write idiomatic code?

A great source of inspiration is the [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines) which was launched in 2015 at C++ Con. This set of guidelines is edited by [Bjarne Stroustrup](https://www.stroustrup.com/) and [Herb Sutter](https://herbsutter.com/), but it's open for everyone on [Github](https://github.com/isocpp/CppCoreGuidelines) to create a pull request or review them.

[Kate Gregory](https://twitter.com/gregcons) and [J. Guy Davidson](https://twitter.com/hatcat01) were so much inspired by these guidelines that [they decided to write a book about them](https://www.amazon.com/Beautiful-Core-Guidelines-Writing-Clean/dp/0137647840?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=4a574de3a8d52ce616b3bdddd26e0c8c&camp=1789&creative=9325). Luckily they didn't decide to go through all the approximately 300 guidelines, but they picked 30 that they organized into 5 groups and explained them and some related matters in detail. Their goal in sharing these 30 guidelines is not to teach you the C++ syntax but rather how to improve your style.

The 5 groups are:

- Bikeshedding Is Bad
- Don't Hurt Yourself
- Stop Using That
- Use This New Thing Properly
- Write Code Well By Default

I think most of these titles are self-evident, except for the first.

At least to me. 

I had to look up what bikeshedding means. It turns out that Parkinson observed that a committee whose job is to approve plans for a nuclear power plant may spend the majority of its time on relatively unimportant but easy-to-grasp issues, such as what materials to use for the staff bikeshed while neglecting the design of the power plant itself, which is far more important but also far more difficult to criticize constructively.

Having a look at the rules Kate and Guy chose for this section, I still don't understand what exactly they meant. It's probably that unimportant issues shouldn't bog you down.

Just like a section title! ;)

Apart from this section title, I think the book is very clear. And after all, not understanding the title is more about my level of English ...

## Getting to the details

Let's have a deeper look at 4 chapters of the book.

### Where there is a choice, prefer default arguments over overloading

I often find people mixing up the words *parameters* and *arguments*. Sometimes they don't realize it. Sometimes they are well aware that something is probably not okay. Before they have to use the word, they slow down, they say it slowly, they look around and then they continue. I used to be like that.

Reading this chapter fixes that knowledge gap for good.

> *"Before we start, we want to remind you of the difference between a parameter and an argument: an argument is passed to a function. A function declaration includes a parameter list, of which one or more may be supplied with a default argument. There is no such thing as a default parameter."*

It was worth already worth reading this chapter just for that. But there is more!

[F.51](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f51-where-there-is-a-choice-prefer-default-arguments-over-overloading
) is about how you should make a choice between default arguments and overloading. The story supporting this chapter is about a function called `make_office()` that grows in complexity over time. With the growing complexity, the number of function parameters also grows and we learn about what can go wrong. Due to the subtleties of overload resolution and unambiguity of default arguments, overloading is discouraged.

One thing surprised me though. They discourage introducing enums instead of `bool` parameters. I find their counterexample actually more readable and I was quite convinced by [Matt Godbolt's talk that also touched this point](https://youtu.be/nLSm3Haxz0I?t=1110).

Still, I perfectly agree with their final conclusion. If you have a chance, instead of new overloads, extra `bool` or `enum` parameters, default arguments, prefer to introduce new functions with clear and descriptive names.

### Avoid trivial getters and setters

In the early days of C++, it was perfectly normal to write classes that exposed all their private variables with getter and setter functions. I'm not that old, but even I saw that a lot. Moreover, I saw IDEs - mostly for Java - generating those for you.

But does that help emerge proper abstraction levels and interactions between classes?

I leave that here as a theoretical question.

The only reason how this might help you is that you can set breakpoints with your debuggers reporting when a member is accessed or modified.

As [C.131 says, we should avoid trivial getters and setters](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#c131-avoid-trivial-getters-and-setters). They add nothing meaningful to the interface, they are nothing but noise.

If you really want to go with fully exposed members then prefer using a struct where they will be public by default and avoid adding any business logic.

Otherwise, use better names than simple setters and getters. Come up with abstractions that don't only do the trivial but ensure having proper class invariants. For example instead of `void Account::setBalance(int)`, introduce `void Account::deposit(int)` and `void Account::withdraw(int)`.

### Specify concepts

One of the flagship features of C++20 is concepts. They let you formalize requirements towards template arguments. This is a feature that we should definitely use as much as possible. The core guidelines go as far as [T.10](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t10-specify-concepts-for-all-template-arguments) says that one should specify concepts for **all** template arguments.

We should formalize how a template argument will be used, and what kind of characteristics an API, a type must have. Doing so will help the reader in two ways.

First, the reader will understand easier with what kind of types a template can be used. Second, the compiler will check earlier if an argument is valid for a given template and it will generate error messages at the point of the call, not at the time of instantiation. As such, the developer will get errors in a more timely manner. Besides, the errors due to unsatisfied requirements are more readable than the good old errors of failed template instantiations.

If you want to learn more about concepts check out my book on [C++ Concepts](https://leanpub.com/cppconcepts/).

### Prefer immutable to mutable data

Last but not least, let's talk about constness a bit.

[P.10](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p10-prefer-immutable-data-to-mutable-data) is about constness from a philosophical approach. By that I mean that it's not about how and when you make variables `const`. It's simply about the fact that it's easier to reason about immutable data. You know that no matter what, it will not change.

And in fact, [P.10](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#p10-prefer-immutable-data-to-mutable-data) goes only so far. On the other hand, the chapter dedicated to it goes much further. The authors suggest making objects and member functions `const` **wherever** you can. They also explain the differences between `const` pointers and pointers to `const`s. They speak about the differences between *east `const`* and *`const` west*.

It's a bit like a short version of my book [How to use `const` in C++](https://leanpub.com/cppconst/).

In a subsequent chapter, they also discuss [ES.22](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#es22-dont-declare-a-variable-until-you-have-a-value-to-initialize-it-with) which suggests *not to declare a variable until you have a value to initialize it with*. While this is not strongly about constness, they also show techniques for how to turn variables following the [initialize then modify anti-pattern] into `const`-initialized ones. Someone, it's as easy as declaring the variable later, but you might have to add a new constructor, use a ternary operator or even [immediately invoked lambda expressions](https://www.sandordargo.com/blog/2020/02/19/immediately-invoked-lambda-functions).

All in all, [Beautiful C++](https://www.amazon.com/Beautiful-Core-Guidelines-Writing-Clean/dp/0137647840?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=4a574de3a8d52ce616b3bdddd26e0c8c&camp=1789&creative=9325) offers lots of ways to make your code more `const`-correct.

## Conclusion

[Beautiful C++](https://www.amazon.com/Beautiful-Core-Guidelines-Writing-Clean/dp/0137647840?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=4a574de3a8d52ce616b3bdddd26e0c8c&camp=1789&creative=9325) is a very interesting book about how to write more readable, more maintainable C++ code. You'll find 30 handpicked guidelines from the Core Guidelines in the book. The authors explained each of those in detail how and why to apply them.

If you're looking for your first C++ book, probably this is not the one to choose. It won't teach you the basics of the language. But it's a perfect second book. If you follow the pieces of advice of the authors, you'll write better code than most of your fellow developers.

A highly recommended read!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
