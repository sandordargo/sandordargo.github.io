---
layout: post
title: "C++23: How lambdas are going to change?"
date: 2022-11-23
category: books
tags: [cpp, cpp23, lambda]
excerpt_separator: <!--more-->
---
C++23 is coming soon and it will change how lambdas work in 3 different ways. They will not only become simpler in certain circumstances but they will be also aligned more with other features of the language.

Okay, let's go through these changes.

## Make `()` more optional for lambdas

What is the simplest lambda function?

Is it `[](){}`?

Nope! It's `[]{}`.

If a lambda has no parameters (*in standardese: if the parameter declaration clause is empty*), we can omit the parentheses!

Or can't we?

It depends. In regular lambda functions we can, but there has been another not-very-intuitive rule. When you have some lambda template parameters, `constexpr`, `consteval`, `mutable` or `noexcept` specifiers, or when you benefit from attributes or trailing return types (see the next section!) or a `requires` clause, you cannot omit the empty parentheses.

So the following lambda declaration is illegal in C++20:

```cpp
// warning: parameter declaration before lambda declaration specifiers only optional with '-std=c++2b' or '-std=gnu++2b' [-Wc++23-extensions]
auto l = [] mutable {};
```

Thanks to [P1102R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1102r2.html), omitting empty parentheses in such circumstances becomes legitim. A small proposal that doesn't change how existing code behaves but makes the language a bit more comfortable and more importantly, it makes the language more consistent.

## Change the scope of lambda trailing-return-type

I found overly interesting this change and it definitely made me learn something important I had no idea about!

Let's assume that you have such a piece of code:

```cpp
double j = 42.0;
// ...
auto counter = [j=0]() mutable -> decltype(j) {
    return j++;
};
```

I know the trailing return type makes not much sense here, but this is just a simple example to showcase the problem.

What do you expect the return type of `counter` to be?

You might argue that in the lambda capture we declare `j` and initialize it with `0`. The `0` literal without any suffix is an `int`, therefore when we invoke `counter` it should return an `int`.

That's wrong, the right answer is `double`!

Even though the `j` introduced in the capture is physically closer to the trailing return type and most probably developers would first think about the captured `j` rather than some other variable from the outer scope, still the answer is that the type of `j` comes from the outer scope. Whatever is in the capture list, it's not visible in the trailing return type.

This also means that the following snippet on its own would not compile:

```cpp
auto counter = [j=0]() mutable -> decltype(j) {
    return j++;
};
```

To be fair, this is the luckier case. It's better to have a clear error rather than an unexpected and often undetected behaviour like in the original example.

[P2036R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2036r3.html) is going to change this situation. Name lookups for trailing return types are going to consider captures before looking outside. While this is not going to be a backward-compatible change, it will almost always match the developer's intent. 

## Attributes on lambdas

After reading this section title, you might ask whether it's already possible to use attributes with lambdas. The answer is yes, it is possible. The place for adding an attribute is in the lambda declarator, either before or after the parameter declaration clause, but always between the optional noexcept specifiers and trailing return types.

The attributes sequence belongs to the type of the corresponding function call operator. The paper [P2173R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2173r1.pdf) argues - and rightly so - that this should not necessarily be the case. Attributes should be allowed to belong to the function call operator.

After all, why couldn't the operator and with that almost always the lambda be `[[nodiscard]]`, `[[noreturn]]` or `[[deprecated]]`?

According to the proposed and accepted change, regardless of the other optional elements of a lambda expression, now we can declare the attribute sequence right after the lambda introducer (and its optional capture) or right after the template parameter list including its requires clause.

The proposed wording says that an attribute specifier sequence in a lambda declarator appertains to the type of the function call operator, but if it comes before the lambda declarator, then it belongs to the function call operator itself, not its type. You might say that you cannot attach a `[[nodiscard]]` attribute to a type, it wouldn't make sense. You're right, but don't think only about the standard attributes, even the proposal mentions vendor-defined ones.

With [this change](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2173r1.pdf), the following piece of code becomes valid, where the function call of the operator is `[[nodiscard]]`:

```cpp
auto lm = [][[nodiscard]]()->int { return 42; };
```

Meaning that this would emit a warning:

```cpp
auto lm = [][[nodiscard]]()->int { return 42; };
// ...
lm(); // warning: ignoring return value of 'main()::<lambda()>', declared with attribute 'nodiscard' [-Wunused-result]
```

Please note that GCC and Clang already implemented this behaviour, GCC already in version 9! 

## Conclusion

Lambdas were introduced in C++11 and each standard brought some new features. It's not going to be different with C++23. It will bring better attributes, a more reasonable trailing return type deduction and more consistent rules for omitting an empty parameter list. Stay tuned for more articles about the coming standard!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
