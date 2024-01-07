---
layout: post
title: "DR20 - The Equality Operator You Are Looking For"
date: 2024-1-3
category: dev
tags: [cpp, cpp20, dr, equality]
excerpt_separator: <!--more-->
---
When I see *DR*, I immediately think about Disaster Recovery. That's due to my first corporate job where I worked as a Database Administrator and we had regular exercises to simulate events when datacenters would be unavailable.

When you see DR in the title of a C++ proposal, it's not about a disaster, it's more about a bug. *DR* stands for defect report. But the paper itself is usually not about reporting that there is a problem - that has been already done -, but more about proposing a solution.

What's more important is that defect reports are not becoming part of the latest standard, they retrospectively change the one that introduced the defect.

## A defect of C++20's equality operator

[P2468R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2468r2.html) is addressing a problem that was introduced by C++20. C++20 brought us the spaceship operator (`operator<=>`), but it further changed the logic of object comparisons. It changed the meaning of `==` and `!=` and how overload resolution applies to them.

C++20 introduced the concept of "rewrites" or "rewrite targets". What this means is that if you implement a conforming `operator==`, the compiler will make sure that there is also `operator!=` available. If you use `operator<=>`, it will be used for rewrites. With "rewrites", certain logical operators are implemented if certain others are available as they can be expressed with the help of the other.

For example, if you have an `operator==` that checks whether two members are equal, the compiler can rewrite it and provide `operator!=`. `operator!=` will not check whether the members differ, but it will negate the result of `operator==`.

As this is a new behaviour since C++20, you might run into some surprises, some unintended behaviour.

Problems can arise when you migrate to C++20 and your `operator==` and `operator!=` are not matching. It might happen that the rewritten form of that operator is a better match and you have ambiguity errors or a silent change in the behaviour when migrating. How can those operators not match, you might ask? It can be intentional, but more probably it will be about a missing `const` qualifier on one of the operators.

