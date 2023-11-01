---
layout: post
title: "C++23: Removing garbage collection support"
date: 2023-11-1
category: dev
tags: [cpp, cpp23, constexpr, compiletime]
excerpt_separator: <!--more-->
---
If we go through the list of C++23 features, we can stumble upon the notion of garbage collection twice. Once among the language and once among the library features. Both entries refer to the same paper ([P2186R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2186r2.html)): garbage collection (*GC* in short) support is getting removed from C++. Just to make it clear, it's not getting deprecated, but it's getting completely removed. As it's an unimplemented and unsupported feature, removing it is a nice act of language cleanup. It also wouldn't be surprising if you never heard about GC in C++.

The first surprise this paper might give you is the fact that it talks about Garbage Collection and C++. When I was learning about C++ in comparison with Java a long long time ago, one advantage of C++ was deterministic memory management due to the lack of garbage collection.

As it turns out, in 2008 a [minimal support for garbage collection and reachability-based leak detection](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2670.htm) was added to C++0x. This paper was based on two earlier ones, you'll find the reference for those in the previously referenced one. Standard-conform garbage collectors should take into account *strict pointer safety*. In fact, there is no such garbage collectors in major standard libraries.

> [N2670](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2670.htm)) introduced the `std::pointer_safety` enumeration with three enumerators: `relaxed`, `preferred`, `strict`. When `std::get_pointer_safety()` is called, an implementation will return a value indicating how it treats pointers that are not safely derived (see below).
> - `pointer_safety::relaxed` is returned if non-safely-derived pointers will be treated the same as pointers that are safely derived for the duration of the program
> - `pointer_safety::preferred` is returned if non-safely-derived pointers will be treated also the same as safely-derived pointers, but at the same time, the implementation is allowed to hint that it's desirable to avoid dereferencing such pointers
> - `pointer_safety::strict` is returned if non-safely-derived pointers might be treated differently than pointers that are safely derived.

> A pointer value is a *safely-derived pointer* to a dynamic object only if it has pointer-to-object type and it is:
>
> - the value returned by a call to the C++ standard library implementation of ::operator new(std::size_t);
the result of taking the address of a subobject of an lvalue resulting from dereferencing a safely-derived pointer value;
> - the result of well-defined pointer arithmetic using a safely-derived pointer value;
> - the result of a well-defined pointer conversion of a safely-derived pointer value;
> - the result of a reinterpret_cast of a safely-derived pointer value;
> - the result of a reinterpret_cast of an integer representation of a safely-derived pointer value;
> - the value of an object whose value was copied from a traceable pointer object, where at the time of the copy the source object contained a copy of a safely-derived pointer value.

The standard lets implementations get away with basically any, even no-op implementations of GC and therefore it makes very little sense to have them in the standard. This doesn't mean though that there haven't been any successful garbage collectors for C++. The most often use case for GC implemented in C++ is having virtual machines written in C++ for other languages that are garbage collected. Different JavaScript VMs are like that and Lua as well.

But those garbage collectors also have their own set of requirements which are influenced much more by the language they help as their runtime than the C++ standard.

While [P2186R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2186r2.html) lists more concerns, I find it important to highlight two of them.

When you read above about `std::pointer_safety`, did you understand the difference between `relaxed` and `preferred`? If you did, please explain in the comments section. It's totally unclear what well-behaved programs should do when they are given this information. It only brings confusion with any benefits.

The other problem is the definition of *safely-derived pointers*. In order to get a *safely-derived pointer*, you must use the global `::operator new` and there are no other implementation-defined ways to get such pointers. This is a problem because people often want to use other `new`s or want to avoid their usage completely to create objects in local arrays while avoiding heap usage.

As a consequence, garbage collection has been removed from C++23 with a great majority as well as pointer safety. The following names are removed from `std::` 

- `declare_reachable`
- `undeclare_reachable`
- `declare_no_pointers`
- `undeclare_no_pointers`
- `get_pointer_safety`
- `pointer_safety`

## Conclusion

Garbage collection and related features are being removed from C++23. If you are surprised to learn that the C++ standard had support for GC, you are not alone. It was unimplemented, confusing and pretty useless hence it's removed. While there are existing garbage collectors, mostly for VMs written in C++ as a runtime for other languages, they are not impacted as they were not relying on the standard, exactly because of the reasons it's getting removed.

A bit of simplification to the standard never hurts.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!