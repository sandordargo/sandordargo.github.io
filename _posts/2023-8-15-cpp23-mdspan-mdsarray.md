---
layout: post
title: "C++23: mdspan"
date: 2023-8-15
category: dev
tags: [cpp, cpp23, static, calloperator]
excerpt_separator: <!--more-->
---
Recently, we discussed that C++23 introduces the [multidimensional subscript operator](https://www.sandordargo.com/blog/2023/08/09/cpp23-multidimensional-subscription-operator) which will work - for the time being - only with new containers. In C++23 that basically means `mdspan`. In C++26, a similar type `mdarray` will follow, but it's also possible that existing types will start supporting the multidimensional subscript operator.

In this post, let's review what this new type - `mdspan` - is about.

## An immense work to introduce a new container

First of all, let's remind ourselves what a big work certain people put in into the standardization, or in other words, how much certain people work on making C++ better and safer. They deserve our gratitude.

If you look at this page (and at the below screenshot), you'll see 5 different papers listed next to mdspan. Those 5 papers don't even include the one for the [multidimensional subscript operator](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2128r6.pdf)!

![The 5 papers of mdspan in C++23]({{ site.baseurl }}/assets/img/mdspan-papers.png "The 5 papers of mdspan in C++23")
_The 5 papers of mdspan in C++23_

Why are there five different papers and what are they about? The reason behind this is that we are human beings and we cannot make perfect designs on the first run. [To be fair, not even ChatGPT can](https://www.sandordargo.com/blog/2023/07/05/trip-report-cpp-on-sea-2023#my-favourite-ideas).

If you check [the first paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0009r18.html), you'll see 18 (!) revisions spanning over 7 years (pun intended)! That's a lot of time and work introducing a new header! The other 4 papers contain minor fixes about [changing some types](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2599r2.pdf), [renaming certain members](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2604r0.html), [adding a missing `empty()` member function](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2613r1.html) and [modifying constructor behaviour](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2763r1.html).

Despite all the work, not every originally proposed feature made it to C++23, for example, `submdspan` (a view of a subset of an existing `mdspan`) should follow in the next standard.

At the time of writing, only Clang 17 supports it which I don't have access to and Compiler Explorer doesn't support it yet either. But we might play with [kokkos/mdspan reference implementation](https://github.com/kokkos/mdspan) even on [Compiler Explorer](https://godbolt.org/z/Mxa7cej1a).

## Main characteristics

First of all, `mdspan` is similar to span in the sense that it's a non-owning array view. A multidimensional array can map a tuple of indexes to one single element of the underlying container. Each indice can go from zero to the indice's extent - 1 where the product of extents is the size of the underlying array.

*Please note that the `stdex` namespace is to refers to the reference implementation coming from [kokkos/mdspan](https://github.com/kokkos/mdspan)*

```cpp
std::array numbers {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
stdex::mdspan mdspanOfNumbers {numbers.data(), stdex::extents{2, 5}};
for (size_t rowIndex=0; rowIndex < mdspanOfNumbers.extent(0); ++rowIndex) {
    for (size_t columnIndex=0; columnIndex < mdspanOfNumbers.extent(1); ++columnIndex) {
        std::cout << mdspanOfNumbers[rowIndex, columnIndex] << ' ';
    }
    std::cout << '\n';
}

/*
1 2 3 4 5 
6 7 8 9 10 
*/
```

Multidimensional arrays have long been useful for representing large amounts of data, describing points in physical space, or expressing approximations of functions. They are a natural way to represent mathematical objects like matrices and tensors.

C++, so far, has not provided good enough support for multidimensional arrays. Although there are different options available for the developers ([detailed here](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0009r18.html#why-are-existing-c-data-structures-not-enough)), they either lack some runtime-aspects, or they don't provide the desired attributes in terms of type homogeneity or memory allocations.

In terms of memory layout, `mdspan` is really flexible coming with three different layouts. In `layout_right`, the rightmost index gives access to the row, so it starts with the column on the left - that's how we used to do things in C and C++. `layout_left` provides the opposite behaviour where the leftmost index is the row and the rightmost is the column - this emulates Fortran's and Matlab's behaviour.

```cpp
std::array numbers {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
stdex::mdspan<int, stdex::extents<int, 2, 5>, stdex::layout_right> mdspanOfNumbers {numbers.data()};
for (size_t rowIndex=0; rowIndex < mdspanOfNumbers.extent(0); ++rowIndex) {
    for (size_t columnIndex=0; columnIndex < mdspanOfNumbers.extent(1); ++columnIndex) {
        std::cout << mdspanOfNumbers[rowIndex, columnIndex] << ' ';
    }
    std::cout << '\n';
}

/*
1 2 3 4 5 
6 7 8 9 10 
*/

std::array numbers {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
stdex::mdspan<int, stdex::extents<int, 2, 5>, stdex::layout_left> mdspanOfNumbers {numbers.data()};
for (size_t columnIndex=0; columnIndex < mdspanOfNumbers.extent(0); ++columnIndex) {
    for (size_t rowIndex=0; rowIndex < mdspanOfNumbers.extent(1); ++rowIndex) {
        std::cout << mdspanOfNumbers[columnIndex, rowIndex] << ' ';
    }
    std::cout << '\n';
}
/*
1 3 5 7 9 
2 4 6 8 10 
*/
```

Note that in the above two examples, the outputs are the same, but we changed the layout and the order of columns and rows. You can also observe in the above example that while previously we passed the extents as part of the constructor call, we can also pass them as class template arguments.

There is also `layout_stride` available which provides a generalization of the other two which lets you specify a different stride for each extent.

As you might have guessed, the reason behind providing different layouts is language interoperability.

In `mdspan` not only the layout is customizable, but also the accessors. Different hardware can have different behaviour and different libraries or language extensions can have different concepts of memory accesses. In order to support the different needs, one can customize the data accessors. [You can find an elaborate example here](https://github.com/kokkos/mdspan/blob/stable/examples/restrict_accessor/restrict_accessor.cpp).

Now you might ask, why do we get a view (`mdspan`) before a multidimensional container?!

There is ongoing work also for creating a standalone container called [`mdarray`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/wg21.link/p1684), but the span had the priority. The reason behind this is that we use span-like features more often than the full features containers. We more often pass around containers by reference just to access elements or call the `.size()` member function than actually performing memory allocations. This is even truer in the world of parallel programming, *"where multiple processing units access different elements of the same array in parallel [...] memory allocation and deallocation are “synchronization points” for parallel processing units.*

## Conclusion

In this post, we discovered another new feature of C++23, multidimensional array views, a.k.a. `mdspan`. We saw the motivations behind creating this new view and why it's even more important than an owning container. We saw some of the main characteristics of `mdspan`, how we can create and use it. We also saw, how customizable it is to provide compatibility with different languages and hardware.

We also saw what an immense work it has been to create this new standardized view. You might look for more features that you might have heard about (such as `submdspan`), but despite all these papers and revisions, not all the parts made it to C++23. 

Thanks for the work of the authors and thanks to you for reading this article!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!