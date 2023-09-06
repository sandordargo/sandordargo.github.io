---
layout: post
title: "C++23: some changes related to templates"
date: 2023-9-6
category: dev
tags: [cpp, cpp23, constexpr, compiletime]
excerpt_separator: <!--more-->
---
I know the above title is a bit vague. As we move forward with the introduction of C++23 features, there are going to be some articles like that. At the same time, there are more than two features that are related to templates in C++23. But some of them were already presented, such as [`if consteval`](https://www.sandordargo.com/blog/2022/06/01/cpp23-if-consteval) or [the explicit object parameter (a.k.a. deducing this)](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23).

Today, we are going talk about something related to and needed by deducing this, and the other topic is going to be class template argument deduction (*CTAD*).

## CTAD for inherited constructors

[P2582R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2582r1.pdf) is about class template argument deduction (CTAD) from inherited constructors. If you check the paper and you don't speak standardese well (and I don't), it'll be difficult to understand what it is about as it only contains the proposed wording.

Luckily, it refers to another document ([P1021R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1021r6.html)), that contains the rationale behind this and many other changes.

From [P1021R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1021r6.html), we can learn that CTAD that was introduced in C++17 had and still has some limitations in its usability. Some were already fixed in C++20, but obviously, the inherited constructors use case.

Let's take the example from the paper to demonstrate this shortcoming.

Let's assume that we have these two classes (we talked about C++17 so far, but we are potentially using C++20 here if we uncomment the requires clause):

```cpp
#include <memory>
#include <concepts>
#include <iostream>

template <typename T>  /* requires std::invocable<T> */ struct CallObserver { 
  CallObserver(T &&) : t(std::forward<T>(t)) {}
  virtual void observeCall() { t(); }
  T t;
};
 
template <typename T> struct CallLogger : public CallObserver<T> { 
  using CallObserver<T>::CallObserver; 
  virtual void observeCall() override { 
    std::cout << "calling";
    CallObserver<T>::t();  }
};

int main()


{

   CallObserver observer([]() { /* ... */ }); // OK
   CallLogger logger([]() { /* ... */ });

}

```

In C++17 you can use `CallObserver` without passing any type as a template parameter and CTAD will just work fine.

```cpp
CallObserver observer([]() { /* ... */ }); // OK
```

CallLoger inherits the constructors of CallObserver, but still `CallLogger logger([]() { /* ... */ });` would fail as there is no viable constructor or deduction guide available.

According to [P2582R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2582r1.pdf), this is going to be fixed in C++23 and `CallLogger` will inherit the deduction guidelines too. At the moment, no compiler has implemented it yet.

Until then, we have to define an explicit deduction guideline if we want to make it work:

```cpp
template <typename T> CallLogger(T) -> CallLogger<T>;
```

Here is the full example:

```cpp
#include <memory>
#include <concepts>
#include <iostream>

template <typename T>  /* requires std::invocable<T> */ struct CallObserver { 
  CallObserver(T &&) : t(std::forward<T>(t)) {}
  virtual void observeCall() { t(); }
  T t;
};
 
template <typename T> struct CallLogger : public CallObserver<T> { 
  using CallObserver<T>::CallObserver; 
  virtual void observeCall() override { 
    std::cout << "calling";
    CallObserver<T>::t();  }
};


/*
In C++23 this will not needed anymore
*/ 
template <typename T> CallLogger(T) -> CallLogger<T>;

int main() {

   CallObserver observer([]() { /* ... */ }); // OK in C++17
   CallLogger logger([]() { /* ... */ }); // OK only with the explicit deduction guideline untill C++23
}
```

## `std::forward_like`

The implementation of [Deducing this](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23) used a hypothetical `std::forward_like<decltype(self)>(variable)` facility. (It was not referenced in the aforementioned article). [P2445R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2445r1.pdf) contains the necessary proposal for this utility.

`std::forward_like` is (going to be) part of the `<utility>` header. As `std::forward`, it is also a type cast that only influences the value category of an expression. It forwards the value category of an object expression based on the value category of the owning object expression.

If we talk about an owning object (`o`), member object (`m`) relationship, thus when `o.m` is valid, it would be spelt as `std::forward<decltype(o)>(o).m` up until C++20.

But - for example with members of lambda closures - `o.m` is not always a valid expression and that's when this new facility comes in handy.

The authors considered three different models for the implementation. According to the language model, the behaviour of `forward_like` would have followed what `std::forward<decltype(Owner)>(o).m` does. According to the tuple, we would have got what `std::get<0>(tuple<Member> Owner)` does. However, the authors decided to go with a so-called `merge` model in which the `const` qualifiers of the owner and the member are merged and the value category of the Owner is adopted.

There are some nice tables representing the common parts and the differences of the different approaches in the [8. section of the paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2445r1.pdf).

There are 4 different use cases mentioned in the proposal for `std::forward_like`. The first one is a lambda that forwards its capture. Notice that it also uses deducing this, even though here we don't talk about a recursive lambda, only the value type of the enclosing lambda is needed.

```cpp
auto callback = [m=get_message(), &scheduler](this auto &&self) -> bool {
  return scheduler.submit(std::forward_like<decltype(self)>(m));
};
callback(); // retry(callback)
std::move(callback)(); // try-or-fail(rvalue)
```

In the second use case, a member is forwarded that is owned by the Owner, but not directly contained by it. Look at this example below to understand what it means. The value stored in `m_ptr` is owned by the struct `Owner`, but it's not directly contained in it, because there is a unique pointer (`m_ptr` in fact) in between.

```cpp
struct Owner {
  std::unique_ptr<std::string> m_ptr;

  auto getPtr(this auto&& self) -> std::string {
    if (m_ptr) {
      return std::forward_like<decltype(self)>(*ptr);
    }
    return "";
  }
};
```

In the paper, you'll find the third use case showing why it's good to merge `const` qualifiers and also that `forward_like` can be useful even without deducing this.

## Conclusion

In this article, we reviewed how class template argument deduction is extended in C++23 in order to support inherited constructors. We also learned about `std::forward_like` which is a feature needed by [deducing this](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23).

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
