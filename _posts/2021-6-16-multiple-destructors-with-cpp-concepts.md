---
layout: post
title: "Multiple destructors with C++ concepts"
date: 2021-6-16
category: dev
tags: [cpp, cpp20, concepts, templates]
excerpt_separator: <!--more-->
---
We probably all learnt that one cannot overload the destructor. Hence I write about ***"the"*** destructor and *a* destructor... After all, it has no return type and it doesn't take parameters. It's also not really `const` as it destroys the underlying object.
<!--more-->

Yet, there were techniques existing to have multiple destructors in a class and those techniques are getting simplified with C++20.

# The need for multiple destructors

But first of all, why would you need multiple destructors?

For optimization reasons for example! 

Imagine that you have a class template and you want to have destruction depending on the traits of the template parameters. Trivially destructible types can work with the compiler-generated destructor and it's much faster than the user-defined ones...

Also, while RAII is great and we should write our classes by default with that paradigm having in mind, with a good wrapper we can make non-RAII classes at least to do the clean-up after themselves.

These are already two reasons to have multiple destructors, but I'm sure you can name others, feel free to do so in the comments section.

# Multiple destructors before C++20

So how to do this?

As I've learned from [C++ Weekly](https://www.youtube.com/watch?v=A3_xrqr5Kdw), you can use [std::conditional](https://en.cppreference.com/w/cpp/types/conditional).

[std::conditional](https://en.cppreference.com/w/cpp/types/conditional) lets us choose between two implementation at compile-time. If the condition that we pass in as a first parameter evaluates to `true`, then the whole call is replaced with the second parameter, otherwise with the third.

Here comes the example:
```cpp
#include <iostream>
#include <string>
#include <type_traits>

class Wrapper_Trivial {
  public:
    ~Wrapper_Trivial() = default;
};

class Wrapper_NonTrivial {
  public:
    ~Wrapper_NonTrivial() {
        std::cout << "Not trivial\n";
    }
};

template <typename T>
class Wrapper : public std::conditional_t<std::is_trivially_destructible_v<T>, Wrapper_Trivial, Wrapper_NonTrivial>
{
    T t;
};

int main()
{
    Wrapper<int> wrappedInt;
    Wrapper<std::string> wrappedString;
}
```
So, our `Wrapper` class doesn't include a destructor, but it inherits it either from `Wrapper_Trivial` or `Wrapper_NonTrivial` based on a condition, based on whether the contained type `T` is trivially destructible or not.

It's a bit ugly, almost write-only code. Plus supporting the second case - cleanup after non-RAII code - is even uglier.

# Multiple destructors with C++20

C++ concepts help us to simplify the above example. Still with no run-time costs, and probably with cheaper write costs.

```cpp
#include <iostream>
#include <string>
#include <type_traits>

template <typename T>
class Wrapper
{
    T t;
 public:    
    ~Wrapper() requires (!std::is_trivially_destructible_v<T>) {
        std::cout << "Not trivial\n";
    }
    
    ~Wrapper() = default;
};

int main()
{
    Wrapper<int> wrappedInt;
    Wrapper<std::string> wrappedString;
}
/*
Not trivial
*/
```
We still have a class template, but instead of using the cumbersome to decipher `std::conditional`, first, we wrote a destructor with a `requires` clause. Then we defaulted the unconstrained one.

In the `requires` clause, we constrain that implementation so that the wrapped type cannot be trivially destructible.

The above example [works fine with gcc](godbolt.org/z/rKeETW5jE). We receive the expected output. On the other hand, if you try to compile it [with the latest clang](godbolt.org/z/rqME8W1rn) (as of June 2021, when this article was written), you get a swift compilation error.

```cpp
<source>:19:18: error: invalid reference to function '~Wrapper': constraints not satisfied
    Wrapper<int> wrappedInt;
                 ^
<source>:10:26: note: because '!std::is_trivially_destructible_v<int>' evaluated to false
    ~Wrapper() requires (!std::is_trivially_destructible_v<T>) {
                         ^
1 error generated.
ASM generation compiler returned: 1
<source>:19:18: error: invalid reference to function '~Wrapper': constraints not satisfied
    Wrapper<int> wrappedInt;
                 ^
<source>:10:26: note: because '!std::is_trivially_destructible_v<int>' evaluated to false
    ~Wrapper() requires (!std::is_trivially_destructible_v<T>) {
                         ^
1 error generated.
```
Basically, the error message says that the code is not compilable, because `int` is trivially destructible, therefore it doesn't satisfy the requirements of the first destructor which requires a not trivially destructible type.

It's sad because `int` should use the other destructor...

While I was looking at the code, I realized that I dislike something about it - apart from the compilation failure. We started with the most specific, with the most constrained overload, instead of going from the general implementation towards the specific.

So I updated the order of the two destructors:

```cpp
#include <iostream>
#include <string>
#include <type_traits>

template <typename T>
class Wrapper
{
    T t;
 public:     
    ~Wrapper() = default;

    ~Wrapper() requires (!std::is_trivially_destructible_v<T>) {
        std::cout << "Not trivial\n";
    }
};

int main()
{
    Wrapper<int> wrappedInt;
    Wrapper<std::string> wrappedString;
}
```

Lo and behold! It compiles with clang! But it doesn't produce the expected output. In fact, what happens is that just like previously, only the first declared destructor is taken into account.

We can draw the conclusion that clang doesn't support - yet - multiple destructors and cannot handle concepts well in the context of destructors. [Mr. K.](https://twitter.com/GeekyMrK) - who we were experimenting with - [filed a bug for LLVM](https://bugs.llvm.org/show_bug.cgi?id=50570).

*Just for the note, I asked a colleague who was access to MSVCC, the above examples work fine not only with gcc but with the MS compiler too.*

# Conclusion

Today we learned that while in general, a class should always have one destructor, for class templates there have been ways to provide different implementations for that destructor based on the characteristics of template arguments.

The old way of doing this is using `std::conditional`, but it's not as readable as using C++20 concepts.

We have also seen that while C++20 provides an extremely readable way to do this, it's not yet fully supported not even by all the major compilers. gcc and msvcc provide a correct implementation, but clang is a bit behind on this.

**If you want to learn more details about C++ concepts, [check out my book on Leanpub](https://leanpub.com/cppconcepts)!**