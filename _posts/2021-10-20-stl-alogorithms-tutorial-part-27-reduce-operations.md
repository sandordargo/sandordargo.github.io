---
layout: post
title: "The big STL Algorithms tutorial: reduce operations"
date: 2021-10-20
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), it's time to move forward and start discussing the `<numeric>` header. We discussed all the non-range functions of the `<algorithm>` header.
<!--more-->

Today we're going to discuss:

* `accumulate`
* `reduce`
* `transform_reduce`

## `std::accumulate`

The C++ standard library doesn't have a `sum` function that you could call to add up all the elements of a container and get the sum of its items. What you'll likely end up with - unless you write a raw `for` loop - is `std::accumulate.`

It takes a range by its begin and end iterators, an initial value and then it uses `operator+` first on the initial value and the first element of the range, then on their sum and the next value and so on, till there are no more elements to add.

As an initial value, we take the identity property of addition, which for numbers is 0. I say for numbers because you can define `operator+` on any type. For a `std::string`, it would be the empty string.

```cpp
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector nums {1, 2, 3, 4};
    std::cout << "sum: " 
              << std::accumulate(nums.begin(), nums.end(), 0) 
              <<'\n';
}
/*
sum: 10
*/
```

It's also possible not to use `operator+` with `accumulate`, but to provide a custom binary operation. Let's showcase it still with addition.

```cpp
#include <iostream>
#include <numeric>
#include <vector>

int main()
{

    std::vector nums {1, 2, 3, 4};
    std::cout << "sum: " 
              << std::accumulate(nums.begin(), nums.end(), 0,  [] (int previousResult, int item) {
                    return previousResult + item;
                  })
              <<'\n';
}
/*
sum: 10
*/
```

It's worth noting that in the lambda, the first parameter is the so far accumulated result (the initial value in the first iteration) and as a second parameter, the next element of the container is passed.

The accumulated result can be a different type than each element. Let's try to join numbers into a string with a separator.

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main()
{

    std::vector nums {1, 2, 3, 4};
    std::cout << std::accumulate(nums.begin(), nums.end(), std::string(),  [] (std::string previousResult, int item) {
                    return previousResult + '-' + std::to_string(item);
                  })
              <<'\n';
}
/*
-1-2-3-4
*/
```

Now the problem is that our result is prefixed with a dash, that we might not want.

There are two ways to handle this. One is through the lambda:

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main()
{

    std::vector nums {1, 2, 3, 4};
    std::cout << std::accumulate(nums.begin(), nums.end(), std::string(),  [] (std::string previousResult, int item) {
                    if (previousResult.empty()) {
                      return std::to_string(item);
                    }
                    return previousResult + '-' + std::to_string(item);
                  })
              <<'\n';
}
/*
1-2-3-4
*/
```
If the `previousResult` is empty which is the initial value, we don't add a separator and we return early. Otherwise, business as usual.

The other is through the initial element and the beginning point of the accumulation:

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main()
{

    std::vector nums {1, 2, 3, 4};
    std::cout << std::accumulate(nums.begin()+1, nums.end(), std::to_string(nums[0]),  [] (std::string previousResult, int item) {
                    return previousResult + '-' + std::to_string(item);
                  })
              <<'\n';
}
/*
1-2-3-4
*/
```

Note that in this example we both had to modify the beginning of the range and in initial value, while in the previous solution we only modified the lambda. But we do an extra check for each iteration.

I think the first one is more readable (for my eyes at least), and in terms of performance  - [according to Quick Bench](https://quick-bench.com/q/-3peTd5FqQhG1xgw95szkH3B29c) - there is no significant difference.

## `reduce`

`std::reduce` is very similar to `std::accumulate`. The differences are:
- `std::reduce` was only introduced with C++17
- While `std::accumulate` is basically a left fold operation, `std::reduce` doesn't guarantee any order
- As elements can be rearranged and grouped during execution, it makes sense that `std::reduce` can take an `ExecutionPolicy` in the *"0th"* position

To demonstrate the main difference, let's run the previous example with `reduce` instead of `accumulate`:

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main()
{

    std::vector nums {1, 2, 3, 4};
    std::cout << std::reduce(nums.begin()+1, nums.end(), std::to_string(nums[0]),  [] (std::string previousResult, int item) {
                    return previousResult + '-' + std::to_string(item);
                  })
              <<'\n';
}
```

