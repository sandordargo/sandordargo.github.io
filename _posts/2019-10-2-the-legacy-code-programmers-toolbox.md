---
layout: post
title: "The Legacy Code Programmer's Toolbox by Jonathan Boccara"
date: 2019-10-2
category: books
tags: [books, refactoring, legacycode]
excerpt_separator: <!--more-->
---
[The Legacy Code Programmer's Toolbox](https://leanpub.com/legacycode) is the quite fresh e-book of [Jonathan Boccara](https://twitter.com/joboccara), the person behind [Fluent C++](https://www.fluentcpp.com/).

Not surprisingly it's about how to attack legacy code. The author gives practical examples of how we should digest such code and make it better for good. He doesn't just propose to make any legacy code better, but to decide which parts are worth to change.
<!--more-->

I've been reading just before the [Code as a crime scene](), and at certain points, I was not sure which book I was reading. I think that's a good sign for Jonathan.

When we speak about legacy code, it's always important to define what we mean by legacy code. [Micheal Feathers](https://michaelfeathers.silvrback.com/) defines such code as any code without tests. Boccara in this book took a bit different approach. In his, legacy code is defined by 3 qualities:

- hard for you to understand
- you are not comfortable changing it
- and you are somehow concerned with

Personally, probably I would still go with Feather's approach. I think I understand why the author focuses on that you have to be concerned with the code. We tend to go to the extremes, either we prefer to refactor nothing or to refactor all the legacy code we encounter. We simply cannot afford to refactor just any code without tests, so he pulls in to the definition that the developer in charge must be concerned by that piece of code.

Still, to me, legacy code remains legacy code even if we are not concerned with it. If a building is collapsed, but you have nothing to do with it, it's still a collapsed building. Do you have to rebuild it? Maybe not. Maybe nobody has to. Or not that moment, but the fact remains a fact.

Otherwise, I liked the book a lot. I'd like to highlight 3 pieces of advice he gave.


> "Avoid complaining if you don’t intend to improve the code."

Probably this is the most useful piece of advice he gives and it would be even better if we changed the word _code_ by the word _situation_.

> "One way to write better code is by reading code."

This is something I used to do. I used to keep [a pomodoro or two every week](http://sandordargo.com/blog/2018/02/28/setting-yourself-up-to-succeed) to read code, but I stopped due to some changing priorities on my side. Probably I should reconsider this. You might ask how to find good code, how do you know that something is good. Jonathan gives some ideas in his book. But you might look around your standard library whatever language you use.

> "As C++ expert [Tony van Eerd](https://github.com/tvaneerd) says, the minimum number of occurrences to justify putting a piece of code in its own function is 1

I love this one so much. I'm a huge fan of readable code involving highly descriptive variable and function names. I tend to extract functions even if I'd use them only once. This thought, this quote was exactly what I needed in a recent code review.

There is much more in [the book](https://leanpub.com/legacycode), but I cannot and don't want to uncover everything. I'd highly encourage to read the book, or if you'll attend [CppCon](https://sched.co/Sfnq), you can also go and check out author's presentation [10 Techniques to Understand Existing Code](https://sched.co/Sfnq) which will cover many of the ideas presented in the book.

Happy reading!