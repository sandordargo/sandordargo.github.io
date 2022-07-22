---
layout: post
title: "Why to use C++ in 2022"
date: 2022-X-X
category: books
tags: [cpp, beginners]
excerpt_separator: <!--more-->
---
C++ is a programming language that is roughly 40 years old and it's still inescapable. In this article, we are going to see where and why it's used and whether it has a future or not.

Let's get into it!

## Where C++ is used nowadays?

C++ is everywhere. Code written in C++ is on your phone, in your washing machine, in your car, in the aeroplanes, in your banks, really everywhere.

Let's get a bit more specific. Many applications for image manipulations are, like Adobe Photoshop or Illustrator are written in C++. 3D games are also often written in C++. 3D animations, simulations, and rendering software are also mainly coded in C++. Image manipulation is a somewhat complex and resource-intensive field and it needs the fastness and hardware closeness of C++.

But images are not the only field, there is a fair chance that the browser you are using to read this was also written in C++, such as Chrome or Firefox.

If we go even lower and look into compilers or the operating system, many of them are written in C++. If not, it'll most probably be C.

But that was only the desktop world.

In the world of enterprise software, you'll of course find other languages, but where performance is critical, C and C++ are the default choice for a good reason.

In the embedded world, where both memory and CPU are more limited than on a desktop, C++ thrives. No matter if you look at your smartwatch, your phone, or you turn on your washing machine or get into your car and turn on the ignition, you should feel a bit of grate for the unknown C++ developer who managed not to crash it with a segmentation fault right after startup.

## Why C++ is used?

So we saw that C++ is still used pretty much anywhere. But why? There are so many sceptics out there who think it's legacy and should be removed from most of the modern company's codebase.

### Because of legacy?

So is C++ still used only because it's the inherited technology of old applications? By old, I often mean decades-old software.

The answer is partly yes.

