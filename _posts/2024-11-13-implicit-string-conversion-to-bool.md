---
layout: post
title: "Implicit string conversions to booleans"
date: 2024-11-13
category: dev
tags: [cpp, builds, staticlinking, dynamiclinking]
excerpt_separator: <!--more-->
---
From [C++ Brain Teasers by Anders Schau Knatten](https://www.sandordargo.com/blog/2024/10/16/cpp-brain-teasers), I learned about a compiler warning offered by Clang, called `-Wstring-conversion`. It emits a warning when a string literal is implicitly converted into a boolean.

You might even be surprised that it is even possible. And why would anyone do that?

Let's start with the first part by explaining why such an implicit conversion is possible. A string literal is an array of `const char`s. Arrays can be converted into pointers, something [we talked about last week when we discussed why `span`s are so useful](https://www.sandordargo.com/blog/2024/11/06/std-span). This is also called decay. Furthermore, pointers can be converted into booleans. That is how a string literal can be converted into a `bool`.

```cpp
static_assert(!!"" == true);
static_assert(static_cast<bool>("") == true);
```

What might be surprising though is that even an empty string literal is converted into `true`. The reason is that only a `nullptr` would be converted into `false`, but an empty string literal is an array of a size of one so it's not a `nullptr`. As a result, `""` converted to `true`. The possible confusion is that the one character in that array of one is the `\0` terminator. But this shouldn't really matter. You shouldn't use such shady implicit conversions.

We could end this article right here. But life is not ideal and I tried to turn on `-Wstring-conversion` in a production codebase where I found a few different cases of string literals conversions.

## Fail with a message

The most frequent usage of these conversions was tied to constantly failing assertions.

```cpp
assert(!"Invalid argument")
```

Considering that a string literal can be converted into `true`, the above assertion will always fail and the literal `"Invalid argument"` will appear in the logs.

The problem with the above assertion is that if you don't know about such implicit conversions, it's hard to understand this line of code. We can replace it in a very simple way.

```cpp
assert(false && "Invalid argument")
```

With the expanded version, the `false &&` communicates even to non-C++ developers that the assertion will always fail and it's easier the deduce that the second part is just an error message.

## Fail conditionally with a message

The second case is very similar to the previous one, just a bit more complicated. Before the negated literal, there is another condition combined with an `OR`.

```cpp
assert((arg != 42) || !"Invalid argument");
```

This combination means that if the condition fails, then the assertion will fire. Otherwise, if the condition is true, then the whole expression is true and we can move on.

To make it more readable, we should make the same breakdown in code that we just did in words. We should extract the condition and invert it, then we can make the same transformation on the negated literal that we did previously.

```cpp
  if (arg == 42) {
    assert(false && "Invalid argument");
  }
```

We end up with slightly more code, but it raises way less questions. Our code becomes more readable.

Remember, when we write production code, we don't write it to win in a code golf where every character counts. We simply want to make the future maintainers' lives easier while doing the right thing.

Now, let's have a look at a third case.

## Allow additional implicit conversions

The last case is plain horror. Let's have a look at it first:

```cpp
void foo(long l) {
}

void bar(const char* arg) {
    foo(arg); // this fails to compile
}

void baz(const char* arg) {
    foo(!!arg);
}
```

We have a function called `foo` that takes a `long`, but it can take any other integer-like parameter that we want to call with an argument of `const char*`. A `const char*` can be implicitly converted to a `bool` which can be implicitly converted to a `long`. But the compiler cannot invoke both on its own.

But if we force an implicit conversion to a bool with a negation, the compiler will take care of the other. Of course, due to the negation, we wouldn't get the value we wanted, so we have to negate the negated value.

It will always be the same result, unless `arg` is `nullptr`. But let's say, we don't want to eliminate this argument, we want to keep it. How should we do it without relying on the implicit conversion to bool?

Instead of a double negation, let's use a `static_cast` which is longer, but it's very clear on its intent

```cpp
void bar(const char* arg) {
    foo(static_cast<bool>(arg));
}
```

We still rely on an implicit conversion from `bool` to `long`, but that's obviously not reported by `-Wstring-conversion`. If it really bothered us, we could use another `static_cast`. Though I guess nobody really wants that.

## Conclusion

In this article, we learned about `-Wstring-conversion`, something I learned from C++ Brain Teasers by Anders Schau Knatten](https://www.sandordargo.com/blog/2024/10/16/cpp-brain-teasers). Clang offers this compiler warning which fires on implicit conversions from C-strings to `bool`s.

I presented you with three different scenarios where I saw that developers relied on this kind of implicit conversion in production codebases. Two of them are related to assertions and one is about a chain of conversions, ultimately to integral numbers. None of them are difficult to get rid of.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
