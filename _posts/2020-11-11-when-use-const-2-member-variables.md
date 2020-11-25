---
layout: post
title: "When to use const in C++? Part II: member variables"
date: 2020-11-11
category: dev
tags: [cpp, tutorial, const]
excerpt_separator: <!--more-->
---
_Just make everything `const` that you can! That's the bare minimum you could do for your compiler!_

This is a piece of advice, many _senior_ developers tend to repeat to juniors, while so often even the preaching ones - we - fail to follow this rule.
<!--more-->

In this series of articles, we discuss about:
- [`const` functions](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` local variables](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` member variables](https://www.sandordargo.com/blog/2020/11/11/when-use-const-2-member-variables)
- [`const` return types](https://www.sandordargo.com/blog/2020/11/18/when-use-const-3-return-types)
- [`const` parameters](https://www.sandordargo.com/blog/2020/11/25/when-use-const-4-parameters)

In the first episode, we covered `const` functions and `const` local variables. Today we'll speak about the members.

Originally, I didn't plan this post. I simply wanted to speak about `const` variables regardless if they have a local scope or if they are members of an object.

Then I saw [this tweet from Tina Ulbrich](https://twitter.com/_Yulivee_/status/1310812389743435776) who I met at [C++OnSea2020](https://www.youtube.com/watch?v=y2OGpAqD-f8) and I was horrified. Yet another thing in C++, I had no idea about and something I've been doing considering that it's a good practice.

Truth to be told, I didn't do anything harmful, but that's only by chance.

Ok, let's get to it.

Why would you have `const` members at the first place?

Because you might want to signal that they are immutable, that their values should never change. Some would claim that you have private members for that purpose and you simply should not expose a setter for such members, then there is no need to explicitly make them `const`.

I get you, you're right. In an ideal world.

But even if you are a strong believer of the [Single Responsibility Principle]() and small classes, there is a fair chance that others later will change your code, your class will grow, and someone might accidentally change the value inside, plus you haven't given the compiler a hint for optimization due to immutability.

To me, these are good reasons to make a member const. At least to show the intention.

But unfortunately, there are some implications.

The first is that classes a const member are not assignable:
```cpp
class MyClassWithConstMember {
public:
  MyClassWithConstMember(int a) : m_a(a) {}
private:
  const int m_a;
};

int main() {
  MyClassWithConstMember o1{666};
  MyClassWithConstMember o2{42};
  o1 = o2;
}
/*main.cpp: In function 'int main()':
main.cpp:11:8: error: use of deleted function 'MyClassWithConstMember& MyClassWithConstMember::operator=(const MyClassWithConstMember&)'
   11 |   o1 = o2;
      |        ^~
main.cpp:1:7: note: 'MyClassWithConstMember& MyClassWithConstMember::operator=(const MyClassWithConstMember&)' is implicitly deleted because the default definition would be ill-formed:
    1 | class MyClassWithConstMember {
      |       ^~~~~~~~~~~~~~~~~~~~~~
main.cpp:1:7: error: non-static const member 'const int MyClassWithConstMember::m_a', cannot use default assignment operator
*/
```
If you think about it, it makes perfect sense. A `variable` is something you cannot change after initialization. And when you want to assign a new value to an object, thus to its members, it's not possible anymore.

As such it also makes it impossible to use move semantics, for the same reason.

From the error messages, you can see that the corresponding special functions, such as the assignment operator or the move assignment operator were deleted.

Let's implement the assignment operator. It will compile, but what the heck would you do?

```cpp
MyClassWithConstMember& operator=(const MyClassWithConstMember&) {
  // ???
  return *this;
}
```

Do you skip assigning to the const members? Not so great, either you depend on that value somewhere, or you should not store the value.

And you cannot assign to a const variable, can you? For a matter of fact, you can...

```cpp
#include <utility>
#include <iostream>

class MyClassWithConstMember {
public:
  MyClassWithConstMember(int a) : m_a(a) {}
  MyClassWithConstMember& operator=(const MyClassWithConstMember& other) {
    int* tmp = const_cast<int*>(&m_a);
    *tmp = other.m_a; 
    std::cout << "copy assignment \n";
    return *this;
  }
  
int getA() {return m_a;}
  
private:
  const int m_a;
};

int main() {
  MyClassWithConstMember o1{666};
  MyClassWithConstMember o2{42};
  std::cout << "o1.a: " << o1.getA() << std::endl;
  std::cout << "o2.a: " << o2.getA() << std::endl;
  o1 = o2;
  std::cout << "o1.a: " << o1.getA() << std::endl;

```

As you cannot cast the constness away from value, you have to turn the member value into a temporary non-const pointer and then you free to rampage.

Is this worth it?

You have your const member, fine. You have the assignment working, fine. Then if anyone comes later and wants to do the same "magic" outside of the special functions, for sure, it would be a red flag in a code review.

Speaking of special functions. Would move semantics work? Well, replace the assignment with this:

```cpp
o1 = std::move(o2);
```

You'll see that it's still a copy assignment taking place as the [the rule of 5](https://www.fluentcpp.com/2019/04/19/compiler-generated-functions-rule-of-three-and-rule-of-five/) applies. If you implement one special function, you have to implement all of them. The rest is not generated.

In fact, what we have seen is rather dangerous. You think, you have a move and you're efficient due to having a const member as using move semantics, but in fact, you are using the old copy assignment.

Yet, performance-wise, it seems hard to make a verdict. I ran a couple of tests in [QuickBench](https://quick-bench.com/q/58tnSzx0Hjm6t3KyIX9OcSHe9WE) and there is no significant difference between the above version and the one with non-const member and generated special assignment operator. On low optimization levels (None-O1) it depends on the compiler and its version. With higher optimization levels set there seems to be no difference.

## Conclusions

Having const local variables is good. Having const members... It's not so obvious. We lose the copy assignment and the move semantics as const members cannot be changed anymore.

With "clever" code, we can run a circle around the problem, but then we have to implement all the special functions. For what?

No performance gain. Less readability in the special functions and for slightly higher confidence that nobody will change the value of that member.

Do you think it's worth it?

Stay tuned, next time we'll discuss `const` return types.