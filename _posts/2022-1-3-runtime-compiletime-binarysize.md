---
layout: post
title: "The run-time, compile-time, binary size triangle"
date: 2023-1-4
category: dev
tags: [cpp, binarysizes, executables, optimization]
excerpt_separator: <!--more-->
---
Often we start software development, we only care about writing some code. Or actually about writing the most amount of code that is possible. We often don't really know how to build C++ files, but nevertheless, it's semi-automated. We simply don't have to deal with it. Efficiency also doesn't matter as long as somebody starts to complain. Even if they do, you might be able to prove that it's not because of you, not because of the code you wrote. These complaints will be almost always about run-time efficiency and very rarely about the compilation time or the size of the binary.

But in each C++ developer's life, there is a moment when these start to matter.

In order to be able to talk about binary sizes, in the next weeks and months, let's start talking about why all these matters and how our executable gets generated.

My goal today is not to go into deep details on these questions, but to give you an overview, to put you in context.

## The run-time, compile-time, binary size triangle

We often read about run-time and compile-time efficiency, but we should also read about binary sizes, why all of these matter and how they correlate to each other.

It's pretty obvious why run-time performance matters. Who wants to wait more for the program to execute? Numbers are showing that the more you have to wait for an application to start, the more you have to wait for your video or song to start playing, and the more likely that you will not use the app ever again. If your server cannot handle the necessary amount of booking requests, customers will simply spend their money somewhere else. At the same time, there are applications where it doesn't really matter whether they take 10% longer to execute them. There are cases when you can sacrifice a bit of runtime.

What is better, having something slower once (to compile) and faster a million times (to execute) or the other way around? You'd rightly argue that having something slower once and faster every time ever after is the better choice. But again, it depends!

If your repository is huge and it's getting built over and over again hundreds or thousands of times a day, you might want to optimize for the compile time performance. Optimizing for the compile time does not necessarily mean avoiding techniques that are a bit slower to compile - for example, templates in C++. It can also mean organizing the code in a way that you can minimize the amount of code to rebuild each time. It can also mean introducing build caches to further decrease the chance to rebuild something.

I used to work in an environment where compile-time hardly mattered, and waiting 10-15 minutes for a CI pipeline to finish was not a problem. But when you work on bigger components where the CI can run for an hour or a lot more, compile-time counts a lot. While making the build faster by a few seconds does not change the world, continuous growth should still be avoided as all the little increases add up quickly and make the situation much worse. 

The size of the executable also raises similar questions. You might work with big servers where this question barely matters or actually it comes up in a different way. Maybe you think that the size of a binary does not matter, but you have to store several versions of the same shared library because you don't upgrade systematically all your components to use the same version of a big shared library. Instead, they rely on many different versions and soon you have storage issues. In such a scenario, the problem is both about binary sizes and about how you manage the dependencies of your components.

Or maybe you work in a constrained environment. Maybe your code has to fit the limited resources of a microcontroller and while run-time performance is important, the size of a binary is (also) a real bottleneck. Or maybe you work in a mobile environment where the size of the executable directly influences how long it takes to install the application and how long it takes every time to start it and load everything necessary into memory. You simply cannot afford longer startup times.

In the coming weeks and maybe months, let's study how the change of one of these factors influences the other two.

My assumption is that when we optimize for run-time performance, both the compilation time and the binary size will go up.

On the other hand, I also assume that we can make our executable smaller and at the same time, we can gain some compilation time while sacrificing a bit of our runtime performance.

My assumption - assuming that we are not removing horrible things, but we optimize decent code - is that we cannot improve all three measurements. Let's see if I'm right and if not, what can we do better?

## Conclusion

This week, I introduced a new series in which we are going to learn about executable formats, what kind of C++ code is going to increase the size of the executable and what is going to simply make runtime performance worse.

Today we discussed that runtime performance is not always the most important, or at least not the only bottleneck. I also shared my assumption that in normal circumstances we cannot improve compile-time, runtime and binary size at the same time.

In the next episode, we are going to discuss the structure of an executable file. Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!