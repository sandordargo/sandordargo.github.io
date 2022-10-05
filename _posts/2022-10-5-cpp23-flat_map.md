---
layout: post
title: "C++23: flat_map, flat_set, et al."
date: 2022-10-5
category: dev
tags: [cpp, cpp23, flat_map, flat_set]
excerpt_separator: <!--more-->
---
[C++23 is introducing some more data structures](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0429r9.pdf), some more associative containers. We are going to get the *flat* versions of `map`/`set`/`multimap`/`multiset`:

* `flat_map`
* `flat_set`
* `flat_multimap`
* `flat_multiset`

These new types will work as drop-in replacements for their non-flat types. The goal of these new types is to provide different time and space complexities compared to the original containers. The non-flat versions' implementations are using balanced binary trees under the hood which more-or-less defines their main characteristics. C++11 introduced the unordered versions of these containers, and even though in most cases they should be preferred, they are often neglected. As the name suggests, unordered containers are not sorted.

Now we are getting sorted containers that are more effective in a big chunk of our use cases.

As mentioned, the original containers use binary search trees, the unordered versions use hashmaps. The flat ones use sequence containers. In fact, the flat versions are not even containers, they are container adapters.

>*[Container adapters](https://stackoverflow.com/questions/3873802/what-are-containers-adapters-c)* are interfaces created by limiting functionality in a pre-existing container and providing a different set of functionality. When you declare a container adapter, you have the option of specifying which sequence container should be the underlying container.

The underlying data structure is configurable through template parameters, but they must be sequence containers with random access iterators. Why do I speak about template parameters in plural? Because for a `flat_map`, you can use different containers for keys and values. From now on, I'll simply write about `flat_map`, but the observations are also valid for the other *flat* container adapters unless I explicitly write so.

## Specialties of a `flat_{map|set}`

So what are the specialities of these container adapters over the usual associate containers? What are those different time and space complexities that it has to provide? Let's start with enumerating the disadvantages, just to avoid thinking that a `flat_map` is too good to be true.

- Insertion and deletion are going to be slower
- On insertion and deletion iterators will become unstable
- The exception safety is weaker because moves and copies do happen, we don't just pass around pointers anymore
- And we cannot store non-copyable, non-movable types in *flat* structures

And now let's see what we get for this price. We'll get:

- Faster iteration
- Random access iterators instead of bidirectional iterators
- Smaller memory consumption
- Improved cache-friendliness due to contiguous memory layout

Actually, all this comes from the fact that under a `map` you'll find a balanced search tree, while under a `flat_map`, you'll find a sequence container, like a `std::vector`.

Let's have a closer look at some of these items above.

What happens when we insert into a `map` or when we delete from it and the tree has to be rebalanced? There will be no copies or moves, because each node in the `map` has a handle, a pointer to the data and only these handles will be moved around not the pointed objects.

Once we understand this, we also take it evident that the flat versions have a better memory footprint. When you work on a contiguous memory area when you deal with a sequence container, you have no handles, no metadata to store. You simply have to deal with the data. You also have to work more on them when you insert/delete as it's not enough to move around the handles anymore.

If you want to go deeper into the cache friendliness topic, I'd recommend watching the talk of [Bjorn Fahller at C++OnSea](https://www.youtube.com/watch?v=yyNWKHoDtMs). He explained that even in use cases when we might think that a linked list would serve us better than a sequence container, the latter might be a better choice. Even if it has to perform more work. More work is sometimes faster as the bottleneck is not the CPU anymore, but the cache. With sequence containers, the CPU has to access contiguous parts of memory, which is particularly cache-friendly.

## One more word on speed

As it was already mentioned, `flat_map` is an ordered map. If you give some inputs to it either at construction time or later, it will take care of keeping itself sorted. That requires some resources. But what if your data is already sorted? After all, C++ has the concept of not paying for what you don't use.

`flat_map` provides a solution for that!

The standard library provides a tag type called `sorted_unique_t` and the `flat_map` constructors have overloads taking that tag as a first parameter. Not only the constructors but also those overloads of the `insert` method that take multiple elements to insert. If you construct a `flat_map` from elements that are already sorted, or when you `insert` a range of items that are already sorted, don't forget to use the overloads with `sorted_unique_t`, because you can benefit from a significant performance gain.

But beware! If you use a `sorted_unique_t` overload with unsorted data, the behaviour is undefined! All bets are off!

## How it differs in its API

`flat_map` stores separately the keys and values, and the storage for those can be of different types. Because of that, there is a bunch of new constructors available. In addition, there are several new overloads for the constructor and the `insert` method taking the previously explained `sorted_unique_t` tag so that we don't resort to already sorted items.

The `extract` member method moves out both underlying storage containers. The `extract` function is overloaded with `&&` showing that the original object should be used anymore.

The other direction of moving data is also possible through the `replace` function. It takes containers as *rvalue* references and replaces the underlying containers with what was passed in.

## What if you want to use the new adaptors?

You'll still have to wait for the standard versions. At the moment of writing this article, [no compiler supports the flat container adapters](https://en.cppreference.com/w/cpp/compiler_support/23).

At the same time, the proposal didn't just come out of the blue. [`boost` has had this feature for quite a while](https://www.boost.org/doc/libs/1_80_0/doc/html/boost_container_header_reference.html#header.boost.container.flat_map_hpp), which served as a basis for the standardization. You can go and experiment with it, if not on your local, you go to [Compiler Explorer](https://godbolt.org/z/x3vY1f6rz) or [Coliru](http://coliru.stacked-crooked.com/a/84be54a775297036).

![Flat map in use]({{ site.baseurl }}/assets/img/flat_map.png)

## Conclusion

In C++23, we are going to get some exciting new container adaptors in the standard library! The `flat_{map|multimap|set|multiset}` containers offer different space and time complexities compared to their normal, original versions. It favors fast iteration, lookup and lower memory consumption at the expense of potentially slower writes. At the same time, it still offers the benefits of having sorted containers, unlike the `unordered_*` versions.

Although there is no compiler support for the time being, we can already learn about [both from the standard](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p0429r9.pdf) or by using [the boost versions](https://www.boost.org/doc/libs/1_80_0/doc/html/boost_container_header_reference.html#header.boost.container.flat_map_hpp)!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!