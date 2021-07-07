---
layout: post
title: "2 ways to use static with functions in C++"
date: 2021-7-7
category: dev
tags: [cpp, tutorial, static]
excerpt_separator: <!--more-->
---
I've been doing a code review lately and I saw the following piece of code (I anonymized it) in a `.cpp` file:
<!--more-->
```cpp
static bool isWineColour(const std::string& iWineCoulour) {
  static const std::array<std::string, 3> wineCoulours{ "white", "red", "rose" };
  return std::find(wineCoulours.begin(), wineCoulours.end(), iWineCoulour)
         != wineCoulours.end();
}
```
I read the code and it made sense, but I didn't really get it. WTF. Do we return a `static bool`? What? I've never seen anything like that in a `cpp` file and it wouldn't make sense, would it?

But it said `static bool` and we are not in the header. There is no `isWineColour()` function declared in the header at all.

At this point, I understood that either there is something very wrong here or I am missing the point. Given that the code compiled, the tests succeeded and SonarQube didn't report any code smells, it was pretty clear that I was missing the point.

Just to make it clear, before I reveal the *big secret* (no, there is no big secret...) there is no such thing as a `static` return type. When the keyword `static` appears in front of the return type, it might be mean one of these two possibilities:

- a member function is `static`
- a free-function cannot be accessed by any other translation unit

So the difference between the two usages is that in one case, we use `static` with a member function in the other we use it with a free-function.

Let's get into details.

## `static` member functions

Okay, probably this one you already knew. If you make a class member function static, it means that you can call it without going through an instance of the class.

```cpp
#include <iostream>
#include <type_traits>

class A {
public:
  static void Foo() {
      std::cout << "A::foo is called\n"; 
  }
    
};

int main() {
  A a;
  a.Foo();
  A::Foo();
}
/*
A::foo is called
A::foo is called
*/
```

As you can see, it's possible to call `Foo()` both via an instance (`a.Foo()`) or just via its enclosing class (`A::Foo()`).

There are a couple of characteristics to keep in mind:

- `static` member functions don't have `this` pointer
- A `static` member function can't be virtual
- `static` member functions cannot access non-`static` members
- The `const`, `const volatile`, and `volatile` declarations aren't available for `static` member functions

As `this` pointer always holds the memory address of the current object and to call a static member you don't need an object at all, it cannot have a `this` pointer.

A `virtual` member is something that doesn't relate directly to any class, only to an instance.  A *"`virtual` function"* is (by definition) a function that is dynamically linked, i.e. it's chosen at runtime depending on the dynamic type of a given object. Hence, as there is no object, there cannot be a virtual call.

Accessing a non-`static` member requires that the object has been constructed but for static calls, we don't pass any instantiation of the class. It's not even guaranteed that any instance has been constructed.

Once again, the `const` and the `const volatile` keywords modify whether and how an object can be modified or not. As there is no object...

Probably we all got used to `static` member functions already. Let's jump to the other usage of `static` with functions.

## `static` free functions

Normally all functions declared within a `cpp` file have external linkage by default, meaning that a function defined in one file can be used in another `cpp` file by forward declaration.

As I recently learned, we can declare a free-function `static` and it changes the type of linkage to internal, which means that the function can only be accessed from the given translation unit, from the same file where it was declared and from nowhere else.

With internal linkage, the linker can ignore the `static` free-functions entirely bringing a couple of advantages:
- the free-function can be declared in a `cpp` file and we have a guarantee that it will not be used from any other place
- speeds up the link-time as there is one less function to take care of
- we can put a function with the same name in each translation unit and they can be implemented differently. For example, you could create a logger that is implemented differently in each translation unit.

## Conclusion

Today I shared with you what I learned recently from a code review that I was doing for someone else. I learnt that we can declare `static` not only class member functions, but free-functions as well. 

Having a class member function static means that it's part of the class, but there is no instance needed to call it, hence it cannot interact with members of the class.

Declaring a free-function static is about its visibility and the type of linkage. If you declare a free-function static, it will have an internal linkage and will not be accessible from any other file.

Have you ever used static free functions?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
