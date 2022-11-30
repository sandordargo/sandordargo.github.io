---
layout: post
title: "C++23: auto(x) and decay copy"
date: 2022-11-30
category: books
tags: [cpp23, cpp, auto, decay]
excerpt_separator: <!--more-->
---
When I first saw `auto` I thought that woah, not so fast, how am I going to know the type of my variables? Then I started to understand that `auto` helps in so many different ways. It helps remove the clutter, think about iterator declarations! It removes code duplication, just think about `make_shared` or `make_unique`. But it doesn't only reduce visual pollution, it also keeps the code more maintainable. If you extensively use `auto` and you decide to change the type of your container you won't have to update so many parts of your code.

We haven't even mentioned templates. And that's not the goal today.

I just wanted to state that using `auto` is useful.

And it also evolved a lot since it was introduced by C++11. Since C++14, we can use with lambdas, then C++17 made it possible to have `auto` template types and with C++20, we can constrain them [with concepts](https://leanpub.com/cppconcepts). With C++23, we are going to get one more way to use it: `auto(x)`.

## What problem is it going to solve?

Sometimes we want to make a copy of a variable before we'd pass it on to the next function.

Let's take this example:

```cpp
void pop_front_alike(auto& x) {
    std::erase(x.begin(), x.end(), x.front());
}
```

This function does not compile as `std::vector::front` returns a reference, while `erase` expects a value. It's true that `erase` could also take a `const&` and `front` could return that. But if `x` is `const&` then `erase` cannot take it as it wants to modify `x`.

So we should rather make a copy of `x`.

We can write `auto tmp = x.front();` and then pass along `tmp`. Sadly, this has a couple of issues:

- It's still an extra line for nothing, just to create a copy
- You have to name that variable somehow... In a meaningful way... But what can be a meaningful name? `tmp`? `copy`? Really?
- We get an lvalue copy, not an rvalue copy in this case.

In order to solve all these problems, we could directly pass `T(x.front())`. But we don't have `T`, we used `auto` to take `x`! We could fall back from `auto` to an old-school templated solution, but to be fair, we don't even want the type passed in, we want the [decay](https://en.cppreference.com/w/cpp/types/decay) type.

> Unless T appears in the context of an array or a function type, the decay of `T` is the original type with removed reference and removed cv-qualifiers. For (C-style) arrays, it's the pointer type of T; for functions, it's the "pointerized" function type.

So if we wanted to get the above `T`, we should do it like this:

```cpp
using T = std::decay_t<decltype(x.front())>;
std::erase(x.begin(), x.end(), T(x.front()));
```

This is probably not something we want in our code, and definitely not repeatedly. So ideally we'd put into a function that extracts both the type and returns the copy to us. But as it's written in [the paper P0848R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0849r8.html
) such a function would also have its own issues. In particular, in terms of accessibility of non-public class members. So the accepted proposal in the end is not about a library feature of `std::decay_copy`, but a language feature.

`auto(x)` will return you a decaying copy of the passed in variable and as an optimization, it's a no-op if `x` is already a prvalue.

> [A prvalue is an expression whose evaluation initializes an object or a bit-field, or computes the value of the operand of an operator, as specified by the context in which it appears.](https://learn.microsoft.com/en-us/cpp/cpp/lvalues-and-rvalues-visual-cpp?view=msvc-170)

In the paper, you read further interesting considerations about alternative implementations using existing functions and features and why it's not called `prvalue_cast`. [This one](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0849r8.html
) is a fairly readable proposal even for people not speaking standardese.

## Conclusion

`auto` is one of the most useful features of modern C++ as it helps keep the code more concise and more maintainable. It evolved a lot since C++11, as we could use it in more and more different contexts. With C++23, we can use it to make a `prvalue` copy and we can use it to pass a copy to a function we want to call, without bothering to find a good name for the copy. Probably we won't use this feature that frequently, but when we'll need it, it'll be really useful.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
