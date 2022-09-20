---
layout: post
title: "The 4 use of using in C++"
date: 2022-4-27
category: dev
tags: [cpp, bestpractices, using]
excerpt_separator: <!--more-->
---
When I write code I don't only want to write code that is correct. I also want to write code that is understandable, and maintainable. I want to deliver code that is easy to read not only for the compiler but also for other human beings. After all, humans will read my code more frequently than compilers.

I have been thinking what are the single most important keywords that help us write readable code. Probably this question doesn't make much sense, but `const` and `using` are definitely among these. We [already discussed `const` a lot](https://leanpub.com/cppconst/), this time it's time to see how using `using` can improve our code.

We are going to review the 4 ways we can use it:

- type aliasing with `using`
- introducing complete namespaces with `using`-directive
- introducing members of another namespace with `using`-declaration 
- importing class members with `using`-declaration 

## Aliasing

In old C++ we could use `typedef` to give another name, to give an alias for our types. Sometimes you might want to use it instead of [strong typing](https://www.sandordargo.com/blog/2020/10/14/strong-types-for-containers), just to benefit from more meaningful names like `int`.

```cpp
typedef int Horsepower;
```

Other times you want to shorten long types for easier usage:

```cpp
typedef std::vector<std::string>::iterator Iterator;
```

Since C++11 we can use `using` instead of `typedef` to achieve the same results.

```cpp
using Horsepower = int;
using Iterator = std::vector<std::string>::iterator;
```

Why would you use `using` over the good old `typedef`? Just read the above statements! Exactly like the [T.43 core guideline](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t43-prefer-using-over-typedef-for-defining-aliases) says, it's more readable! The keyword has a very clear meaining, then the name comes first and the old comes after a `=`. 

Besides, `using` can be used more generally. It can be used for template aliases where `typedef` would lead to a compilation error.

```cpp
template<typename T>
typedef std::map<int, T> MapT;      // error

template<typename T>
using MapT = std::map<int, T>;   // OK
```

## Using-directive in namespace and block scope

You've probably seen many code examples that right after the `#include` statements contain the line `using namespace std`. 

You've probably seen lots of such application code.

You've probably been told that it's bad.

It's particularly bad if you do in at the global scope in a header file, just like [SF.7 from the Core Guidelines says]:

```cpp
// bad.h
#include <iostream>
using namespace std; // bad 

// user.cpp
#include "bad.h"

// some function that happens to be named copy
bool copy(/*... some parameters ...*/);

int main()
{
  // now overloads local ::copy and std::copy, could be ambiguous
  copy(/*...*/);
}
```

In my opinion, even the fact that as a reader you cannot be sure where a function comes from is bad. This is a simplistic example, but when you use `using namespace` in a long `.cpp` file it's hard to keep track of where certain objects come from. I prefer having `using`-declarations instead and I also often introduce alias namespaces.

```cpp
//some.h
#include <other.h>

using mcs = mynamespace::component::subcomponent;

msc::Class foo();
//some.cpp
msc::Class foo() {
  using msc::AnotherClass;
  AnotherClass bar;
  // ...
}
```
As such, I don't pollute the global namespace. What you have to keep in mind is that when you introduce a `using`-directive into a header file at the global namespace header, you don't just mess things up in the current scope.

If you include the header file in other files, you'll also bring the inclusion of all those introduced symbols. If you introduce different header files with different global levels `using`-directives, the situation becomes even worse and the results of name lookup might depend on the order of inclusion.

To avoid all such problems, just follow [SF.7](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#sf7-dont-write-using-namespace-at-global-scope-in-a-header-file) and *don’t write using namespace at global scope in a header file*.


## Using-declaration in namespace and block scope

While the `using`-directive brings all the symbols of a namespace into the current scope, a `using`-declaration will bring only one item!

```cpp
using std::string;
string foo{"bar"};
```

In the above example, we just demonstrated how it works. After `using std::string`, we can refer to `std::string` without mentioning the `std` namespace.

It's still something not to overuse! A `using`-declaration may also expand an overload set. It's less dangerous to use it at a file scope than having a `using`-directive at the same scope, but risks still remain.

Starting from C++20, you can also introduce scoped enumerators into a namespace of block scope!

```cpp
enum class Color { red, green, blue };

class MyClass {
  using Color::red;
  Color c = red; // This is OK from C++20
};
```

In fact, it would also work with the old-style unscoped `enum`, but why would we do that?

## Importing base class members with `using`-declaration

With `using`-declaration, you can introduce base class members - including constructors - into derived classes. It's an easy way of exposing `protected` base class members as `public` in the derived class. It can be used both for functions and variables. 

```cpp
#include <iostream>

class Base {
 protected:
  void foo() {
    std::cout << "Base::foo()\n";
  }
 
 int m_i = 42; 
};


class Derived : public Base {
 public:
  using Base::foo;
  using Base::m_i;
};

int main() {
  Derived d;
  d.foo();
  std::cout << d.m_i << '\n';
}
/*
Base::foo()
42
*/
```

If you try to modify the above example and remove any of the two `using`-declarations, you'll see the compilation failing.

If the derived class already has a member with the same name, the compilation will not. The imported symbol from the base class will be hidden.

```cpp
#include <iostream>

class Base {
 protected:
  void foo() {
    std::cout << "Base::foo()\n";
  }
};


class Derived : public Base {
 public:
  using Base::foo;
  
  void foo() {
    std::cout << "Derived::foo()\n";
  }
};

int main() {
  Derived d;
  d.foo();
}
/*
Derived::foo()
*/
```

I find this technique really useful for unit testing. When you're writing a mock by hand, you often have to expose protected member functions from the base class, from the class that you are about to mock.

One way of doing it is forwarding the call.

Hopefully, the function's name in the mock is not changed, but I've seen it a couple of times. It really puts an extra burden on the maintainers when they realize that there is a better option.

```cpp
class ClassUnderTest {
 public:
  virtual void testMe() {
  }
  
  virtual void testMeToo() {
  }
};

class MockClassUnderTest : public ClassUnderTest {
 public:
  void testMe() override {
     ClassUnderTest::testMe(); 
  }
  
  void mockedTestMeToo() {
      ClassUnderTest::testMeToo(); 
  } 
};
```

Apart from tying a lot of unnecessary code, the problem above is that if the parameter list of `testMe` or `testMeToo` changes, you'll also have to update `MockClassUnderTest`. You can get rid of that need by using `using`.

```cpp
class MockClassUnderTest : public ClassUnderTest {
 public:
  using ClassUnderTest::testMe; 
  using ClassUnderTest::testMeToo;
};
```

Now we have less code and it's more understandable what's happening. As a bonus, even the maintenance is simplified. 


## Conclusion

In this article, we discussed the 4 different ways that we can use the `using` keyword. It's the right way to create aliases and import base class members in derived classes. At the same time, they can be also used to introduce whole namespaces into the current scope which can be particularly dangerous. Last but not least, `using` can also introduce single types to the current scope which is a less dangerous option than introducing whole namespaces, still, it should be used with care.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!