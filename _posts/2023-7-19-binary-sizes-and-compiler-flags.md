---
layout: post
title: "Binary sizes and compiler flags"
date: 2023-7-19
category: dev
tags: [cpp, binarysizes, compiler, linker]
excerpt_separator: <!--more-->
---
During the last few months, we discussed a lot about how to write code to limit the size of our binary. [We should think twice if we want to turn a class into polymorphic](https://www.sandordargo.com/blog/2023/02/08/binary-sizes-and-virtual), [exceptions also take a heavy toll](https://www.sandordargo.com/blog/2023/03/29/binary-size-and-exceptions) and we might end up [defaulting special functions in the cpp file](https://www.sandordargo.com/blog/2023/02/01/special-functions-and-binary-sizes) which is quite unintuitive.

There are practices going against common sense and also that can be considered a best practice in any case. I think it's good to know about them.

But how you compile your code is also important! Something I briefly mentioned in one of the first parts when it comes to optimization levels. But at that time we only talked about `-Os` and `-Oz`. Interestingly, they didn't work well in my local experiments. `-O3` almost always produced better results and never worse. Maybe my scenarios were not big enough.

A few weeks ago, I presented my findings at [MUC++](https://www.meetup.com/mucplusplus/events/293377912/), and I was focusing on the programming techniques. During the after-talk discussion, I was told that if binary size is really a concern for me, then I should definitely try certain not-so-well-known compiler and linker settings and they kindly gave me a few suggestions. Many of them had been already applied by others, but now at least I understand what they mean!

Let me share those with you too, so that you also know and can experiment with them.

## Optimize for space: `-Os`

I already mentioned `-Os` at the beginning of the series, but I think it's important to bring it up once again. One usually builds with `-O0` for debug builds to have the most information available for an eventual debugging session.

On the other hand, for release builds, one most often uses `-O3` to have an executable that is fast as possible - thanks to compiler optimizations. But if your goal is to have the smallest binary possible, you should consider `-Os` (or `-Oz` depending on the compiler). The compiler will *o*ptimize for *s*pace. It doesn't give up performance though. It'll do most of the optimizations that are done on `-O2` except for a few that would enlarge your binary, such as loop unrolling (we'll see that later).

## Link Time Optimization

What does the name Link Time Optimization (*LTO* in short) tell us? Not much to be honest. Some optimization would happen during link time, right?! Fair enough. It has another name which is a bit more revealing: Inter Procedural Optimization! So when LTO/IPO is enabled, the linker looks beyond individual object files and performs optimizations such as removing functions and even related global code and data that are never used. [Here you can find a nice example](https://llvm.org/docs/LinkTimeOptimization.html#example-of-link-time-optimization).

When LTO is enabled, source files are not directly translated to ELF/Mach-O or other object files, but they are translated into an intermediary bitcode. Out of those files, the linker can extract all the dependency information that it needs to finally optimize away some of the code and then it can create the optimized ELF/Mach-O/etc files that are optimized by the linker.

The fundamental way to trigger it is to pass `-flto`, but it has further options which are worth looking into when you decide to use it. These go beyond the scope of this article, but we might look into it in a future post.

## `-fdata-sections` / `-ffunction-sections` and `-Wl,--gc-sections`

First of all, let me explain why I mention 3 options right away in one section. `-fdata-sections` and `-ffunction-sections` are similar in their nature and they won't help you unless you pass `-Wl,--gc-sections` as well.

`-fdata-sections` and `-ffunction-sections` place each piece of global data item and function respectively into its own section in the object file. If you only use these, your final binary will be larger. But if you pass `-Wl,--gc-sections`, the unused sections will not be part of your compiled and linked file and you can end up with a smaller binary.

Now let's have a couple of remarks.

`-Wl,--gc-sections` seems odd as a parameter when you first see it. Passing `-Wl` to gcc or clang means that what you're passing after is meant to be an option for the linker, and not for the compiler. If you invoke the linker separately, you will have to pass `--gc-sections` on its own.

My second remark is about unused sections. If you read about these options, [people often mention "unused code"](https://stackoverflow.com/questions/4274804/query-on-ffunction-section-fdata-sections-options-of-gcc). That's misleading. You might think that this is a tool then to identify code that is written but nowhere used. That's not the case.

When the linker has to resolve a certain symbol - let's say `foo()` - in another object file, then once it finds it, it'll include the entire `.text` section. Potentially with lots of code that you don't use. Unused code in that sense. If each function and piece of global data have its own section, the linker will only include the needed sections thus you might gain a considerable amount of space.

And here is the third remark. Don't forget to measure. Depending on your code, you might not gain a lot of space and if that's your case, then don't use these because they make compilations and linking slower and if there are some "magic" sections that are needed for your program but they are nowhere referenced in the functions that you use, your program will misbehave. You can find more info on this [here starting from slide 9](https://elinux.org/images/2/2d/ELC2010-gc-sections_Denys_Vlasenko.pdf).

All-in-all, these options are really useful, just don't forget to measure and test first!

## `-Wl,--icf=all` or `-Wl,--icf==safe`

We often have functions whose bodies are almost identical. You might say that's not good and one should follow the DRY principle and don't repeat the same code over and over again, but it's not always the case, you might not want to reuse the same code in completely different contexts. In any case, you might end up with identical code.

Of course, when you want to bring down binary size, it feels like a big waste to store the generated code for identical functions.

Luckily, compiler and linker implementers already thought about that! What you might look for is _**i**dentical **c**ode **f**olding__, in short *icf*.

According to some Google experiments, this technique might reduce the size of your binary by 6%!

You can turn it on by passing `--icf=all` to your linker, or `-Wl,--icf=all` if you go through your compiler. We must note that ICF might cause some issues if you use functions as keys in a map for example. Then the same piece of code - which would normally be two different pieces of code - would practically be used as a map key twice which is obviously a problem. To avoid such issues, you can use `--icf=safe`. It will decrease a bit the gain, but the benefits will be still significant.

> When you want to benefit from ICF, you should also consider setting `-faddrsig`. This switch controls whether Clang (have you found something similar in GCC?) emits an address-significance table into object files. These tables allow the linkers to implement ICF without too many false positives. 

## `-mllvm -inline-threshold=<n>`

Clang lets you drive - or at least give a hint - how much it should inline functions through the `-mllvm -inline-threshold=<n>` setting where *n* is set to 255 by default. The corresponding compiler setting for gcc is `-finline-limit`.

The smaller *n* is, the less inlining will happen, thus the smaller your binary will be. On the other hand, by increasing the value of *n*, you will end up with more inlining, a bigger binary and potentially faster code.

Don't just play with this setting blindly, measure the effects to avoid bad surprises, both in terms of binary size and also regarding run-time performance.

## `-fno-unroll-loops`

A well-known optimization technique used by all compilers is to replace loops with their body repeated the necessary number of times.

```cpp
// the original loop

for (size_t i = 0; i < 5; ++i) {
	foo();
}

// the unrolled version
foo();
foo();
foo();
foo();
foo();
```

If someone would write the latter code, you'd never let it merge. Still, the compiler might do this optimization. It's bad for the binary size, but it might be better for the run-time there is no need to jump around and maintain a loop variable.

If you're desperate to gain a few bytes, you might try this!

## Conclusion

In this article, we reviewed compiler and linker options that help us reduce the size of the binary. Just don't forget to measure the effects, because these are usually compromises between the space your executable needs and the time it requires to execute. Which ones you can take are always based on your constraints.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