It doesn't compile!

```
main.cpp:10:84: note: candidate: 'main()::<lambda(std::string, int)>'
   10 |     std::cout << std::reduce(nums.begin()+1, nums.end(), std::to_string(nums[0]),  [] (std::string previousResult, int item) {
      |                                                                                    ^
main.cpp:10:84: note:   no known conversion for argument 2 from 'std::__cxx11::basic_string<char>' to 'int'
```

That is very interesting. It complains that a `string` cannot be converted to an integer. That's true, but we didn't have such a problem with `accumulate`! So there must be another difference!

So what does the documentation say about `BinaryOp`:

> "`T` must meet the requirements of `MoveConstructible`. and `binary_op(init, *first)`, `binary_op(*first, init)`, `binary_op(init, init)`, and `binary_op(*first, *first)` must be convertible to `T`.

Clearly, our binary operation doesn't satisfy these requirements.

What does the documentation say for `accumulate`?

> **op**: binary operation function object that will be applied. The binary operator takes the current accumulation value `a` (initialized to `init`) and the value of the current element `b`.
> The signature of the function should be equivalent to the following:
> `Ret fun(const Type1 &a, const Type2 &b);`
> The type `Type1` must be such that an object of type `T` can be implicitly converted to `Type1`. The type Type2 must be such that an object of type `InputIt` can be dereferenced and then implicitly converted to `Type2`. The type Ret must be such that an object of type `T` can be assigned a value of type `Ret`.

The only things missing are 
- that `T` is the type of the `accumulate`'s return value and the type of `init`
- `InputIt` is the type of the begin and end iterators.

So there is this extra - explicitly - unsaid difference between `accumulate` and `reduce`.

With `accumulate`, you fold all the elements to get a result in whatever type, but with `reduce` you fold the elements in a way that the result must stay convertible to the type of the elements.

I think that the reason behind this is that `reduce` can take items in whatever order and even the result of the previous iteration can appear in both positions of the `BinaryOp`.

So let's see a working example.

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main()
{

    std::vector nums {1, 2, 3, 4};
    std::cout << std::accumulate(nums.begin(), nums.end(), 0) <<'\n';
    std::cout << std::reduce(nums.begin(), nums.end()) <<'\n';
}
```

As you can see, `reduce` can default even the initial value to the default constructed value of the underlying type. This is dangerous because the default constructed type might not always be identity value.

Now let's see another example, where we can see a potential difference in the outputs:

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>
#include <execution>

int main()
{

    std::vector nums {32,16,8, 4, 2, 1};
    std::cout << std::accumulate(nums.begin()+1, nums.end(), *nums.begin(), std::minus<>{}) <<'\n';
    std::cout << std::reduce(nums.begin()+1, nums.end(),*nums.begin(), std::minus<>{}) <<'\n';
    std::cout << std::reduce(std::execution::seq, nums.begin()+1, nums.end(),*nums.begin(), std::minus<>{}) <<'\n';
    std::cout << std::reduce(std::execution::unseq, nums.begin()+1, nums.end(),*nums.begin(), std::minus<>{}) <<'\n';
    std::cout << "======\n";
    std::cout << std::reduce(std::execution::par, nums.begin()+1, nums.end(),*nums.begin(), [](int a, int b){
        std::cout << a << " " << b << '\n';
        return a-b;
    }) <<'\n';
}
/*
1
25
25
1
======
16 8
4 2
8 2
32 6
26 1
25
*/
```