Think about the [Cobol Cowboys](http://cobolcowboys.com/). Few people know Cobol so if there is a need, they can earn loads of money. 

And there is a need!

Cobol is still widely used in the financial industry. Those systems were written long decades ago and they still work pretty well. Maybe they don't support all the modern requirements, but they are robust, reliable and so complex that nobody dares to rewrite them.

C++ is not so bad, it's not as old as Cobol, and there are more people learning it and knowing how to use it.

But it's sometimes only used because companies are so heavily invested in it, they have their whole ecosystems evolved around C++ that it'd be very costly to migrate away and whatever some people think who don't find it sexy enough is economically unjustifiable.

But is C++ such a legacy?

### C++ is evolving

Not at all! C++ is predictably evolving. As I explained in detail in one of my previous articles, since 2011 C++ follows a train-like model. Every three years there is a new version released with new language and library features and with bug fixes of earlier features.

The release schedule and the standardized work guarantee that the versions are the results of well-thought additions, instead of ad hoc decisions. Compiler implementors have the time to implement them properly and also the community has the time to adapt.

At the same time, one of the C++ superpowers is backward compatibility - which is a double-edged sword - coded that compiled yesterday will most probably compile tomorrow. In fact, code that was compiled in 1985 will most probably compile in 2025.

The evolution of C++ has been targetting to remove the pain points of the developers and write modern code easier.

One of the most important features of C++ is predictable memory management. There is no garbage collection that happens whenever (or not). It's deterministic when and how memory will be released and given back to the operating system. While it's perfectly deterministic it was also quite easy to shoot yourself in the leg and mess it up by not freeing up the memory or trying to release it twice or even more...

Modern C++ introduced smart pointers that made dynamic memory management way less error-prone by introducing pointers that can clean up after themselves. 

Another pain point for many developers has been related to templates. SFINAE, incredibly long and difficult to read error messages are less and less of a problem with the introduction of C++20 concepts that help us constrain the types accepted by templates and provide relevant and relatively easy to read error messages if something still goes wrong.

During the last years, there were some big works ongoing to introduce the `<ranges>` library with which we can replace very procedural loops with a functional style.

### The economic advantage...

C++ is close to the hardware, can easily manipulate resources, provide procedural programming over CPU intensive functions and is fast. It is also able to override the complexities of 3D games and provides multilayer networking. All these benefits of C++ make it a primary choice to develop gaming systems as well as game development suites.

If you use a so-called modern language such as Python or Javascript, often you'll fall back to writing some critical functionalities or libraries in C or in C++ just to make its speed acceptable.

There are very few languages that can compete with C++ in terms of speed and one of them is C.

But speed is not everything.

You might say that well, you don't care about the speed that much. I only want to serve that many transactions, I don't have such constraints on speed. I prefer code that is easy to develop.

Understandable.

As we saw earlier, C++ is becoming easier and easier to develop. Of course, the ease of writing modern C++ is nowhere like that of Python, but things are not black and white.

Some modern languages provide the ease of writing code, some others provide great functionalities.

When you choose a car, you don't only think about comfort or speed, though those can be quite important. More often than not, you'll also have to think about fuel consumption. Again, the C/C++/Rust trio is way better than all the other languages. Basically, they play a different game.

PIC 

The above numbers are quite impressive.

Now let's look at this slide that was presented at CPPP by Damien Buhl.

PIC 

By using C++, we can save lots of CO2 emissions which is quite shocking.

So it seems that in many cases even if your performance requirements do not, energy consumption and environment protection do suggest/imply the usage of C++.

## What are the drawbacks?

If C++ is evolving and becoming easier to write and in addition, it's even good for energy bills and therefore the planet, what's the problem? Why are people reluctant to use it?

Let's see a couple of things.

### Bad press...

Let's face it, C++ has a bad reputation.

If you read [Coders At Work](), there are many who wrote that C and C++ are too difficult to use and there are only a few reasons to do. With C it's very easy to shoot yourself in the leg, with C++ it's a bit more difficult, but when you do, you blow it completely off.

Not the nicest things to say.

They were definitely true, but they are less and less so.

But the books and the interviews will not go away and it's very difficult to change public opinion, especially among those who don't code anymore. Like most of the decision-makers.

### While the language is evolving, it's getting more difficult to learn

As I wrote several times earlier, C++ is evolving, it gets more and more features, and it's becoming easier and easier to write expressive code.

What used to be a raw loop today can be written in such a functional way:
```cpp
// instead of
const std::vector<int> numbers = {1, 2, 3, 4, 5};

auto count = 0;
for (const auto& n : numbers) {
  if ( n % 2 == 0) {
    ++count;
  }
}

// know we can write
auto count = std::ranges::count_if(
    numbers, 
    [](auto n){
        return n % 2 == 0;
    }
);
```

While this is all nice, it also means that those who want to write better C++ code, have to learn more. Many think that C++'s biggest superpower is the fact that it's almost completely backwards compatible. Such an important feature that Matt Godbolt dedicated to it almost all [his keynote at CPPP 2021](https://www.codereckons.com/keynotes/c%2B%2B's-superpower)!

Certain practices that used to be considered the best ones became worst over time. But they still compile, they are still valid syntax, usually basic syntax, so we have to learn them. Maybe you don't need to do pointer arithmetics anymore at least not so much, but you still need to know about it. The same goes for manual memory management, C-style arrays, end so on. 

I do think that such topics should not be taught in-depth, but from what I can see, most universities, teach legacy C++ and people have to relearn modern C++ at companies. If they are even reminded that there is a more modern version.

### Intellectual neglige 

TO BE WRITTEN

### Ecosystem

And the final point, probably the most important one. If you work with Python, you know exactly how and from where you should get your packages. That's similar to Java, not to mention Javascript. These languages are not standardized, but they have standard ways to easily deliver and use libraries, to share and build code in a way that doesn't take a long time to grasp.

That's clearly not the case for C++.

Writing make files is difficult. Many consider CMake as the de facto standard to write build scripts, but it's clearly not the case. Many don't like it due to its syntax and there are many different ways to generate your build scripts. Many companies have their own systems - including Amadeus.

What about package management?

Well, there is Conan, vcpkg, but they are not having the maturity as yarn, npm, PyPI or maven is.

C++ has still a long way to go.

## So the future?

I asked around among some of the prominent figures of the C++ community, here is what they said:

*C++, today, remains truer than ever to its original mission of providing zero-cost abstractions over low level systems code, where possible, and low-cost abstractions that you only pay for when you use them the rest of the time. It does this while maintaining compatibility with C and earlier versions of C++ - despite constantly evolving and adopting surprisingly modern language features.* - Phil Nash, author of Catch2, main organizer of C++ On Sea

*C++ is used in game development extensively, for console and PC games particularly. It allows direct access to hardware through zero cost abstractions. The power and flexibility it grants makes it a hard language to learn, because more decisions are placed in your hands about what to do. As an international standard with a commitment to backwards compatibility, you know that there isn't going to be a Python2/Python3 situation. The future is looking bright, with concurrency and networking looking promising for C++26 and a host of features designed to streamline and simplify the language.* - J. Guy Davidson, Head of Engineering Practice at Creative Assembly, Co-author of Beautiful C++, ISO C++ Committee voting member

MORE TO COME, WRITE A CONCLUSION
