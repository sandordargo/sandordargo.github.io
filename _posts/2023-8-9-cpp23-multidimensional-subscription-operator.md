---
layout: post
title: "C++23: multidimensional operator[]"
date: 2023-8-9
category: dev
tags: [cpp, cpp23, static, calloperator]
excerpt_separator: <!--more-->
---
Two weeks ago we talked about [how C++23 allows for the call and subscription operators to be `static`](https://www.sandordargo.com/blog/2023/07/26/cpp23-static-call-and-subscript-operator). This time, we stay in the land of operators and we'll continue discussing the subscript operator. We are going to see how the subscription operator becomes multidimensional thanks to the authors of [P2128R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2128r6.pdf).

## Motivations

If you ask C++ developers about how they access the elements of multidimensional arrays, it's likely that you'll get several different answers depending on their experience.

If you ask someone who is not-so-experienced or works in a non-mathematical domain, there is a fair chance that the answer will be that you should use multiple subscription operators in a row: `myMatrix[x][y]`.

There are a couple of problems with this approach:
- it implies that `myMatrix[x]` is a valid expression
- due to failure to inline it has negative performance implications
- `myMatrix[x]` might even perform a deep copy!

If you ask someone who is familiar with scientific libraries, the answer to how to access a multi-dimensional array will likely be to use the call operator `myMatrix(x, y)`.

While it can solve the problems mentioned for chained subscript operators, it has other issues.
- it's far from being intuitive to read `myMatrix(x, y) = 42`. Are we assigning a value to a function call or what?
- it's not consistent with the one-dimensional access of `myMatrix[x]`
- it might be difficult to distinguish invocables and multi-dimensional arrays. Even for the compiler
- even library authors often consider this as a workaround

There is yet another method. You might pass a tuple to the subscript operator: `myMatrix[{x, y}]`. Well, this is not very intuitive either and is still inconsistent with one-dimensional access

Using the subscript operator with several parameters (`myMatrix[x, y]`) would solve these issues.

## What is changing?

C++20 [deprecated uses of the comma operator in subscripting expressions](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1161r3.html). Let's see what changed through an example:

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

// The output of this in C++17:
// warning: left operand of comma operator has no effect [-Wunused-value]
// 2
std::cout << numbers[0, 1] << '\n';


// The output of this in C++20:  
// warning: top-level comma expression in array subscript is deprecated [-Wcomma-subscript]
// warning: left operand of comma operator has no effect [-Wunused-value]
// 2
std::cout << numbers[0, 1] << '\n';
```

In the above example, the comma operator is used and the first parameter is simply ignored. You get a warning for this by the compiler and C++20 added a second commit showing deprecation.

As this deprecation is quite new, the changes presented by [P2128R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2128r6.pdf) are only valid for *new* standard and user types. Starting from C++23, `operator[]` should be able to accept zero or more arguments, including variadic arguments.

C-arrays, `vector`s, `array`s, and other already existing containers do not benefit from this change. At least, not yet:

```cpp
std::vector<std::vector<int>> numbers = {
  {1, 2, 3, 4, 5},
  {11, 12, 13, 14, 15}
};

std::cout << numbers[0, 1] << '\n';
/*
<source>:10:25: warning: top-level comma expression in array subscript changed meaning in C++23 [-Wcomma-subscript]
   10 |     std::cout << numbers[0, 1] << '\n';
<source>:10:15: error: no match for 'operator<<' (operand types are 'std::ostream' {aka 'std::basic_ostream<char>'} and '__gnu_cxx::__alloc_traits<std::allocator<std::vector<int> >, std::vector<int> >::value_type' {aka 'std::vector<int>'})
   10 |     std::cout << numbers[0, 1] << '\n';
*/
```

This example fails, because `numbers[0, 1]` for a `vector` of `vector`s still tries to access `numbers[1]` instead of `numbers[0][1]`.

On the other hand, new types such as [`mdspan`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p0009r10.html) in C++23 or [`mdarray`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1684r0.pdf) most probably in C++26 will get this feature automatically.

For existing types, the new meaning might get adopted starting from C++26.

For the time being, it's not so easy to showcase this example. One solution is to use the `mdspan` library from *kokkos* ([Godbolt](https://godbolt.org/#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,selection:(endColumn:27,endLineNumber:11,positionColumn:27,positionLineNumber:11,selectionStartColumn:27,selectionStartLineNumber:11,startColumn:27,startLineNumber:11),source:'%23include+%3Cvector%3E%0A%23include+%3Chttps://raw.githubusercontent.com/kokkos/mdspan/single-header/mdspan.hpp%3E%0A%23include+%3Ciostream%3E%0A%0Aint+main()%0A%7B%0A++std::vector+v+%3D+%7B1,2,3,4,5,6,7,8,9,10,11,12%7D%3B%0A%0A++//+View+data+as+contiguous+memory+representing+2+rows+of+6+ints+each%0A++auto+multispan+%3D+std::experimental::mdspan(v.data(),+2,+6)%3B%0A++std::cout+%3C%3C+multispan%5B1,+1%5D+%3C%3C+!'%5Cn!'%3B%0A%7D%0A'),l:'5',n:'0',o:'C%2B%2B+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((g:!((h:compiler,i:(compiler:clang_trunk,deviceViewOpen:'1',filters:(b:'0',binary:'1',binaryObject:'1',commentOnly:'0',debugCalls:'1',demangle:'0',directives:'0',execute:'0',intel:'0',libraryCode:'0',trim:'1'),flagsViewOpen:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,libs:!((name:fmt,ver:trunk)),options:'-std%3Dc%2B%2B2b',overrides:!(),selection:(endColumn:1,endLineNumber:1,positionColumn:1,positionLineNumber:1,selectionStartColumn:1,selectionStartLineNumber:1,startColumn:1,startLineNumber:1),source:1),l:'5',n:'0',o:'+x86-64+clang+(trunk)+(Editor+%231)',t:'0')),k:50,l:'4',m:50,n:'0',o:'',s:0,t:'0'),(g:!((h:output,i:(editorid:1,fontScale:14,fontUsePx:'0',j:1,wrap:'1'),l:'5',n:'0',o:'Output+of+x86-64+clang+(trunk)+(Compiler+%231)',t:'0')),header:(),l:'4',m:50,n:'0',o:'',s:0,t:'0')),k:50,l:'3',n:'0',o:'',t:'0')),l:'2',n:'0',o:'',t:'0')),version:4)):


```cpp
#include <vector>
#include <https://raw.githubusercontent.com/kokkos/mdspan/single-header/mdspan.hpp>
#include <iostream>

int main()
{
  std::vector v = {1,2,3,4,5,6,7,8,9,10,11,12};

  // View data as contiguous memory representing 2 rows of 6 ints each
  auto multispan = std::experimental::mdspan(v.data(), 2, 6);
  std::cout << multispan[1, 1] << '\n';
}

/*
8
*/
```

## Conclusion

Today we discussed the different options we currently have in order to access items of multi-dimensional arrays. As we saw, using chained subscript operators, or the function call operator have their own shortcomings as well as passing a tuple to the subscript operator.

Starting from C++23, at least for new types, the subscript operator will take several arguments comma-separated thanks to [P2128R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2128r6.pdf).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
