---
layout: post
title: "The rule of 5 and inheritance"
date: 2024-8-7
category: dev
tags: [cpp, cpp23, binarysizes, move_only_function]
excerpt_separator: <!--more-->
---
Last week, we talked about the rule of five and we discovered what it means for move operations if we only declare a destructor and not the rest of the special member functions. In that case, move operations are not declared, any move would automatically downgraded to a copy.

You might say that it doesn't happen often that you only declare a destructor without the others and if it is so, you should either remove it or declare the rest. However, there is one very common scenario in C++ that requires our attention.

I've seen plenty of classes like these two below:

```cpp

class Base {
public:
    virtual ~Base() = default; 

    /*
    The rest
    */

private:
    /*
    Possibly some data members
    */
};

class Derived {
public:
    ~Derived() override = default;

private:
    /*
    Possible some  - more - data members
    */
};
```

So today, we are going to see what to do with the special member functions when we have to deal with dynamic polymorphism.

## What happens behind the scenes in terms of SPMs

Let's use Compiler Explorer without turning any optimization on. The assembly will show us
- what's in the virtual tables
- what special member functions are called for the line `Derived s2 = std::move(s1);`.

If we take the above example and [look at it in Godbolt](https://godbolt.org/z/3s6KdvGY6), we can make the following observations:
- the `vtable` for `Derived` includes its destructor
- the line where we try to move calls `Derived::Derived(Derived const&)` which calls `Base::Base(Base const&)`. In other words, copy constructors are involved.

This should not surprise us after all. Having the virtual destructor is normal and using copy semantics instead of move semantics when there is a user-provided destructor is [exactly what we saw last week](https://www.sandordargo.com/blog/2024/07/31/rule-of-5-once-again).

Now, let's remove the user-provided destructor from `Derived` and [check the generated assembly](https://godbolt.org/z/GWWxGcM15):

- the `vtable` for `Derived` still includes its destructor
- the move now invokes `Derived::Derived(Derived&&)` which invokes `Base::Base(Base const&)`.

We removed the explicit declaration (and definition) of Derived's destructor, but it's still in the vtable. We often forget that if there is a virtual destructor in any of the base classes, the compiler will automatically generate it for the derived classes as well.

Besides, now that we have not provided a destructor for Derived, move operations are available and used as intended. On the other hand, it still invokes the copy constructor in the Base class. That makes also sense, given that `Base` has a user-declared destructor which prevents the automatic generation of move operations.

Let's make the last step. We obviously cannot remove the virtual destructor of `Base`. Technically, we could, but if we intend to use a class as a base class in a polymorphic structure we should always provide the destructor in order to avoid undefined behaviour when we delete a derived object through the base class pointer. So practically that's not an option.

The only real option is to follow the rule of five in the base class.

```cpp
#include <type_traits>
#include <utility>
#include <string>

class Base {
public:
    virtual ~Base() = default; 

    Base() = default;
    Base(const Base&) = default;
    Base(Base&&) = default;
    Base& operator=(const Base&) = default;
    Base& operator=(Base&&) = default;

    /*
    The rest
    */

private:
    /*
    Possibly some data members
    */
};

class Derived : public Base {
public:

private:
    /*
    Possible some  - more - data members
    */
};

static_assert(std::is_default_constructible_v<Derived>);
static_assert(std::is_copy_constructible_v<Derived>);
static_assert(std::is_copy_assignable_v<Derived>);
static_assert(std::is_move_constructible_v<Derived>);
static_assert(std::is_move_assignable_v<Derived>);

int main() {
    Derived s1;
    Derived s2 = std::move(s1);

    return 0;
}
```

Let's check the assembly for [the above example](https://godbolt.org/z/ad47vscW1):

- we have all the destructors in the `vtable`s for `Derived`
- the move now invokes `Derived::Derived(Derived&&)` which now correctly invokes `Base::Base(Base&&)`.

## How should we define our base classes?

If we want to create a class that will serve as a base class, we need to provide it with a virtual destructor. That's for sure.

In addition, if we want the base class to benefit from move semantics, we should also follow the rule of five and provide all the special member functions. Otherwise, if we only provide a `virtual` destructor - and maybe a default constructor - every move will automatically be handled as a copy.

Is that a problem?

The answer is - as always - "it depends". When you have a base class without any data members serving simply as an interface, it's not an issue.

On the other hand, if the base class has data members and has to be moved around a lot, that can be a problem.

If you want to go for sure, just follow the rule of 5. It doesn't stop where the keyword `virtual` comes in to picture.


## Conclusion

[Last week, we discussed the rule of 5](https://www.sandordargo.com/blog/2024/07/31/rule-of-5-once-again) and what really happens if you provide a destructor, but not the rest. We saw that it was a bad idea, it would lead to a fallback of  all moves to copies.

This week, we checked what happens under the hood if we deal with virtual destructors and inheritance. We've seen that essentially nothing changes. If we fail to follow the rule of 5 and we only provide a virtual destructor, we can say goodbye to move operations.

Therefore in derived classes we should not deal with destructors anyhow, they are generated and they are virtual given that a base class has a virtual destructor. On the other hand, in the base class, we should follow the rule of 5 to ensure the compiler can move objects instead of copying them whenever possible.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  