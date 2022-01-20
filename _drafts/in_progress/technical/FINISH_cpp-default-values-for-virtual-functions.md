## Listen to your scanners and keep things simple.

Some time ago I wrote about [virtual functions and default values](https://www.sandordargo.com/blog/2021/01/27/virtual-functions-with-default-arguments). In brief, they don't work well together, because the right overload of a virtual function is selected dynamically, at runtime, based on the dynamic type of the object. But the default value is determined based on the static type of your object.

I found a similar case in our code, it looked like pretty much like this, but of course in much bigger classes:

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

This example bleeds from many wounds, has problems with readability and it's misleading. In this case, let's ignore we should avoid using booleans as function arguments and we should either replace them with enums and/or we should create two separate functions.

Having that bool parameter with its default value might suggest that it's actually used. But the instances of `Derived` were never used through a reference or pointer of that type. They are always passed around ans used through a `Base` class pointer, so the default value is never used and instead the empty implementation of `Base::foo(std::string)` is taken.


With the good intentions of removing the default value, I walked right into this trap and I made a small change and I ended up with this code:

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

I removed the default value in `Derived::foo(string, bool)` and introduced another overload `Derived::foo(string)` that calls the other the two-parameter overload with the previous default value. So you don't have to think about default values, you see in the implementation how the call is forwarded.

So what's the problem? I didn't only change the code, but I also changed the behaviour.

When we have a `Derived` object that we access through a `Base` pointer and we only pass in a string and not the boolean, then before we invoked `Base::foo(string)`, but with the changed version we call `Derived::foo(string)`.

Luckily this change was caught by component tests.

I was thinking how to fix it in a way that doesn't break the behaviour, but also increases the readability of the code leaving space for less confusion.

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

You might claim that with this, I break the API and that's true, but even that can be fine in some cases. Like in this one:

- The API in internal, used only by one application that is maintained by the very same people as the API
- Breaking an internal API can be still cumbersome, but it turned out that nobody was using it!
- But even if someone used it, it could have been fixed easily by performing a find-n-replace.

What did I learn?

- Tests can save your life and you cannot simply rely on unit tests, each layer of the test pyramid is important
- When your static code analyzer complains that something is too complex, it probably is too complex.
- When something seems complex, think about twice what you do and add more tests!
