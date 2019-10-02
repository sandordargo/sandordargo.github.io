---
layout: post
title: "Lambda Expressions in C++"
date: 2018-12-19
category: dev
tags: [cpp, tutorial, cpp11, stl, cleancode]
excerpt_separator: <!--more-->
---
Reading through [Scott Meyer](https://www.aristeia.com/)'s [Efective Modern C++](https://amzn.to/2EiL37U) helped me discover a lot of features of modern C++, including [right value references](http://sandordargo.com/blog/2018/11/25/override-r-and-l0-values), the [trailing return type declaration](http://sandordargo.com/blog/2018/11/07/trailing-return-type) and lambda expressions. Let's talk about those lambdas in this post.
<!--more-->

You might think, come on, this is old stuff, every serious developer should know about lambda expressions. You might be right, yet, it's not the case. Recently I made a brown bag session on lambdas and out of about 15 developers, two of us have already used lambdas in C++ and two others in Java. So the need is out there.

## What are lambda expressions?

Lambda expressions are anonymous functions. They are small snippets of code that provide a better readability in most cases if they are not hidden into an enclosing class. By the way, in C++, those enclosing classes would be called functors or function objects. We are going to cover them in a minute.

So we can say, that lambda expressions are here for us to replace functors and to make the code more expressive. Through their ease of usage and extreme expressivity, they boost the usage of the Standard Template Library.

At this point, I have to make a confession. I used to be very bad at C++. I knew the basic syntax and of course, I kept improving the readability of my code, but my knowledge was very poor on the STL, the standard library, on everything that is beyond the basic syntax. [When I was looking for a new team](http://sandordargo.com/blog/2018/01/03/new-year-new-start), moving to a pure/mostly C++ team was a compromise to me. I preferred Java and python much more. Probably because I moved around their ecosystems' more comfortably.

In my new team even though I worked some weeks in Java parts too, I ended up on C++ projects and I made up my mind. I decided to learn C++ better, at least to an advanced-medium level this year. This journey helped me a lot to <strike>fall in love with C++</strike> like it better than before. Lambdas are one important part of this new relationship.

Enough is enough. Let's go back to our topic.

## What do lambdas replace? Fu...

Functors, that's right. Functors, or by their maiden name, function objects are instances of classes where the `operator()` is overridden. So you can call them like this:

```
FunctorClass aFunctor;
aFunctor();
```

Or if it takes a parameter:

```
FunctorClass aFunctor;
aFunctor(42);
```

Defining them is pretty easy. They are normal classes, they just override `operator()`.

Let's sketch up quickly a functor that will decide if a given number is between 0 and 10.

```
class IsBetweenZeroAndTen {
  public:
  bool operator()(int value) {
    return 0 < value && value < 10;
  }
};
```

Fairly easy, but sometimes you really don't care about reusability and you don't want to find an _appropriate_ place for this function. You just want to define it once and on the fly. Lambdas, here they come!

## Syntax

Let's learn a bit about C++ lambda syntax. First, we are going to have a small overview then we go into details.

### Overview

```
[/* capture */] (/* parameters*/) { /* body */ }
```

It's that simple. So let's rewrite our functor as a lambda expression:

```
[](int value) {
  return 0 < value && value < 10;
}
```

As it's something very simple, just looking at the code, you can easily understand it without a name. You don't have to place a class somewhere, you just declare it on the fly. Yet, you might think that adding a name to it might help you increase code readability. That's fine, there are such cases, still, you don't need to write a class, you can save it in a variable:

```
auto isBetweenZeroAndTen = [](int value) {
  return 0 < value && value < 10;
}
```

Yes, it's that easy. Are you interested in its type? Try using `decltype` to get it.

Let's move on.

### Capture

Something that is really nice about C++ lambdas is that you can practice English. You have all types of brackets in it. You will have to deal with parentheses or round brackets (`()`), square or box brackets (`[]`) and braces or curly brackets (`{}`). Let's start with the square ones;

In the scope of lambda expressions, they are called a capture. So far you only saw them empty. What do they capture? They might capture variables that are not passed to the lambdas as a parameter and they are also not created inside.

Let's go back to our example of `isBetweenZeroAndTen`. Let's say we want to upper bound to vary.

```
auto upperBound = 42;
[](int value) {
  return 0 < value && value < upperBound; // doesn't compile, WTF is upperBound?
}
```

This will not compile, because in the scope of the lambda `upperBound` is unknown. It has to capture it. Let's see how!

#### Capture nothing

Well, when they are empty (`[]`), they capture nothing. That's stupid simple.

#### Capture by value

Write `[upperBound]` and our lambda will have the value of it.

```
auto upperBound = 42;
[upperBound](int value) {
  return 0 < value && value < upperBound;
}
```

#### Capture by reference

With the [well-known ampersand]() you can capture the variable by its reference, instead of the value.

```
auto upperBound = 42;
[&upperBound](int value) {
  return 0 < value && value < upperBound;
}
```
This implies - at least - two important things:
- The value of the captured variable can be modified even for the outside world
- You must make sure that the referenced variable still exists once the lambda is executed

#### Capture all by value

`[=]` will save "all" the variables needed in the body of the lambda by value. Sounds fun? Have you noticed that I wrote _all_ between double quotes? I did so because we have to understand what "_all_" variables mean. All means all the non-static local variables. So for example, if you reference a member variable in the lambda, even if you used it just next to the lambda declaration, it will not work.

```
m_upperBound = 42;
[=](int value) {
  return 0 < value && value < m_upperBound; // doesn't compile, m_upperBound is not a non-static local
}
```

How to fix this? There are two simple ways. One is that you make a local copy and capture that.

```
m_upperBound = 42;
auto upperBound = m_upperBound;
[=](int value) {
  return 0 < value && value < upperBound;
}
```

The other way is to pass in the whole surrounding object, `this`, we'll see it later.

#### Capture all by reference
`[&]` with this capture block, all the necessary and available variables will be captured by reference. Same notions apply here as for capturing all variables by value.

And don't forget. If a captured variable went out of scope since you captured it, you are in deep trouble.

#### Capture all by value, but
With using `[=, &divisor]` as a capture, everything will be captured by value except for the variable that is explicitly listed preceded with an `&`.

#### Capture all by reference, but
With using `[&, divisor]` as a capture, everything will be captured by value except for the variable that is explicitly listed.

#### Capture `this`
As we previously said, an only non-static local variable can be saved with the capture block. But as so frequently in life, there is a difference. You can also save the surrounding object like this: `[this]`. `this` is a pointer to the enclosing object, so if you capture `this`, you'll have access to the members for example:


```

[this](int value) {
  return 0 < value && value < this->m_upperBound;
}
```

But we shall not forget that `this` is a pointer.  If it ceases to exist between the time we capture it and the time our lambda is executed, we'll have to face undefined behaviour.

### The list of parameters

The list of parameters, as usual, come in between parentheses (`()`). Some remarks:
- In C++11 you cannot use `auto` as a type-specifier. But since C++14, you may.
- If there are no parameters passed to a lambda, the empty list can be omitted. Meaning that `[]{}` is a valid lambda expression. Though for readability reasons, it's better not to remove the empty parenthesis.

### The return type

Hmmm... There was no return type in our example so what does this section do here? And why after the list of parameters?

The return type of lambda expressions can be and most often is omitted when
- it is void
- or if it deducible (so if you could use `auto`)

As such, in practice most of the time the return type is omitted. In fact, in production code, I have never seen lambdas with an explicit return type.

If you do have to or want to declare them, you must use the [trailing return type syntax] meaning that you will declare the type between the parameter list and the body, putting the type after an arrow like this:

```
[](int value) -> bool {
  return 0 < value && value < 10;
}
```

### The body

It's just a normal body. As a best practice, it should be a quite lean one. If you need something longer, heavier, maybe a lambda is not the way you go.

As a reminder let's mention that you can work with the following variables:
- local variables declared in the body
- parameters passed into the lambda
- non-static local variable captured within the square brackets called a _"capture"_

Again, just to emphasize, if you go with the option of capturing references you must be sure that the referenced variable will be still alive when the lambda would be executed.

## Advantages

I already mentioned some of the advantages of using lambdas:
- no need for writing a full class
- no need to find an appropriate name for the class
- no need to find a good place for the class
- enhanced readability for simple use-cases.

And there is one more to mention. [Here](https://cppinsights.io/lnk?code=I2luY2x1ZGUgPGNzdGRpbz4KI2luY2x1ZGUgPHZlY3Rvcj4KI2luY2x1ZGUgPGFsZ29yaXRobT4KCmNsYXNzIGZ1bmN0b3J7CiAgcHVibGljOgogIGJvb2wgb3BlcmF0b3IoKShpbnQgdmFsKSB7CiAgICByZXR1cm4gMCA8IHZhbCAmJiB2YWwgPCAxMDsKICB9Cn07CgppbnQgbWFpbigpCnsKICAgIHN0ZDo6dmVjdG9yPGludD4gY29udGFpbmVyIHsyLDQsNiw4LCAyNX07CgogIAlzdGQ6OmZpbmRfaWYoY29udGFpbmVyLmJlZ2luKCksIGNvbnRhaW5lci5lbmQoKSwgZnVuY3RvcigpKTsKICAKICAgIHN0ZDo6ZmluZF9pZihjb250YWluZXIuYmVnaW4oKSwgY29udGFpbmVyLmVuZCgpLAogICAgICAgCQkgICAgICBbXShpbnQgdmFsKSB7IHJldHVybiAwIDwgdmFsICYmIHZhbCA8IDEwOyB9KTsKICAKfQ==&rev=1.0) you can check how much code will be generated for a functor. Default constructors, move constructor, copy constructor, destructor and nothing for a lambda apart from the operator overload. Oh, and there is one more. The compiler will not find out if you forgot to declare `operator()` overload as const. No problem for a lambda. 



## Some examples

Now that we understand the syntax of C++ lambda expressions, let see a couple of examples for their usage. I'll stick with the C++11 syntax, meaning that I won't use the `auto` keyword in the parameter list, and in the STL algorithms, I won't use [ranges](https://www.fluentcpp.com/2018/02/09/introduction-ranges-library/).

### Do the same thing on all elements of a list

Let's say we have a list of `Widget`s and you want to call their `resize()` method.

Non-lambda way:

```
auto widgets = std::vector<Widget> { … }; // a bunch of widgets
for (auto& widget : widgets) {
  widgets.resize();
}
```

Lambda way:


```
#include <algorithm>
// ...

auto widgets = std::vector<Widget> { … }; // a bunch of widgets

std::for_each(std::begin(widgets), std::end(widgets), 
  [](Widget& widget) {
  widgets.resize();
} );
```

In this case, it's debatable if you really want to use lambdas. The syntax is a bit more clunky, but it's generic for all std containers and you define the range you want to iterate over.

If we'd take the good old C++0x way, we can see even a readability advatage:

```
for(std::vector<Widget>::iterator it = widgets.begin(); it != widgets.end() ; ++it)
{
   widgets.resize();
}
```
Those iterators are just ugly to manage.

But with this example, we might already get the idea, that among the STL algorithms, lambdas will become handy.

### Get all the integers of a string

I know, I know, you could easily do this with a regular expression. But let's say you don't want to.

```
#include <string>
#include <algorithm>
#include <cctype>

auto another = std::string{};
std::copy_if(std::begin(input), std::end(input),
            std::back_inserter(another),
            [](char c) {
                return std::isdigit(c);
            }
);

```

The `copy_if` function will iterate over a range defined by the first two parameters. The third one defines where to copy the upcoming character if the condition defined by the last parameter is true.

In the last parameter, we defined a lambda expression. It gets a character as a parameter and returns `true` or `false` depending on whether the passed in character is a digit or not. Luckily in the standard library, there is a function to do, meaning that we don't have to try to cast it, nor to check its ASCII value.

### Write a function checking if a string is lowercase

Again this could be done with a regex, but it's more fun to do it with a lambda (or not...). If it's faster or not that should be measured.

```
#include <string>
#include <cctype>
#include <algorithm>

auto isLower(const std::string& phrase) -> bool {
    return std::all_of(std::begin(phrase), std::end(phrase), [](char c){return std::islower(c);});
}
```

`std::all_of` iterates over the range defined by the first two parameters and returns `true` if the lambda defined in the third parameter returns `true` for all the values. If there is at least one that evaluates to `false` the whole expression returns `false`. Again, luckily the `cctype` header has something helping us decide if a given character is lowercase.

### Use custom deleters for smart pointers

As a last example let's go to the shady world of pointers.

Probably we all heard that we should use smart pointers instead of new and all. If we have to deal with dynamic memory allocation and ownership it's better to choose an appropriate smart pointer either from boost or from the standard library depending on which version of C++ we are using.

When our shiny smart pointer reaches the end of its lifetime, the raw pointer it holds inside gets deleted. But what if it's not the only thing we want to do? 

What else we would want to do you might ask. Let's say we want to log. If you want to see more use cases, read [this article](https://www.bfilipek.com/2016/04/custom-deleters-for-c-smart-pointers.html).

In the case of some extra work needed, we have to define a deleter for the smart pointer and pass it as a parameter to the declaration. 

You can either define a deleter class, a functor, or as you might have guessed, you can just pass a lambda like this:

```
std::shared_ptr<Widget> pw1(new Widget, [](Widget *w){ ... });
```
The downside is that you cannot use `make_shared`, but that's another story and not the fault of lambdas.

## Conclusion

I hope you enjoyed this short journey to the - not so - new world of C++ lambdas. We covered not just why we should use lambdas, but we went into details regarding their syntax and saw a couple of examples.

If you only learned C++0x, you should keep in mind that C++ got a lot of features "recently" and it's getting more and more expressive just like lambdas show us.

Happy coding!