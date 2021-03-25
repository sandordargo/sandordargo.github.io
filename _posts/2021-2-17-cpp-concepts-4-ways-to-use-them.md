---
layout: post
title: "4 ways to use C++ concepts in functions"
date: 2021-2-17
category: dev
tags: [cpp, concepts]
excerpt_separator: <!--more-->
---
Welcome back to the series on C++ concepts. [In the previous article](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations) we discussed what are the motivations behind concepts, why we need them. Today we are going to focus on how to use existing concepts. There are a couple of different ways.
<!--more-->

## The 4 ways to use concepts

To be more specific, we have four different ways at our disposal.

For all the ways I am going to share, let's assume that we have a concept called `Number`. We are going to use a very simplistic implementation for it. I include it so that if you want to try the different code snippets, you have a concept to play with, but keep in mind that it is incomplete in a functional sense. More about that in a next episode.

```cpp
#include <concepts>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;
```

### Using the `requires` clause

In the first of the four presented ways, we use the `requires` clause between template parameter list and the function return type - which is `auto` in this case.

```cpp
template <typename T>
requires Number<T>
auto add(T a, T b) {
  return a+b;
}
```
Note how we use the concept, how we define in the `requires` clause that any `T` template parameter must satisfy the requirements of the concept `Number`.

In order to determine the return type we simply use `auto` type deduction, but we could use `T` instead as well.

Unfortunately, we can only add up two numbers of the same type. We cannot add a `float` with an `int`

If we tried so, we'd get a bit long, but quite understandable error message:

```
main.cpp: In function 'int main()':
main.cpp:15:27: error: no matching function for call to 'add(int, float)'
   15 |   std::cout << add(5,42.1f) << '\n';
      |                           ^
main.cpp:10:6: note: candidate: 'template<class T>  requires  Number<T> auto add(T, T)'
   10 | auto add(T a, T b)  {
      |      ^~~
main.cpp:10:6: note:   template argument deduction/substitution failed:
main.cpp:15:27: note:   deduced conflicting types for parameter 'T' ('int' and 'float')
   15 |   std::cout << add(5,42.1f) << '\n';
      |                           ^

```

If we wanted the capability of adding up numbers of multiple types, we'd need to introduce a second template parameter.

```cpp
template <typename T,
          typename U>
requires Number<T> && Number<U>
auto add(T a, U b) {
  return a+b;
}
```

Then calls such as `add(1, 2.14)` will also work. Please note that the concept was modified. The drawback is that for each new function parameter you'd need to introduce a new template parameter and a requirement on it.

With the requires clause, we can also express more complex constraints. For the sake of example, let's just "inline" the definition of number:

```cpp
template <typename T>
requires std::integral<T> || std::floating_point<T>
auto add(T a, T b) {
  return a+b;
}
```

Though for better readability, in most cases, I consider a better practice to name your concept, especially when you have a more complex expression.

### Trailing `requires` clause

We can also use the so-called *trailing `requires` clause* that comes after the function parameter list (and the qualifiers - `const`, `override`, etc. - if any) and before the function implementation. 

```cpp
template <typename T>
auto add(T a, T b) requires Number<T> {
  return a+b;
}
```

We have the same result as we had with the `requires` clause we just wrote it with different semantics. It means that we still cannot add two numbers of different types. We'd need to modify the template definition similarly as we did before:

```cpp
template <typename T, typename U>
auto add(T a, U b) requires Number<T> && Number<U> {
  return a+b;
}
```
Still, we have the drawback of scalability. Each new function parameter potentially of a different type needs its own template parameter.

Just as for the `requires` clause, you can express more complex constraints in the *trailing `requires` clause*.

```cpp
template <typename T>
auto add(T a, T b) requires std::integral<T> || std::floating_point<T> {
  return a+b;
}
```

### Constrained template parameter

The third way to use a concept is a bit terser than the previous ones, which also brings some limitations.

```cpp
template <Number T>
auto add(T a, T b) {
  return a+b;
}
```

As you can see, we don't need any `requires` clause, we can simply define a requirement on our template parameters right where we declare them. We use a concept name instead of the keyword `typename`. We'll achieve the very same result as with the previous two methods.

