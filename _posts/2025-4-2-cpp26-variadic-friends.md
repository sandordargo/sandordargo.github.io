---
layout: post
title: "C++26: variadic friends"
date: 2025-4-2
category: dev
tags: [cpp, cpp26, variadic, friends]
excerpt_separator: <!--more-->
---
Up until C++23, functions, classes, function and class templates could be declared as friends. Starting from C++26, thanks to Jody Hagins' and Arthur O'Dwyer's proposal, [P2893R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2893r3.html), friendship can also be granted to a *pack of types*.

## Who does a pack of friends look like?

In earlier standards, we must declare friend class templates one by one, just like in the example below.

```cpp
template<class T=void,
         class U=void>
class Foo {
  friend T;
  friend U;
};
``` 

Note the default values for the template parameters. That makes it possible to accept a variadic number of template arguments with the maximum number of arguments fixed by how many parameters we added.

With C++26, that is simplified and you can directly use a pack of types with friendship granted.

```cpp
template<class... Ts>
class Foo {
  friend Ts...;
};
```

Now let's see two practical usages from the [proposal](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2893r3.html).

## The Passkey idiom

If you ever used `friends` you might have found it problematic that a friend declaration's scope is for the whole class. If you declare a friend within a class, it will have access to all the private parts of the class. But what if you wanted to limit the access to a couple of member functions?

One option is to move those to another class and grant friendship within that class. Though it's highly probable that from a design perspective that's not such a great idea.

Another - more elegant solution - is the passkey idiom.

```cpp
template<class T>
class Passkey {
  friend T;
  Passkey() = default; // note that this ctor is private!
};

class A;
class B;

class C {
  friend A;
private:
  void internal();
public:
  void intentionalA(Passkey<A>);
  void intentionalB(Passkey<B>);
};

class A {
  void m(C& c) {
    c.internal(); // OK
    c.intentionalA({}); // OK
    c.intentionalB({}); // Error, Passkey<B>'s constructor is inaccessible
  }
};

class B {
  void m(C& c) {
    c.intentionalB({}); // OK
  }
};
```

You can see that in order to call `C`'s public functions, you need to pass a `Passkey` object specialized with the right class. But only `T` can instantiate `Passkey<T>` due to its private constructor combined with friendship.

It seems a bit cumbersome that you need two different member functions if you want to give access to two different classes, `intentionalA` for `A` and `intentionalB` for `B`. The name could be the same, but you'd still need two function bodies (definitions) and you'd have to type out the `Passkey`s to avoid ambiguous function calls:

```cpp
template<class T>
class Passkey {
  friend T;
  Passkey() {}
};

class A;
class B;

class C {
  friend A;
private:
  void internal();
public:
  void intentional(Passkey<A>);
  void intentional(Passkey<B>);
};

class A {
  void m(C& c) {
    c.internal(); // OK
    c.intentional(Passkey<A>{}); // OK
    c.intentional(Passkey<B>{}); // Error, Passkey<B>'s ctor is inaccessible
  }
};

class B {
  void m(C& c) {
    c.intentional(Passkey<B>{}); // OK
  }
};
```

With variadic friends, this code can be simplified. We have to make `Passkey` accept a variadic number of arguments and make them all friends. Then `C::intentional` can accept any allowed specialization.

```cpp
template<class... T>
class Passkey {
  friend T...;
  Passkey() {}
};

class A;
class B;

class C {
  friend A;
private:
  void internal();
public:
  void intentional(Passkey<A, B>);
};

class A {
  void m(C& c) {
    c.internal(); // OK
    c.intentional({}); // OK
  }
};

class B {
  void m(C& c) {
    c.intentional({}); // OK
  }
};
```

It's worth noting that you don't need access to a class' constructor to use it as a template type parameter. `Passkey<A, B>{}` works fine for both `A` and `B`. With this change, you only need to define `intentional` once.

## CRTP and access to private parts of derived classes

If you are not familiar with the *Curiously Recurring Template Pattern (CRTP)*, [read this article](https://www.sandordargo.com/blog/2019/03/13/the-curiously-recurring-templatep-pattern-CRTP) before moving on.

When you use this pattern, it might happen that in the base class, you want to access private parts of the derived class. As you don't want to expose those private parts, you grant friendship to the base class.

```cpp
// The base class
template<class Crtp, class MsgT>
class Receiver { 
  void receive(MsgT) {
    static_cast<Crtp*>(this)->private_ += 1;
  }
};


// The derived class
template<class MsgT>
struct Dispatcher :
  public Receiver<Dispatcher<MsgT>, MsgT>
{
  //_private member exposed to the base class, called Receiver
  using Receiver<Dispatcher, MsgT>::Receiver;
  friend Receiver<Dispatcher, MsgT>;

private:
  int private_;
};
```

Fair enough. But what if there are several base classes? What if the `Dispatcher` takes a variadic number of message types, inheriting from a pack of base classes?

Today, there is no nice and simple solution, because only inheritance and the `using` directive support pack expansion, but not friendships. With C++26, the solution is simple, because friendship can be granted to a pack of friends.

```cpp
// The base class
template<class Crtp, class MsgT>
class Receiver {
  void receive(MsgT) {
    static_cast<Crtp*>(this)->private_ += 1;
  }
};

// The derived class
template<class... MsgTs> // notice the variadic template parameters
struct Dispatcher :
  public Receiver<Dispatcher<MsgTs...>, MsgTs>... // Inheritance supports pack expansion
{
  using Receiver<Dispatcher, MsgTs>::Receiver...;  // The using directive support pack expansion
  friend Receiver<Dispatcher, MsgTs>...; // Error pre-C++26, accepted from C++26

private:
  int private_;
};
```

## Conclusion

Thanks to the acceptance of [P2893R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2893r3.html), friendship can also be granted to a *pack of types*. While you won't need this feature every day, as you can see from the examples, in the right circumstances, it can significantly simplify your code.


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)