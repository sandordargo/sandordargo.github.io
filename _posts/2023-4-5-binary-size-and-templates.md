---
layout: post
title: "Binary sizes and passing functions to functions"
date: 2023-4-5
category: dev
tags: [cpp, binarysizes, functions, templates]
excerpt_separator: <!--more-->
---

So why templates are interesting for our series on binary sizes? It's because as mentioned, they do not represent callable code. They represent templates to generate callable code. The more you generate, the more different input sets you use with them, the bigger binary you get.

> *If you want to learn about templates more I'd advise you to read [Template Metaprogramming with C++ by Marius Bancila](https://www.sandordargo.com/blog/2022/10/28/template-metaprogramming-with-cpp-by-marius-bancila).*

I'm not telling you not to use templates. Not at all. They are a useful tool in your belt, if used properly. I simply want to contribute to that needed understanding, so that you don't shoot yourself in the leg. Now let's see what we can do to minimise the harm.

## Use minimal templates

In [Clean Code](http://amzn.to/2Fj14wo), we learned that we should write small functions and those small functions should build up small classes. What is small? It depends, and I don't want to get into a numbers game here. Then in [A Philosophy of Software Design](https://dev.to/sandordargo/deep-vs-shallow-modules-5fj3) John Ousterhout wrote that we should write deep methods and classes and not be afraid of having long entities. You might tend to one or the other opinion, but when it comes to templates, you should really consider having some classes.

Let's have a look at an example. As I'm learning nowadays about wine tasting and plan to get a certification in the next few months, I'm using wine bottling as an example.

```cpp
#include <vector>

class Wine {};
class Bottle {};
void storeTemporarily(Bottle bottle) {}
void moveToCellar(Bottle bottle) {}
void uploadToStore(Bottle bottle) {}

template <typename Callable>
void prepareWineForSell(Callable bottlerFunction, Wine wine) {
  auto bottles = bottlerFunction(wine);
  
  for (const auto& bottle: bottles) {
    storeTemporarily(bottle);
    moveToCellar(bottle);
    uploadToStore(bottle);
  }
}

int main() {
    auto bottleWithScrewlock = [](const Wine& wine) { return std::vector<Bottle>(10);};
    auto bottleWithCork = [](const Wine& wine) { return std::vector<Bottle>(10);};
    auto bottleWithSyntethicCork = [](const Wine& wine) { return std::vector<Bottle>(10);};
    auto bottleWithWirehood = [](const Wine& wine) { return std::vector<Bottle>(10);};

    Wine wine;
    prepareWineForSell(bottleWithScrewlock, wine);
    prepareWineForSell(bottleWithCork, wine);
    prepareWineForSell(bottleWithSyntethicCork, wine);
    prepareWineForSell(bottleWithWirehood, wine);   
}
```

In the above example, we can see that `prepareWineForSell` is a function template that takes a callable. It only uses it in the first line of the function body, no more. But if you instantiate this template with different callables, the compiler has to generate the code for the rest of the function that doesn't use the callable 4 different times.

This we can easily optimize, by extracting the part that doesn't depend on the callable into a separate function. As such, those lines don't have to be included in the function 4 different times but the function exists only once.

```cpp
#include <vector>

class Wine {};
class Bottle{};
void storeTemporarily(Bottle bottle) {}
void moveToCellar(Bottle bottle) {}
void uploadToStore(Bottle bottle) {}

void postProcessBottles(const std::vector<Bottle>& bottles) {
  for (const auto& bottle: bottles) {
    storeTemporarily(bottle);
    moveToCellar(bottle);
    uploadToStore(bottle);
  }
}

template <typename Callable>
void prepareWineForSell(Callable bottlerFunction, Wine wine) {
  auto bottles = bottlerFunction(wine);
  
  postProcessBottles(bottles);
}

int main() {
    auto bottleWithScrewlock = [](const Wine& wine) { return std::vector<Bottle>(10);};
    auto bottleWithCork = [](const Wine& wine) { return std::vector<Bottle>(10);};
    auto bottleWithSyntethicCork = [](const Wine& wine) { return std::vector<Bottle>(10);};
    auto bottleWithWirehood = [](const Wine& wine) { return std::vector<Bottle>(10);};

    Wine wine;
    prepareWineForSell(bottleWithScrewlock, wine);
    prepareWineForSell(bottleWithCork, wine);
    prepareWineForSell(bottleWithSyntethicCork, wine);
    prepareWineForSell(bottleWithWirehood, wine);   
}
```

If we check the generated assembly in compiler explorer, we can see that there is more than a 10% difference between the two versions and the one with the smaller template function is also the smaller at the end, just as expected.

In this example, we could go even further and make `prepareForSell` return `std::vector<Bottle>` and call `postProcessBottles` right with these results. Or we could probably even remove `postProcessBottles` completely. On the one hand, I wanted to showcase when and how to reduce your templates. But on the other hand, you should indeed have a look into going even further if you can. Not only to reduce your binary size but mainly to reduce the complexity of your code.

## Realize when you are using templates

My next point is a small, yet important one. Often when you ask inexperienced C++ programmers if they use templates, many will say no. But it's almost 100% sure that they do. The standard library is full of templates. There is a reason why people often mix up the notions of the standard library and the standard template library.

Even a `std::string` is a template! It's just an alias for `std::basic_string<char>` where `std::basic_string` is `template<class CharT, class Traits = std::char_traits<CharT>, class Allocator = std::allocator<CharT>> class basic_string;`. So keep in mind that instantiating templates has a cost. How much? It depends on your use case.

But there are two more things I want you not to forget. Whenever you see `auto` in the function parameter list, don't forget that you're dealing with a template with all its advantages and costs. For example `auto add(auto, int)` is a template.

And it's worth mentioning that `std::function` is also a template and a costly one as we saw in [The observer pattern and binary sizes
](https://www.sandordargo.com/blog/2023/03/15/binary-sizes-and-observer-pattern#in-comparison).

Let's discuss this one more time!

## `std::function` vs function pointers vs templates

If you have a function that takes another function (or function object or lambda) as a parameter, how should you do it? One option is to take it as a `std::function`.

```cpp
#include <iostream>
#include <functional>

int performOperation(std::function<int(int, int)> op, int lhs, int rhs) {
    return op(lhs, rhs);
}

int main() {
    auto add = [](int lhs, int rhs) {return lhs + rhs;};
    auto subtract = [](int lhs, int rhs) {return lhs - rhs;};
    auto multiply = [](int lhs, int rhs) {return lhs * rhs;};
    std::cout << performOperation(add, 42, 51) << '\n';
    std::cout << performOperation(subtract, 42, 51) << '\n';
    std::cout << performOperation(multiply, 42, 51) << '\n';
    return 0;
}
```

A `std::function` is a type-erased wrapper around any kind of callable. A function pointer will take a free function or something that can be implicitly converted to it. Such as a lambda without a capture. `std::function` has quite some overhead, but it provides flexibility.

If we want flexibility, we can also take a template. While it might be too wide for us and we want to avoid too late template instantiation error messages, since C++20 we can and should use concepts to constrain the accepted arguments.

```cpp
#include <iostream>
#include <concepts>

template <std::invocable<int, int> Callable>
int performOperation(Callable op, int lhs, int rhs) {
    return op(lhs, rhs);
}

int main() {
    auto add = [](int lhs, int rhs) {return lhs + rhs;};
    auto subtract = [](int lhs, int rhs) {return lhs - rhs;};
    auto multiply = [](int lhs, int rhs) {return lhs * rhs;};
    std::cout << performOperation(add, 42, 51) << '\n';
    std::cout << performOperation(subtract, 42, 51) << '\n';
    std::cout << performOperation(multiply, 42, 51) << '\n';
    return 0;
}
```

This version doesn't check the return type, so let's refine it a bit more. We should create a new concept that takes into consideration both the arguments and the return type and use only that concept.

```cpp
template <typename Callable>
concept RestrictedCallable = std::invocable<Callable, int, int> && requires (
    Callable callable, int lhs, int rhs) {
        { callable(lhs, rhs) } -> std::same_as<int>;
};

template <RestrictedCallable Callable>
int performOperation(Callable op, int lhs, int rhs) {
    return op(lhs, rhs);
}
```

This above version doesn't even allow implicit conversions on the returned type, if you wanted to allow that, you could use `std::convertible_to`. If you want to learn more about concepts, [check this series](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations).

Then there is the third option which is just taking a good old function pointer. If we don't need the feature of a capture list then it should be a good enough option.

```cpp
#include <iostream>

int performOperation(int(* op)(int, int), int lhs, int rhs) {
    return op(lhs, rhs);
}

int main() {
    auto add = [](int lhs, int rhs) {return lhs + rhs;};
    auto subtract = [](int lhs, int rhs) {return lhs - rhs;};
    auto multiply = [](int lhs, int rhs) {return lhs * rhs;};
    std::cout << performOperation(add, 42, 51) << '\n';
    std::cout << performOperation(subtract, 42, 51) << '\n';
    std::cout << performOperation(multiply, 42, 51) << '\n';
    return 0;
}
```

My biggest take against function pointers is the readability. I find it extremely inconvenient that the parameter name is buried somewhere in the function pointer signature. Let's use a `typedef` to improve the situation.

```cpp
using MyFunctionPtr = int(*)(int, int);

int performOperation(MyFunctionPtr op, int lhs, int rhs) {
    return op(lhs, rhs);
}
```

It's better like that.

Now let's compare how these three solutions compare against each other.


|   Version            | Binary size  | 
|----------------------|--------------|
|  std::function -O3| 40,189          |
| templates -O3     | 35,259         |  
|  function pointer -O3| 35,341          |


As we could expect, `std::function` resulted in a slightly bigger binary, the difference compared to the other two variants was above 10%. On the other hand, templates and function pointers had a similar result. 

We cannot yet infer a long-term conclusion. What we see is that when you have a minimal template and few instantiations, then templates and function pointers have similar results. But we can expect that the template solution will scale worse. Each additional type of callable will add its own instantiation, and in addition, the longer the template is, the more space each generated function will take.

You have to keep those in mind. But if your templates are small and the number of instantiations is limited, probably the template version offers better readability. In particular, if you can use C++20's concepts.

## Conclusion

In this article, we discussed about how templates might bloat the size of a binary. It's worth keeping in mind that each template instantiation adds to our binary. It's also important to remember that we use more templates than we'd often think to. Containers are templates, including strings! Don't forget, if you see `auto` in function parameter list, you see a function template!

Speaking about functions, let's not forget that `std::function` is also a template, one of the more costliers! If you want to pass a callable and amy kind of efficieny matter for you, think about other options such creating a constrained template or just using function pointers!

If the function template that has to be instantiated each time is big, then probably a function pointer will be a good enough option despite it's limited readability.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!