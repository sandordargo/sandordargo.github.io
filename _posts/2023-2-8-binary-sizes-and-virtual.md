---
layout: post
title: "Virtual functions and binary sizes"
date: 2023-2-8
category: dev
tags: [cpp, binarysizes, executables, compilation]
excerpt_separator: <!--more-->
---
In the previous article of this series on binary sizes, we discussed how special functions' implementations - or their lack of - influence the size of the generated binary.

Our conclusion was that if we could, we should follow the rule of 0. Not only because of simplicity for the reader but also for the compiler. If that's not possible, we should not declare destructors virtual in vain. And if it's possible, we should `=default` non-`virtual` destructors in the header both for readability and for binary sizes.

On the other hand, if we must have a non-`virtual` destructor, we should clearly think about implementing (preferably with `=default`) the destructor in the `.cpp` file, in other words, out-of-line. While you might find it strange in terms of readability, it produced a smaller binary.

It's still worth noting that in most circumstances, a class with a `virtual` destructor contributes to a larger binary file.

But does it matter how many functions are virtual? Given the same amount of methods in a class, is there a difference in terms of binary size whether one or all of them are virtual?

That's the question we are going to discuss in this post.

## What does the `virtual` keyword do?

When any of a class-member function is declared with the `virtual` keyword, it means that the compiler cannot know during compile-time which implementation of the `virtual` function will be called.

How could it resolve during runtime which function is to be called?

