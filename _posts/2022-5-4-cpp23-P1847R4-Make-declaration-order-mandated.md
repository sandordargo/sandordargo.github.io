---
layout: post
title: "Make declaration order layout mandated"
date: 2022-5-4
category: dev
tags: [cpp, cpp23, memory]
excerpt_separator: <!--more-->
---
We are soon reaching the middle of 2022 and as such we are getting closer and closer to C++23. I plan to show you more and more new features and fixes from the coming version. The first one was [deducing this](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23), and today we continue with the paper of [Pal Balog](https://stackexchange.com/users/2817566/balog-pal) about [*making declaration order layout mandated*](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1847r4.pdf).
<!--more-->

## What do we mean by (the standard) layout?

When we talk about the layout of a class (in C++) we mean how it is represented in the memory, where and in what order are the different fields stored.

The layout of a class is defined by many different attributes and we are not going to cover each different case, but I want to share with you enough information to understand what [P1847R4](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1847r4.pdf) wants and its motivations.

The simplest layout is called the standard layout. It's sufficiently defined so that it's memcopyable and it can be also consumed by C programs. The requirements for having a standard layout are:

- All non-`static` data members have the same access control
- Has no `virtual` functions or `virtual` base classes
- Has no non-`static` data members of reference type
- All non-`static` data members and base classes are themselves standard layout types
- Has no two (possibly indirect) base class subobjects of the same type
- Has all non-`static` data members and bit-fields declared in the same class (either all in the derived or all in some base)
- None of the base class subobjects has the same type as
-- for non-union types, as the first non-`static` data member (see empty base optimization), and, recursively, the first non-`static` data member of that data member if it has non-union class type, or all non-`static` data members of that data member if it has union type, or an element of that data member if it has array type, etc.
-- for union types, as any non-`static` data members, and, recursively, the first non-`static` data member of every member of non-union class type, and all non-`static` data members of all members of union type, and element type of all non-`static` data members of array type, etc.
-- for array types, as the type of the array element, and, recursively, the first non-`static` data member of the array element if it has non-union class type, or as any non-`static` data member of the array element if it has union type, or as the element type of the array element if it has array type, etc.

That's kind of a long list. If you want to easily check whether your class is having a standard layout or not, you can use [`std::is_standard_layout`](https://en.cppreference.com/w/cpp/types/is_standard_layout).

```cpp
#include <iostream>
#include <type_traits>

class A {
  int a;
  int b;
};

class B {
  int a;
public:
  int b;
};

class C {
  C (int& ib) : b(ib) {}
  int a;
  int& b;
};


int main() {
  std::cout << std::boolalpha;
  std::cout << std::is_standard_layout_v<A> << '\n';
  std::cout << std::is_standard_layout_v<B> << '\n';
  std::cout << std::is_standard_layout_v<C> << '\n';
}
```

## So what is the paper about?

According to the standard, implementations used to have the possibility to reorder members in the layout of a class given that they have different access control.

Let's say that you have a `class MyType`.

```cpp
class MyType {
public:
  int m_a;
private:
  int m_b;
  int m_c;
public:
  int m_d;
};
```

Compilers might decide to give `m_b` and `m_c` a lower address than to `m_a`. Though they cannot change the order between `m_b` and `m_c` and not even between `m_a` and `m_d`. At least not since C++11. In C++03, `m_d` could have been preceded `m_a` in the layout as they were part of two different access control blocks.

The old rule from C++03 said that "nonstatic data members of a (non-union) class declared without an intervening access-specifier are allocated so that later members have higher addresses within a class object. The order of allocation of nonstatic data members separated by an access-specifier is unspecified (11.1)".

Later, in C++11, [N2342](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2342.htm) made some changes were made to trim the level of freedom implementers have. *"The requirement that POD data members have no intervening access-specifiers is changed to require only that such data members have the same access control. This change is believed to also be more in line with programmer expectations than the current requirements."*

The most important implementers confirmed that they don't use this feature. One other that has a configurational option said that they never received and customer report having that option turned on. Based on the pieces of evidence, this right to reorder is not used.

The C++ standard is something quite complex and this paper aims to simplify it a little bit by removing the implementers' license of member reordering in case access control is mixed.

So while the `MyType` is subject to member reordering up to C++20, starting from C++23 it won't be possible anymore.

## Conclusion

[P1847R4](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1847r4.pdf) explains how layouts could be reordered in C++ when the access control is mixed and it proposes to remove the possibility for that reordering. While it doesn't change what standard layout is, it removes a rule that was unused and seemed quite arbitrarily.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!