---
layout: post
title: "When and how variables are initialized? - Part 4"
date: 2024-5-1
category: dev
tags: [cpp, fundamental, initialization, undefinedbehaviour]
excerpt_separator: <!--more-->
---
For the last couple of weeks, we've been learning about the different forms of initializations in C++. As a reminder, we covered so many different forms of initializations. [In part 1](https://www.sandordargo.com/blog/2024/04/10/initializations-part-1), we covered the different initialization syntaxes and copy-initialization. [In part 2](https://www.sandordargo.com/blog/2024/04/17/initializations-part-2), we covered direct-, list- and aggregate-initalization. [In part 3](https://www.sandordargo.com/blog/2024/04/24/initializations-part-3), we discussed default-, value- and zero-initialization.

This week, in this final post on this topic, we are going get into the details of constant- and reference-initialization.

## Constant-initialization

Constant initialization sets the initial values of static variables to a compile-time constant. It is performed before all other initializations. The compiler can and will often perform constant-initialziation at compile-time and thus store the object representations in the binary's `.data` section. If the variable is not only constant-initialized, but it's also `const`, then it might go into a read-only section such as `.rodata`.

Two conditions have to be fulfilled to perform constant-initialization.

A variable either has to have an initializer or needs a default initializer that results in some initialization.

```cpp
#include <string>

static const std::string s; // default initializer is avaialable, constant-initialization happens
static int i; // no constant initialization, undefined value

int main() {
    return 0;
}
```

And its intialization has to be a constant experssion or it has to invoke a `constexpr` constructor.

```cpp
#include <string>

static const std::string s; // default initializer is avaialable, constant-initialization happens
static const int num = 42; // constant initialization happens, 
static constexpr int num2 = s.size(); // constant initialization happens, std::string::size is a constant expression

int main() {
    return 0;
}
```

## Reference-initialization 

Reference initialization binds a reference to an object. The referenced object must be the same type as the reference or implicitly convertable to it. Once initialized, a reference cannot be changed to refer to another object. You have pointers for that purpose.

Reference initialization might apply both to named lvalue and named rvalue references.

What is particular about reference intialization is that the reference can bind to a base class of the target or to any type it has a conversion operator for.

But what happens when we try to initialize a reference?

It depends on what we have on the right side. If list-initialization syntax is used, then the rules of list-initialization are applied.

Otherwise, we can differentiate between direct and indirect binding.

Let's look into direct bindings first.

First, let's see an example of storing a reference to the base class object even though the reference is initialized with the derived object. Direct binding applies if the initialized reference is an lvalue, the target is a non-bit-field lvalue and the type of the reference is reference-compatible with the type of the target.

```cpp
class Base {};

class Derived : public Base {};

int main() {
    Derived d;
    Base& b = d;
}
```

Now let's see an example of storing a reference to an integer as the result of a conversion operator. Direct binding also can apply if the reference to be initialized is an lvalue reference, the target is class type, but the reference to be initialized is not reference-related to the target's type. In this case, it's required that the target can be converted to an lvalue that type is reference-compatible with the type to be initialized.

```cpp
class Wrapper {
public:
    Wrapper(int num): m_num(num) {}
    int& operator()() {
        return m_num;
    }
private:
    int m_num;
};

int main() {
    Wrapper w{42};
    int& num = w();
    return 0;
}
```

Direct initialization can happen also happen for non-bit-field rvalues and function lvalues given that the target's type and the reference to be initialized are reference compatible. Also, if there is a conversion operator guaranteeing the reference compatibility following the same patterns that we saw for lvalue references, direct reference initialization is performed.

If direct binding is not available, indirect binding is considered. In these cases, the reference type and the target are not reference-related to each other. That means that for class types first user-defined conversions are considered. When such conversions are applied, the reference bounds to a temporary object that is copy initialized from the target. The reference is direct-iniatialized from the temporary object that is copy-initialized from the target. Phew.

```cpp
const std::string& rs = "abc"; // rs refers to temporary copy-initialized from char array
const double& rcd2 = 2;        // rcd2 refers to temporary with value 2.0
```

This leads to the question of lifetimes. The lifetime of the temporary will be extended until the lifetime of the reference. At the same time, the extension can only happen once, if the reference is passed on, the lifetime won't be extended a second time.

There are a couple of exceptions though. A temporary bound to a return value of a function in a `return` statement is not extended, you're going to have a dangling pointer.

```cpp
#include <iostream>

int& fun() {
    int a = 66;
    return a;
}


int main() {
    auto b = fun();
}
/*
warning: reference to local variable 'a' returned [-Wreturn-local-addr]

Program returned: 139
Program terminated with signal: SIGSEGV

*/
```

You have to deal with similar consequences if a function returns a temporary that bounds to a reference parameter. It cannot outlive the function call.

Then let's take the following exception from C++ Reference: *a temporary bound to a reference in the initializer used in a new-expression exists until the end of the full expression containing that new-expression, not as long as the initialized object. If the initialized object outlives the full expression, its reference member becomes a dangling reference.*

This is interesting to show. 


```cpp
#include <iostream>
#include <memory>
#include <utility>
 
struct S {
    int mi;
    const std::pair<int, int>& mp; // reference member
};

int main() {
    S a {1, {2, 3}}; 
    S* p = new S{1, {2, 3}};

    std::cout << a.mi << " " << a.mp.first << " " << a.mp.second << '\n';                         
    std::cout << p->mi << " " << p->mp.first << " " << p->mp.second << '\n';
    return 0;
}
```

`a` is correct, if `p` works that's by chance. The temporary references on `p`s initialization are not extended by definition, because that would be a second a extension. The first extension happens in the expression `new S{1, {2, 3}}`, the second would be the assignment. On GCC, this works without a problem, Clang on the other hand emits a warning about it.

## Conclusion

Today, we learned about the last two types of initalizations on our plate, constant- and reference-initialization. The most important takeaway is probably to remember that reference lifetime-extension happens only once.

This also marks the end of this mini-series on initializations. We've seen 9 different kinds of inizitalizations. We saw that even when we talk about one initialization, others are involved. For example when you value initialize an object, members without an explicit intializer will be zero-initialized. Copy-intialization can involve list- or direct-initialization, etc.

C++ has many nuances. In my opinion, the best we can do is to avoid relying on them and write code that is straithforward and works in all circumstances.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

