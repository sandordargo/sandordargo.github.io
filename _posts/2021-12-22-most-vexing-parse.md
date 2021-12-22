---
layout: post
title: "What is the most vexing parse?"
date: 2021-12-22
category: dev
tags: [cpp, tutorial, beginners]
excerpt_separator: <!--more-->
---
The most vexing parse is a specific form of syntactic ambiguity resolution in the C++ programming language. The term was used by [Scott Meyers in Effective STL](https://www.sandordargo.com/blog/2020/08/26/effective-stl). It is formally defined in section 8.2 of the C++ language standard.
<!--more-->
It means that whatever that can be interpreted as a function declaration will be interpreted as a function declaration. It also means long minutes sitting in front of a failed compilation trying to figure out what the heck is going on.

Take the following example:

```cpp
std::string foo();
```

Probably this is the simplest form of the most vexing parse. The unsuspecting coder might think that we just declared a string called foo and called its default constructor, so initialized it as an empty string.

Then, for example, when we try to call `empty()` on it, and we have the following error message (with gcc):
```
main.cpp:18:5: error: request for member 'empty' in 'foo', which is of non-class type 'std::string()' {aka 'std::__cxx11::basic_string<char>()'
```
What happened is that the above line of code was interpreted as a function declaration. We just declared a function called foo, taking no parameters and returning a string. Whereas we only wanted to call the default constructor.

This can give a kind of headache to debug even if you know about the most vexing parse. Mostly because you see the compiler error on a different line, not where you declared your ~~varibale~~ function, but where you try to use it.

This can be fixed very easily. You don't need to use parentheses at all to declare a variable calling its default constructor. But since C++11, if you want you can also use the {}-initialization. Both examples are going to work just fine:

```cpp
std::string foo;
std::string bar{};
```

Now let's have a look at a bit more interesting example:

```cpp
#include <iostream>
#include <string>

struct MyInt {
    int m_i;
};

class Doubler {
public:
    Doubler(MyInt i) : my_int(i) {}
    
    int doubleIt() {
        return my_int.m_i*2;
    }
    
private:
    MyInt my_int;
    
};


int main() {
    int i=5;
    Doubler d(MyInt(i)); // most vexing parse here!
    std::cout << d.doubleIt() << std::endl;
}
```

You might think that we initialize a `Doubler` class with a `MyInt` taking `i` as a parameter. But instead what we just declared is a function called `d` that would return a `Doubler` and it would take a parameter called `i` of type `MyInt`.

Hence the error message:

```
main.cpp: In function 'int main()':
main.cpp:25:20: error: request for member 'doubleIt' in 'd', which is of non-class type 'Doubler(MyInt)'
   25 |     std::cout << d.doubleIt() << std::endl;
      |                    ^~~~~~~~
```

There are 3 ways to fix it:

- Declare the `MyInt` object outside of the call, on the previous line, but then it won't be a temporary anymore.

```cpp
MyInt mi(i);
Doubler d(mi); // no more most vexing parse here!
```

- Replace the any or both of the parentheses with the brace initialization. Both `Doubler d{MyInt(i)};` or `Doubler d(MyInt{i})` would work, just like `Doubler d{MyInt{i}}`. And this third one in consistent at least in how we call the constructors. The potential downside is that this only works since C++11.
```cpp
// these all work
Doubler d1{MyInt(i)};
Doubler d2(MyInt{i});   
Doubler d3{MyInt{i}};
```

- If you are using an older version of C++ than C++11, you can add an extra pair of parentheses around the argument that is meant to be sent to the constructor: `Doubler d((MyInt(i)))`. This also makes it impossible to parse it as a declaration.

```cpp
Doubler d((MyInt(i))); // This also works
```

## Conclusion

The most vexing parse is a form of a subtle bug that most probably you faced as a beginner C++ programmer and it made you lose quite a few minutes while trying to spot the error.

The most vexing parse says that anything that can be interpreted as a function declaration will be interpreted like that. Often, when you meant to declare a local variable, the compiler takes it as a function declaration and therefore it will fail on later lines.

You can fix it in different ways and if you use at least C++11, the `{}`-initialization should be the easiest and most readable way to do it.

If you liked this article, I invite you to [read this one](https://www.sandordargo.com/blog/2019/11/27/does-this-string-declaration-compile) too.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
