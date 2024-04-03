---
layout: post
title: "Upgrading the compiler: undefined behaviour uncovered"
date: 2024-4-3
category: dev
tags: [cpp, fundamental, initialization, undefinedbehaviour]
excerpt_separator: <!--more-->
---
Not so long time ago, I already mentioned the differences between the different kinds of initializations in my article, [Struct Initialization](https://www.sandordargo.com/blog/2023/11/22/struct-initialization).

In the coming weeks, I'm going to revisit the topic. I don't think I'll say a lot of new things, but I have fresh inspiration to write about this.

So let's start with the story.

I recently upgraded the compiler I used to build a project from Clang 14 to Clang 17. It always feels better to compile with something newer, isn't it?

It all went fine, I applied the new compiler and I started to fix the compiler warnings one by one before merging my pull request when I found that a unit test broke. My first reaction was that it must be a flaky test. But it was not. My second reaction was that by fixing the compiler warnings, I must have changed something. But I did not. My third reaction was despair.

Once I recovered from it, I tried to reproduce the error on my local. You might ask why I didn't try to reproduce it before?! I changed so little when I fixed the warnings that it should have really stood out, that's why. Sadly, I couldn't reproduce it on my local. But I use a different compiler than the CI and it was working before on the CI as well, so I was not that much surprised.

I extracted the failing piece of code and simplified it as much as it made sense and I had a look at it on [Compiler Explorer](https://godbolt.org/z/38oj5j6df). This seemed like a good idea because I could immediately try it with different compilers and compiler versions. Needless to say, I couldn't reproduce the problem at all.

Smells like Undefined Behaviour?

Here is the piece of code:

```cpp
#include <cstdint>
#include <iostream>
#include <optional>

enum SomeOption {
  kSomeOptionUnknown = -1,
  kSomeOptionNone,
  kSomeOptionA,
  kSomeOptionB,

  kSomeOptionLast
};

using AnotherOption = uint32_t;
enum {
  kAnotherOptionFoo = 1 << 0,
  kAnotherOptionBar = 1 << 1,
  kAnotherOptionFooBar = 1 << 2,
  kAnotherOptionBaz = 1 << 3,
};

using SomeRules = uint32_t;
enum {
  kSomeRulesA = 1 << 0,
  kSomeRulesB = 1 << 1,
  kSomeRulesC = 1 << 2,
  kSomeRulesD = 1 << 3,
};
  
  struct State {
    SomeOption some_option;
    AnotherOption another_option;
    SomeRules rules;
    bool playful;
    bool isOn() const noexcept;
  };

  bool State::isOn() const noexcept {
  const bool forced = (rules & kSomeRulesA) == 0;
  return !(forced || (some_option == kSomeOptionNone) ||
           (some_option == kSomeOptionUnknown));
}

bool hasStateChanged(const std::optional<State> &old_state,
                     const State &new_state) {
  return old_state->some_option != new_state.some_option ||
                       old_state->isOn() != new_state.isOn() ||
                       ((old_state->another_option ^ new_state.another_option) &
                        kAnotherOptionBaz);
}

int main() {
  State old_state;
  old_state.some_option = kSomeOptionB;
  old_state.rules = kSomeRulesA;
  old_state.playful = false;

  State new_state = old_state;
  new_state.playful = true;

  if(!hasStateChanged(std::optional<State>{old_state}, new_state)) {
    std::cout << "test passed!\n";
  } else {
    std::cout << "test failed\n";
  }

}
```

I know that this example is a bit long, but it's already simplified quite a bit. I let you ponder over it a bit. I even put here a nice image of Castell de Sant Ferran (Figueres) so that you don't see the answer immediately.

![Castell de Sant Ferran (Figueres)]({{ site.baseurl }}/assets/img/light-tunnel.jpg)

So the problem is that `State::another_option` is left uninitialized therefore the behaviour is undefined when its value is read in `hasStateChanged()`. Right after I gave it an initial value, the unit test didn't fail anymore.

Obviously, that's not the long-term solution. We have to make sure that our code doesn't read uninitialized regions of the memory. The best way to do that is to make sure that no variable is left uninitialized. For struct or class members, the easiest way is to provide a default value either through the default constructor or by in-line class member initialization.

I would love to answer what exactly changed between Clang 14 and Clang 17 that provoked this breakdown of the test. But I have absolutely no idea about it.

On the other hand, it's worth noting two things:
- we can often observe that struct members have no default initialization (more often than class members)
- yet the code works as expected

What can be the reason behind - apart from undefined behaviour in our favour?

The answer is how fundamental variables are initialized.

Let's go through some basic rules.

- No matter if we talk about a local, namespace level or member variable, a class-type variable will be default-initialized.
- If a reference is not initialized, the program is ill-formed.
- For fundamental types, it's more nuanced.

Depending on how the enclosing object is initialized, fundamental members might be left in an indeterminate state and we have to face undefined behaviour if we read them, just like in the above case or they might be zero-initialized.

In the above example, given that the declaration `State old_state;` has no initializer sequence, *default-initialization* is performed and that leaves `State::another_option` in an indeterminate state.

If we add an initializer sequence (such as `{}`) to the mentioned declaration and we end up with `State old_state{};`, then we can talk about *value-initialization* which will *zero-initialize* `State::another_option` in the above struct.

I find it more and more important to understand these nuances of C++, at the same time, I also understand more and more why many people think it's too complex. I believe that a way to solve this - without moving to another language - is to write expressive and explicit code. 

If you want to say that a member's default value is zero, then just write it down and don't rely on initialization rules. At the same time, I know I would comment on code that initializes a `std::string` explicitly like `std::string s{""}`.

As a rule of thumb, I would suggest always explicitly initializing variables of fundamental types while not forgetting about enums. When it comes to class types and you want to invoke the default constructor, you can omit the explicit initialization. In my opinion, it's reasonable to expect people to know that their default constructor will be invoked, but it's unreasonable to expect people to know when fundamental types will be initialized and when they will be left with an undefined value. If you really want to stay consistent, just use the braces (`std::string s{};`).

## Conclusion

In this article, I showed you what kind of undefined behaviour was uncovered thanks to a recent compiler upgrade I performed. Then we discussed when started to discuss when we can expect a fundamental data type to be initialized and when not.

In the few articles, we are going to get deeper into that topic and we'll discuss the many different kinds of initialization that exist in C++. Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!