As it's explained on [Johnny's Software Lab](https://johnnysswlab.com/the-true-price-of-virtual-functions-in-c/), the way to do it is not standardized. Yet, most compilers do it in a similar way.

For each class that has at least one `virtual` method, the compiler creates a virtual table. It's usually just referred to as a *vtable*. The *vtable* of each class contains a pointer to each of their `virtual` functions. A virtual table might not contain pointers only to the same class. If a derived class doesn't [override](https://www.sandordargo.com/blog/2018/07/05/cpp-override) a method of a base class, then it will point to that base class implementation. 

When an object is created at runtime, there is also a virtual pointer (*vpointer*) created pointing to the right virtual table.

So there is one *vpointer* for each object created at run-time, but only one *vtable* per type which is created per compile-time.

I think we can rightly expect that therefore a `virtual` method - due to the *vtable* - results in an increased binary size and we could also expect that any additional `virtual` method will add to the size of the binary as the *vtable* grows, but only a little bit.

## Validate our hypothesis

Now let's create two `main.cpp` files. In the first one, we are going to have two classes with a `virtual` destructor and with 3 members each having a non-`virtual` accessor. In the second example, the same class will have those accessors turned into `virtual` ones. I know it's not a particularly elaborate example, but it's a start.

```cpp
// main-single-virtual.cpp
#include <array>

class SingleVirtual {
public:
    SingleVirtual() = default;
    SingleVirtual(int a, int b, int c) : m_a(a), m_b(b), m_c(c) {}
    virtual ~SingleVirtual() = default;

    int getA() const { return m_a; }
    int getB() const { return m_b; }
    int getC() const { return m_c; }

private:
    int m_a = 0;
    int m_b = 0;
    int m_c = 0;
};


std::array<SingleVirtual, 10'000> a;

int main() { }

// main-many-virtuals.cpp
#include <array>

class ManyVirtuals {
public:
    ManyVirtuals() = default;
    ManyVirtuals(int a, int b, int c) : m_a(a), m_b(b), m_c(c) {}
    virtual ~ManyVirtuals() = default;

    virtual int getA() const { return m_a; }
    virtual int getB() const { return m_b; }
    virtual int getC() const { return m_c; }

private:
    int m_a = 0;
    int m_b = 0;
    int m_c = 0;
};

std::array<ManyVirtuals, 10'000> a;

int main() { }
```

Depending on the optimization level, the version where only the destructor is `virtual` was compiled into a binary with the size of `281,488`/`281,520`. The fully virtual version had slightly bigger binaries, `281,615`/`281,647`.

<!--
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtuals-vs-crtp/basic $ bem main-single-virtual                                                                                                                                          17:29
  1.86s user 0.34s system 89% cpu 2.463 total
281,520
  1.84s user 0.34s system 87% cpu 2.492 total
281,488
  1.79s user 0.32s system 86% cpu 2.455 total
281,488
  1.77s user 0.32s system 87% cpu 2.383 total
281,520
  0.04s user 0.14s system 28% cpu 0.622 total
sandord@G60XRLP2XM ~/personal/dev/trynfail/binaries/virtuals-vs-crtp/basic $ bem main-many-virtuals                                                                                                                                           17:30
  1.77s user 0.34s system 88% cpu 2.391 total
281,647
  1.79s user 0.32s system 88% cpu 2.373 total
281,615
  1.79s user 0.32s system 87% cpu 2.406 total
281,615
  1.76s user 0.31s system 87% cpu 2.353 total
281,647
  0.04s user 0.11s system 35% cpu 0.400 total
-->

We can see that the difference is small. In the previous articles, I used an array of 10,000 objects so that we don't have to look into tiny differences. But in this case, it's worth having a look at the exact size of the difference.

It's less than 200 bytes per 10,000 objects. To be more precise, the difference is 127 bytes, way less than one byte per object. It cannot have anything to do with the number of objects. It is only about the size of the *vtable*. It's almost negligible.

First, I ran both examples with a much smaller array of only 10 objects. The difference between the two versions was still exactly 127 bytes.

As we start to remove the `virtual` keywords from the accessors one by one, the difference also shrinks. First by 48 byes to 79 bytes, then by another 32/48 bytes (depending on the optimization level) to 31-47 bytes. And finally, by devirtualizing the third accessor, the difference shrinks by another 48 bytes depending.

Another observation that we can make, is that at the end, `ManyVirtuals` ended up with a smaller binary. That's because its name is shorter. But that only mattered when the optimization level was `-O0`

## What can we learn from this?

It seems that the size of a *vtable* entry is about 48 bytes, at least on Apple Clang 15. It might not seem a big deal at first, but if we have 10 classes with 3 virtual methods each that's already more than 1KB. But these numbers can be much higher, especially if we consider a `virtual` method in a class template. That can quickly explode if we don't pay attention.

At the same time, we can only rest assured that the length of class/function names is not a problem anymore as it was in the past - given that we turn on compiler optimizations.

If we have a look at the assembly code after an unoptimized build (so that it remains somewhat readable), we can see things that seem like a list, probably the virtual table:

```asm
// SingleVirtual.s

    .section    __DATA,__const
    .globl  __ZTV13SingleVirtual            ; @_ZTV13SingleVirtual
    .weak_def_can_be_hidden __ZTV13SingleVirtual
    .p2align    3
__ZTV13SingleVirtual:
    .quad   0
    .quad   __ZTI13SingleVirtual
    .quad   __ZN13SingleVirtualD1Ev
    .quad   __ZN13SingleVirtualD0Ev


// ManyVirtuals.s

    .section    __DATA,__const
    .globl  __ZTV12ManyVirtuals             ; @_ZTV12ManyVirtuals
    .weak_def_can_be_hidden __ZTV12ManyVirtuals
    .p2align    3
__ZTV12ManyVirtuals:
    .quad   0
    .quad   __ZTI12ManyVirtuals
    .quad   __ZN12ManyVirtualsD1Ev
    .quad   __ZN12ManyVirtualsD0Ev
    .quad   __ZNK12ManyVirtuals4getAEv
    .quad   __ZNK12ManyVirtuals4getBEv
    .quad   __ZNK12ManyVirtuals4getCEv
```

For `SingleVirtual`, we can see an entry for the constructors and the destructor. We can also see that it's in the `const DATA` section. But there is nothing for the getter methods. On the other hand, those are there in `ManyVirtuals`. I couldn't figure out why there are entries for the constructors which are not virtual in C++. If you happen to know, please share in the comments or by e-mail.

## Conclusion

In this post, we saw how little it matters whether we add a new `virtual` method to a class that has already a `virtual` destructor. Having a `virtual` destructor ensures that everything necessary is instrumented for polymorphic behaviour which can make a huge difference in your binary size - and runtime performance. At the same time, adding another `virtual` function barely adds to the size of the binary. It'll only mean an extra entry in your *vtable*.

In the next episode, we'll have a look into two classic design patterns. We'll take a classical reference semantic and a modern value semantic implementation and see how much that matters in terms of binary size.
