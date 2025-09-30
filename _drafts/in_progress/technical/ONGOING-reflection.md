 


2.2 Why a single opaque reflection type?
https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#why-a-single-opaque-reflection-type


std::meta::info
https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#stdmetainfo

As of P2996R12
```cpp
namespace std {
  namespace meta {
    using info = decltype(^^::);
  }
}
```

Values and objects

scalar type, with meaningful equality and inequality without ordering

A reflection of an object (including variables) does not compare equally to a reflection of its value. Two values of different types never compare equally.

https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#consteval-only


https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#basic.fundamental-fundamental-types
https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#expr.reflect-the-reflection-operator






## Conclusion

This week, we started to learn about the basics of C++ reflection syntax. We learned that we can enter the reflection world with the `^^` operator and leave that domain with the help of splicers (`[: meta_info :]`).

We also briefly looked into what is between the two, `std::meta::info` being a single opaque type representing reflection information. Next week, we'll dig deeper into what's inside and how to work with it.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!






I can't help but highlight my favorite example, though: access_context::current(). This function gives back a representation of the class, namespace, or function from which it's called; the idea is that you can then use the returned object to ask "Is <member-of-a-class> accessible from this context?" And the P2996R10 revision barely said any more than that (check it out: https://lnkd.in/etj8gjcw). Compare that to the behemoth wording that we ended up with, which has led me to joke to colleagues that the most opaque core wording in the C++ standard now lives in the library part of the Standard 😂 https://lnkd.in/edEEXcrN

But that's the nature of the game! Determination, patience, and a little bit of creativity are the price of entry. Super proud of what my co-authors and I were







If you want to try anything, I propose that you use Compiler Explorer. At the moment of writing (July 2025), you have a branch of clang and EDG available.

: https://github.com/bloomberg/clang-p2996.

We discussed in part 1,what reflection is from a high level perspective. Now we are getting to the C++ related details.

Let's lay down some foundations.

We can talk about introspection and code injection. One is about looking at the current code, the other is about generating new one. Either needs some object to store information about the inspected or to be injected code.

That opaque type is `std::meta::info`. (which header)
> The designers went with one single type instead of type/variable/etc. So that they don't codify the current design. It would be difficult to change it. Think about how the meaning of variables changed in C++11 when it was extended to contain references.

TODO: show how it consists of

`std::meta::info` is the most important building block.

The question is how to get it.

You either construct it yourself - to be discussed later -, or you ask the compiler to retrieve you the reflection value with the help of the reflection operator: prefix `^^`, e.g. `auto reflection_value = ^^MyTpe;`

> I know what you think, `^^` looks horrible, right? And it's just the first element of the syntax of this whole new language. Why can't it be `^` I've heard people asking. There are two reasons, the second being more important. It's not to be confused with `^` from C# which means there ..., besides, in C++ `^` is the binary XOR operator and it's better to have a clear distinction.

Okay, so we know how to query and store the metainformation.

```cpp
#include <experimental/meta> // this will become simply <meta>

struct MyStruct {
    int a;
    float b;
};

int main() {
    [[maybe_unused]] std::meta::info meta_info = ^^MyStruct;
}
```

What can we possible do with it and how?

We can produce grammatical elements from the meta info with so called splicers.

For example, we can get back the type from `meta_info` in the above example in order to instantiate `MyStruct`. A splicer uses the syntax of `[: reflection_value :]`.

```cpp
#include <experimental/meta> // this will become simply <meta>

struct MyStruct {
    int a;
    float b;
};

int main() {
    [[maybe_unused]] typename [: ^^MyStruct :] instace_of_my_struct{.a = 1, .b = 2.0};
}
```

Note the keyword `typename`. We have to use it to show that... Also there is little interest in taking the reflection value of a type and use it directly.

> The typename prefix can be omitted in the same contexts as with dependent qualified names (i.e., in what the standard calls type-only contexts). For example:
```cpp
using MyType = [:sizeof(int)<sizeof(long)? ^^long : ^^int:];  // Implicit "typename" prefix.
```

It would be more interesting to use the `meta_info` variable we created earrlier.

But this doesn't compile.

Let's pause for a second and ask ourselves why.

The error message gives it away.

C++ reflection is static, aka compile time reflection. It works with *compile time*, but meta_info is just a normal variable. We must mark it with `constexpr`.

```cpp
#include <experimental/meta> // this will become simply <meta>

struct MyStruct {
    int a;
    float b;
};

int main() {
    [[maybe_unused]] constexpr std::meta::info meta_info = ^^MyStruct;
    [[maybe_unused]]typename [: meta_info :] instace_of_my_struct{.a = 1, .b = 2.0};
}
```

Okay, we can use `meta_info` with splicers. Where else?


a number of consteval metafunctions to work with reflections (including deriving other reflections), and

let's start with member_number? what is a metaf and and e can write one

```cpp
struct S { unsigned i:2, j:6; };

consteval auto member_number(int n) {
  if (n == 0) return ^^S::i;
  else if (n == 1) return ^^S::j;
}

int main() {
  S s{0, 0};
  s.[:member_number(1):] = 42;  // Same as: s.j = 42;
  s.[:member_number(5):] = 0;   // Error (member_number(5) is not a constant).
}
```

list a couple of functions

--- next week
enum to string
menttion that it comes from the paper

--- next week
binary layout


-- next week
3.10 Struct to Struct of Arrays


-- next weeks
get into the details of the language syntax details


-- next weeks
annotations agains attributes
