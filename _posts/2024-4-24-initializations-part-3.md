---
layout: post
title: "When and how variables are initialized? - Part 3"
date: 2024-4-24
category: dev
tags: [cpp, fundamental, initialization, undefinedbehaviour]
excerpt_separator: <!--more-->
---
For the last couple of weeks, we've been learning about the different forms of initializations in C++. This quest is motivated by [a bug I discovered after a compiler update](https://www.sandordargo.com/blog/2024/04/03/upgrading-the-compiler-and-undefined-behaviour) in a code base that I maintained. While I was looking for an answer to what happened, I realized that there are not only default and value initializations in C++ but many more different forms.

Today, we are going to cover, default-, zero- and value-initialization.

## Default-initialization

This is performed when an object is constructed with no initializer sequence. 

```cpp
#include <iostream>
#include <string>

struct S {
    int m_num;
    std::string m_text;
};

int main() {
    S s1;
    std::cout << "s1.m_num: " << s1.m_num << ", s1.m_text: " << s1.m_text << '\n';

    S* s2 = new S;
    std::cout << "s2->m_num: " << s2->m_num << ", s2->m_text: " << s2->m_text << '\n';
}
```

As you can see, the initializer can be omitted both for variables with automatic and even with dynamic storage duration, but it also works for static and thread-local variables.

There is yet another possibility when a base class or a non-static data member is not mentioned in a constructor initializer list and that constructor is called. That might be a bit complex explanation, so let's look at some examples.

```cpp
struct S {
    S() {};
    std::string m_text;
};
```

In the above piece of code, a non-static data member (`m_text`) is not mentioned in a constructor initializer list. So when we construct an instance of `S`, `m_text` is default initialized.

Now let's cover the other possibility when the base class is not mentioned in the constructor initializer-list.

```cpp
struct Base {
    std::string m_base_text;
};

struct Derived : Base {
    Derived() {};
    int m_num;
    std::string m_text;
};
```

For class types, default-initialization means that their default constructor would be invoked.

For non-class types, the meaning of default-initialization depends on their storage duration. If their storage duration is automatic or dynamic, the value would be undefined. But if their storage duration is static or thread-local, then the variable gets zero-initialized. We'll discuss what zero-initialization means just in a bit.

It's worth noting that references and `const` scalar objects cannot be default-initialized.

> The following types are scalar: arithmetic, enumeration, pointer and pointer-to-member types and all their cv-qualified versions, as well as `std::nullptr_t`.

## Value-initialization

Now let's talk about value-initialization! First of all, let's see the syntax that would invoke it!

```cpp
#include <iostream>
#include <string>

struct S {
    S() : m_num{}, m_text() {}
    int m_num;
    std::string m_text;
};

int main() {
    S s1{};
    std::cout << "s1.m_num: " << s1.m_num << ", s1.m_text: " << s1.m_text << '\n';

    S* s2 = new S();
    std::cout << "s2->m_num: " << s2->m_num << ", s2->m_text: " << s2->m_text << '\n';
}
```

What we can see is that both local variables, `s1` and `s2` are now followed by an empty initializer sequence, either `()` or `{}` can invoke value-initialization.

Value-initialization is also invoked for non-static members when they are initialized in the member initializer list with an empty pair of parentheses or braces.

But what are the effects?

As always, the answer is *"it depends"*.

If it's a class type and it has an implicitly defined or `default`ed constructor, zero-initialization will happen. Unless the default constructor is non-trivial, then default-initialization will happen.

If it's a class type and there is no default constructor, or it's user-provided or even deleted, then we are in the case of default initialization.

If it's an array type, then for each element we get into a recursive loop as they are value initialized.

If it's a non-class type, the object is zero-initialized.

In all cases, if the empty pair of braces (`{}`) is used and `T` is an aggregate type, aggregate-initialization is performed instead of value-initialization.

If `T` is a class type that has no default constructor but has a constructor taking `std::initializer_list`, list-initialization is performed just as we saw last week.

## Zero-intialization

By now, we can clearly see that initialization is a complex topic in C++. If you're not convinced, the next phrase will probably change that: *zero-initialization does not have a dedicated syntax, but it might happen in certain situations*.

But first of all, what is zero-initialization?

It depends on the type.

If `T` is a union type then padding bits are initialized to zero bits and the first non-static named data member is zero-initialized.

For arrays, each element is zero-initialized and for reference types, nothing is done.

So far so good, but what does really happen?

If `T` is a non-union class type, then padding bits are initialized to zero bits, all non-static members are zero-initialized, all non-virtual base class subobjects are zero-initialized just like each virtual base class subobjects, unless `T` is also a base class subobject itself.

If `T` is scalar, then the object is initialized to the value obtained by explicitly converting the integer literal `0` to `T`. So `bool b` would be false, `int i` 0, `double d` 0.0 and a `char c` would contain the null character (`\0`).

In the end, we are getting back to the fact that every type deep down is composed of built-in types and zero initialization will boil down to zero-initialize those.

But when can it happen?

- For every named variable with static or thread-local storage duration that is not subject to constant initialization (another type of initialization!), before any other initialization. 
- When an array of any character type is initialized with a string literal that is too short, the remainder of the array is zero-initialized. So the rest will contain `\0` characters.
- As part of the value-initialization sequence for non-class types and for members of value-initialized class types that have no constructors, including value initialization of elements of aggregates for which no initializers are provided.

This last one might need an example.

```cpp
#include <iostream>

struct S {
    int a, b, c;
};

class C {
public:
    int a, b, c;
};
 
int main() {
    S a = S(); // the effect is same as: A a{}; or A a = {};
    std::cout << "a = {" << a.a << ' ' << a.b << ' ' << a.c << "}\n";

    S b; // the effect is same as: A a{}; or A a = {};
    std::cout << "b = {" << b.a << ' ' << b.b << ' ' << b.c << "}\n";

    C c;
    std::cout << "c = {" << c.a << ' ' << c.b << ' ' << c.c << "}\n";

    C d = C{};
    std::cout << "d = {" << d.a << ' ' << d.b << ' ' << d.c << "}\n";
}
/*
a = {0 0 0}
b = {-11906462 32551 1651076199}
c = {32551 0 0}
d = {0 0 0}
*/
```

We can see that neither `S` nor `C` has a constructor (only the default generated one). Yet, for `a` and `d` we see zero-initialization happening, while for `b` and `c` what we observe is undefined behaviour and some garbage values.


The difference is that zero-initialization might happen as part of value initialization, which is clearly the case for `a` and `d` due to the usage of initializer sequences. On the other hand, for `b` and `d`, no initializer sequence is used and as such we are in the case of default-initialization which leaves the uninitialized members in an undefined state.

## Conclusion

This week, we've discussed default-, value- and zero- initializations of objects. It's important to keep in mind that relying on default-initialization might leave data in an undefined state but with zero/value initialization that will never happen.

Next week, we'll put the final pieces together and talk about constant and reference initialization.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

