---
layout: post
title: "How to write your own C++ concepts?"
date: 2021-XX-XX
category: dev
tags: [cpp, concepts]
excerpt_separator: <!--more-->
---
During the previous weeks, we discussed the motivations behind C++ concepts and how to use them with functions and with classes. But we have hardly written any. We defined a functionally incomplete concept called `Number` for the sake of example, but that's it. Now are going into details on what kind of constraints we can express in a concept.
<!--more-->

Let's define the simplest concept we can imagine first, just to see the syntax.

```cpp
template<typename T> 
concept Any = true;
```

First, we list the template parameters, in this case, we have only one, `T`, but we could have multiple ones separated by commas. Then after the keyword `concept` we declare the name of the concept and then after the `=` we define the concept.

In this example, we simply say `true`, meaning that for any type `T` the concept will be evaluated to `true`; any type is accepted. Should we wrote `false`, nothing would be accepted.

Now that we saw the simplest concept, let's check what building blocks are at our disposal to construct a more detailed concept.

## Use already defined concepts

Arguably the easiest way to define new concepts is by combining existing ones.

For example, in the next example we are going to create - once again - a concept called `Number` by combining integers and floating point numbers.

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

As we saw earlier, there are plenty of concepts defined in the different headers of the standard library so there is an endless way to combine them.

But how to define truly unique concepts?

## Write your own constraints

In the coming sections, we are going to delve into how to express our own unique requirements without using any of the predefined concepts. 

### Requirements on operations

You can simply express that you require that a template parameter support a certain operation or operator by *wishful writing*.

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

struct A {
  int b;
};

int main () {
  std::cout << add(4, 5) << '\n';
  std::cout << add(true, true) << '\n';
  // std::cout << add(A{4}, A{5}) << '\n'; // error: use of function 'auto add(auto:11, auto:12) [with auto:11 = A; auto:12 = A]' with unsatisfied constraints
}
/*
9
2 
*/
```
We can observe that when `add()` is called with parameters of type `A` - as they do not support `operator+` - the compilation fails with a rather descriptive error message (not the whole error message is copied over into the above example).

Writing the `Addable` concept seems rather easy, right? After the `requires` keyword we basically wrote down what kind of syntax we expect to compile and run.

### Simple requirements on the interface

Let's think about operations for a little longer. What does it mean after all to require the support of a `+` operation?

It means that you constrain the accepted types to those having a function `T T::operator+(const T& other) const` function. Or it can even be `T T::operator+(const U& other) const`, as maybe you can add an instance of another type, but that's not the point here. My point is that we made a requirement on having a specific function.

So we should be able to define a requirement on any function call, right?

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

In this example, we have a function `printSquare` (1) that requires a parameter satisfying the concept `HasSquare` (2). In that concept we can see that it's really easy to define what interface we expect. After the `requires` keyword, we have to write down how what calls the interface of the accepted types should support.

Our expectations are written after the `requires` keyword. First, there is a parameter list between parantheses - like for a function - where we have list in all the template parameters that would be constrained and any other parameters that might appear in the contraints. More on that later.

If you expect that any passed in type have a function called `square`, you simply have to write `(T t) {t.square();}`. `(T t)` because you want to define a constraint on an instance of `T` template type and `t.square()` because you expect that `t` instance of type `T` has a public function `square()`.

If you have requirements on the validity of multiple function calls, you just have to list all of them separated by a semicolon like if you called them one after the other.

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

As `Exponent` is explicitely listed as a template type parameter, there is no need for the `auto` keyword after it. On the other hand, `auto` is needed after `HasPower`, otherwise how would we know that it's a concept and not a specific type?!

Now `printPower` can be called the following way - given that we renamed `IntWithSquare` to `IntWithPower` following our API changes:

```cpp
printPower<int>(IntWithPower{5}, 3);
printPower<long>(IntWithPower{5}, 4L);
```

At the same time, the call `printPower<float>(IntWithPower{5}, 3.0);` will fail because the type `float` does not satisfy the constraint on integrality.

Do you like what we have just written?

I don't. We have to pass type information to `printPower` about the exponent, and in addition we can't use `IntWithPower` as an exponent.

Let's attack the two problems one by one.

First we should remove the need of explicit type information. Concepts can help us. As an exponent we have to require a type satisfies the `std::integral` concept.

```cpp
template<typename Exponent> 
requires std::integral<Exponent>
void printPower(HasPower<Exponent> auto number, Exponent exponent) {
  std::cout << number.power(exponent) << '\n';
}
```

Now we can call `printPower` with any `integral` type without passing the type explicitely: `printPower(IntWithPower{5}, 3);`.

The other problem is bit more difficult to get rid off. We want to be able to call `Base::power()` with a custom type, like `IntWithPower` and for that we need two things:

- `IntWithPower` should be considered an `integral` type
- `IntWithPower` should be convertible to something accepted by `pow` from the `cmath` header.

Let's go one by one.

By explicitely specifying the `type_trait` `std::is_integral` for `IntWithPower`, we can make `IntWithPower` an integral type. Of course, if we plan to do so in real life, it's better to make sure that our type has all the characteristics of an integral type, but that's beyond our scope here.

```cpp
template<>
struct std::is_integral<IntWithPower> : public std::integral_constant<bool, true> {};
```

Now we have to make sure that `IntWithPower` is convertible into a type that is accepted by [`pow`](http://www.cplusplus.com/reference/cmath/pow/). It accepts floating point types, but when it comes to `IntWithPower`, in my opinion it's more meaningful to convert it to an `int` and let the compiler perform an implicit conversion to `float` - even though it's better to avoid implicit conversions in general. But after all, `IntWithPower` might be used in other contexts as well - as an integer.

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

If we check our example now, we'll see that both `printPower(IntWithPower{5}, IntWithPower{4});` and `printPower(IntWithPower{5}, 4L);` will compile, but `printPower(IntWithPower{5}, 3.0);` fails because `3.0` is not integral.

Right, as we just stated, [`pow`](http://www.cplusplus.com/reference/cmath/pow/) operates on floating-point numbers but we only accept integrals. Let's update our corresponding requirements!

```cpp
template <typename Base, typename Exponent>
concept HasPower = (std::integral<Exponent> || std::floating_point<Exponent>) && requires (Base base, Exponent exponent) { 
    base.power(exponent);
};

