---
layout: post
title: "Struct initialization"
date: 2023-11-22
category: dev
tags: [cpp, struct, initialization, warnings]
excerpt_separator: <!--more-->
---
This article is inspired by a compiler warning that I fixed recently. The warning is `-Wmissing-field-initializers`. This flag will report you potentially uninitialized fields. Sometimes it's overly paranoid, but nevertheless, you'll end up with more readable code if you follow what the compiler tells you.

## The lack of parentheses matters

Let's start from the beginning with a simple `struct` that has two members. One member is of a fundamental type and the other is a class type. We'll see what happens if we instantiate the `struct` and try to print the values of the members.

```cpp
#include <iostream>
#include <string>

struct S {
    int m_num;
    std::string m_text;
};

int main() {
    S s;
    std::cout << "s.m_num: " << s.m_num << ", s.m_text: " << s.m_text << '\n';
}
/*
s.m_num: 1600677166, s.m_text: 
*/
```

Our small program will print some garbage value for `s.m_num` and nothing for `s.m_text`.

That's all right. When it comes to class members in case they are not explicitly initialized, their default constructor is called, but fundamental types are left uninitialized. The default constructor will not do anything for us.

That's what we were taught about member initialization. Or at least that's what I learned.

Let's change `main()` a little bit.

```cpp
int main() {
    S s();
    std::cout << "s.m_num: " << s.m_num << ", s.m_text: " << s.m_text << '\n';
}
```

