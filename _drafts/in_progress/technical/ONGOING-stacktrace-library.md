---
layout: post
title: "C++23: The stacktrace library"
date: 2022-5-4
category: dev
tags: [cpp, cpp23, stacktrace]
excerpt_separator: <!--more-->
---
So far, there was no way in C++ to get runtime information on the current call sequence. Other popular programming languages such as Java, C# or Python provide this possibility. Thanks to P0881R7 and the people behind, with C++23, now we will also have a similar feature.

Let's discover in this article what and how we can use!

<!--
Shar the motivation, from 
https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0881r7.html
-->

## Key features!

Some design decisions

- All *stack_framce* functions and contructors are lazy, no information will be decoded until it's needed to keep the library fast
- Frames are stored in a dynamically sized storage as the most important piece of information is often at the bottom of the stacktrace. This also means that the stacktrace should not be constructed performance critical hot paths. Or at least, it should be constructed with a custom allocator.


The `<stacktrace>` header provides us essentially with two classes. `stacktrace_entry` is the representation of one evaluation in a stacktrace and that evaluation might be empty. You can check its emptiness with `operator bool`. What's in an evaluation? That's a good question. (check boost maybe?)

In order to get more information about the evaluation, you can get 3 queries.
- `description()`
- `source_file()`
- `source_line()`

These names are quite self-evident, once we understand what does an evaluation of a stacktrace mean.

The other class in the header is `basic_stacktrace` and it consists of multiple stacktrace entries. It's either the representation of the full stacktrace or just a given part of it. `std::basic_stacktrace` works pretty much as a standard contianer with iterator and element access functions, and in addition, there is a `static` function called `current` that obtains the current stacktrace and there is also `get_allocator` returning the associated allocator.



  + hash

  https://en.cppreference.com/w/cpp/utility/stacktrace_entry

## Is it already available?

(What is pmr? https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2301r1.html)

## How does it work?

basic stacktrace is a list of stacktrace events

can I query it anywhere?
perf impact?


https://en.cppreference.com/w/cpp/header/stacktrace

https://en.cppreference.com/w/cpp/utility/basic_stacktrace

```cpp
      auto currentStacktrace = std::stacktrace();
      for (const auto& entry : currentStacktrace) {
          std::cout << entry;
        // std::cout << entry.description() << std::endl;
        // std::cout << entry.source_file() << std::endl;
        // std::cout << entry.source_line() << std::endl;
      }
```

not workging yet

## Conclusion

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
