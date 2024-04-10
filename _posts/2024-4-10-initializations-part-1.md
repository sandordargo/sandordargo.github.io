---
layout: post
title: "When and how variables are initialized? - Part 1"
date: 2024-4-10
category: dev
tags: [cpp, fundamental, initialization, copyinitialization]
excerpt_separator: <!--more-->
---
Recently, [I shared a story with you about a bug, about a manifestation of undefined behaviour a compiler upgrade uncovered](https://www.sandordargo.com/blog/2024/04/03/upgrading-the-compiler-and-undefined-behaviour). There we briefly looked into why a member was left uninitialized, but the topic of initialization deserves a deeper look.

Let's look first at C++ Reference. It matters where you check, but you might even find 8 different types of initialization.

I had a bit of a hard time understanding how all of them relate to each other. So I decided to go over each of them and just see if they mention/reference others. Here is what I found:

- *Default-initialization*: zero-initialization
- *Zero-initialization*: value-initialization, non-local initialization, constant-initialization
- *Value-initialization*: aggregate-initialization, list-initialization, default-initialization, zero-initialization, copy-initialization
- *Aggregate-initialization*: list-initialization, copy-initialization, direct-initialization
- *List-initialization*: direct-list-initialization, copy-list-initialization, aggregate-initialization, copy-initialization, direct-initialization
- *Direct-initialization*: list-initialization, aggregate-initialization, value-initialization, copy-initialization
- *Copy-initialization*: list-initialization, aggregate-initialization, direct-initialization
- *Constant-initialization*: default-initialization
- *Reference initialization*: list-initialization, copy-initialization, direct-initialization

That's plenty of connections. How can we categorize initializations? How can we start learning them?

I had two ideas in mind, a top-down and a bottom-up approach. The top-down in this case would mean that we start with those that reference others and as we need more details, we get deeper. With the bottom-up approach, we'd start with the details and once we understand them, we use them to build up higher-level concepts.

The problem is that the above list is full of cycles, only reference-initialization is not referenced by others.

So let's use another approach, which is based on C++ reference's.

## 3 initialization syntaxes 

C++ reference lists 4 distinct syntaxes to perform initialization, but two forms use the same rules, the same list-initialization-syntax.

The first listed syntax involves an `=` and an expression right after. That is the *copy-initialization* syntax and will invoke copy-initialization of the object on the left of the equation sign operator. *E.g. `T obj = foo();`*

The second syntax still involves braces. It doesn't matter whether those braces follow an `=`, the syntax is called the list-initialization syntax and they will invoke the rules of list initialization. The pair of braces can be either empty, contain an initializer list or a designated initializer list. *E.g. `T obj = {.foo = 42, .bar = 13}`*

There is another syntax where the variable name is followed by a pair of parentheses (`()`) with an initializer-list in between. This syntax is called direct-initialization syntax and therefore the object will be direct-initialized. *E.g. `T obj("foo", bar)`*

In this article, we are going to cover the first one from that list, *copy-initialization* and during the next weeks, we'll cover the rest.

## Copy-initialization

Previously, we saw that copy-initialization is invoked by the copy-initialization syntax such as `T obj = foo()`. But copy initialization will also happen when you pass an argument by value to a function or when you return a variable by value from a function. Let's not forget about throwing or catching an exception by value also uses copy-initialization! And we'll also see copy-initialization later, as part of aggregate initialization.

Let's see the effects of copy-initialization.


```cpp
class MyClass { /* */ };

MyClass foo() {
    return MyClass();
}

// ...
MyClass mc = foo();
``` 

In the above snippet, `mc` is copy-initialized from the return value of `foo()`. Since C++17 copy elision is guaranteed, so only one `MyClass` object is instantiated, there are no temporary objects created.

```cpp
class MyClass { /* */ };

// ...
MyClass mc;
MyClass mc2 = mc; 
```

This is also a copy-initialization and it would still be called a copy initialization if `mc` was moved (`MyClass mc2 = std::move(mc)`). In these cases, the copy and move constructors are invoked. Otherwise, when there are the same types on both sides or on the left side there is a derived type of the type to be initialized, the compiler would examine all the non-explicit constructors of the type on the left to find the best match by overload resolution and call it.

If the types are not the same on both sides, the left side is also not a derived type of the left side and at least one side is not a class type then the compiler will consider user-defined conversion sequences.

> A user-defined conversion consists of zero or one non-explicit single-argument converting constructor or non-explicit conversion function call.

The above definition means that explicit constructors are not considered user-defined conversions. Let's expand our MyClass to see the above in action.

```cpp
#include <string>

class MyClass {
public:
    MyClass() = default;
    explicit MyClass (const std::string& s): m_s(s){}
private:
    std::string m_s;
};

int main() {
    // MyClass mc = "not OK";
    using namespace std::string_literals;;
    // MyClass mc2 = "also not OK"s;
    MyClass mc = MyClass("OK"); // just to keep the copy-initialization syntax, but normally you simply call MyClass mc("OK");
}
```

We can see that we must explicitly invoke an explicit constructor, otherwise, it's not considered even if it wouldn't require a conversion from `const char*` to `std::string`. On the other hand, when we invoke the constructor, an implicit conversion is performed for the parameter.

If there are non-class types on both sides, the compiler will consider standard conversions if necessary.

```cpp
int num = true; // OK num is 1
const float val = num; // OK val is 1.0
```

## Conclusion

Last week, piggybacking a bug caused by undefined behaviour, we discussed why knowing about the different types of initializations in C++ is important. This week, we started a discovery of the different types and syntaxes of initialization and we discovered the details of copy-initialization.

Next week, we'll continue with list- and direct-initialization. Stay tuned.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
