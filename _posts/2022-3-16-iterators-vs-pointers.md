---
layout: post
title: "C++ basics: Pointers vs iterators"
date: 2022-3-16
category: dev
tags: [cpp, tutorial, iterators, pointers]
excerpt_separator: <!--more-->
---
Do you sometimes feel that you cannot explain the most basic things of a language you work with? You're asked a simple question and suddenly you can only say *"eeeeeeeh, I have to check, sorry.*"

Don't worry. Often we take things for granted, and until a less experienced person asks such a question, we don't even think about them. But sometimes it's worth going back to the basics and deepening or simply refreshing our knowledge.
<!--more-->

Today, let's discuss pointers and iterators.

## Pointers

Let's start with the dreaded pointers which can make C and C++ difficult to learn compared to other languages.

### What is a pointer?

First of all, a pointer is a type of variable that is meant to store a memory address.

I say meant to, because if it's correctly initialized it either stores `nullptr` or the address of another variable - it can even store the address of another pointer -, but if it's not correctly initialized, it will contain random data which is quite dangerous, it can lead to undefined behaviour.

### How can you initialize a pointer?

You have three different ways!

- Take the address of another variable:

```cpp
#include <iostream>

int main(){
  int v = 42;
  int* p = &v;
}
```

- Point it to a variable on the heap
```cpp
#include <iostream>

int main(){
  int* p = new int {42};
}
```

- Or just take the value of another pointer
```cpp
#include <iostream>

int main(){
  int* p = new int {42};
  int* p2 = p;
}
```

### Pointer values and pointed values

In any case, if you print the value of a pointer, it will be a memory address. If you want to get the pointed value, you have to dereference the pointer with `operator*`.

```cpp
#include <iostream>

int main(){
  int* p = new int {42};
  int* p2 = p;
  std::cout << p << " " << *p << '\n';
  std::cout << p2 << " " << *p2 << '\n';
  std::cout << &p << " " << &p2 << '\n';
}
/*
0x215dc20 42
0x215dc20 42
0x7fff77592cb0 0x7fff77592cb8
*/
```

In this example, we can see that both `p` and `p2` stores the same memory address and therefore they locate the same value too. At the same time, the addresses of the pointers themselves are different - taken by `operator&`.

### Memory deallocation

If an allocation happens with the `new` operator, in other words, if an allocation is on the heap, someone has to deallocate the allocated memory which happens with `delete`. Should you forget to do it when the pointer goes out of scope and you will have a memory leak. 

You'll have no more access to that place of memory and as it's not deallocated nobody else can use it. Should your code run long enough and create enough memory leaks, it might crash as it won't have access to enough memory anymore. So make sure you deallocate all allocated memory.

```cpp
#include <iostream>

int main() {
  int* p = new int {42};
  std::cout << p << " " << *p << '\n';
  delete p; 
}
```

If you try to access the pointer after the deletion, or if you try to delete it a second time, that's undefined behaviour and you'll most probably face a core dump.

Such errors often happen in legacy code, for example in such scenarios:

```cpp
#include <iostream>

int main(){
  int* p = new int {42};
  std::cout << p << " " << *p << '\n';
  
  bool error = true;
  
  if (error) {
    delete p; 
  }
  
  // ...
  delete p; 
}
```

`error` obviously is assigned from a more complex computation and usually, the 2 deletions are not added to the code at the same time.

The poor man's defence technique is to assign `nullptr` to `p` after deletion. If you try to delete the pointer again, it won't have any effect as deleting a `nullptr` is a no-op.

```cpp
#include <iostream>

int main(){
  int* p = new int {42};
  std::cout << p << " " << *p << '\n';
  
  bool error = true;
  
  if (error) {
    delete p;
    p = nullptr;
  }
  
  // ...
  delete p; 
  p = nullptr;
}
```

The other thing to do is to always check for ptr validity before you access one. But even if we ignore the problems of thread safety, we cannot feel safe. What if a pointer was deleted already and not set to `nullptr`? Undefined behaviour, potentially a crash. Or even worse...

```cpp
#include <iostream>

int main(){
  int* p = new int {42};
  if (p != nullptr) {
    std::cout << p << " " << *p << '\n';
  }
  
  delete p; // we forget to set it to nullptr
  if (p != nullptr) { // we pass the condition
    std::cout << p << " " << *p << '\n';
  }
}
/*
0x22f3c20 42
0x22f3c20 0
*/
```

Or what if you made a copy of the pointer? You delete one pointer and set it to `nullptr`. The copied sibling will not know that the other was deleted:

