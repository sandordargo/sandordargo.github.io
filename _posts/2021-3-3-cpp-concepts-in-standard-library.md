---
layout: post
title: "Concepts shipped with the C++ standard library"
date: 2021-3-3
category: dev
tags: [cpp, concepts]
excerpt_separator: <!--more-->
---
Welcome back to the series about C++ concepts. In the previous episodes, we discussed what are the motivations behind concepts, and then how to use them. Today we are going to have an overview of what kind of concepts are shipped with the C++ standard library.
<!--more-->

C++20 hasn't only given us the ability to write powerful concepts, but it also comes with more than 50 concepts part of the standard library and shared across three different headers.

## Concepts in the `<concepts>` header

In the `<concepts>` header you will find the most generic ones expressing core language concepts, comparison concepts and object concepts.

We are not going to explore all of them here for obvious reasons, you can find [the full list here](https://en.cppreference.com/w/cpp/concepts). Let me just pick three concepts so that we can get the idea.

### `std::convertible_to` for conversions with fewer surprises

`std::convertible_to` helps you to express that you only accept types that are convertible to another type - a type that you specify. The conversion can be both explicit or implicit. For example, you can say that you only accept types that can be converted into a `bool`. As the first parameter, you pass the type you want a conversion to be valid `From` and as the second, the type you want to be able to convert `To`, in our case, `bool`.

```cpp
#include <concepts>
#include <iostream>
#include <string>

template <typename T>
void fun(T bar) requires std::convertible_to<T, bool> {
  std::cout << std::boolalpha << static_cast<bool>(bar) << '\n';
}

int main() {
 fun(5); // OK an int can be converted into a pointer
//  fun(std::string("Not OK")); // oid fun(T) requires  convertible_to<T, bool> [with T = std::__cxx11::basic_string<char>]' with unsatisfied constraints
}
```

### `std::totally_ordered` for defined comparisons

`std::totally_ordered` helps to accept types that specify all the 6 comparison operators (`==`,`!=`,`<`,`>`,`<=`,`>=`) and that the results are consistent with a [strict total order](https://en.wikipedia.org/wiki/Total_order#Strict_total_order) on T.

```cpp
#include <concepts>
#include <iostream>
#include <typeinfo> 

struct NonComparable {
  int a;
};

struct Comparable {
  auto operator<=>(const Comparable& rhs) const = default; 
  int a;
};


template <typename T>
void fun(T t) requires std::totally_ordered<T> {
  std::cout << typeid(t).name() << " can be ordered\n";
}

int main() {
  NonComparable nc{666};
//   fun(nc); // Not OK: error: use of function 'void fun(T) requires  totally_ordered<T> [with T = NonComparable]' with unsatisfied constraints
  Comparable c{42};
  fun(c);
}
```

In the above example, you can also observe how to easily use the `<=>` (a.k.a. spaceship) operator to generate all the comparison operators.

If you are looking for more information on the `<=>` operator, I highly recommend reading [this article from Modernes C++](https://www.modernescpp.com/index.php/c-20-more-details-to-the-spaceship-operator).

### `std::copyable` for copyable types

`std::copyable` helps you to ensure that only such types are accepted whose instances can be copied. `std::copyable` object must be copy constructible, assignable and movable.

```cpp
#include <concepts>
#include <iostream>
#include <typeinfo> 

class NonMovable {
public:
  NonMovable() = default;
  ~NonMovable() = default;

  NonMovable(const NonMovable&) = default;
  NonMovable& operator=(const NonMovable&) = default;
  
  NonMovable(NonMovable&&) = delete;
  NonMovable& operator=(NonMovable&&) = delete;
};

class NonCopyable {
public:
  NonCopyable() = default;
  ~NonCopyable() = default;

  NonCopyable(const NonCopyable&) = default;
  NonCopyable& operator=(const NonCopyable&) = default;
  
  NonCopyable(NonCopyable&&) = delete;
  NonCopyable& operator=(NonCopyable&&) = delete;
};

class Copyable {
public:
  Copyable() = default;
  ~Copyable() = default;

  Copyable(const Copyable&) = default;
  Copyable& operator=(const Copyable&) = default;

  Copyable(Copyable&&) = default;
  Copyable& operator=(Copyable&&) = default;
};

template <typename T>
void fun(T t) requires std::copyable<T> {
  std::cout << typeid(t).name() << " is copyable\n";
}

int main() {
  NonMovable nm;
//   fun(nm); // error: use of function 'void fun(T) requires  copyable<T> [with T = NonMovable]' with unsatisfied constraints
  NonCopyable nc;
//   fun(nc); // error: use of function 'void fun(T) requires  copyable<T> [with T = NonCopyable]' with unsatisfied constraints
  Copyable c;
  fun(c);
}
```

As you can see in the above example, class `NonMovable` doesn't satisfy the concept as its move assignment and move constructor are deleted.

For `NonCopiable`, it's a similar case, but while the move semantics are available, it lacks the copy assignment and the copy constructor.

Finally, `Copyable` class defaults all the 5 special member functions and as such, it satisfies the concept of `std::copyable`.

## Concepts in the `<iterator>` header

In the [`<iterator>`](https://en.cppreference.com/w/cpp/iterator#Algorithm_concepts_and_utilities) header, you'll mostly find concepts that will come in handy when you deal with [algorithms](https://www.sandordargo.com/blog/2019/01/30/stl-algos-intro). It makes sense if you think about it, as the functions of the `<algorithms>` header operate on the containers through iterators, not directly on the containers.

### `std::indirect_unary_predicate<F, I>`

There are concepts related to callables, e.g. you can specify that you accept only unary predicates. First, what is a predicate? A predicate is a callable that returns either a `bool` value or value that is convertible to a `bool`. A unary predicate is a predicate that takes one parameter as its input.

I know that the following example is not very realistic, it's only for demonstrational purposes.

```cpp
#include <iostream>
#include <iterator>
#include <vector>

template <typename F, typename I>
void foo(F fun, I iterator) requires std::indirect_unary_predicate<F, I> {
    std::cout << std::boolalpha << fun(*iterator) << '\n';
}

int main()
{
  auto biggerThan42 = [](int i){return i > 42;};
  std::vector numbers{15, 43, 66};
  for(auto it = numbers.begin(); it != numbers.end(); ++it) {
      foo(biggerThan42, it);
  }
}
```

In the above example `foo` takes a function and an iterator and the concept `std::indirect_unary_predicate` ensures that the passed-in function can take the value pointed by the iterator and return a `bool` instead.

### `std::indirectly_comparable`

In the [`<iterator>`](https://en.cppreference.com/w/cpp/iterator#Algorithm_concepts_and_utilities) header you'll not only find concepts related to callables but more generic ones as well. Such as whether two types are inderictly comparable. That sounds interesting, let's take a simple example:

```cpp
#include <iostream>
#include <iterator>
#include <string>
#include <vector>

template <typename Il, typename Ir, typename F>
void foo(Il leftIterator, Ir rightIterator, F function) requires std::indirectly_comparable<Il, Ir, F> {
    std::cout << std::boolalpha << function(*leftIterator, *rightIterator) << '\n';
}

int main()
{
  using namespace std::string_literals;
  
  auto binaryLambda = [](int i, int j){ return 42; };
  auto binaryLambda2 = [](int i, std::string j){return 666;};
  
  std::vector ints{15, 42, 66};
  std::vector floats{15.1, 42.3, 66.6};
  foo(ints.begin(), floats.begin(), binaryLambda);
//   foo(ints.begin(), floats.begin(), binaryLambda2); // error: use of function 'void foo(Il, Ir, F) requires  indirectly_comparable<Il, Ir, F, std::identity, std::identity> 
}
```

In this case, I've been left a bit puzzled by the [documentation](https://en.cppreference.com/w/cpp/iterator/indirectly_comparable):
- As a third template parameter it has `class R` which normally would refer to ranges. 
- But then according to its definition, it calls `std::indirect_binary_predicate` with `R` forwarded in the first position. 
- In [`std::indirect_binary_predicate`](https://en.cppreference.com/w/cpp/iterator/indirect_binary_predicate), in the first position, you accept a `class F` and F stands for a callable (often a function).

Why isn't `R` called `F`? Why binary predicates are not mentioned in the textual description?

Probably only because this is still the beginning of the concepts journey. I'm actually going to submit a change request on this item.

## Concepts in the `<ranges>` header

In the [`<ranges>`](https://en.cppreference.com/w/cpp/ranges#Range_concepts) header you'll find concepts describing requirements on different types of ranges.

Or simply that a parameter is a `range`. But you can assert for any kind of ranges, like `input_range`, `output_range`, `forward_range`, etc.

```cpp
#include <iostream>
#include <ranges>
#include <string>
#include <vector>
#include <typeinfo> 

template <typename R>
void foo(R range) requires std::ranges::borrowed_range<R> {
  std::cout << typeid(range).name() << " is a borrowed range\n";
}

int main()
{
  std::vector numbers{15, 43, 66};
  std::string_view stringView{"is this borrowed?"};
//   foo(numbers); // error: use of function 'void foo(R) requires  borrowed_range<R> [with R = std::vector<int, std::allocator<int> >]' with unsatisfied constraints
  foo(stringView);
}
```

The above example checks whether a type satisfies the concept of a `borrowed_range`. We can observe that a `std::string_view` does, while a `vector` doesn't.

If you are curious, having a borrowed range means that a function can take it by value and can return an iterator obtained from it without any dangers of dangling. For more details, [click here](https://en.cppreference.com/w/cpp/ranges/borrowed_range).

## Conclusion

Today we've seen a few examples of concepts shipped with the C++20 standard library. There are about 50 of them shared among 3 headers (`concepts`, `iterators`, `ranges`).

Next week, we are going to see how to implement our own concepts.