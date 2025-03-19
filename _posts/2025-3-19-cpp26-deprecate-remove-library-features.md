---
layout: post
title: "C++26: Deprecating or removing library features"
date: 2025-3-19
category: dev
tags: [cpp, cpp26, remove, deprecate]
excerpt_separator: <!--more-->
---
In the previous article, we discussed [what language features are removed from C++26](https://www.sandordargo.com/blog/2025/03/12/cpp26-removing-language-features). In this one, we are going to cover both language features that are finally removed after a few years of deprecation, and also those that are getting deprecated by C++26.

> As a reminder, a removal from the language usually happens in two steps. First a feature gets deprecated, meaning that its users would face compiler warnings for using deprecated features. As a next step, which in some cases never comes, the compiler support is finally removed.

## Remove Deprecated `std::allocator` Typedef From C++26

The `std::allocator` has a `typedef` that was deprecated in C++20 and now finally it's removed. Though it's a minor corner case, but it's been so easy to misuse it that it was rather embarrassing for the committee, hence the removal by [P2868R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2868r3.pdf).

Classes deriving from `std::allocator` don't synthesise the `typedef` member correctly and the allocator authors have to add their own typedef to ensure correct behaviour. If they knew about it...

## Removing function overload of `std::basic_string::reserve()` that takes no 

I just learned from [P2870R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2870r3.pdf) that `std::basic_string::reserve` had an overload taking no arguments. As it was a poor substitute for `std::basic_string::shrink_to_fit`, it was deprecated in C++20.

Now it's gone.

`reserve` used to have a default value of `0` for its sole argument turning it into a non-binding `shrink_to_fit`. But `shrink_to_fit` was introduced as an independent function in C++11 and as such this behaviour of `reserve` became superfluous and a "few" years later it got deprecated.

If your code uses `reserve()` without any arguments, migration is simple, just replace it it with `shrink_to_fit`.

## Remove Deprecated Unicode Conversion Facets from C++26

The `<codecvt>` header was first provided by C++11 and then deprecated by C++17 due to its underspecification, notably a lack of error handling. It's removed in C++26 by [P2871R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2871r3.pdf).

This library contained several helper classes to convert between different UTF formats. Due to the bad specification, ill-formed UTF strings could be used as an attack vector.

This change is about improving language safety.

## Freestanding: removing `std::strtok`

`std::strtok` has been part of C++ freestanding, in other words, it's a C function and was part of C++ to help with compatibility. As `std::strtok` has been removed from C2X standards, [C++ doesn't need it anymore either](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2937r0.html). 

> *"A freestanding C++ implementation is mostly a superset of a freestanding C implementation, even in the "C" parts of C++. This means that a freestanding C++ implementation can not generally be built on top of a minimal freestanding C implementation. Either the C++ implementation must provide some of the C parts, or the C++ implementation will require a C implementation that provides more than the minimum."* - [source](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2338r4.html#:~:text=A%20freestanding%20C%2B%2B%20implementation%20is%20mostly%20a%20superset%20of%20a,a%20minimal%20freestanding%20C%20implementation.)

## Removing deprecated `strstreams`

C++20 introduced the ability to move strings efficiently out of stringsteams and C++23 brought us the `spanstream` library that we already covered in [*C++23: The rise of new streams*](https://www.sandordargo.com/blog/2023/12/06/cpp23-strtream-strstream-replacement).

Given that C++ now had superior replacements for `char*` stream, now they are finally removed.

Why finally?

Well, apparently, `char*` streams have been the largest and oldest deprecated feature in the standard. They were marked for future deprecation and possible removal almost 30 years ago! Yes, we are talking about C++98.

Will we need to undeprecate these features?

Hopefully not.

## Removing deprecated `std::shared_ptr` Atomic Access APIs

C++11 introduced atomics and smart pointers. Among others, it also introduced a free function API for atomic access to `shared_ptr`. It was an easy-to-use API so it was deprecated by C++20, along with the introduction of its type-safe replacement `std::atomic<shared_ptr<T>>`. C++26 removes the deprecated API thanks to the acceptance of [P2869R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2869r4.pdf).

While the old API expected that the shared object is not used directly, the API made it possible which led to undefined behaviour, typically producing a data race.

To deal with the deprecation, one must perform two steps:
- include the `<atomic>` header
- replace `shared_ptr<T>` with `std::atomic<shared_ptr<T>>`

Alternatively, one might also prefer using `std::atomic` member functions directly, instead of using `std::shared_ptr` overloads. For further details check out section 5.1 in [P2869R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2869r4.pdf).

## Removing `std::wstring_convert`

`std::wstring_convert`, `std::wbuffer_convert` and some related helper functions were introduced by C++11, deprecated with C++17 and finally removed in C++26 by [P2872R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2872r3.pdf). Have you ever used it? I haven't heard about it before.

They help conversions between normal-sized and wide strings.

The reason for their removals is that they are underspecified, there are a handful of open LWG issues related to them, and improving these facilities *"would require more work than the committee wishes to invest to bring it up to the desired level of"*.

## Deprecating `std::is_trivial` and `std::is_trivial_v`

When I saw this accepted proposal I was surprised. Removing `std::is_trivial`? Aren't people talking so much about trivial classes? But that's simply a sign of my shallow knowledge.

The fact is that people don't talk about trivial classes - which notion is also deprecated. People talk about trivially default constructible, trivially copyable or trivially copy assignable classes. Often these properties are not needed at the same.

In addition, `is_trivial` doesn't even check that the required constructors are public.

Use the appropriate `is_trivially_XXX` checks instead of the general `is_trivial`, especially now that the latter is deprecated by [P3247R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3247r2.html)

## Defang and deprecate `std::memory_order::consume`

The problem of `std::memory_oder::consume` has been around for a decade. Its specification is not satisfactory, it's difficult to implement and not even required by most widely-used CPU architectures. Given these problems, most academic work simply ignores its existence and uses other memory models.

But even implementations mostly map it to `std::memory_oder::acquire`. As there is no will and consensus to improve it, the best next step is to deprecate it which [P3475R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3475r1.pdf)

## Conclusion

After having discussed last week [language features removed from C++26](https://www.sandordargo.com/blog/2025/03/12/cpp26-removing-language-features), this week we covered what library features are removed or started their way to be removed through their deprecation.

Deprecation is not always a one-way road. Next week, we'll cover a feature that is going to be underprecated in C++26. Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