Oh, now it doesn't compile. It's just [the most vexing parse](https://www.sandordargo.com/blog/2021/12/22/most-vexing-parse) and we just declared a function. [Read this](https://www.sandordargo.com/blog/2021/12/22/most-vexing-parse) if you're not familiar with this mistake and let's move on. This is what I intended to do:


```cpp
int main() {
    S s = S();
    std::cout << "s.m_num: " << s.m_num << ", s.m_text: " << s.m_text << '\n';
}
/*
s.m_num: 0, s.m_text: 
*/
```

Now `s.m_num` prints `0`. What has changed?

## Default- vs value-initialization vs zero-initialization

When we created `s` by `S s;` we're using [default-initialization](https://en.cppreference.com/w/cpp/language/default_initialization). In that case, all the members of fundamental types will be left uninitialised, while all the class-type members having a default constructor will be initialised through that.

> If a member is class-type and is not default constructible, then the compilation fails as `S::S()` gets implicitely deleted. (Check it on [godbolt](https://godbolt.org/z/PT5163f9e)!)

In the second case (`S s = S()`), we perform [value-initialization](https://en.cppreference.com/w/cpp/language/value_initialization). This means that all class-type members which are default-constructible will be default constructed and all the members of fundamental types are [zero-initialized](https://en.cppreference.com/w/cpp/language/zero_initialization). That last part (if simplified) means that the variable is initialized *"to the value obtained by explicitly converting the integer literal `0`*".

We'd also be in the second case if we replaced `S s = S();` by `S s{}`. Just by adding `= S()` or `{}` we made sure that our object is "properly" initialized.

> I'd personally argue that this is a great solution. I'd provide an explicit initializer for `m_num` to avoid any possibility for undefined behaviour. But for the sake of the example, I haven't do so.

Let's add a twist. Let's say that for some reason you're compelled to provide a default constructor. Let's not care about the why. So you end up with this code.

```cpp
#include <iostream>
#include <string>

struct S {
    S() {};

    // Maybe you have some other constructors as well

    int m_num;
    std::string m_text;
};

int main() {
    S s{};
    std::cout << "s.m_num: " << s.m_num << ", s.m_text: " << s.m_text << '\n';
}
/*
s.m_num: 1600677166, s.m_text: 
*/
```

Now `s.m_num` contains some garbage once again. Instead of *zero-initialization*, it gets *default initialization*. The reason is that `S` now has a *user-provided* default constructor.

Let's assume that you cannot remove the default constructor, but you still want to benefit from zero-initialization. You have to make sure that the default constructor is not *"user-provided"*. The rules say that a function is *user-provided* if it's *user-declared* and not explicitely defaulted or deleted on its first declaration.

This means for us that if we `= default` `S()` when it's declared, we'll benefit from zero-intialization once again.

```cpp
#include <iostream>
#include <string>

struct S {
    S() = default;

    // Maybe you have some other constructors as well

    int m_num;
    std::string m_text;
};

int main() {
    S s{};
    std::cout << "s.m_num: " << s.m_num << ", s.m_text: " << s.m_text << '\n';
}
/*
s.m_num: 0, s.m_text: 
*/
```

On the other hand, if for some reason [we defaulted `S` out-of-line](https://www.sandordargo.com/blog/2023/02/01/special-functions-and-binary-sizes), we'd fall back to default-initialization once again.

## A readable and maintainable initialization

Sadly, the compiler hasn't warned me during the whole quest and not even static code analyzers built-in to Compiler Explorer. Note that I compiled with `-Wall -Wextra -Werror -pedantic -std=c++20 -Wmissing-field-initializers -O3` (I wanted to make `-Wmissing-field-initializers` very explicit in the shared links). Probably `UBSAN` would have helped. But that's not available there, it's not turned on in many project and anyways in the situation that inspired me to write this article, there was no undefined behaviour. What I had was a compiler warning.

Now that we have seen when zero-initialization is applied, let's get back to that warning.

When there is absolutely no constructor - user-provided or even user-declared -, you can initialize the member of a struct (or the public members of a class for that matter) when you declare a new variable and you don't have to provide all the values!

The below code is valid:

```cpp
#include <iostream>
#include <string>

struct S {
    int m_num;
    std::string m_text;
};

int main() {
    S s{42};
    std::cout << "s.m_num: " << s.m_num << ", s.m_text: " << s.m_text << '\n';
}
/*
s.m_num: 42, s.m_text: 
*/
```

But when you turn on `-Wmissing-field-initializers` or `-Wextra` which includes `-Wmissing-field-initializers`, you get a compiler warning which hopefully is treated as an error. In this case, what the compiler just found is only about readability and therefore maintainability.

Even if there are other fundamental members and you provide a value only for one, the rest will be zero-initialized as you can see:

```cpp
#include <iostream>
#include <string>

struct S2 {
    // S2() = default;

    // S2(...) {}
    // Maybe you have some other constructors as well

    int m_num;
    int m_other_num;
};

int main() {
    S2 s{42};
    std::cout << "s.m_num: " << s.m_num << ", s.m_other_num: " << s.m_other_num << '\n';
}

/*
s.m_num: 42, s.m_other_num: 0
*/
```

While with a bit of C++ experience, it seems evident that the initialization of members will happen from top to bottom, I must agree that it's misleading and it's better to be explicit and change the declaration in a way that it passes all the initial values:

```cpp
S s{42, ""}; // assuming we went back to our original S
```

If we want to avoid that, we should start providing constructors to `S`. But we might end up having to provide 4 of them:
- the default constructor
- one that only takes the initial value of the first member
- another for the second member
- and finally one for all the members 

Quite some code and it can become worse if we have more members. Instead, we should just explicitly enumerate all the starting values.

There is another way to get rid of the warning as well as the need to pass in default values at construction time. The explicit in-class member initialization.

```cpp
#include <iostream>
#include <string>

struct S {
    int m_num = 0;
    std::string m_text{};
};

int main() {
    S s{42};
    std::cout << "s.m_num: " << s.m_num << ", s.m_text: " << s.m_text << '\n';
}
/*
s.m_num: 42, s.m_text: 
*/
```

If you're saying that well, there are no warnings, but this solution is still as misleading as the original solution, I have to say, you're partially right. When you look at the declaration of variable `s`, you don't know that there are other members too. But at least when you look up `S` you see directly that all members are explicitly initialized. You don't have to know the rules about default- vs value-initialization.

Let's make two remarks:
- changing `std::string m_text;` to `std::string m_text{};` did not change anything, because object members are initialized with their default constructors anyway in every situation.
- we got rid of the warning, that's already something...

If you want to make the code not just warning free, but more readable as well, you can use [designated initializers](https://en.cppreference.com/w/cpp/language/aggregate_initialization#Designated_initializers) to show explicitly what members you are initializing:

```cpp
S s{.m_num{42}};
```
This makes the code way more readable.

Use it together with in-class member initialization and you'll get a warning-free and readable solution.

## Conclusion

In this article, we looked into how member variables are initialized when they are not assigned to a value through a user-provided constructor. We saw the differences between zero- and value-initialization. Then we checked what `-Wmissing-field-initializers` is and how to get rid of the warnings while keeping our code both concise and readable. In-class member initialization and designated initializers for structs are great tools for such matters.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!