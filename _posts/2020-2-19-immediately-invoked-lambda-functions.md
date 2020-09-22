---
layout: post
title: "Immediately invoked lambda functions"
date: 2020-2-19
category: dev
tags: [cpp, lambda, iilf, performance]
excerpt_separator: <!--more-->
---
Helping is important in life. You help the weak to get stronger, you help the hungry to learn fishing, you help someone to achieve her goals. Why not help your compiler to perform some optimization? As always, help benefits both the one that helps and the one that received a hand.

A good way to help the C++ compiler is to declare whatever variable const that should not change. It requires little effort, right?
<!--more-->

In most cases, it's very easy. But you might run into situations where you are simply not sure what to do.

Let's start with a simple example.

```cpp
// Bad Idea
std::string someValue;
if (caseA) {
    return std::string{"Value A"};
} else {
    return std::string{"Value B"};
}
```

This is bad, because as such `someValue` is not const. Can we make it const? I'm sure if you are a little bit familiar with C++, you can figure out an easy way. You might use a ternary operator.

```cpp
const std::string someValue = caseA ? std::string{"Value A"} : std::string{"Value B"};
```

Easy peasy.

But what to do if there are 3 different possibilities or even more?

```cpp
// Bad Idea
std::string someValue;
if (caseA) {
    return std::string{"Value A"};
} else if (caseB) {
    return std::string{"Value B"};
} else {
    return std::string{"Value C"};
}
```

A not so great idea is to nest ternaries. But it's so ugly that I don't even give you the example, but feel free to try it. I hope you will feel horrified.

Another option is to create a helper function.

```cpp
std::string makeSomeValue() const {
    if (caseA) {
        return std::string{"Value A"};
    } else if (caseB) {
        return std::string{"Value B"};
    } else {
        return std::string{"Value C"};
    }
}

const std::string someValue = makeSomeValue();
```

This is much better because of at least two reasons:
- `someValue` is const now!
- makeSomeValue is also const and given how simple it is, we can benefit copy-elision, return value optimization (TO BE DOUBLECHECKED)

If it's so good, is there any downside?

There are no ups without some downs. You might feel intimidating to find a good place for `makeSomeValue`. Where should it be? Should it be a private helper function? Maybe a static one? Or just a free function? Will it be coherent with the rest of the class?

These are difficult questions to answer and probably not even possible without knowing the exact context.

Since C++11, there is another option. You can use a lambda function that you don't even have to assign to a variable, you can invoke it immediately, hence it's called the immediately invoked lambda function.

```cpp
const std::string someValue = [caseA, caseB] () {
        if (caseA) {
            return std::string{"Value A"};
        } else if (caseB) {
            return std::string{"Value B"};
        } else {
            return std::string{"Value C"};
        }
    }();
```

Is this a magic bullet? Of course not. If the logic is something that you'd have to call any many places, you'd still better think about where to put that helper function. But if it's a one-timer, you have this option now and no problem.

Is it a viable option performance-wise?

First of all, the most important is to write readable and easy to maintain code. If the immediately invoked lambda happens to be your most readable option, go with it. Don't get into [immature optimizaiton]().

You might say that chasing const variables is already such an optimization. That's only half of the truth. Const correctness is not only about the possibility of compiler optimization, but it also helps to write and maintain correct business logic. If you declare something const you make sure that nobody will modify that just by accident. This combination of performance and safety is well worth the tiny bit of extra effort.

Honestly, in most cases, the safety would be worthwhile even if the performance would be worse. But is that the case?

Let's check on the [Compiler Explorer](https://godbolt.org/)!

Below you can find the links for each case compiled with `-O2` optimization flag which I chose deliberately:

* [original non-const version](https://godbolt.org/z/2ZtxMs)
* [const with helper function](https://godbolt.org/z/jg0gKR)
* [const with immediately invoked lambda](https://godbolt.org/z/YeYpRq)

I'm not a master of assembly code, but I can see at least that the const versions are shorter, so they should be also faster.

I made some measurements with [QuickBench](http://quick-bench.com/NnlmXwAOenovIUS4lbQhLJbtrk0), here is the code that you can copy-paste there and the differences were astonishing as you can see.

![Immediately invoked lambda O3]({{ site.baseurl }}/assets/img/immediate-lambdas-o3.png "Immediately invoked lambda O3")

Without optimization or with `-O1`, it's less important, but still significant.

![Immediately invoked lambda O1]({{ site.baseurl }}/assets/img/immediate-lambdas-o1.png "Immediately invoked lambda O1")

We can also see that whether you use a helper function or the immediately invoked lambda, it doesn't make a big difference. Choose based on whether you want to reuse the code or not.

## Conclusion

Today we learned about how we can make seemingly complex variable initializations `const` either with helper functions or with immediately invoked lambda functions. We discussed that enforcing `const`ness is not just an immature optimization, but it also helps to write code that allows fewer mistakes. After just as a curiosity, we checked the performance difference between non-const and const initializations and they are quite important! On the other hand, using lambdas doesn't bring a big performance benefit compared to a helper function, your choice should be based on whether you want to call the same logic more than once.

Next time when you declare a variable think twice if you can make it const. It's worth the price!

Happy const coding!
