---
layout: post
title: "What are type traits?"
date: 2021-4-7
category: other
tags: [cpp, typetraits, templates]
excerpt_separator: <!--more-->
---
Let's start with a more generic question, what is a trait? What does the word *trait* mean?
<!--more-->

According to the [Cambridge Dictionary](https://dictionary.cambridge.org/dictionary/english/trait), a *trait* is "a particular characteristic that can produce a particular type of behaviour". Or simply "a characteristic, especially of a personality".

It's important to start our quest with the generic meaning, as many of us are native English speakers and having a clear understanding of the word *trait* helps us to have a better understanding also on the programming concept.

In C++, we can think about type traits as properties of a type. The `<type_traits>` header was an addition introduced by C++11. Type traits can be used in template metaprogramming to inspect or even to modify the properties of a type.

As we saw in the [C++ concepts series](https://www.sandordargo.com/blog/2021/02/10/cpp-concepts-motivations), you'd often need the information of what kind of types are accepted by a template, what types are supported by certain operations. While concepts are much superior in terms of expressiveness or usability, with type traits you could already introduce compile-time conditions on what should be accepted as valid code and what not.

Though *type traits* can help with even more. With their help, you can also add or remove the `const` specifier, or you can turn a pointer or a reference into a value and so on.

As already mentioned, the library is used in the context of template metaprogramming, so everything happens at compile time. 

## Show me a type trait!

In the concepts series, I already mentioned `std::is_integral` (in fact, I used `std::is_integral_v`, more on that later.) Like other type traits, `std::is_integral` is after all an `integral_constant` that has a static `value` member and some type information.

Let's see how `std::is_integral` is implemented, by looking at the [GCC](https://code.woboq.org/gcc/libstdc++-v3/include/std/type_traits.html#std::is_integral) implementation. While it might be different for other implementations, it should give you the basic idea.

```cpp
template<typename _Tp>
  struct is_integral
  : public __is_integral_helper<typename remove_cv<_Tp>::type>::type
  { };
```

At first glance, we can see that it uses a certain `__is_integral_helper` that is also a template and it takes the passed in type without its `const` or `volatile` qualifier if any.

Now let's have a look at `__is_integral_helper`.

Due to the limitations of this blog post and also due to common sense I won't enumerate all the specialisations of the template `_is_integral_helper`, I'll only show here three just to give you the idea.

```cpp
template<typename>
  struct __is_integral_helper
  : public false_type { };

template<>
  struct __is_integral_helper<bool>
  : public true_type { };

template<>
  struct __is_integral_helper<int>
  : public true_type { };
```

As we can observe, the default implementation of `__is_integral_helper` is a `false_type`. Meaning that in case you call `std::is_integral` with a random type, that type will be handed over to `__is_integral_helper` and it will be a false type that has the value of `false`, therefore the check fails.

For any type that should return `true` for the `is_integral` checks, `__is_integral_helper` should be specialized and it should inherit from `true_type`.

In order to close this circle, let's see how `true_type` and `false_type` are implemented.

```cpp
/// The type used as a compile-time boolean with true value.
typedef integral_constant<bool, true>     true_type;

/// The type used as a compile-time boolean with false value.
typedef integral_constant<bool, false>    false_type;
```

As we can see, they are simple aliased `integral_constants`.

As the last step, let's see how `std::integral_constant` is built. (I omit the #if, etc. directives on purpose)

```cpp
template<typename _Tp, _Tp __v>
  struct integral_constant
  {
    static constexpr _Tp                  value = __v;
    typedef _Tp                           value_type;
    typedef integral_constant<_Tp, __v>   type;
    constexpr operator value_type() const noexcept { return value; }
    constexpr value_type operator()() const noexcept { return value; }
  };
```

So `integral_constant` takes two template parameters. It takes a type `_Tp` and a value `__v` of the just previously introduced type `_Tp`.

`__v` will be accessible as the static `value` member, while the type `_Tp` itself can be referred to as the `value_type` nested type. With the `type` typedef you can access the type itself.

So `true_type` is an `integral_constant` where `type` is `bool` and value is `true`.

In case you have `std::is_integral<int>` - through multiple layers - it inherits from `true_type`, `std::is_integral<int>::value` is `true`. For any type `T`, `std::is_integral<T>::type` is bool.

## How to make your type satisfy a type trait

We've just seen how `std::is_integral` is implemented. Capitalizing on that we might think that if you have a class `MyInt` then having it an integral type only means that we simply have to write such code (I omit the problem of references and cv qualifications for the sake of simplicity):

```cpp
template<>
struct std::is_integral<MyInt> : public std::integral_constant<bool, true> {};
```

This is exactly what I proposed in the [Write your own concepts](https://www.sandordargo.com/blog/2021/03/10/write-your-own-cpp-concepts-part-i) article.

If you read attentively, probably you pointed out that I used the auxiliary "might" and it's not incidental.

I learned that having such a specialization results in undefined behaviour according to the standard [meta.type.synop (1)]:

> *"The behavior of a program that adds specializations for any of the templates defined in this subclause is undefined unless otherwise specified."*
 
What is in that subsection? Go look for a draft standard ([here is one](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4659.pdf)) if you don't have access to a paid version. It's a very long list, and I tell you `std::is_integral` is part of it. In fact, all the primary or composite type categories are in there.

Why?

As Howard Hinnant, the father of `<chrono>` [explained on StackOverflow](https://stackoverflow.com/questions/25345486/why-specializing-a-type-trait-could-result-in-undefined-behaviour) "for any given type T, exactly one of the primary type categories has a value member that evaluates to true." If a type satisfies `std::is_floating_point` then we can safely assume that `std::is_class` will evaluate to false. As soon as we are allowed to add specializations, we cannot rely on this.

```cpp
#include <type_traits>

class MyInt {};

template<>
struct std::is_integral<MyInt> : public std::integral_constant<bool, true> {};

int main() {
    static_assert(std::is_integral<MyInt>::value, "MyInt is not integral types");
    static_assert(std::is_class<MyInt>::value, "MyInt is not integral types");
}
```

In the above example, `MyInt` breaks the explained assumption and this is in fact undefined behaviour, something you should not rely on.

And the above example shows us another reason, why such specializations cannot be considered a good practice. Developers cannot be trusted that much. We either made a mistake or simply lied by making `MyInt` an integral type as it doesn't behave at all like an integral.

This basically means that you cannot make your type satisfy a type trait in most cases. (As mentioned the traits that are not allowed to be specialized are listed in the standard).

## Conclusion

Today, we learned what type traits are, how they are implemented and we also saw that we cannot explicitly say about a user-defined type that it belongs to a primary or composite type category. Next week, we'll see how we can use type traits.