```cpp
#include <iostream>

int main(){
  int* p = new int {42};
  int* p2 = p;
  
  if (p != nullptr) {
    std::cout << p << " " << *p << '\n';
  }
  
  delete p; // we forget to set it to nullptr
  p = nullptr;
  
  if (p != nullptr) { // p is nullptr, we skip this block
    std::cout << p << " " << *p << '\n';
  }
  
  
  if (p2 != nullptr) { // we pass the condition and anything can happen
    std::cout << p2 << " " << *p2 << '\n';
  }
}
/*
0x1133c20 42
0x1133c20 0
*/
```

This case can easily happen when you have classes managing resources via raw pointers and their copy/move operations are not correctly implemented.

### Iterate over arrays

One more thing to mention about pointers is the operations you can perform on them. We often refer to them as pointer arithmetics. Meaning that you can increment or decrement them (perform addition and subtraction). But in fact, you can add or subtract any integer... Using the increment/decrement feature, pointers can be used to iterate over arrays or to access any element of them.

```cpp
#include <iostream>

int main(){
  int numbers[5] = {1, 2, 3, 4, 5};
  int* p = numbers;
  
  for(size_t i=0; i < 5; ++i) {
    std::cout << *p++ << '\n';
  }
  for(size_t i=0; i < 5; ++i) {
    std::cout << *--p << '\n';
  }

  std::cout << '\n';
  std::cout << *(p+3) << '\n';
}
/*
1
2
3
4
5
5
4
3
2
1

4
*/
```
Nice, but in 2022 should we use pointers to iterate over arrays?

The answer is clearly no. It's not safe, a pointer can just point anywhere and it doesn't work with all the container types.

You might have noticed in the previous example that in the first loop we use post-fix increment and in the second loop a pre-fix decrement. After counting up, the pointer already points to an invalid location, so we have to decrement it before dereferencing, otherwise, we risk undefined behaviour.

### Do not use raw pointers

In fact, nowadays there is not much reason to use raw pointers at all. Especially not raw pointers that are allocated with new, raw pointers that are owning their resources. Passing around resources via a raw pointer is still okay, but owning those resources or using pointers as iterators or expressing that a value might or might not be there is something you shouldn't tolerate in your codebase anymore.

We have different better options.

First of all, we can use smart pointers to replace owning raw pointers. 

When we use non-owning pointers, we might use references if something cannot be `nullptr` or if we want to express that something might or might not be present, we could try `std::optional`. But more on this another day.

Let's focus on iterating over an array now and let's see some other options, what can we do with iterators?

## What is an iterator?

Iterators are an essential part of the Standard Template Library. The STL has 4 main building blocks:

- [algorithms](https://www.sandordargo.com/blog/2019/01/30/stl-algos-intro) (`std::rotate`, `std::find_if`, etc.)
- containers (`std::vector<T>`, `std::list<T>`, etc.)
- function objects (`std::greater<T>`, `std::logical_and<T>`, etc.)
- iterators (`std::iterator`, `std::back_inserter`, etc.)

Iterators are the result of the generalization of the concept of a pointer. They can be used to iterate over the elements of an STL container and provide access to the individual elements.

The mention of the STL containers also means that they cannot be used with C-style arrays. It's fine, we should not use C-style arrays at all in 2021.

### The 5 categories of iterators

There are essentially 5 categories of iterators:
- input iterators
- output iterators
- forward iterators
- bidirectional iterators
- random access iterators

*Input iterators* are the simplest form of iterators. They are supporting read-operations and can only move forward. You can use input iterators for (in)equality comparisons and they can be incremented. An example would be the iterator of a `std::list`.

*Output iterators* are also forward iterators, but they are used to assign values in a container, they are write-only iterators. You cannot use them to read values. Such an iterator is the `std::back_inserter` iterator.

*Forward iterators* are the combination of input and output iterators. They let us both access and modify values. `std::replace` uses forward iterators for example. Forward iterators are default constructible and they can access/dereference the same positions multiple times.

*Bidirectional iterators* are like forward iterators, but they can be also decremented, so they can move both forward and backward. `std::reverse_copy` uses such iterators as it both has to reverse values of a container (decrement) and put results in a new container one after the other (increment).

*Random access iterators* are capable of anything that bidirectional iterators can do. In addition, they cannot only be incremented or decremented but their position can be modified by any value. In other words, they support `operator+` and `operator-`. Different random access iterators can also be compared with the different comparison operators (not just with equality/inequality). Random access means that containers accepting random-access iterators can be simply accessed with the offset operator. An algorithm that needs random-access iterators is `std::random_shuffle()`.

### Usage of iterators

Iterators can be obtained from containers essentially two different ways:
- through member functions such as `std::vector<T>::begin()` or `std::vector<T>::end()`
- or via free functions such as `std::begin()` or `std::end()`

There are different variations of iterators, from a practical point of view, they can be `const` or reversed direction as well.

Just like pointers, iterators can be incremented or decremented which makes them suitable for loops. Though before C++11 they were a bit verbose to use:

```cpp
#include <iostream>
#include <vector>

int main(){
  std::vector<int> v {1, 2, 3, 4, 5};
  for (std::vector<int>::const_iterator it=v.begin(); it != v.end(); ++it) {
    std::cout << *it << " ";
  }
}
```

With C++11 and the introduction of the keyword `auto`, the usage of iterators was simplified quite a bit.

```cpp
#include <iostream>
#include <vector>

int main(){
  std::vector<int> v {1, 2, 3, 4, 5};
  for (auto it=v.begin(); it != v.end(); ++it) {
    std::cout << *it << " ";
  }
}
```

Of course, you might argue that [range-based for loops](https://en.cppreference.com/w/cpp/language/range-for) are easier to use, and you are right. It's worth mentioning though that range-based for loops are also implemented with the help of iterators.

## How an iterator differs from a pointer

Now that we discussed both pointers and iterators separately, let's collect the differences between the two categories.

While we are using pointers to hold a memory address, whatever memory address, an iterator is always used with containers. An iterator is used to go through the elements of a container and the items of the container don't need to be stored on a contagious memory area. Even if the items are scattered in the memory, such as for a linked list, an iterator would still work. 

Given that the pointer is always storing a memory address, it can be always be converted to an integer (which is the address). Most iterators cannot be converted into integers.

As we saw there are 5 different categories of iterators and not all of them support all the different pointer arithmetic operations. At the same time, pointers don't have any such distinction. A pointer is a pointer and you can do all the operations with them - which is often quite dangerous.

If you declare a pointer to a type, it can point to any object of the same type. Luckily, iterators are more restricted and they work only inside a certain type of container.

If you ever used raw pointers, you know that they can be deleted, moreover, the owning ones must be deleted in order to avoid memory leaks. Iterators on the other hand cannot be, should not be deleted. An iterator is not responsible for memory management, its sole responsibility is to provide a handle to the elements in the container.

## When to use one and when the other?

Whenever you need to iterate over a standard container, use an iterator over a pointer. As it was designed exactly for that, it's safer and that's what you'd get anyway if you'd call `begin()` or `end()` on the container. Moreover, it's iterators that STL algorithms are taking as inputs, not pointers and also that's what they often return.

There are two reasons not to use iterators:
- using a [range-based for loop](https://en.cppreference.com/w/cpp/language/range-for) that you should indeed prefer, but under the hood, in most cases, they use iterators anyway
- using a C-style array. But in 2021, don't use a C-style array, you can use `std::array` or another STL container.

Don't use pointers for iterations. Use pointers only when you need to pass the address of a variable to another function and when it might be null so you cannot use a reference instead. 

Pointers also come in handy when you have to deal with polymorphism and you need dynamic dispatching, you need to determine which version of a `virtual` function should be called only during runtime.

For memory handling, don't use (raw) pointers. If you need to use dynamic memory allocations, if you need the heap, use a smart pointer instead of a raw pointer so that you can avoid memory leaks or double frees.

## Conclusion

I wish I understood the basics of C++ at the beginning of my developer career. 

I wish I understood them today.

With this piece, I'm a bit closer to understanding the basics of pointers and iterators, I hope you do too.

## References
- [Apache C++ Standard Library User's Guide: Varieties of Iterators](https://stdcxx.apache.org/doc/stdlibug/2-2.html)
- [University of Helsinki: STL Iterators](https://www.cs.helsinki.fi/u/tpkarkka/alglib/k06/lectures/iterators.html)
- [GeeksForGeeks: Difference between Iterators and Pointers in C/C++ with Examples](https://www.geeksforgeeks.org/difference-between-iterators-and-pointers-in-c-c-with-examples/)
- [Microsoft: Raw pointers (C++)](https://docs.microsoft.com/en-us/cpp/cpp/raw-pointers?view=msvc-160)
- [Stackoverflow: Why should I use a pointer rather than the object itself?](https://stackoverflow.com/questions/22146094/why-should-i-use-a-pointer-rather-than-the-object-itself)

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!