With `accumulate` we get `1` as expected, but `reduce` produces different outputs except for with the `unsequenced_policy`. The last call, where we pass in a lambda doing an identical operation compared to `std::minus`, reveals the reason. Subtraction is not commutative and associative, therefore when the items are evaluated in a different order, you won't have the same result.

So when you make a decision between `accumulate` and `reduce`, you have to take into account that as well.

## `transform_reduce`

`std::transform_reduce` is also a recent addition to the STL, we can use it starting from C++17.

It has quite a few overloads. It takes either one range denoted by its begin and end iterators, or two ranges where the second range is defined only by its input iterator.

Then it takes an initial value that is non-defaultable, unlike for `std::reduce`.

The following parameter is a binary reduction operation that might be defaulted to addition (`std::plus<>()`) if the last parameter is also defaulted. The last parameter is either a unary or a binary transform operation (depending on the number of ranges passed in) and that can be defaulted to `std::multiplies` only for binary transformations.

But what would be the output of such an algorithm?

Let's start with the one range example. It'll take each element and apply the transform operation on them, then they will be reduced to one single value.

```cpp
#include <iostream>
#include <numeric>
#include <vector>


int main() {
    std::vector v {1, 2, 3, 4, 5};
    std::cout << std::transform_reduce(v.begin(), v.end(), 0,
                   [](int l, int r) {return l+r;},
                   [](int i) {return i*i;}) 
              << '\n';
}
/*
55
*/
```

In this example, we square each element and then they are summed up.

Now let's have an example for the double range version.

```cpp
#include <iostream>
#include <numeric>
#include <vector>


int main() {
    std::vector v {1, 2, 3, 4, 5};
    std::vector v2 {10, 20, 30, 40, 50};
    std::cout << std::transform_reduce(v.begin(), v.end(), v2.begin(), 0,
           [](int l, int r) {return l+r;},
           [](int f, int s) {return f*s;}) 
              << '\n';
}
/*
550
*/
```
In this other example, we passed in also `v2` and the second lambda that includes the transformation takes two parameters, one from both ranges. We take the product of the items and sum up these products.

Let me share three thoughts on `transform_reduce`.

First, like for `std::reduce`, you have to keep in mind that if the reduce or transform operations are not associative and commutative, the results are non-deterministic.

Second, I find it strange that while the algorithm is called `transform_reduce`, first you pass in the reduction algorithm and then the transformation. I think the name is good because first the transformation is applied, then the reduction, but it should take the two operations in a reversed order.

Third, I said that first the transformation is applied and then the reduction. It's only logically true, but the implementation is more optimal. Imagine, if first all the transformations are applied, then each transformed value has to be stored. Instead, whenever there are two values available to be reduced, reduction happens so that fewer values have to be stored.

You can see this if you add some print statements into the transformation and reduction operations.

```cpp
#include <iostream>
#include <numeric>
#include <vector>


int main() {
    std::vector v {1, 2, 3, 4, 5};
    std::vector v2 {10, 20, 30, 40, 50};
    std::cout << std::transform_reduce(v.begin(), v.end(), v2.begin(), 0,
           [](int l, int r) {
               std::cout << "reduce\n";
               return l+r;
           },
           [](int f, int s) {
               std::cout << "transform\n";
               return f*s;
           }) 
              << '\n';
}
/*
transform
transform
reduce
transform
transform
reduce
reduce
reduce
transform
reduce
550
*/
```

Instead of storing `n` temporary results, the algorithm only needs to track 3 values! Two for the transformations and 1 for the reduction.

## Conclusion

This time, we learnt about three algorithms from the `<numeric>` header. `accumulate`, `reduce` and `transform_reduce` all help us to reduce a range of items into one single value. Using them can simplify your codebase and introduce more constness.

Next time we'll continue with iota another 3 functions from the same header.

Stay tuned!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
