---
layout: post
title: "Why to use C++ in 2022"
date: 2022-11-9
category: books
tags: [cpp, watercooler]
excerpt_separator: <!--more-->
---
C++ is a programming language that is roughly 40 years old and it's still unavoidable. In this article, we are going to see where and why it's used and whether it has a future or not.

Let's get into it!

## Where C++ is used nowadays?

C++ is everywhere. Code written in C++ is on your phone, in your washing machine, in your car, in airplanes, in your banks, and really everywhere.

Let's be a bit more specific. Many image-manipulating applications like Adobe Photoshop or Illustrator are written in C++. 3D games are also often written in C++. 3D animations, simulations, and rendering software are also mainly coded in C++. Image manipulation is a somewhat complex and resource-intensive field and it needs the fastness and hardware closeness of C++.

But images are not the only field, there is a fair chance that the browser you are using to read this was also written in C++, such as Chrome or Firefox.

If we go even lower and look into compilers or the operating system, many of them are written in C++. If not, it'll most probably be C.

But that was only the desktop world.

In the world of enterprise software, you'll of course find other languages, but where performance is critical, C and C++ are the default choice for a good reason.

In the embedded world, where both memory and CPU are more limited than on a desktop, C++ thrives. No matter if you look at your smartwatch, your phone, or you turn on your washing machine or get into your car and turn on the ignition, you should feel a bit of grate for the unknown C++ developer who managed not to crash it with a segmentation fault right after startup.

## Why C++ is used?

So we saw that C++ is still used pretty much anywhere. But why? There are so many skeptics out there who think it's a legacy and should be removed from most of the modern company's codebase.

### Because of legacy?

Some people claim that C++ is still used only because it's the inherited technology of old applications. By old, I often mean decades-old software.

It's only partly true.

