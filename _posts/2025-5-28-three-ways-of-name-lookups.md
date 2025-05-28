---
layout: post
title: "Three types of name lookups in C++"
date: 2025-5-28
category: dev
tags: [cpp, lookups, languagemechanics, namespaces]
excerpt_separator: <!--more-->
---
Let's get back to some basics this week and talk about name lookups in C++. In other words: when you refer to a symbol in your code, how does the compiler find it?

Essentially, we can differentiate between three kinds of lookups:

- Qualified name lookup  
- Unqualified name lookup  
- Argument-Dependent Lookup (ADL)

Let's explore them in that order.

## Qualified Name Lookup

The term *qualified* refers to symbols that are explicitly scoped using the `::` operator. In other words, these are names that appear to the right of a `::`, such as `x` in `a::b::x`.

Before the compiler can perform a qualified name lookup, it must first resolve the left-hand side of the `::` operator. This identifies the namespace or class being referenced.

Qualified name lookup is relatively simple: it only searches the explicitly named scope. It **does not** search enclosing or outer scopes.

Here's an example:

```cpp
#include <print>

namespace c {
int x = 3, z = 13;
}  // namespace c

namespace a {
int x = 1, w = 66;

namespace d {
int y = 42;
}  // namespace d

namespace b {
using namespace c;
int x = 2;
int y = d::y;
}  // namespace b

}  // namespace a

int main() {
    std::println("{}", a::b::x);  // 2
    // std::println("{}", a::b::w); // ERROR
    std::println("{}", a::b::z);  // 13
}
```

- `a::b::x` is `2` because it’s declared directly in `a::b`, which takes precedence over `c::x` imported via using namespace.
- `a::b::z` resolves to `13` from `c::z`.
- `a::b::w` fails to compile because qualified lookup does not consider outer scopes like `a::w`.


## Unqualified Name Lookups

Unqualified name lookup applies to names that are not qualified with `::`. For example, a plain `x` within a function is subject to unqualified lookup.

Look at the below example and the inline comments:

```cpp
int x = 2;
int func() {
    int x = 3;
    int y = x; // Unqualified name lookup finds local x = 3
    int z = ::x; // Qualified name lookup finds global x = 2
    return y * z; // 3 * 2 
}
```

Unqualified name lookup starts in the innermost scope and proceeds outward. Here's the order:

- Local scopes (from innermost to outer):

```cpp
int func() {
    int x = 3;
    int y;
    {
        // No x here; lookup continues to the enclosing function scope
        y = x;
    }
    return y;
}
```

- Class or struct scope:
```cpp
class Widget {
   public:
    int func() {
    	// No local x; lookup finds member x
        int y = x;
        return y;
    }
    int x = 3;
};
```

- Namespace scopes (innermost to outermost). In the below example, we can observe that within `a::b::f()` the lookup of `n` goes to outer and outer levels, from local scope to `a::b` and then to `a` where it can first find `n` declared.

```cpp
#include <print>

namespace a {

int n = 1;

namespace b {

int f() {
    return n;  // Finds a::n (outer namespace)
}

}  // namespace b
}  // namespace a
int main() {
    std::println("{}", a::b::f());  // 1
}
```
- Global scope, if still nothing found. Notice that in the below slightly modified example the declaration of `n` from namespace `a` to the global scope.

```cpp
#include <print>

int n = 1;

namespace a {

namespace b {

int f() {
    return n;  // Finds n in the global scope
}

}  // namespace b
}  // namespace a
int main() {
    std::println("{}", a::b::f());  // 1
}
```

- Namespaces introduced via using namespace directives. These are considered with the same priority as global scope. If both introduce the same symbol, it causes ambiguity. In the below example, both `g()` returns *7* and `h()` returns *2*, but if we uncomment the using directive in `h()`, then the compiler cannot decide between `::x` and `c::x` and the compilation fails due to this disambguouity.

```cpp
#include <print>

int x = 2;

namespace c {
int z = 7;
int x = 8;
}  // namespace c

namespace a {

namespace b {

int g() {
    using namespace c;
    return z;
}
int h() {
    
    // using namespace c;
    // If uncommented, causes ambiguity between ::x and c::x
    
    return x;
}
}  // namespace b
}  // namespace a
int main() {
    std::println("{}", a::b::g());  // 7
    std::println("{}", a::b::h());  // 2
}
```

## Argument-Dependent Lookups (ADL)

Argument-Dependent Lookup (also known as Koenig lookup) is a special kind of unqualified lookup that only applies to function calls. It considers the namespaces of function arguments when resolving the function name.

This allows function calls to “just work” without requiring fully qualified names, especially useful in generic programming and operator overloading.

Let's look into some examples.

```cpp
namespace math {

struct Vec {};

void normalize(Vec) {}

bool operator==(Vec, Vec) {
 return true; 
}

}  // namespace math

void foo() {
  math::Vec a, b;
  normalize(a);   // OK via ADL
  if (a == b) {}  // OK via ADL
}
```

Even though `normalize` and `operator==` are in the `math` namespace, they are found without qualification because the arguments are of type `math::Vec`. Thanks to ADL, function within the `math` namespace are brought into scope.

As mentioned, ADL only works for functions - not for types.

```cpp
namespace math {
struct Vec {};

struct Bar {
    Bar(Vec v): _v(v){} 

    Vec _v;
};
}  // namespace math

void foo() {
  math::Vec a;
  Bar c(a); // ERROR: ADL does not apply to class names like math::Bar
}
```

## Conclusion

Today we learned about an important aspect of C++, name lookup rules.
- *Qualified name lookup* only checks the explicitly named scope and ignores outer scopes.
- *Unqualified name lookup* starts from the innermost scope and walks outward, eventually checking global scope and any using namespace directives.
- *Argument-Dependent Lookup* brings in additional function candidates from the namespaces of the function’s arguments — but only for function calls.

Understanding these lookup strategies helps you predict and control symbol resolution in your code, and prevents obscure bugs caused by ambiguity or unexpected name hiding. It’s also key to mastering templates and overloads.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)