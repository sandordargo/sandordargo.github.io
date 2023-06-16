---
layout: post
title: "Fold expressions in C++"
date: 2023-6-14
category: dev
tags: [cpp, cpp23, fold, reduce]
excerpt_separator: <!--more-->
---
In this article, we are going to talk about time expressions. First, we are going to take some time to define what they are in case you're not familiar with them. We'll do this using C++ syntax, but we must keep in time that the concept is not exclusive to our favourite language.

Then we are going to see what the standard library offers us in terms of algorithms and we'll also see how they evolved so far since they were introduced in C++17 and what new functions we are going to get with C++23.

## What are fold expressions?

When you have several values of the same type and you'd like to apply a binary function to each of these values and produce one single result then you essentially want to fold or reduce them. When you sum up several values and return the result, then you use might use a fold expression.

Using a fold expression lets you [not use an ugly loop](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms), it gives you a concise way to perform any binary operation and give back a single result.

We differentiate between left and right fold operations depending on from which direction we apply the operation.

When we start from the left, we talk about left folding and when we start from the right, we talk about right folding.

Using the example of summing up 5 numbers, let's represent the difference between left and right folding.

```cpp
operator+ 1 2 3 4
// equivalent to left folding
(((1 + 2) + 3) + 4)
// equivalent to right folding
(1 + (2 + (3 + 4)))
```

While the direction doesn't matter if an operation is associative, in other cases, it's important, such as for division.

A fold expression can be also called *parameter pack expansion*.

## The syntax in C++

There are two ways to approach fold expressions in C++. The first is through the syntax supporting writing fold expression and we can use fold expressions thanks to algorithms in the standard library.

In this section, let's talk about the syntax that was introduced in C++17.

