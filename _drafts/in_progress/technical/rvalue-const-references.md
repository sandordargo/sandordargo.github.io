---
layout: post
title: "`const` rvalue references"
date: 2021-X-X
category: dev
tags: [cpp, tutorial, const, rvalue]
excerpt_separator: <!--more-->
---
Recently I facilitated a workshop at [C++OnSea](https://cpponsea.uk/2021/sessions/workshop_how-to-use-correctly-the-const-qualifier.html). It went well, but there was one topic that I couldn't deliver as well as I wanted. You might have guessed it right, it was about `const` rvalue references.

## What are rvalue references?

Rvalue references were introdued to C++ with C++11. Since then, we refer to the traditional references (marked with one `&`) as lvalue references.

With the use of rvalue (`&&`) references we can avoid logically unnecessary copying and implement perfect forwarding functions, thus achieving higher performance and more robust libraries.

If we try to define rvalue references in contrast with lvaule references, we can say that an lvalue is an expression whose address can be taken, as such an lvalue reference is a locator value. 

At the same time, an rvalue is an unnamed value that exists only during the evaluation of an expression.

In other terms, *"an lvalue is an expression that refers to a memory location and allows us to take the address of that memory location via the `&` operator. An rvalue is an expression that is not an lvalue."* ([source](http://thbecker.net/articles/rvalue_references/section_01.html))

From one point of view we might say that if you have a temporary value on the right, why would anyone want to modify it.

But on the other hand, we said that rvalue references are used for removing unnecessary copying, they are used with move semantics. If we "move away" from a variable, it implies modification.

Why would anyone (and how!) make such move-away variables `const`? 

## Binding rules

Given the abpve constraint, not surprisingly, the canonical signatures of the move assignment operator and of the move constructor use non-`const` rvalue references.

But that does't mean that `const T&&` doesn't exist. It does, it's syntactically completely valid.

It's not simply syntactically valid, but the language has clear, well-defined binding rules for it.

For our binding examples, we'll use the following four overloads of a simple function `f`.

```cpp
void f(T&) { std::cout << "lvalue ref\n"; }  // #1
void f(const T&) { std::cout << "const lvalue ref\n"; }  // #2
void f(T&&) { std::cout << "rvalue ref\n"; } // #3
void f(const T&&) { std::cout << "const rvalue ref\n"; } // #4
```

If you have a non-`const` rvalue reference, it can be used with any of these, but the non-`const` lvalue reference (#1). The first choice is `f(T&&)`, then `f(const T&&)` and finally `f(const T&)`.

But if none of those are available, only `f(T&)`, you'll get the following error message:

```cpp
#include <iostream>

struct T {};

void f(T&) { std::cout << "lvalue ref\n"; }  // #1
// void f(const T&) { std::cout << "const lvalue ref\n"; }  // #2
// void f(T&&) { std::cout << "rvalue ref\n"; } // #3
// void f(const T&&) { std::cout << "const rvalue ref\n"; } // #4


int main() {
    f(T{}); // rvalue #3, #4, #2
}
/*
main.cpp:12:8: error: cannot bind non-`const` lvalue reference of type 'T&' to an rvalue of type 'T'
   12 |     f (T{}); // rvalue        #3, #4, #2
      |    
*/
```

So an rvalue can be used both with rvalue overloads and a const lvalue reference. It's a little bit of a mixture.

If we have an lvalue, that can be used only with `f(T&)` and `f(const T&)`. 

```cpp
#include <iostream>

struct T {};

void f(T&) { std::cout << "lvalue ref\n"; }  // #1
void f(const T&) { std::cout << "const lvalue ref\n"; }  // #2
void f(T&&) { std::cout << "rvalue ref\n"; } // #3
void f(const T&&) { std::cout << "const rvalue ref\n"; } // #4


int main() {
    T t;
    f(t); // #1, #2
}
```

There is a little bit of assymetry here.

Can we "fix" this assimetry? Is there any option that can be used only with the rvalue overloads?

No. If we take a `const` rvalue reference, it can be used with the `f(const T&&)` and `f(const T&)`, but not with any of the non-`const` references.

```cpp 
#include <iostream>

struct T {};

void f(T&) { std::cout << "lvalue ref\n"; }  // #1
void f(const T&) { std::cout << "const lvalue ref\n"; }  // #2
void f(T&&) { std::cout << "rvalue ref\n"; } // #3
void f(const T&&) { std::cout << "const rvalue ref\n"; } // #4

const T g() { return T{}; }

int main() {
    f(g()); // #4, #2
}
```

*By the way, don't return `const` values from a function, because you make it impossible to use move semantics. [Find more info here.](https://www.sandordargo.com/blog/2020/11/18/when-use-const-3-return-types)*

## When to use const rvalue references?

Let's turn it around a bit. A lvalue overload can accept both lvalues and rvalues, but an rvalue overload can only accept rvalues. 

The goal of rvalue references is sparing copies and using move semantics. At the same time, we cannot move away from const values. Therefore the usage of `const` rvalue references communicates that 
- a given operation is only supported on rvalues
- but we still make a copy, as we cannot move.

We haven't seen a lot the need for this. For a potential example with unique pointers [check out this StackOverflow answer](https://stackoverflow.com/a/60587511/3238101).

What is important to note is that `f(const T&&)` can take both `T&&` and `const T&&`, while `f(T&&)` can only take the non-`const` rvalue reference and not the const one.

Therefore if you want to prohibit rvalue references, you should delete the `f(const T&&)` overload. 

What would happen otherwise?

If you delete the non-`const` overload, the compilation will fail with rvalue references, but even though it doesn't make much sense in general to pass `const` rvalue references, the code will compile.

```cpp
#include <iostream>

struct T{};

void f(T&) { std::cout << "lvalue ref\n"; }
void f(const T&) { std::cout << "const lvalue ref\n"; }
void f(T&&) = delete; //{ std::cout << "rvalue ref\n"; }
// void f(const T&&) { std::cout << "const rvalue ref\n"; }

const T g() {
 return T{};
}

int main() {
    f(g());
}
/*
const lvalue ref
*/
```

On the other hand, if we delete the `const T&&` overload, we make sure that no rvalue references are accepted at all.

```cpp
#include <iostream>

struct T{};

void f(T&) { std::cout << "lvalue ref\n"; }
void f(const T&) { std::cout << "const lvalue ref\n"; }
// void f(T&&) = delete; //{ std::cout << "rvalue ref\n"; }
void f(const T&&) = delete; //{ std::cout << "const rvalue ref\n"; }

const T g() {
 return T{};
}

int main() {
    f(g());
    f(T{});
}
/*
main.cpp: In function 'int main()':
main.cpp:15:6: error: use of deleted function 'void f(const T&&)'
   15 |     f(g());
      |     ~^~~~~
main.cpp:8:6: note: declared here
    8 | void f(const T&&) = delete; //{ std::cout << "const rvalue ref\n"; }
      |      ^
main.cpp:16:6: error: use of deleted function 'void f(const T&&)'
   16 |     f(T{});
      |     ~^~~~~
main.cpp:8:6: note: declared here
    8 | void f(const T&&) = delete; //{ std::cout << "const rvalue ref\n"; }
      |      ^
*/
```

So due to the binding rules, we can only make sure by deleting the `const` version that no rvalue references are accepted.

You can observe this the standard library too, for example with [`std::reference_wrapper::ref` and `std::reference_wrapper::cref`](https://github.com/microsoft/STL/blob/1ece8a0352327397997c3f4b649a228c66da3ce1/stl/inc/type_traits#L1967-L1981).

## Conclusion

Today we discussed about `const` rvalue references. We saw that although at a first glance they don't make much sense, they are still useful. Rvalue references in general are used with move semantics which implies modifying the referred object, but in some rare cases it might have a good semantic meaning. At the same time, it's also used with `=delete` to prohibit rvalue references in a bullet proof way.

Let me know if you've ever used `const` rvalue references in your code!


## References
- [Lvalues and Rvalues By Mikael Kilpeläinen](https://accu.org/journals/overload/12/61/kilpelainen_227/)
- [C++ Rvalue References Explained by Thomas Becker](http://thbecker.net/articles/rvalue_references/section_01.html)
- [A Brief Introduction to Rvalue References by Howard E. Hinnant, Bjarne Stroustrup and Bronek Kozicki](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html)
- [What are const rvalue references good for? by Boris Kolpackov](https://www.codesynthesis.com/~boris/blog/2012/07/24/const-rvalue-references/)
