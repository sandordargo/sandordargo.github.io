---
layout: post
title: "Functions to be called only once in C++"
date: 2021-10-27
category: dev
tags: [cpp, functions, oop]
excerpt_separator: <!--more-->
---
In this article, we are going to discuss how we can make sure member functions are called no more than once while their enclosing object is alive.
<!--more-->

There can be different motivations for having such functions. Certain operations might be very costly, so we don't want to recompute the results several times, or maybe logically it doesn't make sense to call a function twice.

As we're going to see, the different motivations can lead to different solutions.

## Very costly operations

Imagine that a function has something very costly to perform. Maybe it has to retrieve something from the database, or from the network, maybe simply it's just very CPU intensive and we want to avoid doing that work twice.

### Cache the data

A simple and clean solution can be that when you call the costly function the first time, it saves the data in a private member. For later calls, the result is simply retrieved from that member instead of repeating the computation.

```cpp
class MyClass {
public:
  // ...
  CostlyResult getCostly() {
    if (m_result.empty()) {
      computeCostlyResult();
    }
    return m_result;
  }
private:
  CostlyResult m_result;
}

```

In this simple example, we default initialize `m_result` of type `CostlyResult` which has the means to check whether it already stores the outcome of the costly operations. For practical reasons, it's called `CostlyResult::empty` but there can be other ways to make such checks. You might even use a helper member to track if the function has been already called.

```cpp
class MyClass {
public:
  CostlyResult getCostly() {
    if (!m_result_initialized) {
      computeCostlyResult();
      m_result_initialized = true
    }
    return m_result;
  }
private:
  CostlyResult m_result;
  bool m_result_initialized{false};
};
```

The goal is clearly to avoid the computation being done twice. If the object lives long and the result might change, you might want to provide means to trigger a refresh of the data. As long as it doesn't happen automatically but the user of the class had to pass in a special flag or call a function, it's okay. The computation won't be triggered accidentally.

But what if you really want to restrict the number of calls and not just the computations?

### Have a counter

Instead of checking whether the `m_result` member was initialized, we can have a counter that counts how many times `getCostly` was called.

We can set a threshold and if there are more calls than that, we can raise an error like in the below example. Note that if the threshold is only one, meaning that the function can be called only once, instead of using a counter, we can fall back to a `bool` that is set after the first call - like in the previous example.

```cpp
#include <stdexcept>

class CostlyResult{};

class MyClass {
public:
  // ...
  CostlyResult getCostly() {
    if (number_of_costly_calls > 0) {
      throw std::runtime_error("MyClass::getCostly() can be called only once");
    }
    ++number_of_costly_calls;
    return computeCostlyResult();
  }
private:
  CostlyResult computeCostlyResult() {
    return {};
  }
  int number_of_costly_calls;
};

int main() {
  MyClass mc;
  mc.getCostly();
  mc.getCostly(); // ERROR THROWN
}
```
In this example, you can also see that we called `getCostly()`, yet we didn't store the result. That's probably a mistake and a waste of resources. Since C++17 we shall use `[[nodiscard]]` to have a compile-time warning in such situations and change `getCostly` as such:

```cpp
[[nodiscard]] CostlyResult getCostly();
```

Now let's jump to our other main motivation to avoid multiple calls to the same functions.

## Multiple calls are illogical

What can we do if it logically doesn't make sense to call a function more than once?

For sure, caching is not needed, we want to completely avoid multiple calls.

Then we have to ask ourselves a question. Will the call to the constrained function be the very last call on the object?

If no...

### Implement a flag

If the given function call is not the last one on the object, we can take the idea of the counter from the previous sections and implement it strictly with a flag, and of course with the `[[nodiscard]]` attribute in case it returns something.

Let's also have a runtime error in case we go against the rule we set:

```cpp
#include <stdexcept>

class CostlyResult{};

class MyClass {
public:
  // ...
  [[nodiscard]] CostlyResult getCostly() {
    if (getCostly_already_called) {
      throw std::runtime_error("MyClass::getCostly() can be called only once");
    }
    getCostly_already_called = true;
    return computeCostlyResult();
  }
private:
  CostlyResult computeCostlyResult() {
    ;
  }
  bool getCostly_already_called{false};
};

int main() {
  MyClass mc;
  auto r = mc.getCostly();
  //r = mc.getCostly();
}
```

### Destructive separation: move away and call

This solution is borrowed by [Matt Godbolt and his talk at C++ On Sea 2020](https://youtu.be/nLSm3Haxz0I?t=2523).

We can go this way if the function call should be the last one on the object. After this call, our object won't - necessarily - be in a usable shape.

The first thing to do is to add a `[[nodiscard]]` attribute if it has any return type so that people don't accidentally forget to save the results in a variable.

The other step is something more interesting and at the first sight even esoteric.

We have to add the `&&` qualifier to the function declaration - something I wrote about [here](https://www.sandordargo.com/blog/2018/11/25/override-r-and-l0-values#use--or--for-function-overloading).

This means that the function can only be called if the object:
- is temporary
- is about to fall out of scope
- has been moved from

In other words, the object is gone after the call.

Let's have a look at an example:

```cpp
#include <iostream>

class CostlyResult{};

class MyClass {
public:
  // ...
  [[nodiscard]] CostlyResult getCostly() && {
    return {};
  }
private:
};

int main() {
  MyClass mc;
  auto r = mc.getCostly();
}
```

The compiler says now that we are ignoring the `&&` qualifier. We even got a compile-time check so that it should be called only once!

```language
main.cpp: In function 'int main()':
main.cpp:16:24: error: passing 'MyClass' as 'this' argument discards qualifiers [-fpermissive]
   16 |   auto r = mc.getCostly();
      |            ~~~~~~~~~~~~^~
main.cpp:8:30: note:   in call to 'CostlyResult MyClass::getCostly() &&'
    8 |   [[nodiscard]] CostlyResult getCostly() && {
      |                              ^~~~~~~~~

```

Not so fast. The easiest way to get rid of the error message is to move away from `mc`:

```cpp
auto r = std::move(mc).getCostly();
```

We can do the same thing again!

```cpp
auto r = std::move(mc).getCostly();
auto r2 = std::move(mc).getCostly();
```

Of course, you should not do this, but it's possible and the compiler will not shout. At least, when you see the first line, the `std::move` should ring a bell that you shouldn't use that object anymore. But nothing prevents you.

A better way would be to wrap the call in a function and have the call at the last statement:

```cpp
CostlyResult getThatCostly() {
  MyClass mc;
  return std::move(mc).getCostly();
}
```

Note that in the video, std::move is not used in this case, (but with the compiler I use,) it doesn't work without the `move`. Anyway, it's the last line, so for sure, you're not going to reuse the object.

## Conclusion

In this article, we've seen different solutions to prevent functions to be called more than once, or at least to trigger their computations more than once.

Depending on the motivations, there are different solutions, such as caching, throwing exceptions or using function overloads.

Do you have other solutions in mind?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!