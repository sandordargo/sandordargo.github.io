---
layout: post
title: "When to use const in C++? Part I: functions and local variables"
date: 2020-11-4
category: dev
tags: [cpp, tutorial, const]
excerpt_separator: <!--more-->
---
_Just make everything `const` that you can! That's the bare minimum you could do for your compiler!_

This is a piece of advice, many _senior_ developers tend to repeat to juniors, while so often even the preaching ones - we - fail to follow this rule.
<!--more-->

It's so easy just to declare a variable without making it `const`, even though we know that its value should never change. Of course, our compiler doesn't know it.

It's not enough that we fail to comply with our own recommendations, we are also not specific enough. So if others just blindly follow our recommendations without much thinking, then it just messes things up. Compilation failures are easy to spot early on, but dangling references or worse performance due to extra copies are more difficult to identify. Hopefully, those are caught not later than [the code review](https://www.sandordargo.com/blog/2018/03/28/codereview-guidelines).

But don't be mad at the people following your words blindly. If you share pieces of advice without much thinking if you don't expect critical thinking from yourself, why would you expect more from others?

I digressed, let's get back to our topic. So what kind of `const`s are out there?

In this series of articles, we'll discuss about:
In this series of articles, we discuss about:
- [`const` functions](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` local variables](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` member variables](https://www.sandordargo.com/blog/2020/11/11/when-use-const-2-member-variables)
- [`const` return types](https://www.sandordargo.com/blog/2020/11/18/when-use-const-3-return-types)
- [`const` parameters](https://www.sandordargo.com/blog/2020/11/25/when-use-const-4-parameters)

In this post, we are going to explore const functions and const local variables.

Let's get started.

## `const` functions
You can declare a non-static member function `const` if it doesn't change the value of the underlying object. This is recursive in a sense, that it cannot modify any of the members. To guarantee that, it cannot call non-const functions on its members.

```cpp
#include <iostream>

class A {
  public:
  void bar() {
  // ...
  } 
};

class B {
public:
  void foo() const {
    a.bar(); // A::bar() is not const, so this call generates a compilation error!
  }
private:
 A a{};
};

int main() {
  auto b{B{}};
  b.foo();
}
```
On the other hand, we can call non-const functions on locally initialized objects or on function parameters.

In case a function has two overloaded versions where one is `const` and the other is not, the compiler will choose which one to call based on whether the object itself is const or not.

The feature of `const` functions is something you should use all the time. Making the function `const` is _meaningful_. It helps the compiler to use optimizations and in addition, it clarifies the intent of the author. It shows the reader that if he calls such a function it will not have any effect on the members' state.

Use it without moderation.

## `const` variables

If you declare a local variable `const`, you simply mark it immutable. It should never ever change its value. If you still try to modify it later on, you'll get a compilation error. For global variables, this is rather useful, as otherwise, you have no idea who can modify their value. Of course, you should not use global variables and then you don't face the problem...

Those global `const`s can introduce coupling among your classes or even components that you should avoid otherwise. You might even face the static initialization order fiasco, but this a problem for another day...

Otherwise, declaring variables as `const` also helps the compiler to perform some optimizations. Unless you explicitly mark a variable `const`, the compiler will not know (at least not for sure) that the given variable should not change. Again this is something that we should use whenever it is possible.

In real life, I find that we tend to forget the value making variables const, even though there are [good examples at conference talks](https://youtu.be/zBkNBP00wJE?t=1614) and it really has no bad effect on your code, on maintainability.

This is such an important idea that in Rust, all your variables are declared as `const`, unless you say they should be mutable.

We have no reason not to follow similar practices.

Declare your local variables `const` if you don't plan to modify them. Regarding global variables, well, avoid using then, but if you do, also make them `const` whenever possible.

## Conclusion

Today, we started a new series about when and how to use the `const` keyword in C++. In this episode, we learned about `const` local/global variables and `const` functions. They come for free and even let the compiler to make some optimizations. At the same time, they increase the readability of your code. Use them without moderation.

On the other hand, I've never simply said variables. It's because not the same considerations apply to member variables.

Stay tuned, next time we'll learn about whether having `const` member variables is a good idea or not.