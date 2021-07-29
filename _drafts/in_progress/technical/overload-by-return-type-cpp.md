---
layout: post
title: "Is it possible to overload by return type in C++"
date: 2021-6-x
category: dev
tags: [cpp, overload, template, variant]
excerpt_separator: <!--more-->
---
A friend of mine asked me if it's posible to return different types from a function depending on the value (and not the type!) of an input parameter.
<!--more-->

My gut asnwer was that it's not possible to overload a function based on its return type. But as this friend of mine is teaching C++ at a university, I was cautious. He must have something going on in his mind.

But the definition is crystal clear. The return type is not considered when you overload functions.

You can overload functions  based on:
- the type and number of parameters
- cv qualifiers

Is there anything else, I'm missing?

He told me he knows, but there must be a way around. Maybe with some generic function object involved, maybe with some templates and...

...and as usual where there is a will, there is a way. Even if it's not nice, or you might say it's not worth it.

## Using non-type template arguments

Let's make our first steps. In this next example, we have the same function parameter (none, in fact), but we also take a template parameter and specialize based on that:

```cpp
#include <iostream>
#include <string>
#include <type_traits> 

class A {
public:
  template <typename T>
  auto get() const;     
};

template<>
auto A::get<int>() const {
  return 42;
}

template<>
auto A::get<std::string>() const {
  return "FortyTwo";
}


int main() {
  A a;
  std::cout << a.get<int>() << '\n';
  std::cout << a.get<std::string>() << '\n';
}
```
We could also use concepts to achieve something similar, but it wouldn't let us evolve in the direction we want to go. 

Let's say it was a first step.

If we use non-type template arguments, we can basically provide specializations depending on values of the same type and we can even return different types:

```cpp
#include <iostream>
#include <string>
#include <type_traits> 

class A {
public:
  template <int>
  auto get() const;     
};

template<>
auto A::get<1>() const {
  return 42;
}

template<>
auto A::get<2>() const {
  return "FortyTwo";
}


int main() {
  A a;
  std::cout << a.get<1>() << '\n';
  std::cout << a.get<2>() << '\n';
}
/*
42
FortyTwo
*/
```
As you can see, by passing different integer values to the function template in this example, the differet specializations will return different types.

The caveat is that while this works, it's only useable with constant expressions, meaning that we need compile-time values.

In case, we want to somehow make this runtime friendly, we have to wrap the calls:

```cpp
auto aRunTimeValue = std::stoi(argv[1]);
A a;
switch (aRunTimeValue) {
  case 1: 
    std::cout << a.get<1>() << '\n';
    break;
  case 2:
    std::cout << a.get<2>() << '\n';
    break;
  default:
    std::cout << "value not supported\n";
    break;
}
```
So in order to benefit from the different return types, based on run-time values, we had to introduce a branching point to our code, which might or might not be acceptable to you. Nevertheless, having too many of such points reduces maintainability.

## Using `std::variant`

Following the above piece of code, you might think that there is a type in the standard library that might help us. You might think about `std::variant`.

That's right, since C++17 we have `std::variant` available and even before we had the boost version.

What is `std::variant` about?

Above we used a function template, to return different types, `std::variant` is a class template represting a type-safe union.

When you declare a variant you define which types it might hold and it will hold only one of them at the same time. In some special cases, it might hold nothing to show an error.

As you can see, by default it holds a zero-initialized value of the first altenative (given that it's default constructible/initializable):

```cpp
#include <iostream>
#include <string>
#include <variant>

int main() {
  std::variant<int, std::string> v;
  if (std::holds_alternative<int>(v)) {
    std::cout << "int\n";
  } else if (std::holds_alternative<std::string>(v)) {
    std::cout << "str\n";
  }
}
```
A variant can hold a type exactly once, but it can hold both a `const` and a non-`const` version of the same type. Meanwhile, it cannot hold references, or void.

If you want to get the value of a variant the simplest solution is to use the `std::get` either with the type or with the index.

Updating the above example it would look like this:

```cpp
#include <iostream>
#include <string>
#include <variant>

std::variant<int, std::string> foo(int bar) {
  if (bar % 2 == 0) {
    return 42;
  } else {
    return "FortyTwo";
  }  
}

int main() {
  auto v =  foo(66);
  if (std::holds_alternative<int>(v)) {
    std::cout << "int\n";
    std::cout << std::get<0>(v) << "\n";
    std::cout << std::get<int>(v) << "\n";
  } else if (std::holds_alternative<std::string>(v)) {
    std::cout << "str\n";
    std::cout << std::get<1>(v) << "\n";
    std::cout << std::get<std::string>(v) << "\n";
  }
}
```

The only problem with the above piece of code is that all these ways also require compile-time values.

In other words, you still need to intorduce branching points, if you want to handle the different values.

At least there is a more elegant way than a `switch` statement, you can use `std::visit`.

```cpp
#include <iostream>
#include <string>
#include <variant>

struct Visitor {
  void operator()(int& i) { std::cout << "int: " << i << '\n'; }
  void operator()(const std::string& s) { std::cout << "string: " << s << '\n'; }
};

std::variant<int, std::string> foo(int bar) {
  if (bar % 2 == 0) {
    return 42;
  } else {
    return "FortyTwo";
  }  
}

int main() {
  auto v = foo(66);
  std::visit(Visitor(), v);
  std::visit([](auto&& arg){ std::cout << arg << '\n';}, v);
}
```

`std::visit` takes a visitor which might be a _"visitor"_ object proposing an overloaded `operator()` for each possible type in the variant, but it can also take a lambda template or a set of lambdas with the [overload pattern](https://www.bfilipek.com/2018/09/visit-variants.html).

As second parameter, then you pass in the variant or even variants to visit.

There is [a very nice article on this](https://www.bfilipek.com/2018/09/visit-variants.html) by Bartlomiej Filipek, check it out for more details.

## Conclusion

Overloading a function simply by its return type is not possible, but there are different ways, to achieve something alike.

Without any standard library tools, you can rely on non-type template arguments. You can have the same name, same list of non-template parameters and different return types. Though in order to rely on runtime values to pick the right return type, you have to wrap the call, because the non-type template argument can only be a constant expression.

If you work with at least C++17, you can benefit from `std::variant` and `std::visit`. With `std::variant` you wouldn't overload the function anyhow, it would simply be able to return different types, types that you list in the variant declaration. 

In order to handle the different types, you can rely on compile-time values either in the form of indexes or types; or you can use a visitor that will take care of extracting and using the run-time value of the variant.

Do you see any other way? Use the comments section below to share your ideas!