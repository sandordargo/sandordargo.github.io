---
layout: post
title: "C++26: a placeholder with no name"
date: 2025-1-8
category: dev
tags: [cpp, cpp26, placeholder, cleancode]
excerpt_separator: <!--more-->
---
Let's continue exploring C++26. In this post, we are going to discuss a core language feature proposed by Corentin Jabot and Micheal Park in [P2169R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2169r4.pdf). With the new standard we get a cool unnamed placeholder.

## Motivations

By convention, when we have a variable whose value we don't want to use or care about, we often name it `_`. The problem is that with higher warning levels (`-Wunused-variable`), our compilation might fail because `_` is unused.

```cpp
int foo() {
return 42;
}

auto _ = foo();
/*
error: unused variable '_' [-Werror,-Wunused-variable]
*/
```

To avoid this problem, we must mark it `[[maybe_unused]]`.

```cpp
int foo() {
return 42;
}

[[maybe_unused]] auto _ = foo(); // OK now
``` 

But what if we want to ignore several of them?

```cpp
int foo() { return 42; }
char bar() { return 'c'; }

[[maybe_unused]] auto _ = foo();
[[maybe_unused]] auto _ = bar();
```

In this case, even `[[maybe_unused]]` doesn't help because `_` is just a normal variable and is introduced twice. We can use `std::ignore`!

```cpp
#include <utility>
int foo() { return 42; }
char bar() { return 'c'; }

std::ignore = foo();
std::ignore = bar();
```

But even this solution won't work all the time. We cannot use it with structured bindings, which are probably the most often used place where `_` is used as variable names. And if you want to use `_` with various structured bindings in the same (nested) scope, you have to look for other solutions.

```cpp
std::map<int, std::tuple<std::string, std::string, std::string>> m{
        {1, {"one", "I", "foo"}}, {2, {"two", "II", "bar"}}, {3, {"three", "III", "baz"}}};

for(const auto& [_, v]: m) {
    const auto& [e, r, _] = v; // ERROR:  error: redefinition of '_'!
    std::cout << e << " in Roman format " << r << '\n';
}
```

The above piece of code will not compile, because we try to redefine `_`! In some cases, we can easily remove one of the ignored variables with the help of ranges!

```cpp
#include <ranges>

std::map<int, std::tuple<std::string, std::string, std::string>> m{
        {1, {"one", "I", "foo"}}, {2, {"two", "II", "bar"}}, {3, {"three", "III", "baz"}}};

for(const auto& v: m | std::views::values) {
    const auto& [e, r, _] = v;
    std::cout << e << " in Roman format " << r << '\n';
}
```

But this won't be possible all the time.

In addition, there are some variables such as locks and `scope_guard`s that are only used for their side effects. We don't want to store them in (not-so-)nicely named variables.

## The new solution

The solution that is brought to us with the acceptance of [P2169R4](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2169r4.pdf) is simple. We can use `_` as a placeholder in the same scope as many times as we want.

This solution is also similar to other languages' features or conventions and it's not unfamiliar at all from existing C++ practices. It already carries the meaning of _"I don't want to use this variable"_.

In more - but not too - technical terms, if we introduce `_` as a variable, a non-static class member, a lambda capture or in a structured binding, it will implicitly get the `[[maybe_unused]]` attribute. In addition, we can also redeclare it as many times as we want it.

On the other hand, if we try to use `_` in any expression, the program is ill-formed!

The new placeholder has certain _limits_, it cannot be used in template parameter lists or in requires clauses. For more details on the *why*s, refer to the [accepted proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2169r4.pdf)  

The authors also investigated the effects on existing code. The library that might come into most of our minds is *GMock*, where `_` is used to match any input passed to a function. There is little risk that this new feature will cause problems to *GMock* users as long as `using namespace testing;` appears before any declaration of `_`.

This change has been already implemented in GCC 14 and Clang 18.

## Conclusion

C++26 is going to bring us an unnamed placeholder, `_`, that can be redeclared as many times in the same scope as you need it. It is implicitely bears the `[[maybe_unused]]` attribute to avoid getting warnings on unused variables.

You can already try this feature with a fresh version of GCC and Clang. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
