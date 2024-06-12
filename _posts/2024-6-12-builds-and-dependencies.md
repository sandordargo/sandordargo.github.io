---
layout: post
title: "Limit the number of library dependencies"
date: 2024-6-12
category: dev
tags: [cpp, builds, dependencies, architecture]
excerpt_separator: <!--more-->
---
First, let's discuss what a dependency is.

When we talk about dependencies, we can talk about different approaches. When hearing the word "dependency", many people first think about [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) and therefore they think about dependencies of their interfaces. A dependency is an object that the dependee depends on and to break this dependency, the dependee can expect the dependency to be passed through the interface.

```cpp
void withoutInjection(/* various params*/) {
	// some code
	Dependency d;
	d.doStuff(/* some params*/)
	// some more code
}

void withInjection(const Dependency& d, /* various params */) {
	// some code
	d.doStuff(/* some params*/)
	// some more code
}
```

But that's not the kind of dependency that we talk about today.

## Library dependencies

Let's assume that you have this interface:

```cpp
// foo.h
#include <bar/bar.h>

class Foo {
public:
	Foo(Bar b) : m_b(b) {}

	// ...

private:
	Bar m_b;
};
```

The question is where is `Bar` defined? Is it part of the same library? Does it come from somewhere else?

Let's assume that `Bar` is part of a different library. Now `Foo`'s library (`foo`) depends on `Bar`'s library (`bar`). What does that mean compile-wise?

Whether `bar` is shared or dynamic, at one point we need access to the header file `bar/bar.h`, so the include statement can be replaced with the contents of the header file. It doesn't need access to the `cpp` files though. It doesn't need the definitions of the symbols.

On the other hand, those undefined symbols must be resolved at link time. If `bar` is a *static library*, the compiled code will be copied over to your executable. If you use a *dynamic library*, the compiled code won't be copied over, but the linker will only add information that is needed during runtime to load the library into memory.

Now let's talk about the situation when your library depends on other libraries, but some other libraries also depend on your library and all links are dynamic.

But first, let's define two terms.

## The direction of dependencies

- a **downstream dependency** of your library is another library that yours depends on
- an **upstream dependency** of your library is another library that depends on yours

Your library has to be rebuilt every time your downstream dependencies change in a binary-incompatible way. As long as a library changes in a [binary-compatible way](https://community.kde.org/Policies/Binary_Compatibility_Issues_With_C%2B%2B), its upstream dependencies don't have to be rebuilt.

Without going through the do's and don'ts of binary compatibility, in a very simplified way, this means that as long as you don't change the `virtual` interface of a class or you don't modify the existing public non-`virtual` functions, you're binaries will stay compatible.

Now let's get back to dependencies and their publicness. In the CMake world, a dependency should be declared public (marked as `LINK_PUBLIC`) if it's used in your publicly shared header files. If a public header file includes a header from a library, then you depend publicly on that library. If the library you depend on has a binary incompatible change, your library will have to be rebuilt. That change, that need for rebuilt will propagate to your upstream dependencies. Therefore it is also called a transient dependency.

That's why it's important to limit your public dependencies. You want to avoid forcing your clients to rebuild more frequently than absolutely necessary.

## What to do to limit your dependencies?

This means that we have very good reasons to limit our publicly linked dependencies or even if we don't live in the CMake world and we do not distinguish between public and non-public links, we should strive to limit the number of header files we include in our publicly-exposed header files. Our build times and library sizes will go down if we follow these efforts.

But what can we do in order to limit our dependencies and the number of rebuilds?

First of all, we should make sure that we only include in our headers what is absolutely necessary. In other words, delete what you don't use and delete a bit more. Use [forward declarations](https://www.learncpp.com/cpp-tutorial/forward-declarations/) when you don't need the full definitions of a type and include the header only in the `.cpp` file.

Also, if many people depend on your library, try to stabilize your API as soon as possible! With that, people don't have to update their code forcing them to perform some manual work. At the same time, they don't have to rebuild just if they upgrade the version of your library. They will be thankful.

If you're really serious about these efforts, you might think about separating a library into two different parts. One will be a rarely-changing API and the other an implementation. The API will only contain data structures used in your public interface. They shouldn't contain any business logic and they should have only minimal dependencies on any other libraries. Besides, this component will only contain abstract interfaces.

The other library will depend on the API component and it will contain all the implementations of the abstract interfaces. Many of your users will only have to link against your API component as they won't be required to instantiate the direct types on their own. Think about users who get your types as return values of other libraries and they will only have to manipulate those returned values but don't have to construct their own instances.


```cpp
// somewhere in lib barapi

class Storage {
public:
	virtual ~Storage() = default;
	//...
	virtual bool addItem(Item) = 0;
};

// somewhere in lib foo

void baz(std::unique_ptr<Storage> s) {
	// You don't need the implementation of Storage::addItem here 
	auto res = s->addItem(Item{"whatnot", 42});
	// ..
}
```

The clients will have to depend on a much smaller library with hopefully fewer dependencies. Make sure that even in your implementation library you limit the number of exposed headers and the number of exposed dependencies. In other words, you don't have to publish all the header files of your library. If you create some classes in separate `.h`/`.cpp` files that are only used within your library and they *only* exist for better readability, don't publish them.

Even though, if the implementation libraries are only used in leaf components, meaning that they are not used by any other component, it does not matter that much how many of the dependencies are exposed. Still, it's better to be consistent and follow good habits all the time.

## Conclusion

In this article, we discussed why it's important to limit the number of dependencies your APIs have. The more libraries you link, the more time it will take. But also the more libraries you link, the more events can force you to recompile, which in general we want to avoid.

Limiting the number of dependencies can happen in different ways. You have to be very mindful in terms of what you include in your headers and whenever you can, you should rely on forward declaration. But you should go beyond and if build times matter a lot - because you have a big system that is often changed - make sure that dependencies are limited by design design. A great way to achieve that is by splitting a library into a separate API and an implementation library. The API which will be used by most users will only have a small amount of dependencies and the other one can do the heavy lifting.

What are your best practices to limit the number of dependencies your library has or exposes?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!