template<typename Exponent> 
requires (std::integral<Exponent> || std::floating_point<Exponent>)
void printPower(Number<Exponent> auto number, Exponent exponent) {
  std::cout << number.power(exponent) << '\n';
}
```

Now we can call `printPower` with any type for base that satisfies the `HasPower` concept and both with integral and floating-point numbers as an exponent.

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
requires (std::integral<Exponent> || std::floating_point<Exponent>)
void printPower(HasPower<Exponent> auto number, Exponent exponent) {
  std::cout << number.power(exponent) << '\n';
}


int main() {
  printPower(IntWithPower{5}, IntWithPower{4});
  printPower(IntWithPower{5}, 4L);
  printPower(IntWithPower{5}, 3.0);
}
```
### Requirements on return types (a.k.a compound requirements)

We've seen how to write a requirement expressing the need of a certain API, a certain function.

But did we also constrain the return type of those functions?

No, we didn't. `IntWithSquare` satisfies the `HasSquare` concept both with `int square()` and `void square()`.

If you want to specify the return type, you must use something that is called a compound requirement.

Here is an example:

```cpp
template <typename T>
concept HasSquare = requires (T t) {
    {t.square()} -> std::convertible_to<int>;
}; 
```

Notice the following:
- The expression on what you want to set a return type requirement must be surrounded by braces (`{}`), then comes an arrow (`->`) followed by a constraint of the return type.
- A constraint cannot simply be a type. Had you written `int`, you would have received an error message: *return-type-requirement is not a type-constraint.* The original concepts TS allowed the direct usage of types, so if you experimented with that, you might be surprised by this error. This possibility was removed by [P1452R2](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1452r2.html).

There are a number of reasons for this removal. One of the motivations was that it would interfere with a future direction of wanting to adopt a generalized form of `auto`, like `vector<auto>` or `vector<Concept>.`

So instead of simply naming a type you have to choose a concept! If you want to set the return type one of the two following options will satisfy your needs:

```cpp
{t.square()} -> std::same_as<int>;
{t.square()} -> std::convertible_to<int>;
```

I think that the difference is obvious. In case of `same_as`, the return value must be the same as specified as the template argument, while with `convertible_to` conversions are allowed.