Think about the [Cobol Cowboys](http://cobolcowboys.com/). Few people know Cobol so if there is a need, they can earn loads of money. 

And there is a need!

Cobol is still widely used in the financial industry. Those systems were written long decades ago and they still work pretty well. Maybe they don't support all the modern requirements, but they are robust, reliable and so complex that nobody dares to rewrite them.

C++ is not so bad, it's not as old as Cobol, and there are more people learning it and knowing - more or less - how to use it.

But it's sometimes only used because companies are so heavily invested in it. They have their whole ecosystems evolved around C++. It'd be very costly to migrate away. Even decision-makers who don't find C++ sexy enough would find such a migration economically nonsense.

But is C++ such a legacy?

### C++ is evolving

Not at all! C++ is predictably evolving. As I explained in detail [in one of my previous articles](https://www.sandordargo.com/blog/2022/06/29/cpp-standardized), since 2011 C++ follows a train-like model. Every three years there is a new version released with new language and library features and with bug fixes of earlier features.

The release schedule and the standardized work guarantee that the versions are the results of well-thought additions, instead of ad hoc decisions. Compiler implementors have the time to implement them properly and also the community has the time to adapt.

At the same time, one of the C++ superpowers is backward compatibility. Code that compiled yesterday will most probably compile tomorrow. In fact, code that compiled in 1985 will most probably compile in 2025.

The evolution of C++ has been targetting to remove the pain points of the developers and write safer code easier.

One of the most important features of C++ is predictable memory management. There is no garbage collection that happens eventually (or not). It's deterministic when and how memory will be released and given back to the operating system. While it was always perfectly deterministic, it was also quite easy to shoot yourself in the leg and mess it up by not freeing up the memory or trying to release it twice or even more times...

Modern C++ introduced smart pointers that made dynamic memory management way less error-prone by adding pointers that can clean up after themselves. 

Another pain point for many developers has been related to templates. [SFINAE](https://en.cppreference.com/w/cpp/language/sfinae), incredibly long and difficult-to-read error messages are less and less of a problem with the introduction of [C++20 concepts](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations) that help us constrain the types accepted by templates and provide relevant and relatively easy-to-read error messages if something still goes wrong.

During the last years, there were some big works ongoing to introduce the [`<ranges>` library](https://en.cppreference.com/w/cpp/ranges) with which we can replace very procedural loops with a functional style.

### The economic advantage...

C++ is close to the hardware, can easily manipulate resources, provide procedural programming over CPU-intensive functions, and is fast. It is also able to handle the complexities of 3D games and it provides multilayer networking. All these benefits of C++ make it a primary choice for developing gaming systems as well as game development suites.

If you use a so-called modern language such as Python or Javascript, often you'll fall back to writing some critical functionalities or libraries in C or in C++ just to make their speed acceptable.

There are very few languages that can compete with C++ in terms of speed and [one of them is C](https://thenewstack.io/which-programming-languages-use-the-least-electricity/?s=09).

But speed is not everything.

You might say that well, you don't care about the speed that much. You only want to serve that many transactions, you don't have such constraints on speed. You prefer code that is easy to develop.

Understandable.

As we saw earlier, C++ is becoming easier and easier to develop. Of course, the ease of writing modern C++ is nowhere like that of Python, but things are not black and white.

Some modern languages focus on the ease of writing code, some others focus on great functionalities.

When you choose a car, you don't only think about comfort or speed, though those can be quite important. More often than not, you'll also have to think about fuel consumption. Do we do the same thing when we write an application? Do we think about how much energy they would consume? In that sense, the C/C++/Rust trio is performing way better than all the other languages. Basically, they play a different game.

![Carbon footprint]({{ site.baseurl }}/assets/img/energy-time-memory.png)

The above numbers are quite impressive.

Now let's look at this slide that was presented at CPPP by Damien Buhl.

![Carbon footprint]({{ site.baseurl }}/assets/img/carbon_footprint.png)

By using C++, we can save lots of CO2 emissions which is quite shocking.

So it seems that in many cases even if your performance requirements do not, energy consumption and environment protection point towards the usage of C++.

## What are the drawbacks?

If C++ is evolving and becoming easier to write and in addition, if it's even good for energy bills and therefore the planet, what's the problem? Why are people so reluctant to use it?

Let's see a couple of things.

### Bad press...

Let's face it, C++ has a bad reputation.

If you read [Coders At Work](http://amzn.to/2wKEeVt), there are many who wrote that C and C++ are too difficult to use and there are only a few reasons to do so. With C it's very easy to shoot yourself in the leg, with C++ it's a bit more difficult, but when you do, you blow your leg completely off.

Not a very reassuring thing to say.

These comments were definitely true, but they are less and less so.

The language evolves, but the old books and the interviews will not go away. It's very difficult to change public opinion, especially among those who don't code anymore. Like most of the decision-makers.

### While the language is evolving, it's getting more difficult to learn

As I wrote several times earlier, C++ is evolving. It gets more and more features, and it's becoming easier and easier to write expressive code.

What used to be a raw loop today can be written in such a functional way:
```cpp
const std::vector<int> numbers = {1, 2, 3, 4, 5};

// instead of
auto count = 0;
for (const auto& n : numbers) {
  if ( n % 2 == 0) {
    ++count;
  }
}

// now we can write
auto isEven = [](auto number) { return number % 2 == 0; };
auto count = std::ranges::count_if(numbers, isEven);
```

While this is all fine and dandy, it also means that those who want to write better C++ code, have to learn more. Many think that C++'s biggest superpower is the fact that it's almost completely backward-compatible. Such an important feature that Matt Godbolt dedicated to it almost all [his keynote at CPPP 2021](https://www.codereckons.com/keynotes/c%2B%2B's-superpower)!

It's true that certain old best practices became antipatterns over time. But they still compile, they are still valid syntax, usually basic syntax, so we have to learn them. Maybe you don't need to do pointer arithmetics anymore at least not so much, but you still need to know about it. The same goes for manual memory management, C-style arrays, and so on. 

I do think that such topics should not be taught in-depth, but from what I can see, most universities teach legacy C++ and people have to relearn modern C++ at companies. If the company happens to use a more modern version...

### Intellectual neglige 

As [Marek Krajewski shared with me on Twitter](https://twitter.com/mrkkrj/status/1467798371670925315), some people would simply not use C++ out of intellectual laziness. Yes, it's more difficult to learn than Python or Javascript. Yes, you can build great things with the simpler-to-learn alternatives. And in fact, you don't always need the powers of C++. That's all true.

***You should use the right tool for the right job.***

The problem is that many are simply lazy to learn those tools or accept that sometimes those are the right tools. This happens often because of bigotry, because of narrowmindedness, and basically because of intellectual negligence.

It's our job to show, to explain when C++ (or Rust...) is an overkill, and when it is the right solution. More importantly, we have to show that it's not the same language that it was.

### Ecosystem

In his talk, [C++ MythBusters](https://cpponsea.uk/2022/sessions/plenary-cpp-mythbusters.html), [Victor Ciura](https://twitter.com/ciura_victor) busted the myth that C++ is not easily toolable. It is toolable and we have plenty of tools. Victor thinks that we are never going to have *"standardized"* tools, we always have to find the right tool and understand how it works and leverage it. 

While I share his view, we must admit that other languages have simpler solutions for simple problems. If you work with Python, you know exactly how and from where you should get your packages. That's similar to Java, not to mention Javascript. These languages are not standardized, but they have standard ways to easily deliver and use libraries, to share and build code in a way that doesn't take a long time to grasp.

That's clearly not the case for C++.

Writing *makefiles* is difficult. Many accept [CMake](https://cmake.org/) as the de facto standard to write build scripts, but it's clearly not the case. Many don't like it due to its syntax and there are many different ways to generate your build scripts. Many companies have their own systems - including Amadeus.

What about package management?

Well, there is [Conan](https://conan.io/), [vcpkg](https://github.com/microsoft/vcpkg), but they don't have the maturity as [yarn](https://yarnpkg.com/), [npm](https://www.npmjs.com/), [PyPI](https://pypi.org/) or [maven](https://mvnrepository.com/) have.

C++ has still some way to go.

## So the future?

I asked around among some of the prominent figures of the C++ community, and here is what they said:

*C++, today, remains truer than ever to its original mission of providing zero-cost abstractions over low-level systems code, where possible, and low-cost abstractions that you only pay for when you use them the rest of the time. It does this while maintaining compatibility with C and earlier versions of C++ - despite constantly evolving and adopting surprisingly modern language features.* - Phil Nash, author of Catch2, main organizer of C++ On Sea

*C++ is both our legacy and our future.  For all its warts and historical problems, it has an abundance of modern features many of them specially designed to mitigate/replace old idioms/constructs. C++ programmers nowadays can easily write programs completely avoiding such perilous old things. [...] The C++ STL has grown a lot with the ISO standards 11,14,17,20 and more valuable additions are coming in C++23. From new algorithms and ranges to various utilities and support libraries for IO, networking, coroutines, concurrency, heterogenous parallelism and more. Yes, there are more specialized things a programmer might need, but this is where the C++ echo-system comes in to fill the gaps with a plethora of industry-grade (and stable) libraries for all sorts of necessities. Every important piece of software we use today has some C++ in it: maybe it’s all C++, maybe it has some important components in C++, maybe its library is natively compiled in C++, maybe it’s compiler/runtime is written in C++, ...
C++ is still the King of programming languages. Long live the King!* - [Victor Ciura](https://ciura.ro/blog/why-cpp), Senior Software Engineer in Visual C++ team at Microsoft

*C++ is used in game development extensively, for console and PC games particularly. It allows direct access to hardware through zero-cost abstractions. The power and flexibility it grants make it a hard language to learn because more decisions are placed in your hands about what to do. As an international standard with a commitment to backwards compatibility, you know that there isn't going to be a Python2/Python3 situation. The future is looking bright, with concurrency and networking looking promising for C++26 and a host of features designed to streamline and simplify the language.* - J. Guy Davidson, Head of Engineering Practice at Creative Assembly, Co-author of [Beautiful C++](https://devreads.sandordargo.com/beautiful-cpp-by-kate-gregory-and-guy-davidson/), ISO C++ Committee voting member


## Conclusion

C++ might be considered legacy by those who were familiar only with the old patterns, with the old standards, but the language is evolving. Since 2011, since C+11, we get a new version every 3 years with bugfixes and new features. The ecosystem is growing, even though it's far from being as straightforward as some of the other newer languages where for example package management is done the same way everywhere.

Still, the language, the ecosystem is growing, the community is strong and C++ is everywhere, it's unavoidable. It's somehow part of almost every written software. I'm not saying that C++ is the hammer that should make everything a nail around you, but it's something still worth learning and mastering. Even in 2022 and onwards. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
