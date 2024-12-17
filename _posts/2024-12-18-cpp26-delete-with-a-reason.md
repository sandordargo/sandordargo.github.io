---
layout: post
title: "C++26: Delete with a reason"
date: 2024-12-18
category: dev
tags: [cpp, cpp26, attributes, delete]
excerpt_separator: <!--more-->
---
Let's start [exploring C++26](https://www.sandordargo.com/blog/2024/12/11/start-exploring-cpp26) with a simple but useful change. Thanks to Yihe Li's proposal ([2573R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2573r2.html)), when we `=delete` a special member function or a function overload, we can specify a reason.

This is a practical and not very surprising change. Why do I say it's not very surprising? Adding the ability to specify reasons for some operations has become a pattern in C++ evolution. Since C++11, we can add an optional message to `static_assert`, C++14 enriched the `[[deprecated]]` attribute with an optional reason, and the same happened in C++20 to the `[[nodiscard]]` attribute.

The reason is always similar, if not the same, to enhance maintainability and readability. When it comes to readability, not only the code but also the diagnostic messages become easier to grasp. Understanding why the compiler emitted a warning or an error becomes easier.

This latter is even more important than the readability of the codebase. I mean you could still add a comment right before or after a deleted function, but it wouldn't be used in the compilers' messages.

The usage is simple, if you want to specify a reason why a function or function overload is deleted, just pass a string literal containing the reason after `delete` surrounded by parentheses.

```cpp
class NonCopyable
{
public:
    // ...
    NonCopyable() = default;

    // copy members
    NonCopyable(const NonCopyable&)
        = delete("Since this class manages unique resources, copy is not supported; use move instead.");
    NonCopyable& operator=(const NonCopyable&)
        = delete("Since this class manages unique resources, copy is not supported; use move instead.");
    
    // provide move members instead
    // ...  
};
```

And if you still try to copy some members, this is the error message you would get:

```
<source>:16:17: error: call to deleted
constructor of 'NonCopyable': Since this class manages
unique resources, copy is not supported; use move instead.
    NonCopyable nc2 = nc;
                ^     ~~
<source>:8:5: note: 'NonCopyable' has been
explicitly marked deleted here
    NonCopyable(const NonCopyable&)
    ^
```

As the message is optional, this is obviously a non-breaking change, even though you might foresee some clean code practices where people update existing `=delete` specifications.

Another usage that the author had in mind is that functions marked `[[deprecated("with reason")]` might get deleted later with the same reason, `= delete("with reason")`. While it's true that without specifying the deletion reason this is already possible, with the message such a practice would become more readable and you don't have to read the source code of a library to find out the reason for the deletion.

This change has already been implemented in GCC 15 and Clang 19.

## Conclusion

In this article, we learned about a simple but very practical change introduced by C++26. When using the `=delete` specification, we'll be able to provide a reason for the deletion. With the latest GCC and Clang versions, we can already increase the readability of our codebases.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
