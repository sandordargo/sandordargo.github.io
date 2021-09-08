---
layout: post
title: "Do we correctly calculate min and max?"
date: 2021-9-8
category: dev
tags: [cpp, tutorial, algorithms]
excerpt_separator: <!--more-->
---
This article is inspired by Walter E Brown's talk at the [Italian C++ Conference 2021: Extrema: Correctly calculating `min` and `max`](https://www.youtube.com/watch?v=e-TNCbX8mOQ).
<!--more-->

Walter brought up several issues with these algorithms starting from the problem of comparing different types to the question of how to pass efficiently parameters, but I only want to focus on one possible issue.

Let's have a look at a simplistic implementation of `min` and `max` in C++20 style that he shared:

```cpp
auto min(auto left, auto right) {
  return left < right ? left : right;
}

auto max(auto left, auto right) {
  return right < left ? left : right;
}
```

So what's the problem?

What happens if `left == right`?

When left and right are equal both `min` and `max` would return the same. That is `right`. 

But is that right?

According to Walter, that is not right. He raised his points, but they didn't find them interesting enough in the Committee in 2014.

They are definitely interesting enough to discuss here. I think he unveils some points we might not think about otherwise.

Some opposed his idea because it should not matter which one is returned, because after all if `left == right`, the values should be indistinguishable.

That is not necessarily the case.

He comes with an example of a student class:

```cpp
struct Student {
  std::string name;
  int id;
  inline static int registrar = 0;
  Student(const std::string& iName):
    name(iName), id(registrar++) {}

  friend bool operator<(const Student& lhs,
                        const Student& rhs) {
    return lhs.name < rhs.name;
  }
};
```

In this case, we can observe that if two students have the same name - which is not impossible - the objects representing them are not indistinguishable. They do have distinct `id`s.

Yet, both `min` and `max` will return, `right` - according to the implementation that Walter shared.

We might argue that if we don't want that, we should implement the comparison operators in a different way. We should, in fact, make the `Student::id` part of the comparison operators and we wouldn't have this problem.

I have a feeling that if we need these logical operators and we are afraid that two objects might be evaluated to be equal while they are not indistinguishable, we should modify the logical operators.

Since C++20, we can use the spaceship operator to automatically define all the comparison operators.

In the case of the `Student` class, it would look like this:

```cpp
auto operator<=>(const Student&) const = default;
```
If it's possible for the compiler to generate the operators, they will take into account "all base classes left to right and all non-static members of the class in their declaration order".

This means that `Student::id` will be taken into account, hence to have two indistinguishable objects requires having two objects with the same values in each field. Then it should really not matter which one is returned.

You might argue, that logically we cannot do that in all cases. You might be right, you might come up with such a case, but I think this was the main reason while Walter's complaints were not taken into account.

Or were they?

Let's have a look at [the MSVCC implementation](https://github.com/microsoft/STL/blob/9a9820df1a1d3fa84100e3169ff37fdd4fa41759/stl/inc/utility#L30-L70).

Here is a simplified excerpt.

```cpp
template <class _Ty>
_NODISCARD _Post_equal_to_(_Left < _Right ? _Right : _Left) constexpr const _Ty& //
    (max) (const _Ty& _Left, const _Ty& _Right) noexcept(noexcept(_Left < _Right)) /* strengthened */ {
    // return larger of _Left and _Right
    return _Left < _Right ? _Right : _Left;
}

template <class _Ty>
_NODISCARD _Post_equal_to_(_Right < _Left ? _Right : _Left) constexpr const _Ty& //
    (min) (const _Ty& _Left, const _Ty& _Right) noexcept(noexcept(_Right < _Left)) /* strengthened */ {
    // return smaller of _Left and _Right
    return _Right < _Left ? _Right : _Left;
}
```

In case `_Left == _Right`, `max` returns `_Left`, and `min` returns also `_Left`.

Let's look at [clang](https://github.com/llvm-mirror/libcxx/blob/master/include/algorithm#L2517-L2629) too:

```cpp
template <class _Tp, class _Compare>
_LIBCPP_NODISCARD_EXT inline
_LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR_AFTER_CXX11
const _Tp&
min(const _Tp& __a, const _Tp& __b, _Compare __comp)
{
    return __comp(__b, __a) ? __b : __a;
}


template <class _Tp, class _Compare>
_LIBCPP_NODISCARD_EXT inline
_LIBCPP_INLINE_VISIBILITY _LIBCPP_CONSTEXPR_AFTER_CXX11
const _Tp&
max(const _Tp& __a, const _Tp& __b, _Compare __comp)
{
    return __comp(__a, __b) ? __b : __a;
}
```

It's essentially the same, but in this case, `__a` is returned for the case when the elements equal which was called `_Left` in MS.

So yes, both for clang and MSVCC the returned value is the same for `min` and `max` if the inputs are equal. The only difference is that one returns the first input, the other the second. Gcc acts like clang, it returns the first, the lefthand-side input.

It would interesting to know what is the reason of  Microsoft having chosen the other value.

But it doesn't change the fact that both are strange. Since Walter raised the point at the Committee, others also called this a bug, for example [Sean Paretn at C++Now](https://www.youtube.com/watch?v=giNtMitSdfQ&t=1448s).

If you are really bothered by this, and you'd expect that min return the first item and max the second, you can use [`std::minmax`](https://www.sandordargo.com/blog/2021/09/01/stl-alogorithms-tutorial-part-24-min-max-algorithms#minmax) since C++11.

It either takes two elements or a list of items, but in our case only the case of two items is interesting.

`std::minmax` returns a pair where the first item is a const reference to the minimal element and the second is the max. In case the two inputs are equal, the first item is the first input, the second is the max.

Yes, this means that with `min` and `max` you cannot model `minmax`.

At least we have a workaround.

## Conclusion

[In his recent talk](https://www.youtube.com/watch?v=e-TNCbX8mOQ), Walter E Brown shared his view that it's incorrect that both `std::min` and `std::max` returns the same value in case its two inputs are equal.

If that matters to you, you have different workarounds. You can manually implement `min` and `max` in a way that you like. You can use `minmax` as well, or you can implement the comparison operator in a way that two values are indistinguishable in case they are equal.

Let me know if you faced this issue in your code.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
