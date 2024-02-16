---
layout: post
title: "Multiple destructors with C++ concepts"
date: 2021-6-16
category: dev
tags: [cpp, cpp20, concepts, templates]
excerpt_separator: <!--more-->
---
We probably all learnt that one cannot overload the destructor. Hence I write about ***"the"*** destructor and *a* destructor... After all, it has no return type and it doesn't take parameters. It's also not `const` as it destroys the underlying object.
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

[std::conditional](https://en.cppreference.com/w/cpp/types/conditional) lets us choose between two implementations at compile-time. If the condition that we pass in as a first parameter evaluates to `true`, then the whole call is replaced with the second parameter, otherwise with the third.

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

It's a bit ugly, almost *write-only* code. Plus supporting the second case - cleanup after non-RAII code - is even uglier.

# Multiple destructors with C++20

C++ concepts help us simplify the above example. Still with no run-time costs, and probably with cheaper write costs.

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
We still have a class template, but instead of using the cumbersome to decipher `std::conditional`, we use the trailing `requires` clause to provide an overload for the destructor.

Remember, we learned earlier that in class templates we can provide function overloads using different constraints. This is true even for constructors and destructors.

In the above example, first, we wrote a destructor with a `requires` clause. Then we also provided the default implementation without specifying any constraint.

In the `requires` clause, we specify a constraint that makes it a valid overload only for types that are not trivially destructible. [`std::is_trivially_destructible_v`](https://en.cppreference.com/w/cpp/types/is_destructible) is true if one of the following conditions apply:
- The destructor is not user-provided, e.g. it's either explicitly defaulted or not provided
- The destructor is not virtual, including all the base classes' destructors
- All direct base classes have trivial destructors
- All non-static data members of class type (or array of class type) have trivial destructors

Given all that, what output do we expect from the above example?

`Wrapper<int> wrappedInt` should be destructed with the default, unconstrained constructor because `int` is a trivially destructible type, therefore the constrained overload is not considered.

On the other hand, `Wrapper<std::string> wrappedString` should use the constrained destructor and therefore print *"Not trivial"* on the console, as `std::string` is not a trivially destructible type.

The above example [works fine with gcc](https://godbolt.org/z/rKeETW5jE). We receive the expected output.

## An already fixed compiler bug

> *The bug I'm going to explain was fixed with Clang 15.0.0. I still kept this section for two reason. 1) If you happen to use an older version and you run into the same error, you know you're not alone. 2) It's a nice memento that even compilers might contain errors and compiler engineers work hard to eliminate them.* 

As of June 2021, when this article was written, if you tried to compile it [with the latest clang](https://godbolt.org/z/rqME8W1rn), you got a a compilation error.

```
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
Basically, the error message said that the code is not ill-formed, because `int` is trivially destructible, therefore it doesn't satisfy the requirements of the first destructor which requires a not trivially destructible type.

It's sad because `int` should use the other destructor as we discussed earlier...

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

*Just for the note, I asked a colleague who has access to MSVC, the above examples work fine not only with GCC but with the MS compiler too.*

# Conclusion

Today we learned that while in general, a class should always have one destructor, for class templates there have been ways to provide different implementations for that destructor based on the characteristics of template arguments.

The old way of doing this is using `std::conditional`, but it's not as readable as using C++20 concepts. With a new enough compiler you won't run into any troubles using C++20's concepts.

**If you want to learn more details about C++ concepts, [check out my book on Leanpub](https://leanpub.com/cppconcepts)!**