If you don't believe it, I'd urge you to check it on [Compiler Explorer](https://godbolt.org/z/sGTsab).

At the same time, it's worth to note that this method has a limitation. When you use the *`requires` clause* in any of two presented ways, you can define an expression such as `requires std::integral<T> || std::floating_point<T>`. When you use the *constrained template parameter* way, you cannot have such expressions; `template <std::integral || std::floating_point T>` __is not valid__.

So with this way, you can only use single concepts, but in a more concise form as with the previous ones.

### Abbreviated function templates

Oh, you looked for brevity? Here you go!

```cpp
auto add(Number auto a, Number auto b) {
  return a+b;
}
```

There is no need for any template parameter list or *`requires` clause* when you opt for *abbreviated function templates*. You can directly use the concept where the function arguments are enumerated.

There is one thing to notice and more to mention.

After the concept `Number` we put `auto`. As such we can see that `Number` is a constraint on the type, not a type itself. Imagine if you'd simply see `auto add(Number a, Number b)`. How would you know as a user that `Number` is not a type but a concept?

The other thing I wanted to mention is that when you follow the *abbreviated function template* way, you can mix the types of the parameters. You can add an `int` to a `float`.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

auto add(Number auto a, Number auto b) {
  return a+b;
}

int main() {
  std::cout << add(1, 2.5) << '\n';
}
/*
3.5
*/
```
So with *abbreviated function templates* we can take different types without specifying multiple template parameters. It makes sense as we don't have any template parameters in fact.

The disadvantage of this way of using concepts is that just like with *constrained template parameters*, we cannot use complex expressions to articulate our constraints.

## How to choose among the 4 ways?

We have just seen 4 ways to use concepts, let's have a look at them together.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

template <typename T>
requires Number<T>
auto addRequiresClause(T a, T b) {
  return a+b;
}

template <typename T>
auto addTrailingRequiresClause(T a, T b) requires Number<T> {
  return a+b;
}

template <Number T>
auto addConstrainedTemplate(T a, T b) {
  return a+b;
}

auto addAbbreviatedFunctionTemplate(Number auto a, Number auto b) {
  return a+b;
}

int main() {
    std::cout << "addRequiresClause(1, 2): " << addRequiresClause(1, 2) << '\n';
    // std::cout << "addRequiresClause(1, 2.5): " << addRequiresClause(1, 2.5) << '\n'; // error: no matching function for call to 'addRequiresClause(int, double)'
    std::cout << "addTrailingRequiresClause(1, 2): " << addTrailingRequiresClause(1, 2) << '\n';
    // std::cout << "addTrailinRequiresClause(1, 2): " << addTrailinRequiresClause(1, 2.5) << '\n'; // error: no matching function for call to 'addTrailinRequiresClause(int, double)'
    std::cout << "addConstrainedTemplate(1, 2): " << addConstrainedTemplate(1, 2) << '\n';
    // std::cout << "addConstrainedTemplate(1, 2): " << addConstrainedTemplate(1, 2.5) << '\n'; // error: no matching function for call to 'addConstrainedTemplate(int, double)'
    std::cout << "addAbbreviatedFunctionTemplate(1, 2): " << addAbbreviatedFunctionTemplate(1, 2) << '\n';
    std::cout << "addAbbreviatedFunctionTemplate(1, 2): " << addAbbreviatedFunctionTemplate(1, 2.14) << '\n';
}
```
Which form should we use? As always, the answer is *it depends*...

If you have a complex requirement, to be able to use an expression you need either the *`requires` clause* or the *trailing `requires` clause*.

What do I mean by a complex requirement? Anything that has more than one concept in it! Like `std::integral<T> || std::floating_point<T>`. That is something you cannot express either with a *constrained template parameter* or with an *abbreviated template function*.

If you still want to use them, you have to extract the complex constraint expressions into their own concept.

This is exactly what we did when we defined the concept `Number`. On the other hand, if your concept uses multiple parameters (something we'll see soon), you still cannot use *constrained template parameters* or *abbreviated template function* - or at least I didn't find a way for the time being.

If I have complex requirements and I don't want to define and name a concept, I'd go with either of the first two options, namely with *`requires`* clause or with *trailing `requires` clause*.

In case I have a simple requirement, I'd go with the *abbreviated function template*. Though we must remember that *abbreviated function templates* let you call your function with multiple different types at the same time, like how we called `add` with an `int` and with a `float`. If that is a problem and you despise the verboseness of the `requires` clause, choose a *constrained template parameter*.

Let's also remember that we talk about templates. For whatever combination, a new specialization will be generated by the compiler at compile time. It's worth to remember this in case you avoided templates already because of constraints on the binary size or compile time.

## Conclusion

Today, we have seen how to use concepts with function parameters. We detailed 4 different ways and saw that the more verbose ones give us more flexibility on the constraints, while the tersest one (*abbreviated function template*) gives extreme flexibility with the types we can call the function with.

~~Next time, we are going to discuss what kind of concepts we get from the standard library before we'd actually start writing our own concepts.~~

The next article is about [how to use concepts with classes](https://www.sandordargo.com/blog/2021/02/24/cpp-concepts-with-classes)!

Stay tuned!