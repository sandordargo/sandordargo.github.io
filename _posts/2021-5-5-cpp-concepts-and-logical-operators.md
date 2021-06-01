---
layout: post
title: "C++ Concepts and logical operators"
date: 2021-5-5
category: dev
tags: [cpp, concepts, cpp20]
excerpt_separator: <!--more-->
---
In February and March, most of my posts were about [C++ concepts](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations) and now I'm amending it with a new article.
<!--more-->

## Why is this new post on concepts?

Because I had a misconception. Better to say, I didn't even think about some important aspects.

I said that obviously, we can use both `&&` and `||` logical operators to combine concepts. Oh, and of course, we can negate(`!`) - I wrote.

It's because I handled the `requires` clause as an ordinary boolean expression. But is that so?

## `!a` is not the opposite of `a`

By asking the above question, you guessed the answer. It's a no.

Let's suppose we have a function `foo()` that takes two parameters, `T bar` and `U baz`. We have some constraints on them. One of them must have a nested type `Blah` that is unsigned.

```cpp
#include <concepts>

template <typename T, typename U>
requires std::unsigned_integral<typename T::Blah> 
      || std::unsigned_integral<typename U::Blah>
void foo(T bar, U baz) {
    // ...
}


class MyType {
public:
    using Blah = unsigned int;
    // ...
};

int main() {
    MyType mt;
    foo(mt, 5);
    foo(5, mt);
    // error: no operand of the disjunction is satisfied
    // foo(5, 3);
}
```
When we call `foo()` with an instance of `MyType` in the first position, the requirements are satisfied by the first part of the disjunction and the second one is shortcircuited. All seems expected, though we could have already noticed something...

Let's go for the second case. We call `foo()` with an integer in the first place. Is its nested type `Blah` unsigned? It doesn't even have a nested type! Com'on, it's just an `int`!

What does this mean for us? It means that having something evaluated as `false` doesn't require that that an expression returns `false`. It can simply not be compilable at all.

Whereas for a normal boolean expression, we expect that it's well-formed and each subexpression is compilable.

That's the big difference.

For concepts, the opposite of a `true` expression is not `false`, but something that is either not well-formed, or `false`!

## What needs parentheses?

In the `requires` clause sometimes we wrap everything in between parentheses, sometimes we don't have to do so.

It depends on the simplicity of the expression. What is considered simple enough so that no parentheses are required?

- `bool` literals
- `bool` variables in any forms among value, `value<T>`, `T::value`, `trait<T>::value`
- concepts, such as `Concept<T>`
- nested requires expressions
- conjunctions (`&&`)
- disjunctions (`||`)

This list means that negations cannot be used without parentheses.

Try to compile this function:

```cpp
template <typename T>
requires !std::integral<T>
T add(T a, T b) {
   return a+b;
}
``` 

It will throw at you a similar error message:

```
main.cpp:8:10: error: expression must be enclosed in parentheses
    8 | requires !std::integral<T>
```
Why is this important?

## Subsumption and negations

All these matters, when the compiler is looking for the most constrained method.

Let's assume that we have a class `MyNumber` with two versions of `add`:

```cpp
class MyNumber {
public:
    MyNumber(T m){}
    T add(T a, T b) requires (not std::floating_point<T>) {
      // ...
      T sum;
      return sum; 
    }
    T add(T a, T b) requires (not std::floating_point<T>) && std::signed_integral<T> {
      // ...
      T sum;
      return sum; 
    }
};

```

The compiler uses boolean algebra to find the most constrained version of `add` to take. If you want to learn more about the theories behind this process that is called subsumption, I'd recommend you to read about [syllogism](https://en.wikipedia.org/wiki/Syllogism).

If we called MyNumber with a signed integer that is both not floating-point and is signed, you expect the compiler to subsume that the first constraints are common and we have to check whether the second one applies to our type or not.

It seems simple.

It's not so simple.

If you call and compile, you'll get an error message complaining about an ambiguous overload.

Even though we used the parentheses!

The problem is that `()` is part of the expression and subsumption checks the source location of the expression. If two expressions are originating from the same place, they are considered the same, so the compiler can subsume them.

As `()` is part of the expression, `(!std::floating_point)` originates from two different points and those 2 are not considered the same, they cannot be subsumed.

They are considered 2 different constraints, hence the call to `add()` would be ambiguous.

That's why if you need negation and thus you need parentheses, and you rely on subsumption, it's better to put those expressions into named concepts.

```cpp
template <typename T>
concept NotFloating = not std::floating_point<T>;

template <typename T>
class MyNumber {
public:
    MyNumber(T m){}
    T add(T a, T b) requires NotFloating<T> {
      // ...
      T sum;
      return sum; 
    }
    T add(T a, T b) requires NotFloating<T> && std::signed_integral<T> {
      // ...
      T sum;
      return sum; 
    }
};
```

Now `NotFloating` has the same source location whenever it is used, therefore it can be subsumed.

Not using negations directly, but putting expressions into named concepts seems to go against [the rule of using standard concepts whenever possible instead of writing our own concepts](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t11-whenever-possible-use-standard-concepts). But due to the subsumption rules, this is necessary.

## Conclusion

In this extra part of the concepts series, we saw that requiring the opposite of a `true` expression in concepts is not necessarily a `false` it can also mean something that would not be well-formed, something that would not compile.

As such, a conjunction or a disjunction is not as simple as a boolean `and` or `or` operation but something more complex. It gives more possibilities to have a concept satisfied.

We saw that negating an expression is not considered such a simple act as combining expressions in conjunction or disjunctions. They require parenthesws and in case you want to rely on subsumption and avoid ambiguous function calls, negated expressions have to be placed into their own concepts.

**If you want to learn more details about _C++ concepts_, [check out my book on Leanpub](https://leanpub.com/cppconcepts)!**