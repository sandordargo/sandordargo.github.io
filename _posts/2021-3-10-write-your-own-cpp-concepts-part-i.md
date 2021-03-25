---
layout: post
title: "How to write your own C++ concepts? Part I."
date: 2021-3-10
category: dev
tags: [cpp, concepts]
excerpt_separator: <!--more-->
---
During the previous weeks, we discussed [the motivations behind C++ concepts](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations) and [how to use them with functions](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them) and [with classes](https://www.sandordargo.com/blog/2021/02/24/cpp-concepts-with-classes). But we have hardly written any. We defined a functionally incomplete concept called `Number` for the sake of example, but that's it. Now are going into details on what kind of constraints we can express in a concept.
<!--more-->

This article would be too long if I included the different kinds of constraints all at once. In this one, we are going start from the simples concepts combining existing ones then we are going to finish with required operations and in general requirements on a class' API.

Next week, I'll show you how to write requirements on return types, how to express type requirements and how to nest constraints.

It's high time to finally get started.

## The simples `concept`

Let's define the simplest concept we can imagine first, just to see the syntax.

```cpp
template<typename T> 
concept Any = true;
```

First, we list the template parameters, in this case, we have only one, `T`, but we could have multiple ones separated by commas. Then after the keyword `concept,` we declare the name of the concept and then after the `=` we define the concept.

In this example, we simply say `true`, meaning that for any type `T` the concept will be evaluated to `true`; any type is accepted. Should we wrote `false`, nothing would be accepted.

Now that we saw the simplest concept, let's check what building blocks are at our disposal to construct a more detailed concept.

## Use already defined concepts

Arguably the easiest way to define new concepts is by combining existing ones.

For instance, in the next example, we are going to create - once again - a concept called `Number` by accepting both integers and floating-point numbers.

```cpp
#include <concepts>

template<typename T> 
concept Number = std::integral<T> || std::floating_point<T>;
``` 

As you can see in the above example, we could easily combine with the `||` operator two concepts. Of course, we can use any logical operator.

Probably it's self-evident, but we can use user-defined concepts as well.

```cpp
#include <concepts>

template<typename T> 
concept Integer = std::integral<T>;

template<typename T> 
concept Float = std::floating_point<T>;

template<typename T> 
concept Number = Integer<T> || Float<T>;
```

In this example, we basically just aliased (and added a layer of indirection to) `std::integral` and `std::floating_point` to show that user-defined concepts can also be used in a combination of concepts.

[As we saw earlier](https://www.sandordargo.com/blog/2021/03/03/cpp-concepts-in-standard-library), there are plenty of concepts defined in the different headers of the standard library so there is an endless way to combine them.

But how to define truly unique concepts?

## Write your own constraints

In the coming sections, we are going to delve into how to express our own unique requirements without using any of the predefined concepts. 

### Requirements on operations

We can simply express that we require that a template parameter support a certain operation or operator by *wishful writing*.

If you require that template parameters are addable you can create a concept for that:

```cpp
#include <iostream>
#include <concepts>

template <typename T>
concept Addable = requires (T a, T b) {
  a + b; 
};

auto add(Addable auto x, Addable auto y) {
  return x + y;
}

struct WrappedInt {
  int m_int;
};

int main () {
  std::cout << add(4, 5) << '\n';
  std::cout << add(true, true) << '\n';
  // std::cout << add(WrappedInt{4}, WrappedInt{5}) << '\n'; // error: use of function 'auto add(auto:11, auto:12) [with auto:11 = WrappedInt; auto:12 = WrappedInt]' with unsatisfied constraints
}
/*
9
2 
*/
```
We can observe that when `add()` is called with parameters of type `WrappedInt` - as they do not support `operator+` - the compilation fails with a rather descriptive error message (not the whole error message is copied over into the above example).

Writing the `Addable` concept seems rather easy, right? After the `requires` keyword we basically wrote down what kind of syntax we expect to compile and run.

### Simple requirements on the interface

Let's think about operations for a little longer. What does it mean after all to require the support of a `+` operation?

It means that we constrain the accepted types to those having a function `T T::operator+(const T& other) const` function. Or it can even be `T T::operator+(const U& other) const`, as maybe we want to add to an instance of another type, but that's not the point here. My point is that we made a requirement on having a specific function.

So we should be able to define a requirement on any function call, shouldn't we?

Right, let's see how to do it.

```cpp
#include <iostream>
#include <string>
#include <concepts>

template <typename T> // 2
concept HasSquare = requires (T t) {
    t.square();
};

class IntWithoutSquare {
public:
  IntWithoutSquare(int num) : m_num(num) {}
private:
  int m_num;
};

class IntWithSquare {
public:
  IntWithSquare(int num) : m_num(num) {}
  int square() {
    return m_num * m_num;
  }
private:
  int m_num;
};


void printSquare(HasSquare auto number) { // 1
  std::cout << number.square() << '\n';
}

int main() {
  printSquare(IntWithoutSquare{4}); // error: use of function 'void printSquare(auto:11) [with auto:11 = IntWithoutSquare]' with unsatisfied constraints, 
                                    // the required expression 't.square()' is invalid
  printSquare(IntWithSquare{5});
}
```

In this example, we have a function `printSquare` (1) that requires a parameter satisfying the concept `HasSquare` (2). In that concept, we can see that it's really easy to define what interface we expect. After the `requires` keyword, we have to write down how what calls should be supported by the interface of the accepted types.

Our expectations are written after the `requires` keyword. First, there is a parameter list between parentheses - like for a function - where we have to list all the template parameters that would be constrained and any other parameters that might appear in the constraints. More on that later.

If we expect that any passed in type have a function called `square`, we simply have to write `(T t) {t.square();}`. `(T t)` because we want to define a constraint on an instance of `T` template type and `t.square()` because we expect that `t` instance of type `T` must have a public function `square()`.

If we have requirements on the validity of multiple function calls, we just have to list all of them separated by a semicolon like if we called them one after the other:

```cpp
template <typename T>
concept HasSquare = requires (T t) {
  t.square();
  t.sqrt();
};
```

What about parameters? Let's define a `power` function that takes an `int` parameter for the exponent:

```cpp
template <typename T>
concept HasPower = requires (T t, int exponent) {
    t.power(exponent);
};

// ...

void printPower(HasPower auto number) {
  std::cout << number.power(3) << '\n';
}
```

The `exponent` variable that we pass to the `T::power` function has to be listed after the `requires` keyword with its type, along with the template type(s) we constrain. As such, we fix that the parameter will be something that is (convertible to) an `int`.

But what if we wanted to accept just any integral number as an exponent. Where is a will, there is a way! Well, it's not always true when it comes to syntactical questions, but we got lucky in this case.

First, our concept `HasPower` should take two parameters. One for the base type and one for the exponent type.

```cpp
template <typename Base, typename Exponent>
concept HasPower = std::integral<Exponent> && requires (Base base, Exponent exponent) { 
    base.power(exponent);
};
```

We make sure that template type `Exponent` is an integral and that it can be passed to `Base::power()` as a parameter.

The next step is to update our `printPower` function. The concept `HasPower` has changed, now it takes two types, we have to make some changes accordingly:

```cpp
template<typename Exponent>
void printPower(HasPower<Exponent> auto number, Exponent exponent) {
  std::cout << number.power(exponent) << '\n';
}
```

As `Exponent` is explicitly listed as a template type parameter, there is no need for the `auto` keyword after it. On the other hand, `auto` is needed after `HasPower`, otherwise, how would we know that it's a concept and not a specific type?! As `Exponent` is passed as a template type parameter to `HasPower` constraints are applied to it too.

Now `printPower` can be called the following way - given that we renamed `IntWithSquare` to `IntWithPower` following our API changes:

```cpp
printPower(IntWithPower{5}, 3);
printPower(IntWithPower{5}, 4L);
```

At the same time, the call `printPower(IntWithPower{5}, 3.0);` will fail because the type `float` does not satisfy the constraint on integrality.

Do we miss something? Yes! We can't use `IntWithPower` as an exponent. We want to be able to call `Base::power(Exponent exp)` with a custom type, like `IntWithPower` and for that, we need two things:

- `IntWithPower` should be considered an `integral` type
- `IntWithPower` should be convertible to something accepted by `pow` from the `cmath` header.

Let's go one by one.

By explicitly specifying the `type_trait` `std::is_integral` for `IntWithPower`, we can make `IntWithPower` an integral type. Of course, if we plan to do so in real life, it's better to make sure that our type has all the characteristics of an integral type, but that's beyond our scope here. (*Update: specializing most of the type traits results in Undefined Behaviour, so don't do this in production code*)

```cpp
template<>
struct std::is_integral<IntWithPower> : public std::integral_constant<bool, true> {};
```

Now we have to make sure that `IntWithPower` is convertible into a type that is accepted by [`pow`](http://www.cplusplus.com/reference/cmath/pow/). It accepts floating-point types, but when it comes to `IntWithPower`, in my opinion, it's more meaningful to convert it to an `int` and let the compiler perform the implicit conversion to `float` - even though it's better to avoid implicit conversions in general. But after all, `IntWithPower` might be used in other contexts as well - as an integer.

For that we have to define `operator int`:
```cpp
class IntWithPower {
public:
  IntWithPower(int num) : m_num(num) {}
  int power(IntWithPower exp) {
    return pow(m_num, exp);
  }
  operator int() const {return m_num;}
private:
  int m_num;
}
```

If we check our example now, we'll see that both `printPower(IntWithPower{5}, IntWithPower{4});` and `printPower(IntWithPower{5}, 4L);` will compile, but `printPower(IntWithPower{5}, 3.0);` will fail because `3.0` is not integral.

Right, as we just stated, [`pow`](http://www.cplusplus.com/reference/cmath/pow/) operates on floating-point numbers but we only accept integrals. Let's update our concept accordingly!

```cpp
template <typename Base, typename Exponent>
concept HasPower = (std::integral<Exponent> || std::floating_point<Exponent>) && requires (Base base, Exponent exponent) { 
    base.power(exponent);
};
```

Now we can call `printPower` with any type for `base` that satisfies the `HasPower` concept and both with integral and floating-point numbers as an exponent.

Let's have a look at the full example now:

```cpp
#include <cmath>
#include <iostream>
#include <string>
#include <concepts>
#include <type_traits>

template <typename Base, typename Exponent>
concept HasPower = (std::integral<Exponent> || std::floating_point<Exponent>) && requires (Base base, Exponent exponent) { 
    base.power(exponent);
};

class IntWithPower {
public:
  IntWithPower(int num) : m_num(num) {}
  int power(IntWithPower exp) {
    return pow(m_num, exp);
  }
  operator int() const {return m_num;}
private:
  int m_num;
};

template<>
struct std::is_integral<IntWithPower> : public std::integral_constant<bool, true> {};

template<typename Exponent> 
void printPower(HasPower<Exponent> auto number, Exponent exponent) {
  std::cout << number.power(exponent) << '\n';
}


int main() {
  printPower(IntWithPower{5}, IntWithPower{4});
  printPower(IntWithPower{5}, 4L);
  printPower(IntWithPower{5}, 3.0);
}
```

In this example, we can observe how to write a concept that expects the presence of a certain function that can accept a parameter of different constrained types. We can also see how to make a type satisfying built-in type traits, such as `std::is_integral`.

## Conclusion

Today we started to discover how to write our own concepts. First, we combined already existing concepts into more complex ones, then we continued with making requirements on the validity of operations on the constrained types then we finished by writing requirements for any function call with or without a parameter list.

[Next time we'll continue with constraining the return types, making type and then nested requirements.](https://www.sandordargo.com/blog/2021/03/17/write-your-own-cpp-concepts-part-ii)

Stay tuned!
