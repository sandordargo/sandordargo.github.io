---
title: "Why default values for virtual functions are dangerous?"
---
First and foremost, because they are probably not what you think they are!

The right overload of a `virtual` function is selected dynamically - at runtime - based on the dynamic type of the object. But the default value of a parameter is determined based on the static type of your object.

The first time I learnt about this problem was when I was analyzing the results of our static code analyzer a few years ago. The next time was when I actually ran into a bug.

## A real-life example of how defaults of virtuals can go wild

Let's have a look at a simplified version of a piece of code of what I ran into in a codebase I worked on.

```cpp
struct Base {
  virtual void foo(std::string s) {/* EMPTY IMPLEMENTATION */}
  virtual void foo(std::string s, bool b) = 0;
};

struct Derived : public Base {
  void foo(std::string s, bool b = false) override {
    if (b) {
      // do something
    } else {
      // do something else
    }
  } 
};
```

What can go wrong?

This example bleeds from many wounds. Sadly, it's real. The code is not readable enough as it's misleading. Let's forget that we should avoid using booleans as function parameters.

Having that `bool` parameter with its default value might suggest that it's actually used. But the instances of `Derived` were never used via a reference or pointer of that type. They are always passed around and used through a `Base` class pointer. The default value is never used and instead the empty implementation of `Base::foo(std::string)` is taken.

With the good intentions of removing the default value, I walked right into the trap. I made a small change and I ended up with this code:

```cpp
struct Base {
  virtual void foo(std::string s) {/* EMPTY IMPLEMENTATION */}
  virtual void foo(std::string s, bool b) = 0;
};

struct Derived : public Base {

  void foo(std::string s) override {
    foo(s, false);
  }

  void foo(std::string s, bool b) override {
    if (b) {
      // do something
    } else {
      // do something else
    }
  } 
};
```

What did I exactly do and why?

I removed the default value from `Derived::foo(string, bool b=false)` and introduced another overload `Derived::foo(string)` that calls the other the two-parameter overload with the previous default value. As such, the maintainers don't have to think about the default value, they see in the implementation how the call is forwarded.

So what's the problem?

The problem is that I didn't only change the code, but I also changed the behaviour.

When we have a `Derived` object that we access through a `Base` pointer and we only pass in a string and not the boolean, then before we invoked `Base::foo(string)`. With the changed version we call `Derived::foo(string)`. 

You can verify it with the following piece of code.

```cpp
#include <iostream>
#include <memory>
#include <string>

struct Base {
  virtual void foo(std::string s) {
      /* OTHERWISE EMPTY IMPLEMENTATION */
      std::cout << "Base::foo(string)\n";
  }
  virtual void foo(std::string s, bool b) = 0;
};

struct NewDerived : public Base {

  void foo(std::string s) override {
    foo(s, false);
  }

  void foo(std::string s, bool b) override {
    std::cout << "NewDerived::foo(string, bool b), b is "<<  b << '\n';
  } 
};

struct OldDerived : public Base {

  void foo(std::string s, bool b = false) override {
    std::cout << "OldDerived::foo(string, bool b), b is "<<  b << '\n';
  } 
};

int main () {
    std::cout << std::boolalpha;
    
    std::unique_ptr<Base> baseWithOld =  std::make_unique<OldDerived>();
    baseWithOld->foo("s");
    
    std::unique_ptr<Base> baseWithNew =  std::make_unique<NewDerived>();
    baseWithNew->foo("s");

}
/*
Base::foo(string)
NewDerived::foo(string, bool b), b is false
*/
```

Luckily the bug I introduced with this change was caught by component tests.

I was thinking about how to fix it in a way that doesn't break the behaviour but also increases the readability of the code leaving space for less confusion.

I decided to remove both the default value and the call forwarding.

```cpp
struct Base {
  virtual void foo(std::string s) {/* EMPTY IMPLEMENTATION */}
  virtual void foo(std::string s, bool b) = 0;
};

struct Derived : public Base {
  void foo(std::string s, bool b) override {
    if (b) {
      // do something
    } else {
      // do something else
    }
  } 
};
```

I was in a lucky situation. Even though I broke the API, it was an internal one. The owners and the users of the API were the same. While it could still have led to some massive updates, nobody was using the removed API.

Let's dig into the root cause.

## Default values are not "inherited"

While it's syntactically perfectly correct to use default argument initializers in virtual functions, there is a fair chance that the code will not be maintained over time. In parallel, the emerging chaos will lead to incorrect polymorphic code and unnecessary complexity in your class hierarchy.

Let's look at the following example:

```cpp
#include <iostream>

class Base {
public:
  virtual void fun(int p = 42) {
    std::cout << p << std::endl;
  }
};

class DerivedLeft : public Base {
public:
  void fun(int p = 13) override {
    std::cout << p << std::endl;
  }
};

class DerivedRight : public Base {
public:
  void fun(int p) override {
    std::cout << p << std::endl;
  }
};
```

What output would you expect from the following piece of code?

```cpp
int main() {
  DerivedLeft *d = new DerivedLeft;
  Base *b = d;
  b->fun();
  d->fun();
}
```

