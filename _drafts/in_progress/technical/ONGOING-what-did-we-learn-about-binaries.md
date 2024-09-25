---
layout: post
title: "With no RTTI your code will be cleaner"
date: 2023-3-29
category: dev
tags: [cpp, binarysizes, noexcept, exceptions]
excerpt_separator: <!--more-->
---
I've dedicated a big part of the articles so far in 2023 to discuss how the size of C++ binaries are affected by coding techniques. As I'm ending this series, it's time to wrap up, it's time to summarize what we learned about binaries.

## Why all this matters at all?

When do we use C++? When we need to be efficient, right?! But there are many different types of efficiency. Most often, we talk about run-time efficiency, or in other words, run-time performance. Even run-time efficiency can be broken down. Do we care about sheer speed? Or maybe about the memory footprint of our software? But we can also talk about compile-time efficiency if we want to measure how fast our software can be [built from the source](https://www.youtube.com/watch?v=t2bCE0erTFI). And there is the third type of efficiceny, binary efficiency.

When we talk about binary efficiency, we want to measure, we want to make sure that our compiled binary doesn't occupy more space than necessary. Making sure that our binary is small is often not that important, but when it's serving mobile applications and our libraries have to be downloaded to install applications, or when they are used on small embedded devices, we must consider size as well. not just performance. 

## Compile-time programming is chic, but has its price

Nowadays it's fashionable to write code where more and more things are done at compile-time. Just check the program of a random C++ conference and you'll see something on this topic. At the same time, we must not forget that this often means that binary will be bigger.

[You can initialize a container compile-time? Great!](https://www.sandordargo.com/blog/2023/01/18/object-initialization-and-binary-sizes) Your code will be faster! And you binary bigger. I'm not saying that it's bad, but you have to understand your needs and constraints. Then think about the storage duration of your objects, think about what container you use. And also don't forget that the initial values of your members matter. Default values can be optimized better by the compiler than non-default ones.

## Prevent inline-ing if size matters

Speaking about defaults... Let's not forget about our special functions. How and where we should implement them? First of all don't forget the rule of 0 and the rule of 5. Try to avoid manually providing any special function. But if you must write your own, write them all.

Oh and of course, if you need to provide a virtual destructor, because you want to show that your class can be used as a base for others, use the `=default` implemenentation whenever you can.

The usual advice is to `=default` right at the declaration, in the header file. It's common sense and hard to argue against it. But when binary size matters a lot, you want to prevent the compiler from inlining functions, including the special ones. [You might go with [`default`ing implementations in the `.cpp` files](https://www.sandordargo.com/blog/2023/02/01/special-functions-and-binary-sizes
).


## Don't use virtuals if you don't need them

A class is considered a polymorphic class if it has at least one virtual function - in the class or in any of its base classes. Such as a virtual destructor.

For each such class, a *vtable* is generated that stores a function pointer for each virtual classes. It helps resolving which implementation should be called at runtime for a given object. Of course, this extra level of indirection decreases the run-time performance, but [it's often not that significant](https://johnnysswlab.com/the-true-price-of-virtual-functions-in-c/). But it also has quite some overhead in terms of your binary.

I'm not saying to avoid using virtual functions just because they will make your binaries bigger, but [don't make your destructors and functions virtual by default](https://www.sandordargo.com/blog/2023/02/08/binary-sizes-and-virtual), because they do have a cost. Even a simple virtual destructor will make the compiler generate a bulky vtable. On the other hand, once the vtable is generated, an additional virtual function doesn't add a lot - but it still does.

Be mindful about introducing polymorphic classes..

## Turn of RTTI if you can

*Run-Time Type Information* is by default turned on by your compiler. It lets us query the run-time, in other words, the dynamic type of your polymorphic objects in different ways. It can add a significant overhead to your binaries.

If you turn RTTI off, all the information that can be queried at run-time does not have to be stored in the binary. So you shave off both from the binary size and from the compilation time as that info doesn't have to be generated nor stored.

More than that, [disallowing RTTI arguably makes your code cleaner](https://www.sandordargo.com/blog/2023/04/26/without-rtti-your-code-will-be-cleaner). You don't have to force yourself to prefer virtual functions over dynamic casts, simply because dynamic_casts are not available with RTTI turned off.

If I started a new project, I'd probaly [turn RTTI off from the beginning](
https://www.sandordargo.com/blog/2023/03/01/binary-sizes-and-rtti).

## Exceptions are not zero-cost concepts

Exceptions are often considered as zero-cost abstractions when they are not thrown. In other words, you shouldn't worry about them, because an exception not-thrown doesn't cost anything. That's simply not true!

The C++ run-time has to be ready to handle exceptions and unwind your stack. The compiler will generate all the necessary information that is known at compile-time and store in the binary. Such as exception tables and stack unwinding information. Even if you do not throw a single exception.

[If you want to reduce the size of your binary, you can consider turning exceptions completely off](https://www.sandordargo.com/blog/2023/03/29/binary-size-and-exceptions). You won't be able to write `try`-`catch` blocks and you cannot `throw` anything, because the compiler would return an error seeing those keywords. But your binary will be smaller. If the libraries that you depend on throw an exception, `std::terminate` will be called.

If you cannot turn exception completely off, consider marking functions `noexcept` where makes sense. A function that is not supposed to throw should be marked as `noexcept`. The compiler will not generate the exception tables for those. If they still throw, `std::terminate` will be called.

## Binary sizes and passing functions to functions

Probably we all learned that the mis- and overuse of templates can result in a binary size bloat. Every time we instentiate a template with different arguments, more and more code is generated and compiled into your binary. It's also worth noting that often we forget that we use templates.

`std::function` comes in handy when we have to pass around functions to other functions. But [`std::function` also a template and a costly one](https://www.sandordargo.com/blog/2023/04/05/binary-size-and-templates). When we don't need all its features, we might consider just use a bare template instead probably constrained with the C++20 feautre of concepts.

Also, when it's enough, you might just go with good old (and not so much readable) function pointers.

## Use the right compiler settings

While how we code matters, using the right compiler and linker settings might matter even more! We can tell our compilers to optimize for space by passing the `-Os` flag, but [there are many other less well-known options](TODO: LINKME). We can use link type optimization, when an extra step is introduced to the compilation/linking process but finally the optimization can happen beyond translation units.

You can even ask the compiler to avoid inlining as much as possible, or to make sure that function bodies with indentical code are stored only once. Make sure you test size and effects before you decide to complicate your pipeline. You don't want to make it more complex without significant gains.

## Conclusion

Thanks for bearing with me over the last few months and for all your comments both under the articles or sent via private channels. We learned about many different ways to keep our binaries smaller. We saw that we code differently to achieve it, we can avoid introducing unnecessary `virtual`s or we can `default` special functions in the implementation file. We can also use compiler or linker settings, but we can also make some efforts in the intersection of these! Turning off exceptions or RTTI are definitely in the intersection of coding differently and changing compiler settings.

Feel free to comment under any article if you have further ideas!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
