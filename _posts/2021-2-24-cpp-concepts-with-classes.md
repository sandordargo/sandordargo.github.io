---
layout: post
title: "C++ concepts with classes"
date: 2021-2-24
category: dev
tags: [cpp, concepts]
excerpt_separator: <!--more-->
---
Last time we discussed [how to use concepts with functions](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them) and this time we are going to see how to use concepts with classes. I know it's not what I promised at the end of the previous article, but I realized that I simply forgot about this episode.
<!--more-->

[We saw last week](https://www.sandordargo.com/blog/2021/02/17/cpp-concepts-4-ways-to-use-them) that there are four ways to use concepts with functions:

- the `requires` clause
- the trailing `requires` clause
- constrained template parameters
- abbreviated function templates

With classes, we have fewer options. The *trailing `requires` clause* wouldn't make much sense as there is no function signature it could follow... 

And the abbreviated function templates also won't work.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

class WrappedNumber {
public:
  WrappedNumber(Number auto num) : m_num(num) {}
private:
  Number auto m_num; // error: non-static data member declared with placeholder
};
```

We cannot declare data members with `auto`, [it's prohibited by the standard](https://stackoverflow.com/questions/11302981/c11-declaring-non-static-data-members-as-auto).

If we remove the `auto`, we'll have a different error message saying that we must use `auto` (or `decltype(auto)`) after the concept `Number`.

So what is left?

- the `requires` clause
- constrained template parameters

For our examples, we are going to use the same incomplete `Number` concept we used last time.

```cpp
#include <concepts>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;
```

## The `requires` clause

We can use *the `requires` clause* to define constraints on a template class. All we have to do is the same as writing a template class and after the template parameter list, we have to put the requires clause with all the constraints we'd like to define.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;

template <typename T>
requires Number<T>
class WrappedNumber {
public:
  WrappedNumber(T num) : m_num(num) {}
private:
  T  m_num;
};

int main() {
    WrappedNumber wn{42};
    // WrappedNumber ws{"a string"}; // template constraint failure for 'template<class T>  requires  Number<T> class WrappedNumber'
}
```

As you can see in the example, apart from the additional line with `requires` it's the same as a template class.

If you use the template type name `T` at multiple places, the replacing values must be of the same type. In case you take two constrained `T`s in the constructor, they must of the same type. You won't be able to call with an `int` and with a `float` despite the fact that they both satisfy the concept `Number`.

In case you need that, for each - potentially different - usage of the template parameter, you need a different declaration in the template parameter list and also in the among the constraints:

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;


template <typename T, typename U>
requires Number<T> && Number<U>
class WrappedNumber {
public:
  WrappedNumber(T num, U anotherNum) : m_num(num), m_anotherNum(anotherNum) {}
private:
  T  m_num;
  U  m_anotherNum;
};

int main() {
    WrappedNumber wn{42, 4.2f};
}
```

This above example also shows that we can use compound expressions as constraints. That's something not possible with the other way to write constrained template classes.

## Constrained template parameters

With *constrained template parameters* it's even easier to use concepts. In the template parameter list, instead of the `typename` keyword you can simply concept you want to use.

Here is an exmaple:

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept Number = std::integral<T> || std::floating_point<T>;


template <Number T>
class WrappedNumber {
public:
  WrappedNumber(T num) : m_num(num) {}
private:
  T  m_num;
};

int main() {
    WrappedNumber wn{42};
    // WrappedNumber ws{"a string"}; // template constraint failure for 'template<class T>  requires  Number<T> class WrappedNumber'
}
```

In this example, you can see how we constrained `T` to satisfy the `Number` concept.

The clear advantage of *constrained template parameters* is that they are so easy to use, they are so easy to read and there is no extra verbosity.

The downside is that you cannot use compound expressions as constraints.

While with the `requires` clause you can write something like this:

```cpp
template <typename T>
requires std::integral<T> || std::floating_point<T>
class WrappedNumber {
  // ...
};
```

With the constrained template parameters something like that would be impossible. If you have to use some complex constraints, you must extract them into their own concept.

Apart from that, it's similar to the `requires` clause, in case you have multiple parameters that need to satisfy `Number`, but they can be different, you must use multiple template parameters:

```cpp
template <Number T, Number U>
class WrappedNumber {
public:
  WrappedNumber(T num, U anotherNum) : m_num(num), m_anotherNum(anotherNum) {}
private:
  T  m_num;
  U  m_anotherNum;
};
```
## Conclusion

Today we discovered the two ways to use concepts with classes. Both with *the `requires` clause* and with *constrained template parameters* we have an easy and readable way to use our concepts to constrain the types our template classes can accept.

With the former, we can even define some complex requirements without having to extract them into separate concepts, while with the latter we can only use one concept per template parameter, but on the contrary, it's very terse. Up to you to choose based on your needs.

Next time, we are really going to discuss [what kind of concepts we get from the standard library](https://www.sandordargo.com/blog/2021/03/03/cpp-concepts-in-standard-library) before we'd actually start writing our own concepts. No other surprises before!

Stay tuned!
