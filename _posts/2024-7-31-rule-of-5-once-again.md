---
layout: post
title: "Once more about the rule of 5"
date: 2024-7-31
category: dev
tags: [cpp, cpp23, binarysizes, move_only_function]
excerpt_separator: <!--more-->
---
[Arne Mertz talked about misused guidelines at C++OnSea](https://www.sandordargo.com/blog/2024/07/10/cpponsea2024-trip-report#my-favourite-talks). Among those, there was the rule of 5. Which made me think about a pattern I've seen.

Let's first repeat what the rule of 5 says.
> The Rule of Five tells us that if we need to define any of a copy constructor, copy assignment operator, move constructor, move assignment operator or destructor then we usually need to define all five. 

Fair enough.

Have you ever seen classes where the default constructor and destructor are explicitly defaulted? Like this?

```cpp
class SomeClass {
public:
    SomeClass() = default;
    ~SomeClass() = default;

    void foo();
private:
    int m_num{42};
};
```

First of all, that's not the best idea. You can simply remove them. But let's assume that you cannot remove the user-provided destructor for some reason. Maybe it's not defaulted and it does something.

What does the famous Hinnant table tell us?

![The Hinnant table]({{ site.baseurl }}/assets/img/hinnant-table.jpg "The Hinnant table")
_The Hinnant table (source: https://howardhinnant.github.io/)_

It says that when the user declares a destructor, the move operations are not declared. Hmmm... What does that mean?

Does it support move operations or not?

If you're not familiar with the details, but you got the idea to test class properties at compile-time, you might add some static assertions.


```cpp
#include <type_traits>

class SomeClass {
public:
    SomeClass() = default;
    ~SomeClass() = default;

    void foo();
private:
    int m_num{42};
};

static_assert(std::is_default_constructible_v<SomeClass>);
static_assert(std::is_copy_constructible_v<SomeClass>);
static_assert(std::is_copy_assignable_v<SomeClass>);
static_assert(std::is_move_constructible_v<SomeClass>);
static_assert(std::is_move_assignable_v<SomeClass>);

int main() {
    return 0;
}
```

They all pass!

Does it mean that we have what we wanted? Does it mean that `SomeClass` really supports move operations?

Well... Either we overlooked something or the rule of five and the above Hinnant-table is not totally correct.

Considering those options, of course, we overlooked something.

If you watch a few minutes of [this talk by Howard Hinnant from 18:20](https://youtu.be/vLinb2fgkHk?si=42VB8-4QEpBNt4CP&t=1100), the picture will be clearer. First of all, if something is mentioned as not declared, it will not be part of the overload resolution. Second, and probably this is more important, for move operations this means that they are not available, but there will be an automatic fallback to copy operations. Otherwise, way too many classes would have been broken by C++11.

So in the above case, the two static assertions about move operations only mean that code would compile if move operations were requested. Not that actual move operations would happen.

That's too bad. How can we still be sure about what is going on?

We can have an indirect proof with the help of C++Insights. Let's include the `<utility>` header and add some explicit move operations to our code first.

```cpp
#include <type_traits>
#include <utility>

class SomeClass {
public:
    SomeClass() = default;
    ~SomeClass() = default;

    void foo();
private:
    int m_num{42};
};

static_assert(std::is_default_constructible_v<SomeClass>);
static_assert(std::is_copy_constructible_v<SomeClass>);
static_assert(std::is_copy_assignable_v<SomeClass>);
static_assert(std::is_move_constructible_v<SomeClass>);
static_assert(std::is_move_assignable_v<SomeClass>);

int main() {
    SomeClass s1;
    SomeClass s2 = std::move(s1);
    return 0;
}
```

And now let's run it in [C++ Insights](https://cppinsights.io/about.html).

> *"C++ Insights is a clang-based tool which does a source to source transformation. Its goal is to make things visible, which normally and intentionally happen behind the scenes. It's about the magic the compiler does for us to make things work. Or looking through the classes of a compiler."*

The above class definition is expanded as below:

```cpp
class SomeClass
{
  
  public: 
  inline constexpr SomeClass() noexcept = default;
  inline ~SomeClass() noexcept = default;
  void foo();
  
  
  private: 
  int m_num;
  public: 
  // inline constexpr SomeClass(const SomeClass &) noexcept = default;
};
```

We can observe some `inline` and `noexcept` specifiers being added to the constructor and destructor, and the constructor is even `constexpr`. What is more important though is the commented out line. It's a copy constructor...

It's sad because we expected a move right? But the Hinnant-table already hinted to us that there would be no move operations declared if we declare our own destructor. Let's see what happens if we remove the user-provided destructor.

```cpp
class SomeClass
{
  
  public: 
  inline constexpr SomeClass() noexcept = default;
  void foo();
  
  
  private: 
  int m_num;
  public: 
  // inline constexpr SomeClass(SomeClass &&) noexcept = default;
};
```

Not surprisingly, the destructor disappeared from the expanded C++ Insight version. We can also observe that the commented out line now doesn't specify a copy, but rather a move constructor.

In other words, by providing a destructor, we lost the ability to use move semantics.

Let's see what Compiler Explorer says! But before, I added another member, a `std::string` so that optimizations aren't so easy to perform.

This is the new example:

```cpp
#include <type_traits>
#include <utility>
#include <string>

class SomeClass {
public:
    SomeClass() = default;
    ~SomeClass() = default;

    void foo();
private:
    int m_num{42};
    std::string m_text;
};

static_assert(std::is_default_constructible_v<SomeClass>);
static_assert(std::is_copy_constructible_v<SomeClass>);
static_assert(std::is_copy_assignable_v<SomeClass>);
static_assert(std::is_move_constructible_v<SomeClass>);
static_assert(std::is_move_assignable_v<SomeClass>);

int main() {
    SomeClass s1;
    SomeClass s2 = std::move(s1);
    return 0;
}
```

Now the assembly for the line `SomeClass s2 = std::move(s1);` is a call to the copy constructor!

```asm
call    SomeClass::SomeClass(SomeClass const&) [base object constructor]
```

If we remove the user-declared destructor, it becomes a call to the move constructor:

```asm
call    SomeClass::SomeClass(SomeClass&&) [base object constructor]
```

## Conclusion

Both C++ Insights and Compiler Explorer confirmed the same. By providing a destructor while not following the rule of five, our classes lose the ability to support move semantics. Yet, due to backward compatibility, they don't fail to compile, they silently fall back to copy semantics.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!  