Take this example on [C++ Insights](https://cppinsights.io/lnk?code=I2luY2x1ZGUgPGlvc3RyZWFtPgoKY2xhc3MgTXlDbGFzcyB7CnB1YmxpYzoKICAgIGludCB2YWx1ZTsKCiAgICBib29sIG9wZXJhdG9yPT0oY29uc3QgTXlDbGFzcyYgb3RoZXIpIGNvbnN0IHsKICAgICAgICByZXR1cm4gdmFsdWUgPT0gb3RoZXIudmFsdWU7CiAgICB9CiAgCiAgCWJvb2wgb3BlcmF0b3IhPShjb25zdCBNeUNsYXNzJiBvdGhlcikgY29uc3QgewogICAgICAgIHJldHVybiB2YWx1ZSAhPSBvdGhlci52YWx1ZTsKICAgIH0KfTsKCmludCBtYWluKCkgewogICAgTXlDbGFzcyBhezQyfTsKICAgIE15Q2xhc3MgYns0Mn07CiAgICAKICAgIGlmIChhID09IGIpIHsKICAgICAgICBzdGQ6OmNvdXQgPDwgImEgaXMgZXF1YWwgdG8gYlxuIjsKICAgIH0KCiAgICBpZiAoYSAhPSBiKSB7CiAgICAgICAgc3RkOjpjb3V0IDw8ICJhIGlzIG5vdCBlcXVhbCB0byBiXG4iOwogICAgfQoKICAgIHJldHVybiAwOwp9&insightsOptions=cpp20&std=cpp20&rev=1.0), play with where you put the `const` and observe how the generated code changes from `if(a.operator!=(b))` to `if(!a.operator==(b))`.

```cpp
#include <iostream>

class MyClass {
public:
    int value;

    bool operator==(const MyClass& other) const {
        return value == other.value;
    }
  
    bool operator!=(const MyClass& other) const {
        return value != other.value;
    }
};

int main() {
    MyClass a{42};
    MyClass b{42};
    
    if (a == b) {
        std::cout << "a is equal to b\n";
    }

    if (a != b) {
        std::cout << "a is not equal to b\n";
    }

    return 0;
}
``` 

Another reason for a surprise can be that you introduce `operator==` before the compiler sees the matching `operator!=` declaration. ([C++ Insights](https://cppinsights.io/lnk?code=I2luY2x1ZGUgPGlvc3RyZWFtPgoKY2xhc3MgTXlDbGFzcyB7CnB1YmxpYzoKICAgIGludCB2YWx1ZTsKCiAgICBib29sIG9wZXJhdG9yPT0oY29uc3QgTXlDbGFzcyYgb3RoZXIpIGNvbnN0IHsKICAgICAgICBzdGQ6OmNvdXQgPDwgIlVzaW5nIG9wZXJhdG9yPT1cbiI7CiAgICAgICAgcmV0dXJuIHZhbHVlID09IG90aGVyLnZhbHVlOwogICAgfQp9OwoKLy8gVGhpcyBmdW5jdGlvbiBpcyBub3QgdmlzaWJsZSBiZWZvcmUgb3BlcmF0b3I9PSBpcyBjYWxsZWQKYm9vbCBvcGVyYXRvciE9KGNvbnN0IE15Q2xhc3MmIGEsIGNvbnN0IE15Q2xhc3MmIGIpIHsKICAgIHN0ZDo6Y291dCA8PCAiVXNpbmcgb3BlcmF0b3IhPVxuIjsKICAgIHJldHVybiAhKGEgPT0gYik7Cn0KCmludCBtYWluKCkgewogICAgTXlDbGFzcyBhezQyfTsKICAgIE15Q2xhc3MgYns0Mn07CgogICAgaWYgKGEgPT0gYikgewogICAgICAgIHN0ZDo6Y291dCA8PCAiYSBpcyBlcXVhbCB0byBiXG4iOwogICAgfQoKICAgIGlmIChhICE9IGIpIHsKICAgICAgICBzdGQ6OmNvdXQgPDwgImEgaXMgbm90IGVxdWFsIHRvIGJcbiI7CiAgICB9CgogICAgcmV0dXJuIDA7Cn0=&insightsOptions=cpp20&std=cpp20&rev=1.0))

```cpp
#include <iostream>

class MyClass {
public:
    int value;

    bool operator==(const MyClass& other) const {
        std::cout << "Using operator==\n";
        return value == other.value;
    }
};

// This function is not visible before operator== is called
bool operator!=(const MyClass& a, const MyClass& b) {
    std::cout << "Using operator!=\n";
    return !(a == b);
}

int main() {
    MyClass a{42};
    MyClass b{42};

    if (a == b) {
        std::cout << "a is equal to b\n";
    }

    if (a != b) {
        std::cout << "a is not equal to b\n";
    }

    return 0;
}
```

So in these cases, the compiler takes the `operator==` and if it cannot find the matching negation, it will create it by rewriting the `operator==`

How do the rewritten versions rank against the ones that are provided - just probably provided not in the right form?

As the economist would say, it depends. As the C++ developer would say, it depends on the compiler and the situation, but the answers can go from one range to another. You can check the [paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2468r2.html) for some concrete examples.

## The new clear programming model

What's more important for us is how to solve the situation.

The authors of the paper considered different solutions, implemented them and ran the new rules against at least 59 open-source projects - by the end of their experiments, the number almost doubled - and checked how many of them would break.

In the beginning, a third of the projects broke, but it was below 8% by the end.

Based on these experiments that you can follow in the details in [§1.3 of P2468R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2468r2.html#existing-code-impact), the proposed and accepted solution is the following.

__If you want the compiler to automatically reverse `operator==` and therefore generate `operator!=`, make sure that you write **only** an `operator==` that returns a `bool`. (Yes, it might return an `int`...)__

__If you don't want your `operator==` to be used for rewrites, make sure that you write a matching `operator!=`.__

No matter what, `operator<=>` will be used for rewrites, that's an essential part of the feature.

With all that considered, __if you're migrating from C++17 and you want to keep behaviour the same as it was, make sure that every `operator==` has a matching `operator!=`__. And once you think that you'd like to benefit from rewrites, remove the `operator!=` and the compiler will provide that for you.

## Conclusion

In this article, we very briefly reviewed what defect reports are in C++ and if a solution is proposed they become part of the standard version that introduced a defect. In this case, we saw how the equality operator's behaviour changed with C++20 and that different compilers went with a different approach. In the end, we saw how the situation is fixed, what rules laid down by the authors of [P2468R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2468r2.html). With this fix, it should be straightforward how to benefit form rewritten `operator==` and how to avoid it.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!