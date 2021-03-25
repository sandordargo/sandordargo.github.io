---
layout: post
title: "How to write your own C++ concepts? Part II."
date: 2021-3-17
category: dev
tags: [cpp, concepts]
excerpt_separator: <!--more-->
---
Last week we started to discuss [how to write our own concepts](https://www.sandordargo.com/blog/2021/03/10/write-your-own-cpp-concepts-part-i). Our first step was to combine different [already existing concepts](https://www.sandordargo.com/blog/2021/03/03/cpp-concepts-in-standard-library), then we continued with declaring constraints on the existence of certain operations, certain methods.
<!--more-->

Today, we are going to discover how to express our requirements on function return types, how to write type requirements (and what they are) and we are going to finish with discussing nested requirements.

## Write your own constraints

Last time, we had an example with the concept `HasSquare`. It accepts any type that has a `square` function regardless of the return type. 

```cpp
#include <iostream>
#include <string>
#include <concepts>

template <typename T>
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


void printSquare(HasSquare auto number) {
  std::cout << number.square() << '\n';
}

int main() {
  printSquare(IntWithoutSquare{4}); // error: use of function 'void printSquare(auto:11) [with auto:11 = IntWithoutSquare]' with unsatisfied constraints, 
                                    // the required expression 't.square()' is invalid
  printSquare(IntWithSquare{5});
}
```
Now let's continue with constraining the return types.

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
- The expression on what you want to set a return type requirement must be surrounded by braces (`{}`), then comes an arrow (`->`) followed by the constraint of the return type.
- A constraint cannot simply be a type. Had you written simply `int`, you would receive an error message: *return-type-requirement is not a type-constraint.* The original concepts TS allowed the direct usage of types, so if you experimented with that, you might be surprised by this error. This possibility was removed by [P1452R2](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1452r2.html).

There are a number of reasons for this removal. One of the motivations was that it would interfere with a future direction of wanting to adopt a generalized form of `auto`, like `vector<auto>` or `vector<Concept>.`

So instead of simply naming a type you have to choose a concept! If you want to set the return type one of the two following options will satisfy your needs:

```cpp
{t.square()} -> std::same_as<int>;
{t.square()} -> std::convertible_to<int>;
```

I think that the difference is obvious. In case of `std::same_as`, the return value must be the same as specified as the template argument, while with `std::convertible_to` conversions are allowed.

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

With type requirements, we can express that a certain type is valid in a specific context. Type requirements can be used to verify that

- a certain nested type exists
- a class template specialization names a type
- an alias template specialization names a type

You have to use the keyword `typename` along with the type name that is expected to exist:

```cpp
template<typename T>
concept TypeRequirement = requires {
  typename T::value_type;
  typename Other<T>;
};
```
The concept `TypeRequirement` requires that the type `T` has a nested type `value_type`, and that the class template `Other` can be instantiated with `T`.

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

A `std::vector` has an inner member type `value_type` (requested on line 8) and the class template `Other` can be instantiated with `std::vector<int>` (line 9).

At the same time, an `int` doesn't have any member, in particular `value_type`, so it doesn't satisfy the constraints of `TypeRequirement`.

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

With type requirements, we can make sure that a class has a nested member type or that a template specialization is possible.

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

### Nested requirements

We can use nested requirements to specify additional constraints in a concept without introducing another named concepts.

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

We can, in fact as probably you noticed, we already did. In the concept `Clonable` we work with a `C cloneable` local parameter. With the nested requirement `requires std::same_as<C, decltype(clonable.clone())>` we express that the clone method should return the same type as the parameters'.

You might argue that there is another way to express this, without the nested clause and you'd be right:

```cpp
template<typename C>
concept Clonable = requires (C clonable) {
    { clonable.clone() } -> std::same_as<C>;
};
```

For a more complex example, I'd recommend you to check the implementation of `SemiRegular` concepts on [C++ Reference](https://en.cppreference.com/w/cpp/language/constraints).

To incorporate one of the requirements of `Semiregular` to our `Clonable` concept, we could write this:

```cpp
template<typename C>
concept Clonable = requires (C clonable) {
    { clonable.clone() } -> std::same_as<C>;
    requires std::same_as<C*, decltype(&clonable)>;
};
```

This additional line makes sure that the address of operator (`&`) returns the same type for the `cloneable` parameter as `C*` is. 

I agree, it doesn't make much sense in this context (it does for `SemiRegular`), but it's finally an example that is not easier to express without a nested requirement than with.

In the next post, we'll see how to use a nested requirement when even the enclosing concept is unnamed.

## Conclusion

Today we continued and finished discussing what building blocks are available for us to write our own concepts. We saw how to make constraints on function return types, how to use type requirements on inner types, template aliases and specialisations and finally we saw that it's possible to nest requirements, even though often there are easier ways to express ourselves.

Next time, we'll continue with [some real life examples](https://www.sandordargo.com/blog/2021/03/24/concepts-in-real-life) of how concepts can make our projects easier to understand. Stay tuned!