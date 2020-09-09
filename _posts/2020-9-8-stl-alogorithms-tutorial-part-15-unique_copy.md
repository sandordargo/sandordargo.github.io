---
layout: post
title: "The big STL Algorithms tutorial: modifying sequence operations - how to get distinct elements"
date: 2020-9-8
category: dev
tags: [cpp, tutorial, stl, algorithms]
excerpt_separator: <!--more-->
---
In this next part of [the big STL algorithm tutorial](http://sandordargo.com/blog/2019/01/30/stl-algos-intro), we will discover the 2 modifying sequence algorithms that will help you get unique elements of a container:
<!--more-->

* `unique`
* `unique_copy`

Let's get started!

## `unique`

`unique` - and as a matter of fact `unique_copy` - could have been implemented as two algorithms, just like `remove` and `remove_if` are two different algorithms.

Consistency is not the strongest feature of the `<algortihms>` header.

In this case, we simply have two separate overloaded signatures, but let's go on the goal of this algorithm.

`unique` will remove all the duplicated elements from a container. But only if they are consecutive. In case, you have two identical elements that are not placed next to each other, both supposed to be kept. But we are going to check that.

The return value is the same in both cases, it points at the new `end()` of the container after the duplicate ones were moved past the new end.

[In the first example](http://coliru.stacked-crooked.com/a/106b2cfee74de5e7), we'll use the simpler signature where we only pass in an input range defined by the usual two iterators pointing in the beginning and the end of the range.


```cpp
#include <algorithm>
#include <iostream>
#include <vector>


int main()
{
    std::vector<int> numbers{9, 1, 3, 3, 3, 5, 1, 6, 1};
    std::cout << "Original values: " << std::endl;
    std::for_each(numbers.begin(), numbers.end(), [](auto i) {std::cout << i << " ";});
    std::cout << std::endl;
    std::cout << std::endl;
    
    std::cout << "size: " << numbers.size() << ", capacity: " << numbers.capacity() << std::endl;
    auto oldEnd = numbers.end();
    auto newEnd = std::unique(numbers.begin(), numbers.end());
    std::cout << "same values are only removed if they are next to each other:" << std::endl;
    std::for_each(numbers.begin(), newEnd, [](auto i) {std::cout << i << " ";});
    std::cout << std::endl;
    std::cout << std::endl;
    
    std::cout << std::boolalpha << "oldEnd == newEnd? :" << (oldEnd == newEnd) << std::endl;
    std::cout << "In fact, the end hasn't changed. oldEnd == numbers.end(): " << (oldEnd == numbers.end()) << std::endl;
    std::cout << "number of elements removed: " << std::distance(newEnd, oldEnd) << std::endl;
    std::cout << "Though if you use the end, stranfe results are there..." << std::endl;
    std::for_each(numbers.begin(), oldEnd, [](auto i) {std::cout << i << " ";});
    std::cout << std::endl;
    std::cout << std::endl;
    
    std::cout << "size: " << numbers.size() << ", capacity: " << numbers.capacity() << ", these values haven't changed" << std::endl;
    numbers.erase(newEnd, oldEnd);
    numbers.shrink_to_fit();
    std::cout << "size: " << numbers.size() << ", capacity: " << numbers.capacity() << ", we should erase what is between the return value of unique() and the old end" << std::endl;
}
```

An interesting fact you might notice is that although the end of the vector has not `changed numbers.end()` is the same before and after calling `std::unique()`, what we have between the returned iterator and the (original) end has become meaningless. We could also say it is dangerous to use.

In fact, this makes perfect sense if we remind ourselves of how the STL is designed. Algorithms do not operate on collections, but on iterators. `std::unique` moves elements around each other, but it doesn't remove anything from the underlying collection. That's the exact same reason why you cannot delete elements with `std::remove`, but you have to use the [remove-erase idiom](https://en.wikipedia.org/wiki/Erase%E2%80%93remove_idiom).

So, I'd say that if we want to use this in-place `unique` algorithm, we should never use that container as a whole anymore. Either we take care of removing the elements beyond the returned iterator or we don't use it anymore.

If we want to reuse the original container, it's better to use `std::unique_copy`, but before, let's have a glance at the other version of `unique` where we can customize how elements are compared.

As an optional third argument, we can pass in a binary predicate. In more understandable English, you can pass in a function, function object, [lambda function](https://www.sandordargo.com/blog/2018/12/19/c++-lambda-expressions) taking two arguments (two elements next to each other in the collection) returning a boolean. The predicate should return true if the two elements are to be considered the same (not unique), false otherwise.

Here is a short example.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

struct Person {
    long id;
    std::string name;
    std::string phoneNumber;
};

int main()
{
    std::vector<Person> people { {1, "John D Smith", "555-1234"}, {1, "John David Smith", "784-1234"}, {2, "Adam Jones", "555-7894"} };
    auto it = std::unique(people.begin(), people.end(), [](auto lhs, auto rhs){ return lhs.id == rhs.id; });
    std::for_each(people.begin(), it, [](auto i) {std::cout << i.name << " " << std::endl;});
}
```

In the above example, we have different Person objects that might reference the very same physical being. So the names can differ a little bit, the phone numbers might be still different, but still want to consider two persons as the same. In this particular example, we can use the `id` for that, we make our comparison based on the `id` field.

Otherwise, there are no differences between the two different signatures.

* `unique_copy`

`std::unique_copy` works similarly to `std::unique`, but while the latter moves values around in the original container, the former copies the values to be kept into a target container.

As we learned for other algorithms, the target container is passed after the input, and while the input is denoted by a pair of operators, the target is only by a single one. This target collection has to be big enough to accommodate all the elements. The simplest way is to use a `back_inserter` for this purpose.

The return value is the same as for `std::unique`, an iterator pointing at right after the last copied element. Does this make sense? It does. First, it's consistent with `unique` and second, passing an inserter iterator as the target is not the only option. Maybe you created a big enough target collection for all the values and there will be some free capacity in the target. By free capacity in this case we mean zero constructed elements. In that case, it's useful to see where the copied values end.

Let's see an example of this case.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> numbers{9, 1, 3, 3, 3, 5, 1, 6, 1};
    std::vector<int> uniqueNumbers(numbers.size());
    
    auto it = std::unique_copy(numbers.begin(), numbers.end(), uniqueNumbers.begin());

    std::cout << "Content of uniqueNumbers: " << std::endl;
    std::for_each(uniqueNumbers.begin(), uniqueNumbers.end(), [](auto i) {std::cout << i << " ";});
    std::cout << std::endl << std::endl;
    
    std::cout << "Content of uniqueNumbers until the returned iterator: " << std::endl;
    std::for_each(uniqueNumbers.begin(), it, [](auto i) {std::cout << i << " ";});
    std::cout << std::endl;
}
```

It the above example we initialize the target vector with the size of the original one with contiguous duplicates. As such, after calling the `unique_copy` there will still zero-initialized elements in the target vector.

We should also see as a reminder that even though we called `unique_copy`, the copied elements are not necessarily unique, as only the neighbouring duplicates got removed - exactly as the contract of the `unique*` algorithms promises.

## Conclusion

Today, we learned about `unique` and `unique_copy`, algorithms that remove duplicated items from a range if the duplicated values are next to each other. That's their biggest catch - that duplicated elements should be next to each other, but it's well documented.

Next time we’ll learn about the algorithms that bring us some randomness. Stay tuned!