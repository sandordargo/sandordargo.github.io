---
layout: post
title: "const and smart pointers"
date: 2021-7-21
category: dev
tags: [cpp, tutorial, const, smart pointers]
excerpt_separator: <!--more-->
---
What does `const` have to do with smart pointers?

There are two ways to approach this. One way is to simply consider that smart pointers are effectively pointers. As such, either they can be const, or the type they hold - or maybe even both.
<!--more-->

In another perspective, we consider that smart pointers are class type objects. After all, they are wrapping pointers. As a smart pointer is an object, the rule of thumb might say that it can be passed around as a `const` reference. We are going to see that it's a bad idea.

Let's see both perspectives.

## `const` and smart pointers as pointers

As we said earlier, smart pointers are effectively pointers. Just smart ones. Therefore when we can use `const` both with the pointer itself or with the pointed value.

There are different smart pointers, but as our default choice should be `std::unique_ptr`, I'll use that one throughout our examples, except when I explicitly need a shared pointer to demonstrate a concept.

### `const std::unique_ptr<T>`

In this case, it's the pointer that is `const` and not what we point to. It means that we cannot reset the pointer, we cannot change what it points to. At the same time, the value it points to can be modified.

```cpp
#include <memory>

int main() {
    const std::unique_ptr<int> p = std::make_unique<int>(42);
    ++(*p); // OK, data is not const
    // p.reset(new int{666}); // ERROR cannot reset const pointer
}
```

It's worth noting that if you have a `const unique_ptr`, you're very limited in what you can do. You cannot even return it as it requires moving away from it.

```cpp
const std::unique_ptr<int> f() {
    const std::unique_ptr<int> p = std::make_unique<int>(42);
    return p;
}
/*
error: use of deleted function 
'std::unique_ptr<_Tp, _Dp>::unique_ptr(const std::unique_ptr<_Tp, _Dp>&) 
[with _Tp = int; _Dp = std::default_delete<int>]'
    |     return p;
    |            ^
*/
```

At the same time, it's also worth noting that having the return type `const` is not a problem starting from C++17. The below snippet produces the same error on C++14 as the above one, but passes with C++17:

```cpp
const std::unique_ptr<int> f() {
    std::unique_ptr<int> p = std::make_unique<int>(42);
    return p;
}
```
### `std::unique_ptr<const T>`

In this case, the value the pointer points to is `const`, but the pointer itself is mutable. In other words, you cannot change the value of the pointed data, but you can change what the pointer points to.

```cpp
#include <memory>

int main() {
    std::unique_ptr<const int> p = std::make_unique<const int>(42);
    // ++(*p); // ERROR, data is const
    p.reset(new int{51}); // OK, pointer is not const
}
```

I have to make a remark. In the expression `std::make_unique<const int>(42)` the `const` is not mandatory, the code will compile even if we forget the `const`. But we should rather not forget it. If you check it in compiler explorer, missing the `const` results in an extra move constructror call and we also have to destruct the temporary object:

```asm
mov     DWORD PTR [rbp-20], 42
lea     rax, [rbp-32]
lea     rdx, [rbp-20]
mov     rsi, rdx
mov     rdi, rax
call    std::_MakeUniq<int>::__single_object std::make_unique<int, int>(int&&)
lea     rdx, [rbp-32]
lea     rax, [rbp-40]
mov     rsi, rdx
mov     rdi, rax
call    std::unique_ptr<int const, std::default_delete<int const> >::unique_ptr<int, std::default_delete<int>, void>(std::unique_ptr<int, std::default_delete<int> >&&)
lea     rax, [rbp-32]
mov     rdi, rax
call    std::unique_ptr<int, std::default_delete<int> >::~unique_ptr() [complete object destructor]
```

In case we don't forget the `const` within `std::make_unique`, the above code simplifies to:

```asm
mov     DWORD PTR [rbp-36], 42
lea     rax, [rbp-48]
lea     rdx, [rbp-36]
mov     rsi, rdx
mov     rdi, rax
call    std::_MakeUniq<int const>::__single_object std::make_unique<int const, int>(int&&)
```

The bottom line, if you want a `const` smart pointer, use `const` both on the left and the right side given that you use the `std::make_*` functions.

### const std::unique_ptr<const T>

Not much surprise in this case, it's a combination of the two `const`s. In this case, both the pointed value and the (smart) pointer are const, therefore no change is accepted.

```cpp
#include <memory>

int main() {
    const std::unique_ptr<const int> p = std::make_unique<const int>(42);
    // ++(*p); // ERROR, data is const
    // p.reset(new int{51}); // ERROR, pointer is const
}
```

Don't forget the `const` on the right-hand side!

## `const` and smart pointers as objects

What is a smart pointer? A(n instance of a) smart pointer is an object consisting of a raw pointer and some data for lifetime management. The exact internal representation doesn't matter for our purpose now.

In other words, having a smart pointer means that we have a wrapper object around a pointer and as such one might think that the rule of thumb for class type parameters kicks in. What's that rule of thumb?

