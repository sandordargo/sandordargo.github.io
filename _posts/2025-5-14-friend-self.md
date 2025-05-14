---
layout: post
title: "Declaring a friendship to self"
date: 2025-5-14
category: dev
tags: [cpp, friends, templates]
excerpt_separator: <!--more-->
---
Recently, I ran into some code where a class declared itself as a friend. I was quite surprised to see that and at a second glance, I realised that there were 2 characters difference in a relatively long class name. But my curiosity was already awakened.

Are there situations when a class should include a friendship declaration to itself?

First of all, if you declare a class friend of itself directly, the compiler might issue a warning.

```cpp
class C {
  friend class C; // warning: class 'C' is implicitly friends with itself
};
```

Otherwise, I found two scenarios which I want to share with you. They are slightly different, you might consider cheating both, but these are the closest ones I found.

## Explicitly Enabling Template Specialization Access

The cheating in the first example is that it uses templates. You'll see `friend class Wrapper` in `Wrapper`, but you'll see more. In fact, `Wrapper` is `Wrapper<T>`. Instantiate `Wrapper` with different types, and they turn out to be distinct, unrelated types. What if you want to have access from a `Wrapper` specialization to private elements of `Wrapper<U>`.

Without declaring a friendship, it's not going to work. 

Let's see the code. 

```cpp
#include <concepts>
#include <iostream>

template <typename T>
class Wrapper {
private:
    int secret = 42;

    template <typename U>
    friend class Wrapper; // Friend itself to allow specializations access

public:
    void showSecret() {
        std::cout << "Secret: " << secret << std::endl;
    }
};

template <>
class Wrapper<int> {
public:
    template <typename T> requires (!std::same_as<T, int>)
    void accessSecret(Wrapper<T>& w) {
        // Thanks to the friend declaration, this specialization can access private members
        std::cout << "Accessing: " << w.secret << std::endl;
    }
};

int main () {
    auto w1 = Wrapper<int>();
    auto w2 = Wrapper<double>();
    w1.accessSecret(w2);
}
```

Notice that the `Wrapper<int>` specialization defines an `accessSecret` method which accepts any instantiation of a `Wrapper` class and tries to access a private member of `Wrapper` called `secret`.

Remove the friendship and you'll see that `Wrapper<int>` has no access to private elements of `Wrapper<double>`. Notice that we didn't simply declare `Wrapper` as a friend, but we used a templated version `template <typename U> friend class Wrapper`.

You can play around with the code [on Godbolt](https://godbolt.org/z/or54b3xs4).

## Declare the outer class as a friend

Let's talk about nested classes. An outer class doesn't have access to the non-public members of an inner class, and an outer class has no access to the non-public members of an inner class. 

Though it would be strange to speak about declaring the inner class as a friend in this article, there is no way to qualify to inner class as self. But to declare `Outer` as a friend of `Outer::Inner`, that's not too strong a bending of my self-made constraints.


```cpp
#include <iostream>

class Outer {
   public:
    class Inner {
        int secret = 42;

        friend class Outer;
    };

    void revealSecret(const Inner& inner) {
        std::cout << "Inner's secret is: " << inner.secret << '\n';
    }
};

int main() {
    Outer::Inner innerObj;
    Outer outerObj;
    outerObj.revealSecret(innerObj);  // Prints: Inner's secret is: 42
}
```

I know this example is a bit simplistic. It's also a bit difficult to imagine a real-life situation when an outer class needs access to non-public members of an inner class. But you can imagine situations where there is an inner class representing some sort of state and only the outer class should be able to manipulate the state, no one else.

In such cases, declaring the outer class a friend of the inner class makes sense.

## Conclusion

Today we discussed cases when a class declares itself as a friend. Well, not exactly, maybe. We saw template classes declaring other specializations as friends, and nested classes where inner classes declare their enclosing classes as friends.

Have you ever had to use such a technique?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