In order to demonstrate this, let's have a look at the following example:

```cpp
#include <iostream>
#include <concepts>

template <typename T>
concept HasIntSquare = requires (T t) {
    {t.square()} -> std::same_as<int>;
};

template <typename T>
concept HasConvertibleToIntSquare = requires (T t) {
    {t.square()} -> std::convertible_to<int>;
};

class IntWithIntSquare {
public:
  IntWithIntSquare(int num) : m_num(num) {}
  int square() const {
    return m_num * m_num;
  }
private:
  int m_num;
};

class IntWithLongSquare {
public:
  IntWithLongSquare(int num) : m_num(num) {}
  long square() const {
    return m_num * m_num;
  }
private:
  int m_num;
};

class IntWithVoidSquare {
public:
  IntWithVoidSquare(int num) : m_num(num) {}
  void square() const {
    std::cout << m_num * m_num << '\n';
  }
private:
  int m_num;
};


void printSquareSame(HasIntSquare auto number) {
  std::cout << number.square() << '\n';
}

void printSquareConvertible(HasConvertibleToIntSquare auto number) {
  std::cout << number.square() << '\n';
}


int main() {
  printSquareSame(IntWithIntSquare{1}); // int same as int
//   printSquareSame(IntWithLongSquare{2}); // long not same as int
//   printSquareSame(IntWithVoidSquare{3}); // void not same as int
  printSquareConvertible(IntWithIntSquare{4}); // int convertible to int
  printSquareConvertible(IntWithLongSquare{5}); // int convertible to int
//   printSquareConvertible(IntWithVoidSquare{6}); // void not convertible to int
}
/*
1
16
25
*/
```

In the above example, we can observe that the class with `void square() const` doesn't satisfy either the `HasIntSquare` or the `HasConvertibleToIntSquare` concepts.

`IntWithLongSquare`, so the class with the function `long square() const` doesn't satisfy the concept `HasIntSquare` as long is not the same as `int`, but it does satisfy the `HasConvertibleToIntSquare` concept as `long` is convertible to `int`.

Class `IntWithIntSquare` satisfies both concepts as an `int` is obviously the same as `int` and it's also convertible to an `int`.

### Type requirements

With type requirements we can express that a certain type is valid in a specific context. Type requirements can be used to verify that

- a certain nested type exists
- a class template specializiation names a type
- an alias template specialization names a type

You have to use the keyword `typename` along with the type name that is expected to exist:

```cpp
template<typename T>
concept TypeRequirement = requires {
  typename T::value_type;
  typename Other<T>;
};
```
The concept `TypeRequirement` requires that the type `T` has a nested member value_type, and that the class template `Other` can be instantiated with `T`.

Let's see how it works:


```cpp
#include <iostream>
#include <vector>
template <typename>
struct Other;

template<typename T>
concept TypeRequirement = requires {
  typename T::value_type;
  typename Other<T>;
};

int main() {
  TypeRequirement auto myVec = std::vector<int>{1, 2, 3};
  // TypeRequirement auto myInt {3}; // error: deduced initializer does not satisfy placeholder constraints ... the required type 'typename T::value_type' is invalid 
}
```

The expression `TypeRequirement auto myVec = std::vector<int>{1, 2, 3}` (line 13) is valid.
A `std::vector` has an inner member `value_type` (requested on line 8) and the class template `Other` can be instantiated with `std::vector<int>` (line 9).

Let's change class template `Other` and make a requirement on the template parameter by making sure that `Other` cannot be instantiated with a `vector` of `int`s.

```cpp
template <typename T>
requires (!std::same_as<T, std::vector<int>>)
struct Other
```

Now, the line `TypeRequirement auto myVec = std::vector<int>{1, 2, 3};` fails with the following error message:

```
main.cpp: In function 'int main()':
main.cpp:16:55: error: deduced initializer does not satisfy placeholder constraints
   16 |   TypeRequirement auto myVec = std::vector<int>{1, 2, 3};
      |                                                       ^
main.cpp:16:55: note: constraints not satisfied
main.cpp:10:9:   required for the satisfaction of 'TypeRequirement<std::vector<int, std::allocator<int> > >'
main.cpp:10:27:   in requirements  [with T = std::vector<int, std::allocator<int> >]
main.cpp:12:12: note: the required type 'Other<T>' is invalid
   12 |   typename Other<T>;
      |   ~~~~~~~~~^~~~~~~~~
cc1plus: note: set '-fconcepts-diagnostics-depth=' to at least 2 for more detail
```