You might expect:
```
42
13
```

If that's the case, congratulations! Especially if it was not by chance. If you expected something else, don't worry. It's not evident and that's the problem with using default parameter values for `virtual` functions. 

`b` points to a derived class, yet `Base`'s default value was used.

What if you had the following piece of code?

```cpp
int main() {
  Base *b2 = new Base;
  DerivedRight *d2 = new DerivedRight;
  b2->fun();
  d2->fun();
}
```

You might expect `42` twice in a row, but that's incorrect. The code won't compile. The overriding function doesn't *"inherit"* the default value, so the empty `fun` call on `DerivedRight` fails.

```
main.cpp: In function 'int main()':
main.cpp:28:11: error: no matching function for call to 'DerivedRight::fun()'
   28 |   d2->fun();
      |           ^
main.cpp:19:8: note: candidate: 'virtual void DerivedRight::fun(int)'
   19 |   void fun(int p) override {
      |        ^~~
main.cpp:19:8: note:   candidate expects 1 argument, 0 provided
```

## Static vs dynamic types

In order to understand better what is happening behind the scenes, let's take a step back. Let's modify a bit our previous example and let's forget about `DerivedRight`.

```cpp
#include <iostream>

class Base {
public:
  virtual void fun(int p = 42) {
    std::cout << "Base::fun " << p << std::endl;
  }
};

class Derived : public Base {
public:
  void fun(int p = 13) override {
    std::cout << "Derived::fun " << p << std::endl;
  }
};

int main() {
  Derived *derived = new Derived;
  derived->fun();
  Base *base = derived;
  base->fun();
}
```

What output do you expect now?

It is going to be:

```
Derived::fun 13
Derived::fun 42
```

You might find it surprising that in both cases the derived version was called, yet with different default parameters.

The reason is that a `virtual` function is called on the dynamic (run-time) type of the object, while the default parameter values are based on the static (compile-time) type. The dynamic type is `Derived` in both cases, but the static type is different, hence the different default values are used.

## Is it really a problem? If so, what to do?

It's definitely not a syntactic issue, after all, it compiles.

It might even work according to your expectations.

The main problem is that it's misleading and easy to misunderstand. The root cause of that difficulty is that the selection of the right overload happens based on the run-time type while determining the parameter's default value is based on the compile-time type. 

It's better to keep the code simple and avoid having default parameters in `virtual` functions.

A way to achieve this is to use a protected so-called forwarding function:

```cpp
#include <iostream>

class Base {
public:
  void fun(int p = 42) {
    fun_impl(p);
  }
protected:
  virtual void fun_impl(int p) {
    std::cout << "Base::fun " << p << std::endl;
  }
};

class DerivedLeft : public Base {
protected:
  void fun_impl(int p) override {
    std::cout << "DerivedLeft::fun " << p << std::endl;
  }
};

class DerivedRight : public Base {
protected:
  void fun_impl(int p) override {
    std::cout << "DerivedRight::fun " << p << std::endl;
  }
};

int main() {
  DerivedLeft *d = new DerivedLeft;
  Base *b = d;
  DerivedRight *d2 = new DerivedRight;

  b->fun();
  d->fun();
  d2->fun();
}
```

In this case, only the `virtual` implementation without a default value is altered. The default value is part of the non-`virtual` interface and therefore the behaviour is exactly what one would expect:

```cpp
DerivedLeft::fun 42
DerivedLeft::fun 42
DerivedRight::fun 42
```
In case you really need a second default behaviour, you can create another non-`virtual` function in the corresponding derived class with the new default argument forward still to `fun_impl`, it will work.

```cpp
#include <iostream>

class Base {
public:
  void fun(int p = 42) {
    fun_impl(p);
  }
protected:
  virtual void fun_impl(int p) {
    std::cout << "Base::fun " << p << std::endl;
  }
};

class DerivedLeft : public Base {
public:
  void fun(int p = 13) {
      fun_impl(p);
  }
protected:
  void fun_impl(int p) override {
    std::cout << "DerivedLeft::fun " << p << std::endl;
  }
};

class DerivedRight : public Base {
protected:
  void fun_impl(int p) override {
    std::cout << "DerivedRight::fun " << p << std::endl;
  }
};

int main() {
  DerivedLeft *d = new DerivedLeft;
  Base *b = d;
  DerivedRight *d2 = new DerivedRight;

  b->fun();
  d->fun();
  d2->fun();
}
/*
DerivedLeft::fun 42
DerivedLeft::fun 13
DerivedRight::fun 42
*/
```

Though in this case, it's highly questionable whether one should use the same signatures in different classes in the same hierarchy without one overriding the other.

The best is to come up with a design that doesn't depend on such default values.

## Conclusion

In this article, we saw a real-life example of how using default values in `virtual` functions can lead to subtle bugs. Then we saw default value selection happens in C++ and how it is different from function overload selection. With the understanding of static vs dynamic or run-time vs compile-time types we came up with a solution that removes that eradicates the possibility of such bugs and keeps the code easy to understand.