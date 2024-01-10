---
layout: post
title: "C++23: Allocator related changes"
date: 2024-1-10
category: dev
tags: [cpp, cpp23, allocators, ctad]
excerpt_separator: <!--more-->
---
In this post, we are going to review two changes related to allocators in C++. One is about providing size information about the allocated memory and the other is about how CTAD should happen for containers with non-default allocators.

Let's get into them.

## Providing size feedback in the Allocator interface
[P0401R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0401r6.html) gives a new way to allocate memory on the heap to limit spurious reallocations.

The new interface of `std::allocator` looks like this, but there is also a free function version of it.

```cpp
template<class Pointer>
  struct allocation_result {
    Pointer ptr;
    size_t count;
  };

// this class already exist:
namespace std {
  template<class T> class allocator {
   public:
   // lots of existing things
   // ...
   // this is the old way to allocate 
   [[nodiscard]] constexpr T* allocate(size_t n);

   // and this is the new way
   [[nodiscard]] constexpr allocation_result<T*> allocate_at_least(size_t n);

   // the interface for deallocation does not change
   constexpr void deallocate(T* p, size_t n);
  };
}
```

As you can see, `allocate_at_least` takes a number and it should allocate enough memory for at least that many instances of `T` on the heap. While `allocate` returns a single pointer to the beginning of the allocated memory, `allocate_at_least` returns a new `struct` called `allocation_result` which has two members, the "usual" pointer to the beginning of the allocated memory (`ptr`) and the number of `T`s memory got allocated for (`count`). `count` must be at least as large as the input parameter `n`, but it can also be more.

`allocate_at_least` might throw two kinds of exceptions. If the memory cannot be obtained, it throws a `bad_alloc` exception. If `n` is larger than the maximum value of `size_t` divided by the size of `T`, it throws a `bad_array_new_length` exception. I don't foresee that happening a lot.

When you write custom allocators, this new interface can come in quite handy in order to limit the number of allocations and copies performed. Let's take a vector for example. If you create a vector with 3 items, the chances are high that there will be more memory allocated. Then every time you push back, you either have to interact with the allocator or you have to perform some guesswork and handle if our guess is wrong.

There were other alternatives considered - [described in the paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p0401r6.html) - but somehow each of them introduces some extra work that is avoided if the function used for allocation returns the actual `count`.

## Non-deduction context for allocators in container deduction guides

[P1518R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1518r2.html) improves how *Class Template Argument Deduction*([CTAD](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction)) works for containers.

The paper explains and fixes two different contexts, in one a deduction guideline is not used when it should be used, and in the other case, a deduction guide is used when it should not be used.

> Deduction guides were introduced in C++17. They simplify the usage of class templates by allowing the compiler to automatically deduce template arguments based on the types of constructor arguments. By providing a deduction guide alongside a class template, developers can create instances of the template without explicitly specifying template parameters. The guide specifies how to deduce the template arguments from the constructor arguments, enhancing code readability and reducing verbosity. This feature streamlines the syntax when working with class templates, promoting cleaner and more concise code.

*Please note, that these problems - in most cases - have been already fixed by the major compilers, so if you want to run the examples, you should use an older compiler. Luckily, with [godbolt](https://godbolt.org/z/Gscj7ocYz), that's a piece of cake. With Clang, I had to go back to v14.*

Let's start with the first problem.

### Overconstrained allocators

```cpp
#include <memory_resource>
#include <stack>

int main()
{
    std::pmr::monotonic_buffer_resource mr;
    std::pmr::polymorphic_allocator<int> a = &mr;
    std::pmr::vector<int> pv(a);


    auto noCtadStack = std::stack<int, std::pmr::vector<int>>(pv, &mr); // #A

    auto ctadStack = std::stack(pv, &mr);  // #B
}
/*
<source>:13:22: error: no viable constructor or deduction guide for deduction of template arguments of 'stack'
    auto ctadStack = std::stack(pv, &mr);  // #B
*/
```

As you can see, the compiler complains that there is no viable deduction guide to create `ctadStack` at *#B*, whereas at line *#A*, we could create a stack with the very same arguments.

The only relevant deduction guideline for `std::stack` is:

```cpp
template<class Container, class Allocator>
stack(Container, Allocator) -> stack<typename Container::value_type, Container>;
```

Yet, it's not used due to `&mr`. The problem is that according to the standard's *[container.adaptors.general]* section, a deduction guide for a container adaptor shall **not** participate in overload resolution if it has an **Allocator** template parameter and a type that doesn't qualify as an allocator is deduced for that parameter. Another restricting factor would be if the deduction guide had both a Container and an Allocator template parameter and `uses_allocator_v<Container, Allocator>` was false.

`uses_allocator_v<std::pmr::vector<int>, std::pmr::monotonic_buffer_resource*>` is `true`, the problem is the first condition. A type `A` can be an allocator, if it has a nested-typedef `A::value_type`. Given that `std::pmr::monotonic_buffer_resource` lacks that typedef, it's not considered an allocator.

### Different, conflicting deduction

Let's have a look at a very similar example. But we are going to use a `vector`, instead of a `stack`. But the problem we will see also occurs for other containers.

```cpp
#include <memory_resource>
#include <vector>

int main()
{
    std::pmr::monotonic_buffer_resource mr;
    std::pmr::polymorphic_allocator<int> a = &mr;
    std::pmr::vector<int> pv(a);

    auto noCtadVector = std::vector<int, std::pmr::polymorphic_allocator<int>>(pv, &mr);

    auto ctadVector = std::vector(pv, &mr);
}
```

As `noCtadVector` compiles, we know that the used parameters are good to initialize a `std::vector`.

Let's look at the relevant `vector` constructor:

```cpp
namespace std {

template<class T, class Allocator>
class vector {
    vector(const vector<T, Allocator>&, const Allocator&);
    // ...
};

} 
```

From the first constructor parameter, `T` is `int` and `Allocator` is `std::pmr::polymorphic_allocator<int>`. But from the second parameter `Allocator` is `std::monotonic_buffer_resource*`. Those two are conflicting, therefore the deduction fails. The second parameter shouldn't participate in the deduction, because it doesn't bring any new information and just prevents "natural and useful code from working as desired".

### The solution

To the first problem, to solution is to modify the wording of the standard for container adaptors deduction guides. As we saw earlier, according to the original state, a deduction guide for a container adaptor shouldn't participate in the overload resolution if "it has an `Allocator` template and a type that does not qualify as an allocator is deduced for that parameter". This is modified so that if it has a `Container` template parameter, then this rule doesn't apply and the deduction guide should be part of the overload resolution (unless other rules apply).

To solve the second problem, the constructors for most standard containers (for the full list, [check P1518R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p1518r2.html#wording)) are changing following the same logic.

`constexpr vector(const vector&, const Allocator&);` becomes `constexpr vector(const vector&, const type_identity_t<Allocator>&);`  and `constexpr vector(vector&&, const Allocator&);` becomes `constexpr vector(vector&&, const type_identity_t<Allocator>&);`.

> `std::type_identity` is a C++20 feature that "can be used to establish non-deduced contexts in template argument deduction." That's exactly why and how it is used here.

## Conclusion

In this post, we saw two allocator-related changes in C++23. Thanks to the first change, the new interface of `std::allocator` gives a new way to allocate memory on the heap in order to limit spurious reallocations by returning size information about the allocated memory. The second discussed change fixes some CTAD issues that you can run into if you try to use `std::pmr` (or other custom allocators) with standard containers.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!