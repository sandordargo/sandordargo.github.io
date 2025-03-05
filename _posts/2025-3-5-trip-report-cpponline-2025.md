---
layout: post
title: "Trip report: C++ Online 2025"
date: 2025-3-5
category: books
tags: [cpp, conference, cpponsea, tripreport]
excerpt_separator: <!--more-->
---
Right at the very end of February, the C++ conference season was opened by the second edition of [C++ Online](https://cpponline.uk/). I think it won't take you by surprise that this was an online conference. In a certain way, an online conference offers less than one that happens in real life, one must admit that it has some advantages.

First of all, it's open for more people. It requires less investment money- and time-wise, and people can easily attend from different time zones. You can attend even if you don't manage to get dedicated time from your employer. Some talks - depending on your timezone - were outside of business hours, plus the raw recordings were published very promptly so you could watch something that you missed even on the same day.

This came in handy for me, as I could not dedicate so much time attending talks live as I wanted to. To add a personal side-note, I also wanted to be more active promoting the conference, but a thigh-packed vacation with not much access to the internet prevented me from doing so.

Now it's time to share my favourite talks!

## My favourite talks

Here are my favourite talks from the conference listed in chronological order.

### [The Art of C++ Friendship](https://cpponline.uk/session/2025/the-art-of-cpp-friendship/) by Mateusz Pusz

Mateusz is a professional trainer and also an author of [the quantities and units library for C++](https://github.com/mpusz/mp-units). It is always a pleasure to attend his talks. At C++ Online, he talked about how to use `friend` in C++.

In case you're not familiar with the concept of a `friend` in C++, it grants a function or class access to `private` and `protected` members of another class.

While *"`friend` is a powerful tool, but like art, it requires skill, understanding and careful application"*. It's easy and tempting to let others access your class internals, it will break encapsulation and a good design. But if it's used well, it enables a better design than having getters for example.

One typical usage of `friend` is for binary operations, which *"should typically be overloaded as __non-member functions__ which allows implicit conversion (if any) for both of the arguments."*

The talk starts with the basics of C++ friendship, but it goes in depth and offers valuable parts for more experienced developers as well. Learning about the *Passkey idiom* or the *Attorney-Client idiom* might be valuable for many. We'll cover those in depth in a later article. 

### [Practical Tips for Safer C++](https://cpponline.uk/session/2025/practical-tips-for-safer-cpp/) by Tristan Brindle

C++ was not designed to be a safe language, probably we can agree on that. This - lack of - characteristic has been quite in the center of the C++ community's focus during recent years. Tristan's talk is a useful collection of ideas on how one can use C++ in a safe(r) way.

But what is safety?

He used Dave Abrahams' definition of safety:

> *"A safe operation cannot cause undefined behaviour"*

Tristan covered different types of safety, such as numeric, bounds and memory safety and also went into the question of testing.

I came away with some ideas to dig into more, such as using the `-fwrapv` compiler flag to give wrapping semantics for integers, just like it is in Rust's release mode. 

Another good piece of advice is to write `constexpr` code is a good idea because `constexpr` code cannot have undefined behaviour.

Yet, Tristan reminded us that C++ is inherently memory unsafe. While certain languages use the Law of Exclusivity to ensure memory safety and in principle it would turn C++ to a memory safe language too, it's very difficult to adhere to it on a large scale.

### [Adventures with Legacy Codebases](https://cpponline.uk/session/2025/adventures-with-legacy-codebases/) by Roth Michaels

We spend so much time chasing new features - on this blog as well. At the same time, most of us work with legacy codebases. Thriving in such environments makes someone a good engineer. In his talk, Roth shared a couple of insights based on his experience working with legacy code.

In my opinion, perhaps his most important advice was that when you refactor code before, during or after writing some feature code, separate the refactoring changes from the feature changes. At the bare minimum, commit them separately, but preferably they should be part of separate pull requests so that your changes are easier to review and easier to understand what should and what should not alter functionality. 

Another practical advice was to always have a `.clang-format` file committed in your repository. Even if you don't have hooks that would reformat your code, even if the pipelines don't contain any checks. It will help the contributors to write code in a style that is actually welcome.

Still regarding `.clang-format`, Roth proposed to reformat only new code to avoid having one massive commit which modifies the whole codebase. For the ease of tooling, I always went with the other approach, but I also see the value of his proposition.

An idea that really resonated with me is to have a `#red-diff-appreciation-society` where you can celebrate PRs which mostly or only remove code. Don't forget, each line of code is a liability, something to maintain, something that can potentially be the source of bugs. Celebrate if someone takes the time and effort to remove code. 


## My talk: [Clean Code! Horrible Performance?](https://cpponline.uk/session/2025/clean-code-horrible-performance/)

I also had the opportunity to give a talk, which was about the relationships of clean code performance.

Clean code is just an optimization for readable code. Optimization is always about compromises. While it's true that clean code is not the fastest of all possible codes, most of us don't need the last bits of performance. We have more important aspects to worry about. Maybe it's binary size, maybe it's readability, maybe it's something else. In my opinion, readability is the most important by default. Many of us work in big corporations, where people come and go.

It's also worth considering that the cost of a CPU cycle keeps going down while developers are getting more expensive. Therefore it's worth optimizing for developer productivity rather than CPU cycles. And code readability, clean code, helps making us more productive.

> ![Growing salaries, decreasing CPU costs]({{ site.baseurl }}/assets/img/cpu-cost-developer-salaries.png "Growing salaries, decreasing CPU costs")
> _Growing salaries, decreasing CPU costs_

Even when you have to optimize, do it wisely. Most probably you won't need such low-level optimization [as the developers working on Quake needed](https://www.youtube.com/watch?v=p8u_k2LIZyo). In most cases, optimizing external requests, such as DB operations and network calls will leave you with a good enough solution.

## Conclusion

I was glad to join [C++ Online 2025](https://cpponline.uk/) and attend some great talks. Though I couldn't attend in live as many as I wanted, at online conferences, it's easy to catch up at the end of the day. Also, the platforms used helped the participants to connect with each other. It was great to meet virtually some people I haven't seen since some real-life conferences last year.

Thanks a lot to the organizers for putting C++ Online together.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!