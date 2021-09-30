---
layout: post
title: "Replace std::find_if in 80% of the cases"
date: 2021-9-29
category: dev
tags: [cpp, stl, algorithms]
excerpt_separator: <!--more-->
---
If you've been following the posts on this blog, you know that [I'm a big fan of using standard algorithms](https://www.sandordargo.com/blog/2020/05/13/loops-vs-algorithms) in any programming language, in particular in C++.
<!--more-->

They contain fewer bugs, in general, they have better performance and the standard algorithms are more expressive.

The last point on expressiveness is very important to me and after I saw a not-so-ideal example of using `std::find_if` in our codebase, I wanted to have a deeper look.

So I went through all our usages of `find_if` and I found that it was only used in a proper way in about 20% of all the cases.

This means that the Pareto principle applies here too. In 80% of the cases, `std::find_if` should not have been used.

But what else should have been used? And why?

I brought some simplified examples.

## Is there any such element?

Here is the first example:

```cpp
std::vector numbers {1, 3, 5, 7, 9};

return numbers.end()
           != std::find_if(numbers.begin(), numbers.end(), [](int number) { return number % 2 == 1; });
```

You might also see a close variant of the above example in your code base. Sometimes, there is a temporary variable to store the returned value of `find_if`, even if it's used only once:

```cpp
auto foundElement = std::find_if(numbers.begin(), numbers.end(), [](int number) { return number % 2 == 1; });

return numbers.end() != foundElement;
```

So what goes on here? 

First of all, what does `find_if` return? 

It returns an iterator to the first element of the searched range that satisfies the condition. If there is no such item, it returns an iterator pointing beyond the last element, in other words, to `end()`.

The function's return value in the above examples is a boolean, we simply compare whether `find_if` returns anything else than the `end()` of the examined collection. In other words, it checks whether the `find _if` returns an iterator to any of the elements in `numbers`. Yet in other words, we check if *any of* `numbers`'s elements satisfy the condition passed to `find_if`.

Alright, this last sentence should give us the hint. We can replace the above expression with `std::any_of`:

```cpp
return std::any_of(numbers.begin(), numbers.end(), [](int number) { return number % 2 == 1; });
```

What did we gain? We have a comparison less and potentially a temporary variable less as well. At the same time, our code is shorter, more expressive and we didn't even have to touch the lambda we wrote.

## There is no such element!

A bit different, yet similar example:

```cpp
auto aPotentialItem =
  std::find_if(items->begin(), item->end(), [&iName](const Item& anItem) {
    return inItem._name == iName;
  });
return (aPotentialItem == items->end()) ? nullptr : &(*aPotentialItem);
```

In this example, we don't use `!=` as a comparison between the `end()` of the collection and the return value of `find_if`, but `==` instead. This means that we check whether there is no element in a given range complying with our condition.

In other words, we check whether *none of* the elements satisfy our condition.

Yet, we cannot replace `find_if` in this example with `none_of`, given that we'd have to look up `aPotentialItem` anyway for the other case. (Thanks a lot for your comment [cbuchart](https://disqus.com/by/cbuchart/)!)

At the same time, `find_if` sometimes can be replaced with `none_of`, when you're only looking for the result of the comparison:

```cpp
std::vector numbers {1, 3, 5, 7, 9};

return std::find_if(numbers.begin(), numbers.end(), [](int number) {
    return number % 2 == 1;
  }) == numbers.end();
```

In the above example, we can simplify `std::find_if` with `std::none_of`.

```cpp

std::vector numbers {1, 3, 5, 7, 9, 8};

return std::none_of(numbers.begin(), numbers.end(), [](int number) {
    return number % 2 == 0;
  });
```

## All of the items are the same?

A slightly different case is when you use `find_if_not` and you compare if the returned iterator is the `end()` of the container.

```cpp
std::vector numbers {1, 3, 5, 7, 9};

if (std::find_if_not(numbers.begin(), numbers.end(), [](int i) { return i % 2 == 0;}) == numbers.end()) {
  // do something
}
```

In this case, you're looking for if there is no element matching the predicate.

We can replace it with `all_of` and the result will be much more readable:

```cpp
std::vector numbers {1, 3, 5, 7, 9};

if (std::all_of(numbers.begin(), numbers.end(), [](int i) { return i % 2 == 0;})) {
  // do something
}
```
## So what to do?

Based on the cases I saw, I came up with this rule of thumb for the cases when we don't want to dereference the returned iterator, but we only use it for comparison :
- if the result of `find_if` is compared using `!= end()`, use `any_of`
- if the result of `find_if` is compared using `== end()`, use `none_of`
- if the results of `find_if_not` is compared using `== end()` use `all_of`

Keep `find_if` only if you want to interact with the object pointed by the returned iterator. If the outcome is just a boolean, like in the above examples, you have an alternative still in the standard library.

## Conclusion

This was a short post on how to use the C++ standard algorithms in a better way than often it is used. `std::find_if` is often misused, probably because it's something more people know about than the alternatives.

In the vast majority of the cases I saw, it can be replaced either with `std::any_of` or `std::none_of`, sometimes even with `std::all_of` which improves the readability of the given piece of code a lot.

Go and check in your codebases how `std::find_if` is used. Do you have similar findings?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!