With type requirements we can make sure that a class has a nested type or that a template specialization is possible.

To show that a concept can be used to prove that an alias template specialization names a type, let's take our original example and create a template alias `Reference`:

```cpp
template<typename T> using Reference = T&;
```

And use it in the concept `TypeRequirement`:

```cpp
template<typename T>
concept TypeRequirement = requires {
  typename T::value_type;
  typename Other<T>;
  typename Reference<T>;
};
```

Our example should still compile:

```cpp
#include <iostream>
#include <vector>
template <typename>
struct Other;

template<typename T> using Reference = T&;


template<typename T>
concept TypeRequirement = requires {
  typename T::value_type;
  typename Other<T>;
  typename Reference<T>;
};

int main() {
  TypeRequirement auto myVec = std::vector<int>{1, 2, 3};
}
```

#### Nested requirements

We can use nested requirements to specify additional constraints in a concept without introducing another names concept.

You can think of nested requirements as one would think about lambda functions for STL algorithms. You can use lambdas to alter the behaviour of an algorithm without the need of naming a function or a function object.

In this case, you can write a constraint more suitable for your needs without the need of naming one more constraint that you'd only use in one (nested) context.

Its syntax follows the following form:

```
requires constraint-expression;
```

Let's start with a simpler example. Where the concept `Coupe` uses two other concepts `Car` and `Convertible`.

```cpp
#include <iostream>

struct AwesomeCabrio {
  void openRoof(){}
  void startEngine(){}
};

struct CoolCoupe {
    void startEngine(){}
};

template<typename C>
concept Car = requires (C car) {
    car.startEngine();
};


template<typename C>
concept Convertible = Car<C> && requires (C car) {
    car.openRoof();
};


template<typename C>
concept Coupe = Car<C> && requires (C car) {
    requires !Convertible<C>;
};


int main() {
  Convertible auto cabrio = AwesomeCabrio{};
  //Coupe auto notACoupe = AwesomeCabrio{}; // nested requirement '! Convertible<C>' is not satisfied
  Coupe auto coupe = CoolCoupe{};
}
```

Let's have a look at the concept `Coupe`. First, we make sure that only types satisfying the `Car` concept are accepted. Then we introduce a nested concept that requires that our template type is not a `Convertible`.

It's true that we don't *need* the nested constraint, we could express ourselves without it:

```cpp
template<typename C>
concept Coupe = Car<C> && !Convertible<C>;
```

Nevertheless, we saw the syntax in a working example.

Nested requires clauses can be used more effectively with local parameters that are listed in the outer `requires` scope, like in the next example with `C clonable`:

```cpp
#include <iostream>

struct Droid {
  Droid clone(){
    return Droid{};
  }
};
struct DroidV2 {
  Droid clones(){
    return Droid{};
  }
};

template<typename C>
concept Clonable = requires (C clonable) {
    clonable.clone();
    requires std::same_as<C, decltype(clonable.clone())>;
};


int main() {
  Clonable auto c = Droid{};
  // Clonable auto c2 = DroidV2{}; // nested requirement 'same_as<C, decltype (clonable.clone())>' is not satisfied
}
```

In this example, we have two droid types, `Droid` and `DroidV2`. We expect that droids should be clonable meaning that each type should have a clone method that returns another droid of the same type. With `DroidV2` we made a mistake and it still returns `Droid`.

Can we write a concept that catches this error?

We can easily write one with nested requirements. In the concept `Clonable` we work with a `C cloneable` local parameter. With the nested requirement `requires std::same_as<C, decltype(clonable.clone())>` we express that the clone method should return the same type as the parameters'.

You might argue that there is another way to express this, with the nested clause and you'd be right:

```cpp
template<typename C>
concept Clonable = requires (C clonable) {
    { clonable.clone() } -> std::same_as<C>;
};
```

## Conclusion
In this series we discussed a lot about one of the most important new features of C++: Concepts.

We saw that it's motivated by...
We saw the four different way we can express our requirements...
We had a sneak-peak on what predefined concepts exist
We saw how to write our own concepts which was quite a long story
