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

They contain fewer bugs, in general they have better performance and the standard algorithms are more expressive.

The last point on expressiveness is very important to me and after I saw a not-so-ideal example of using `std::find_if` in our codebase, I wanted to have a deeper look.

So I went through all our usages of `find_if` and I found that it was only used in a proper way in about 20% of all the cases.

This means that the Pareto principle applies here too. In 80% of the cases, `std::find_if` should not have been used.

But what else should have been used? And why?

I brought some examples, where I changed the variable names.

## Is there any such element?

Here is the first example:

```cpp
return myCollection.end()
           != std::find_if(myCollection.begin(), myCollection.end(), [](myType* anItem) {
                return anItem->m_code == ENUM_ELEMENT;
              });
```

You might also see a close variant of the above example in your code base. Sometimes, there is a temporary variable to store the returned value of `find_if`, even if it's used only once:

```cpp
auto foundElement = std::find_if(myCollection.begin(), myCollection.end(), [](myType* anItem) {
            return anItem->m_code == ENUM_ELEMENT;
          });
return myCollection.end() != foundElement;
```

So what goes on here? 

First of all, what does `find_if` return? 

It returns an iterator to the first element of the searched range that satisfies the condition. If there is no such item, it returns an iterator pointing beyond the last element, in other words, to `end()`.

The function's return value in the above examples is a boolean, we simply compare whether `find_if` returns anything else than the `end()` of the examined collection. In other words, it checks whether the `find _if` returns an iterator to any of the elements in `myCollection`. Yet in other words, we check if *any of* `myCollection`'s elements satisfy the condition passed to `find_if`.

Alright, this last sentence should give us the hint. We can replace the above expression with `std::any_of`:

```cpp
return std::any_of(myCollection.begin(), myCollection.end(), [](MyType* anItem) {
          return anItem->m_code == ENUM_ELEMENT;
        });

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

In this example, we don't use `!=` as a comparison between the `end()` of the collection and the return value of `find_if`, but `==` instead. Which means that we check whether there is no element in a given range complying to our condition.

In other words, we check whether *none of* the elements satisfy our condition.

You guessed it right, in such cases we can replace `find_if` with `none_of`:

```cpp
return (std::none_of(items->begin(), item->end(), [&iName](const Item& anItem) {
    return inItem._name == iName;
  })) ? nullptr : &(*aPotentialItem);
```

Our gains are similar. We still spare comparison and potentially a temporary as well. Besides, our code is more terse, more expressive and we didn't even have to touch the lambda we use.

## So what to do?

Based on the cases I saw, I came up with this rule rule of thumb:
- if the result of `find_if` is compared using `!= end()` use `any_of`
- if the result of `find_if` is compared using `== end()` use `none_of`

Keep `find_if` only if you want to interact with the object pointed by the returned iterator. If the outcome is just a boolean, like in the above example, you have an alternative still in the standard library.

## Conclusion

This was a short post on how to use the C++ standard algorithms in a better way than often it is used. `std::find_if` is often misused, probably because it's something more people know about than the alternatives.

In the vast majority of the cases I saw, it can be replaced either with `std::any_of` or `std::none_of` which improves the readability of the given piece of code a lot.

Go and check in your codebases how `std::find_if` is used. Do you have similar findings?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!