The fold expression syntax (`...`) can be used only with [variadic templates???](https://www.sandordargo.com/blog/2023/05/03/variadic-functions-vs-variadic-templates), they cannot be directly used on standard containers.

```cpp
#include <vector>
#include <iostream>

template <typename ...Ts>
auto sum(Ts... numbers) {
    return (numbers + ...);
}

int main() {
    std::cout << sum(1, 2, 3, 4) << '\n';
    return 0;
}
```

In the above example, there are three items to notice.
- In the template parameter list there, there are three dots after the `typename` keyword, indicating that there can be more than one item of the same type
- In the function parameter list, there are also three dots, but after the actual type indicating also that there can be many of them

You could read about the details of those in [this article on variadic templates](https://www.sandordargo.com/blog/2023/05/03/variadic-functions-vs-variadic-templates).

The third thing to notice is the `(numbers + ...)` expression. That is called the fold expression and this particular one is a unary right fold. It will be expanded as `(1 + (2 + (3 + 4)))`. The parameter pack you have on the left will be "combined" by using the operator in the middle. If the fold expression would have started with the three dots (`(... + numbers)`) then we would talk about a unary left fold expression expanded as `(((1 + 2) + 3) + 4)`.

There are a couple of more things here.
- Why did I use the *unary* word?
- Can we pass in a standard container?

I used the unary keyword because the above is not the only type of fold expression that exists. Binary fold expressions are also a thing. The main difference is that with binary fold expressions, you define an initial value so that folding doesn't start by combining the first two elements of the variable pack, but by combining the first item with the initial value.

If we rewrite a unary fold to a binary fold by using the identity element of a given type/operation combination, then we can be sure to have a well-defined and meaningful behaviour even if the variable pack does not contain any items. Binary fold expression can also start from the left (`(0 + ... + numbers)`) or from the right (`(numbers + ... + 0)`).

For standard containers, we'll see how to use the standard algorithms. We could pass a container to the above `sum()` function but it wouldn't sum up the `vector`. The template would be expanded into a function that takes one `std::vector` as `Ts`. But it could effectively use the `operator+` on different vectors combining them into one `std::vector`.

## Fold algorithms

While fold expressions were only introduced in C++17, we already had fold algorithms earlier in the standard. The most well-known is `std::accumulate` which was added in C++11. `std::reduce` is another fold operation added by C++17.

With C++23, we get 6(!) other fold operations, namely:
- `std::ranges::fold_left`
- `std::ranges::fold_left_first`
- `std::ranges::fold_right`
- `std::ranges::fold_right_last`
- `std::ranges::fold_left_with_iter`
- `std::ranges::fold_left_first_with_iter`


### `std::accumulate`

`std::accumulate` takes 4 parameters. The first two define the range of elements it should fold. It also takes an initial value for the folding operation and as a last parameter, it takes a binary operation that it will apply on the running accumulated value (starting with the initial value) and the next item of the input range going from left to right. The binary operation defaults to `operator+`.

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

As mentioned above, `std::accumulate` performs a left fold. In order to apply a right fold, we have to pass in the input range in reverse order.

It's also worth noting that we can fold a range of items into a type that is different from the initial value. So for example we can accumulate a range of `int`s to a single `string`.

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main() {
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

If you want to avoid having an additional initial value and fold items starting right with the first item, just use the first item as an initial value and leave it out of the range you pass in.
```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main() {
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

### `std::reduce`

`std::reduce` was introduced later than `std::accumulate`, only in C++17. While `std::accumulate` is a left-fold operation, `std::reduce` doesn't guarantee any order, it can take pairs of items in just any order. As such, its binary operation can only operate on elements that are at least convertible into the value type of the range. In other words, you cannot fold a container of integers into a string as an integer is not implicitly convertible to a string.

On the other hand, `std::reduce` can be parallelized, and accordingly, it takes an optional `ExecutionPolicy` parameter before all the others.

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main() {
    std::vector nums {1, 2, 3, 4};
    std::cout << std::accumulate(nums.begin(), nums.end(), 0) <<'\n';
    std::cout << std::reduce(nums.begin(), nums.end()) <<'\n';
}
/*
10
10
*/
```

The initial value is missing from the parameters of `std::reduce` as it can default it to the default constructed value of the passed-in range's value type. This might be dangerous because the default constructed type might not always be the identity value.

As `reduce` might take items in just any order, the results might differ from the result of a left-fold operation. 

```cpp
#include <iostream>
#include <numeric>
#include <string>
#include <vector>
#include <execution>

int main() {
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

Now let's move on to algorithms introduced by C++23.

### `std::ranges::fold_left` and `std::ranges::fold_right`

The names of these algorithms are straightforward. They either perform a left- or a right-fold on the passed-in range. As they are part of the *ranges* library they can directly take a range (such as a container), but they also have overloads to take a range defined by its beginning and the end.

Both the initial value and the binary operation must be provided.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector nums {1, 2, 3, 4};
    std::cout << std::ranges::fold_left(nums, 0, std::plus{});
}
/*
10
*/
```

As we saw with `std::accumulate`, these new fold algorithms can also fold items of a range into a different type, such as `int`s to a `string`.

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <vector>

int main() {
    std::vector nums {1, 2, 3, 4};
    std::cout << std::ranges::fold_left(nums, "",
        [](std::string previous, int item) {
            if (!previous.empty()) {
                previous += "-";
            }
            previous += std::to_string(item);
            return previous;
        });
}
/*
1-2-3-4
*/
```

### `std::ranges::fold_left_first` and `std::ranges::fold_right_last`

It might happen that you don't have an identity element for the type you're folding. In those cases, you can start folding starting from the second item and use the first as the initial value. We already saw earlier how to do it with `std::accumulate` when we passed `numbers[0]` as the initial value and `numbers.begin()+1` as the beginning of the input range.

C++23 provides library support for those cases so you don't have to write error-prone code, you simply pass in the range, you skip the initial value and call the function that will take the first value from the range. There are 2 such functions so that we have support both for left and right folding.

```cpp
#include <iostream>
#include <algorithm>
#include <optional>
#include <string>
#include <vector>

int main() {
    std::vector<int> nums {8, 2, 2, 2};
    std::optional<int> r = std::ranges::fold_left_first(nums, std::divides{});
    if (r) {
        std::cout << r.value();
    } else {
        std::cout << "folding has no value!\n";
    }

    std::cout << '\n';
    std::vector<int> reversedNums(nums.rbegin(), nums.rend());
    std::optional<int> r2 = std::ranges::fold_right_last(reversedNums,
        [](int item, int previous) { return previous / item; });
    if (r2) {
        std::cout << r2.value();
    } else {
        std::cout << "folding has no value!\n";
    }
    return 0;
}
```

It's worth noting that in the second example we could not use `std::divides`, because when we fold from the right to the left, we have to also change the order of the parameters for a non-commutative operation.

Sadly, we cannot use these two functions when we want to fold a collection of elements into another type. We cannot fold a vector of integers into a `string`.

### `std::ranges::fold_left_with_iter` and `std::ranges::fold_left_first_with_iter`

The previous functions returned only the folded value. These two functions return two values. In the first position, they return the iterator pointing at the end of the range and in the second position there comes the result. The main difference between these two functions is that `std::ranges::fold_left_with_iter` takes an initial value and therefore it always returns a folded value, while `std::ranges::fold_left_first_with_iter` doesn't take an initializer, uses the first item as the initial value and only returns a `std::optional` as a value.

Why are these functions needed?

A range is not necessarily defined by two iterators. The first iterator denoting the beginning of a range is mandatory, but the second one can also be a sentinel, a condition defining when the iteration should finish.

As the fold functions must compute the end, why not return them just in case?

```cpp
#include <iostream>
#include <algorithm>
#include <ranges>

int main()
{
    auto oneDigitNums = std::views::iota(1) | std::views::take_while([](int i) { return i < 10;});
    std::cout << std::ranges::fold_left(oneDigitNums, 0, std::plus{}) << '\n';

    auto [end, result] = std::ranges::fold_left_with_iter(oneDigitNums, 0, std::plus{});
    std::cout << "Folding finished at " << *end << " with the value " << result << '\n';

    auto [end2, result2] = std::ranges::fold_left_first_with_iter(oneDigitNums, std::plus{});
    std::cout << "Folding finished at " << *end2 << " with the value " << result2.value_or(-1) << '\n';
}
/*
45
Folding finished at 10 with the value 45
Folding finished at 10 with the value 45
*/
```

## Conclusion

In this article, we reviewed how fold expressions and functions evolved in C++. The first fold function (`std::accumulate`) was introduced by C++11. Then C++17 also introduced `std::reduce` and language support as well in the form of fold expressions to simplify folding for variadic templates.

Last, but not least, we reviewed the 6 brand new fold functions C++23 is going to give us. We'll have the chance to 
- fold from both directions,
- use the first or last value as the initial value and 
- we can even get back the iterator pointing just past the last value that was taken into account from a range.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
