**Can virtual functions have default arguments?**

Yes, they can, but you sould not rely on them, as you might not get what you'd expect. 

While it's perfectly legal to use default argument initializers in virtual functions, there is a fair chance that the code will not be maintained over the time will lead to incorrect polymorphic code and unnecessary complexity in a class hierarchy.

Let's see an example:

```cpp
#include <iostream>

class Base {
public:
  virtual void fun(int p = 42) {
    std::cout << p << std::endl;
  }
};

class Derived : public Base {
public:
  void fun(int p = 13) override {
    std::cout << p << std::endl;
  }
};

class Derived2 : public Base {
public:
  void fun(int p) override {
    std::cout << p << std::endl;
  }
};
```

What would you expect from the following `main` function?

```cpp
int main() {
  Derived *d = new Derived;
  Base *b = d;
  b->fun();
  d->fun();
}
```

You might expect:
```cpp
42
13
```

If that's the case, congrats. If not, don't worry. It's not evident. `b` points to a derived class, yet `B`'s default value was used.

Now what about the following possible `main`?

```cpp
int main() {
  Base *b2 = new Base;
  Derived2 *d2 = new Derived2;
  b2->fun();
  d2->fun();
}
```

You might expect 42 twice in a row, but that's incorrect. The code won't compile. The overriding function doesn't _"inherit"_ the default value, so the empty `fun` call to Derived2 fails.

```cpp
/*
main.cpp: In function 'int main()':
main.cpp:28:11: error: no matching function for call to 'Derived2::fun()'
   28 |   d2->fun();
      |           ^
main.cpp:19:8: note: candidate: 'virtual void Derived2::fun(int)'
   19 |   void fun(int p) override {
      |        ^~~
main.cpp:19:8: note:   candidate expects 1 argument, 0 provided
*/
```

Now let's modify a bit our original example and we'll ignore Derived2.

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
    std::cout << "Derived::fun "<< p << std::endl;
  }
};

int main() {
  Derived *d = new Derived;
  Base *b = d;
  b->fun();
  d->fun();
}
```

What output do you expect now?

It is going to be:

```
Derived::fun 42
Derived::fun 13
```
The reason is that a virtual function is called on the dynamic type of the object, while the default parameter values are based on the static type. The dynamic type is `Derived` in both cases, but the static type is different, hence the different default values are used.

Given these differences compared to normal polymorphic behaviour, it's best to avoid any default arguments in virtual functions.

References:
- [GotW.ca: Herb Sutter](http://www.gotw.ca/publications/mill18.htm)
- [SonarSource](https://rules.sonarsource.com/cpp/RSPEC-3719)