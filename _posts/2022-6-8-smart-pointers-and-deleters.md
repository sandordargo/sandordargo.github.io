---
layout: post
title: "Smart pointers and their deleters"
date: 2022-6-8
category: dev
tags: [cpp, smartpointers, performance, typeerasure] 
excerpt_separator: <!--more-->
---
[Bjarne Stroustrup](https://www.stroustrup.com/), the father of C++ once said that  *"C makes it easy to shoot yourself in the foot; C++ makes it harder, but when you do it blows your whole leg off."* Why did he say so? What makes C++ so dangerous?
<!--more-->

In fact, it's probably better to ask what **made** C++ so dangerous. The answer to that question is arguably memory management. Allocating memory on the heap with the `new` keyword and making sure that the memory always gets deallocated with `delete` and exactly once used to be a difficult task. And whenever you failed, you were punished hard at unexpected times. And we haven't even mentioned `malloc` and `free`...

With C++11, we received smart pointers so that it's not an issue anymore. Smart pointers are considered smart because they track their own lifetime and take care of deallocating the memory. No manual actions required.

C++11 did not introduce only one smart pointer, but 3 of them right away. As well informed  C++ developers, we'd better understand which one to choose and why.

Let's dig into the why in this article. 

## What kind of smart pointer should you choose?

Let's not waste too much time on `std::weak_ptr`. They have a specific usecase and we barely need them, but when we do we don't much choice. Let's just say that we use should use them to break the cycle in the case of cyclic ownership. 

> Let's say that you have an instance of a Keyboard, a Logger, and a Screen object. Both the Screen and the Keyboard has a shared pointer of the Logger and the Logger should have a pointer to the Screen.
>
> `Keyboard -> Logger <--> Screen`
>
> What pointer can it use? If we use a raw pointer when the Screen gets destroyed, the Logger will be still alive and the Keyboard still has shared ownership on it. The Logger has a dangling pointer to the Screen.
>
> If it's a shared pointer, there is a cyclic dependency between them, they cannot be destroyed, there is a resource leak.
> Here comes the `std::weak_ptr` to the rescue. There is still a dangling pointer from Logger to the Screen, when the Screen gets destroyed, but it can be easily detected with a weak pointer.

That leaves us with the choice of either a shared or a unique pointer. My experience in big corporate codebases show that people by default choose the `std::shared_ptr`, whereas they should do exactly the opposite.

But why do they choose a shared pointer over a unique one? I think simply because it's easier to use. A `unique_ptr` is not copy-able, thus if you have to pass around you either have to dereference it and pass around the raw pointer, or you have to use `std::move`. With shared pointers, you don't put yourself up to this hassle.

The key for making the right choice is education. 

Let's consider two things.

Types communicate meaning through their names. Is the ownership really shared between different owners or is there only one entity that can *own* a resource? Usually it's the latter case and it's a good enough reason to use the `unique_ptr`. Not to mention that once you're sure that a pointer must be valid, you can simply pass around a reference...

Another thing to take into consideration is performance benefits. Shared pointers are more expensive than unique pointers that essentially do not bring any overhead compared to raw pointers. 

## Why are unique pointers cheaper?

It's way better when we don't only know some facts and take them granted but when we actually understand the reasons behind. Let's dig into why shared pointers are more expensive than unique pointers. Let's start with the answer you probably already heard about before we delve into the more surprising.

### Reference counting

A unique pointer holds a pointer that is referred to by only entity, the owner. Hence it's unique. Once it goes out of scope, the pointer is deleted. But the resource held by the shared pointer can be referred to by other shared pointers and it has to know when to destroy the resource. For that it counts how many others refer to the resource. In fact, it has 2 counters counting the number of shared and weak pointers.

The counters take up some space and maintaining the counters needs some instructions, it needs some time. It has its consequences in terms of performance.

But is that the main and only reason behind why shared pointers are slower than smart ones?

It's definitely not the only reason, and often not even the main one.

### Type erasure / deleters

Both unique and shared pointers can take custom deleters. They can be useful, if you want do something non-conventional while deleting the resource. (Like not deleting it... or maybe logging).

Here is how to use it.

```cpp
#include <iostream>
#include <memory>

template <typename T>
struct FakeDeleter {
  void operator()(T *ptr){
    std::cout << "FakeDeleter doesn't delete\n";
  } 
};

template <typename T>
struct LoggingDeleter {
    void operator()(T *ptr){
    std::cout << "LoggingDeleter is at work\n";
    delete ptr;
  } 
};

int main() {
    std::unique_ptr<int, FakeDeleter<int>> up (new int(42), FakeDeleter<int>());
    std::shared_ptr<int> sp (new int(51), FakeDeleter<int>());
}
```

Notice how the creation of the pointers differ. We pass in both cases the deleter as arguments to the constructor, but it only appears only for the `unique_ptr` as a template argument.

What does this mean for us?

The deleter is part of the type of the unique pointer, for example this expression would not compile as a move assignment between different types - without available implicit conversion - is not permitted.

```cpp
std::unique_ptr<int, FakeDeleter<int>> upFD (new int(42), FakeDeleter<int>());
std::unique_ptr<int, FakeDeleter<int>> upFD2 (new int(51), FakeDeleter<int>());
std::unique_ptr<int, LoggingDeleter<int>> upLD (new int(42), LoggingDeleter<int>());
upFD = std::move(upFD2); // OK
upFD = std::move(upLD); // NOT OK, fails to compile!
```

On the other hand, we have no such issues with shared pointers!

```cpp
std::shared_ptr<int> spFD (new int(51), FakeDeleter<int>());
std::shared_ptr<int> spFD2 (new int(51), FakeDeleter<int>());
std::shared_ptr<int> spLD (new int(51), LoggingDeleter<int>());
spFD = spFD2;
spFD = spLD;
```

How is this possible?

For unique pointers, the deleter is a class template parameter, while for shared pointers it's only a template parameter in the constructor. At the end of the day, a deleter is stored as it was passed for unique pointers, but shared pointers apply type erasure on it which also means an extra allocation on the heap and another layer of indirection.

This also makes shared pointers less performant than unique pointers.

In fact, according to the measurements I saw in [Hands-On Design Patterns with C++](https://www.amazon.com/Hands-Design-Patterns-reusable-maintainable/dp/1788832566?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=c2d690d8e9ee318ad3a6ada4c1a55ec5&camp=1789&creative=9325), the performance overhead due to type erasure is - by default - more significant than reference counting.

On the other hand, most of the negative performance impacts of erasing the deleter type can be optimized away with Local Buffer Optimization. Without going into deep details on it, it means that when the compiler allocated memory for the shared pointer, it allocates a bit more so that it's enough for the deleter too and therefore no second allocation is needed.
Obviously, reference counting cannot be optimized away.


## Conclusion

In this article, after having a small recap on smart pointers, we discussed why unique pointers are cheaper than shared ones. We saw that it's not only about reference counting - which is probably the most well-known cause - but also about the erasure of the deleter type which might add even more to the differences.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!