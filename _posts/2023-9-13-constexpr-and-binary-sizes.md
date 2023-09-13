---
layout: post
title: "Constexpr functions for smaller binary size?"
date: 2023-9-13
category: dev
tags: [cpp, constexpr, binarysizes, undefinedbehaviour]
excerpt_separator: <!--more-->
---
I read recently ["constexpr functions: optimization vs guarantee"](https://andreasfertig.blog/2023/06/constexpr-functions-optimization-vs-guarantee/) by Andreas Fertig. I was a bit surprised by some of his claims about `constexpr` functions regarding binary size so I decided to go after those and check their validity. I don't want to raise the suspense, he was more right than not. When I read his article, I overlooked certain details. Let's get into the details.

## Section header

Let me share some details from [Andreas' article](https://andreasfertig.blog/2023/06/constexpr-functions-optimization-vs-guarantee/).

> What you see here is still an optimization. Yes, if you are interested in a small binary footprint, you can be happy. But, `constexpr` can give you more! You can get guarantees from `constexpr`. Let's explore that.

At this point, I was a little bit surprised. I remember when [we talked about object initializtion](https://www.sandordargo.com/blog/2023/01/18/object-initialization-and-binary-sizes) and we found that if an object can be created and initialized during compile-time, then our binary might become bigger as the variable - depending on its storage duration - might be part of the binary file.

And here I read something contrary. 

You might say that I cited my experience about object initialization but the article is about `constexpr` functions. You're right about that. Let's go further.

> The reason is that `constexpr` implies inline! Try for yourself, make `Fun` `inline`, and you will see exactly the same assembly output as when the function was `constexpr`.
> 
> Because of the implicit `inline`, the compiler understands that `Fun` never escapes the current compilation unit. By knowing that there is no reason to keep the definition around. Then, `Fun` itself is reasonably simple to the compiler, and the parameter is known at compile-time. An invitation for the optimizer, which it happily accepts.

At this point, I really started to scratch my head. [We move special member function definitions to the `.cpp` file](https://www.sandordargo.com/blog/2023/02/01/special-functions-and-binary-sizes), we use different [compiler and linker flags](https://www.sandordargo.com/blog/2023/07/19/binary-sizes-and-compiler-flags) to limit inlining so that we can get a smaller binary. And here I read that inlining `Fun` can help.

I decided to measure the size in different scenarios.

## Using `constexpr` will either not matter or help

Let's measure the implications of `constexpr` functions in three different scenarios. We are going to use the utility in one single place, then in multiple translation units and finally, we'll consume it through a dynamic library.

### In a local scope it won't matter

First, I took the original example and measured the size of the generated binary.

```cpp
constexpr auto Fun(int v) // tried also without constexpr and only with inline
{
  return 42 / v;
}

int main()
{
  const auto f = Fun(6);

  return f;
}
```
The differences were not big, but they proved Andreas' point.

|   Version       | Binary size in bytes | 
|-----------------|--------------|
|  constexpr      | 16,856         |
|  inline         | 16,856      |  
|  no modifiers   | 16,904       |

It's probably worth noting that when you measure binaries with so little difference in size, the filename of the binary file also matters, so I made sure that they are of the same length.

Indeed, the `inline` and `constexpr` versions are a bit smaller. If we have a look at the assembly, we'll see a bigger difference in size and we can observe that the version without `constexpr` and `inline` contains the necessary code for `Fun`, while the other two do not. The necessary calculations happened during compile time, yet the gain in binary size is insignificant.

Fair enough, but we rarely use our functions only once in a `constexpr` environment.

Let's have a look at a more elaborate example.

### Binary sizes are barely affected by multiple translation units

It clearly contradicts my experience what Andreas wrote about `inline`, so I went further and extended his example. I defined a utility header containing the `Fun` function and created three `.h/.cpp` pairs where the utility is included and used by each implementation file.

The results were surprising. Without optimization, the non-`constexpr` version resulted in a smaller binary. When the optimization was turned on, the `constexpr` version was the smaller one. But the difference was below a hundred bytes. In this case that was below even `0.3%`.


|   Version       | Binary size in bytes | 
|-----------------|--------------|
|  constexpr -O0     | 39,761         |
|  non-constexpr -00  | 39,681      |  
|  constexpr -03     | 39,425         |
|  non-constexpr -03  | 39,475      |  

What made this result more interesting was that when I compiled with the `-S` flag to get the intermediary assembly code, the `.s` files of the `constexpr` version were never bigger than those of the non-`constexpr` version.

But when they are compiled together, the relation slightly changes.

At this point, the body of `Fun` was a simple `return` of a division. When I replaced it with 5 different additions to the parameter before returning it, nothing changed.

In the end, we didn't gain anything in terms of binary size.

### `constexpr` functions matter when distributed via a shared library

What we saw in the previous section was probably a bit more realistic usage of a utility function than the original example of a single usage. Even though I think that one single usage can already justify the existence of a function in the name clean code and readability.

The next step is taking the previous example and compiling each `.h` / `.cpp` pair into its own shared library and then linking them together.

This way the `constexpr` version has a clear and significant advantage!

The executable has the exact same size in both cases. Even the `libutils.so` that contains (only) `Fun` has the same size regardless of `Fun` being `constexpr` or not. But all the other shared objects that depend on `libutils.so` are about twice as small if `Fun` is `constexpr` (16,778 bytes vs 33,370 bytes compiled with -O3). As such, the `constexpr` version is overall 119,329 bytes vs. 185,697 bytes of the non-`constexpr` version.

My reasonings about this is the following:
- `libutils.so` has no difference in size as the implementation of `Fun` has to be distributed
- there is a difference in size for the consumers as the computation is done - in our use-case - at compile-time, no code has to be kept for run-time
- the main executable is just calling the intermediary shared objects, their size makes no difference to it.

## Conclusion

Using `constexpr` might matter and according to my measurements, it will never hurt the binary size. In a bigger codebase, when code is distributed through libraries it can significantly help you reduce the binary size.

We also have to keep in mind that using `constexpr` is not only about limiting the binary size. We cannot forget about how much it helps with compile-time evaluation and with template metaprogramming in general. Besides, as [Andreas pointed out](https://andreasfertig.blog/2023/06/constexpr-functions-optimization-vs-guarantee/) `constexpr` also detects undefined behaviour.

Its potential help with binary sizes is only an addition. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!