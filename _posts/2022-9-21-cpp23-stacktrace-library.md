---
layout: post
title: "C++23: The stacktrace library"
date: 2022-9-21
category: dev
tags: [cpp, cpp23, stacktrace, backtrace]
excerpt_separator: <!--more-->
---
So far, there was no way in C++ to get runtime information on the current call sequence. Other popular programming languages such as Java, C# or Python provide this possibility. Thanks to [P0881R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0881r7.html) and the people behind, now we will also get a similar feature with C++23.

Let's discover in this article what exactly we get and how we can use it!

## Is it already available? - the meta how

Before we delve into how to use this new library, we should discuss how to use it or if we can already use it at all. I mean it's a C++23 feature and it doesn't have wide compiler support for the time being. Following a recent [C++ Weekly episode](https://www.youtube.com/watch?v=9IcxniCxKlQ), we can use the `<stacktrace>` library by compiling against at least gcc 12.1 (no trunk is needed), we have to specify `-std=c++23` and we have to add the command line option of `-lstdc++_libbacktrace` to link the library.

As such, we can have early access to this interesting new library!

## What are the key features of the stacktrace library?

Let's pick some interesting and/or important decisions, features from [the accepted papaer](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0881r7.html):

- All *stack_frame* functions and constructors are lazy, no information will be decoded until it's needed to keep the library fast.

- Frames are stored in a dynamically sized storage as the most important piece of information is often at the bottom of the stacktrace. This also means that the stacktrace should not be constructed on performance-critical hot paths. Or at least, it should be constructed with a custom allocator.

- The `<stacktrace>` header provides us essentially with two classes. `stacktrace_entry` is the representation of one evaluation, one frame in a stacktrace and that evaluation might be empty. You can check its emptiness with `operator bool`.

What's in an evaluation? That's a good question. Basically, it's one entry in a backtrace or in other words a stacktrace. If you have a function `foo()` that is called from `main()`, your stacktrace should be composed of two items, two evaluations `foo()` and `main` - life can be a bit more complex though.

In order to get more information about the evaluation, you get 3 queries.
- `description()`
- `source_file()`
- `source_line()`

These names are quite self-evident, once we understand what does an evaluation of a stacktrace mean.

The other class in the header is `basic_stacktrace` and it consists of multiple stacktrace entries. It's either the representation of the full stacktrace or just a given part of it. `std::basic_stacktrace` works pretty much as a standard container with iterators and element access functions.

Keep in mind, that `stacktrace` is just an alias for `basic_stacktrace` with the default allocator.

Beware that this below piece of code might not do what you'd expect:

```cpp
auto currentStacktrace = std::stacktrace(); // Won't work as one might expect!
for (const auto& entry : currentStacktrace) {
  std::cout << entry.description() << std::endl;
  std::cout << entry.source_file() << std::endl;
  std::cout << entry.source_line() << std::endl;
}
```

`std::stacktrace()` is just a constructor call and it instantiates a new `basic_stacktrace` container. If you want to get the stacktrace of the current execution context, call the `current` `static` member function instead of the constructor.

```cpp
auto currentStacktrace = std::stacktrace::current();
```

Once we have a stacktrace, we can obtain the stored information in different ways. The easiest way is to actually just print the whole stacktrace all at once.

```cpp
#include <stacktrace>
#include <iostream>

void foo() {
    auto trace = std::stacktrace::current();
    std::cout << std::to_string(trace) << '\n';
}

int main() {
    foo();
}
/*
   0# foo() at /app/example.cpp:5
   1#      at /app/example.cpp:10
   2#      at :0
   3#      at :0
   4# 
*/
```

We see two interesting things above. `main` is not printed as a description and there are two additional frames on the top that must be related to the execution context.

As `stacktrace` is a container, we can iterate over it and print the items one by one.

```cpp
#include <stacktrace>
#include <iostream>

void foo() {
    auto trace = std::stacktrace::current();
    for (const auto& entry: trace) {
        std::cout << std::to_string(entry) << '\n';
    }
}

int main() {
    foo();
}

/*
foo() at /app/example.cpp:5
     at /app/example.cpp:12
     at :0
     at :0

*/
```

We have pretty much the same output, but now we lost the numbering, which would have to be put back with the help of a loop index.

If we don't want all the information from a trace, we can get the method name (*description*), the source file and the line number separately with the right accessors.

We can iterate over it and print each entry.
We can take the different attributes:

```cpp
#include <stacktrace>
#include <iostream>

void foo() {
    auto trace = std::stacktrace::current();
    for (const auto& entry: trace) {
        std::cout << "Description: " << entry.description() << std::endl;
        std::cout << "file: " << entry.source_file() << std::endl;
        std::cout << "line: " << entry.source_line() << std::endl;
        std::cout << "------------------------------------" << std::endl;
    }
}

int main() {
    foo();
}
/*
Description: foo()
file: /app/example.cpp
line: 5
------------------------------------
Description: 
file: /app/example.cpp
line: 15
------------------------------------
Description: 
file: 
line: 0
------------------------------------
Description: 
file: 
line: 0
------------------------------------
Description: 
file: 
line: 0
------------------------------------
*/

```

An interesting thing I found is that if the current trace is queried when a parameter is defaulted, that function doesn't appear in the stacktrace. Somehow it makes sense because it's not yet executed yet, but it was already called, so I'm not sure if I like this behaviour. But it might be just me.

```cpp
#include <stacktrace>
#include <iostream>

void foo(std::stacktrace trace = std::stacktrace::current()) {
    std::cout << std::to_string(trace) << '\n';
}

int main() {
    foo();
}

/*
   0#      at /app/example.cpp:9
   1#      at :0
   2#      at :0
   3# 
*/
```

## Conclusion

`<stacktrace>` library is a very useful addition to the C++ standard library that lets us query and print the backtrace. The compiler support is very limited for the time being, we can only use gcc and probably the implementation will still change here and there. Still, we can already experiment, we can already learn how to use it. I'm sure it will come in very handy for error handling in C++.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