[It's that you should pass around class type parameters by reference, preferably by `const` reference.](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rf-in)

I personally found that passing smart pointers by reference is the result of syntactical difficulties and a lack of understanding.

A lack of understanding?

Yes, after all, what is the purpose of a smart pointer?

It's proper lifetime management without the need for calling `delete` manually.

When you pass a smart pointer by (`const`) reference, what you don't pass around is the (shared) ownership. In other words, you don't deal with ownership at all. If you don't deal with ownership, there is no need for smart pointers.

If you don't need smart pointers and passing ownership, you should just pass around a raw pointer. It will be faster and more readable.

More readable, because understanding what it means to have a reference of a pointer is quite some mental work.

And it will be also faster because you deal directly with the memory address of the pointed value, you don't have to pass through the layer(s) of a smart pointer.

There are 3 types of smart pointers in C++ and I think that with `std::weak_ptr` there is no problem in general. It's not so widely used, one could even say it's a niche and those who need it and use it, know exactly how to do that.

On the other hand, `unique_ptr` is often used incorrectly. It's just passed around by `const` reference because many don't know how to use it and how to read C++'s lengthy error messages.

When there is a function that creates a `unique_ptr` and returns it, there is no problem.

```cpp
#include <memory>

std::unique_ptr<int> foo() {
  std::unique_ptr<int> p = std::make_unique<int>(42);
  return p;
}

int main() {
  auto p = foo();
}
```

But when you have a function that receives a `unique_ptr` as an argument and later returns it, problems start to arise:

```cpp
#include <memory>

std::unique_ptr<int> bar() {
  std::unique_ptr<int> p = std::make_unique<int>(42);
  return p;
}

void foo(std::unique_ptr<int> ip) {
  ++(*ip);
}

int main() {
  auto p = bar();
  foo(p);
}
/*
main.cpp: In function 'int main()':
main.cpp:14:6: error: use of deleted function 'std::unique_ptr<_Tp, _Dp>::unique_ptr(const std::unique_ptr<_Tp, _Dp>&) [with _Tp = int; _Dp = std::default_delete<int>]'
   14 |   foo(p);
      |   ~~~^~~
*/
```

Bear in mind that this is a simplistic example, but it shows the problem. You want to pass a `uniqe_ptr` to a function and it doesn't work as you have a nasty error message.

What to do?

Some would try to take it by reference in `foo(std::unique_ptr<int>)` and it would work. Unfortunately, it's not the right thing to do, but it's an easy try and it works. 

Let's get back to that error message.

```
main.cpp: In function 'int main()':
main.cpp:14:6: error: use of deleted function 'std::unique_ptr<_Tp, _Dp>::unique_ptr(const std::unique_ptr<_Tp, _Dp>&) [with _Tp = int; _Dp = std::default_delete<int>]'
   14 |   foo(p);
      |   ~~~^~~
```

It says that you try to use a deleted function, namely the copy constructor, that is deleted.

Yes! The `unique_ptr` is not copyable, after all, it's supposed to be unique! So there is no copy constructor. But it doesn't mean we should pass it by (`const`) reference, no. It means we should move it, so the right solution is this:

```cpp
#include <memory>

std::unique_ptr<int> bar() {
  std::unique_ptr<int> p = std::make_unique<int>(42);
  return p;
}

void foo(std::unique_ptr<int> ip) {
  ++(*ip);
}

int main() {
  auto p = bar();
  foo(std::move(p));
}
```

Or if you don't want to pass the ownership to foo, you can simply pass it by a raw pointer:

```cpp
#include <memory>

std::unique_ptr<int> bar() {
  std::unique_ptr<int> p = std::make_unique<int>(42);
  return p;
}

void foo(int* ip) {
  ++(*ip);
}

int main() {
  auto p = bar();
  foo(p.get());
}
```

You might wonder, why would you pass a `shared_ptr` by (`const`) reference. There are no syntactical difficulties like with its unique counterpart. 

I have no right answer. It's probably a lack of understanding combined with good will.

I read an article recently where the author showcased that passing around shared pointers by reference is much faster than by value - as reference counting is expensive. It was even supported by a Quick Bench diagram. 

In fact, it can even happen that the reference counter reaches zero and the pointer gets deleted by the time the reference would refer to the pointer and then bumm... You had a false feeling of security with a `shared_pointer`.

Don't take smart pointers by reference. It won't pass or share the ownership. Pass it by value and if you don't have to deal with the ownership, fine, just use a raw pointer.

If you can avoid dynamic memory allocation, even better.

## Conclusion

Today we discussed smart pointers and `const`. First, we saw what it means to have a `const` smart pointer or a smart pointer to a `const` value.

Later we saw that by misunderstanding, by error, people might pass smart pointers by reference, even by `const` reference instead of value and we should never do that, we should always pass a smart pointer by value.

If you are interested in more details, read [this article by Herb Sutter](https://herbsutter.com/2013/06/05/gotw-91-solution-smart-pointer-parameters/).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
