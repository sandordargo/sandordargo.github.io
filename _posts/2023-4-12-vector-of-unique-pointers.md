---
layout: post
title: "Vectors and unique pointers"
date: 2023-4-12
category: dev
tags: [cpp, vector, movesemantics, unqiueptr]
excerpt_separator: <!--more-->
---
In this post, I want to share some struggles I had twice during the last few months. For one of my examples, I wanted to initialize a `std::vector` with `std::unique_ptr`. It didn't compile and I had little time, I didn't even think about it. I waved my hand and changed my example.

Then I ran into the same issue at work, while we were pairing over an issue. I couldn't just wave anymore, and luckily we also had the time to go a bit deeper.

What we wanted was a little bit different and more complex though. We wanted to return a vector of unique pointers to a struct that holds - among others - a unique pointer.

This is a simplified version of what we wanted:

```cpp
class Resource {
  // unimportant
};

struct Wrapper {
  std::string name;
  std::unique_ptr<Resource> resource;
};

// somewhere in a function
std::vector<std::unique_ptr<Wrapper>> v{
  std::make_unique<Wrapper>(std::move(aName), std::make_unique<Resource>()),
  std::make_unique<Wrapper>(std::move(anotherName), std::make_unique<Resource>())
};
```

That's a bit too complex to start with, so let's dissect the issue into two parts.

## No compiler-generated copy constructor

Let's first omit the external unique pointer and try to brace-initialize a `vector` of `Wrapper` objects.

```cpp
#include <memory>
#include <string>
#include <vector>

class Resource {
  // unimportant
};

struct Wrapper {
  std::string m_name;
  std::unique_ptr<Resource> m_resource;
};


int main() {
  std::string aName = "bla";
  std::vector<Wrapper> v{Wrapper{aName, std::make_unique<Resource>()}};
}
```

The first part of the problem is that we cannot `{}`-initialize this `vector` of `Wrapper`s. Even though it seems alright at a first glance. `Wrapper` is a `struct` with public members and no explicitly defined special functions. Our `{}`-initialization follows the right syntax and the parameters are passed in the right order.

Still, the compiler says stop! Here is a part of the error messages. As `vector` is a template we cannot expect a concise message.

```
/opt/compiler-explorer/gcc-12.2.0/include/c++/12.2.0/bits/stl_uninitialized.h:90:56: error: static assertion failed: result type must be constructible from input type
   90 |       static_assert(is_constructible<_ValueType, _Tp>::value,
```

If we scroll up in the errors we'll find a more explanatory line:

```
In instantiation of 'constexpr bool std::__check_constructible() [with _ValueType = Wrapper; _Tp = const Wrapper&]':
```

So the problem is `Wrapper` cannot be constructed from `const Wrapper&`, in other words, `Wrapper` cannot be copy constructed. That makes sense! It has a *move-only* member, `std::unique_ptr<Resource> m_resource`! Because of this *move-only* member, the compiler cannot automatically generate a copy constructor.

## A `std::vector` always copies its `std::initializer_list`

That's all fine but why do we need a copy constructor? Why cannot we benefit from move semantics?

We can spot the answer on [C++ Reference](https://en.cppreference.com/w/cpp/container/vector/vector)! `std::vector` has only one constructor involving a `std::initializer_list` and there the `initializer_list` is taken by value. In other words, `vector` copies its `initializer_list`. Always.

As the passed in `initializer_list` is going to be copied, the contained type must be copy-constructible. That's clearly not the case for `Wrapper` which we can easily assert.

```cpp
struct Wrapper {
  std::string m_name;
  std::unique_ptr<Resource> m_resource;
};

static_assert(std::is_copy_constructible<Wrapper>());
/*
main.cpp:18:20: error: static assertion failed
   18 | static_assert(std::is_copy_constructible<Wrapper>());
      |                    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
*/
```

### Let's make contained types copy constructible

That's quite easy to fix, we need to provide a user-defined copy constructor, such as `Wrapper(const Wrapper& other): m_name(other.m_name), m_resource(std::make_unique<Resource>()) {}`. At the same time, let's not forget about [the rules of 0/3/5](https://en.cppreference.com/w/cpp/language/rule_of_three), so we should provide all the special functions.

Now we don't have the problem anymore, we can easily use the brace initialization.

```cpp
#include <memory>
#include <string>
#include <vector>

class Resource {
  // unimportant
};

struct Wrapper {
  std::string m_name;
  std::unique_ptr<Resource> m_resource;
  Wrapper() = default;
  ~Wrapper() = default;
  Wrapper(std::string name, std::unique_ptr<Resource> resource) : m_name(std::move(name)), m_resource(std::move(resource)) {}
	
  Wrapper(const Wrapper& other) : m_name(other.m_name), m_resource(std::make_unique<Resource>()) {}
  Wrapper(Wrapper&& other) = default;
	
  Wrapper& operator=(const Wrapper& other) {
    m_name = other.m_name;
    m_resource = std::make_unique<Resource>();
    return *this;
  }
	    
  Wrapper& operator=(Wrapper&& other) = default;
};

int main() {
  std::string aName = "bla";
  std::vector<Wrapper> v{Wrapper{aName, 
  std::make_unique<Resource>()}};
}
```

But what if we cannot or do not want to modify `Wrapper`? Do we have any other options?

Of course!

### We can also extract `vector` initialization

If we have to avoid the brace-initialization of the vector, we can simply default initialize it, probably reserving then enough space for the items we want to add, and use either `vector`'s `emplace_back()` or `push_back()` methods.

There is not a big difference in this case between `emplace_back()` and `push_back()`. `push_back()` will call the appropriate constructor first and then the move constructor. `emplace_back()` will only make one constructor call. At least that's the theory. Let's see if we're right. I amended `Wrapper` so that each of its special functions prints.

```cpp
#include <memory>
#include <string>
#include <vector>

#include <iostream>

class Resource {
  // unimportant
};

struct Wrapper {
  std::string m_name;
  std::unique_ptr<Resource> m_resource;

  Wrapper() {
    std::cout << "Wrapper()\n";
  }

  ~Wrapper() {
    std::cout << "~Wrapper()\n";
  }
	
  Wrapper (std::string name, std::unique_ptr<Resource> resource) : m_name(std::move(name)), m_resource(std::move(resource)) {
    std::cout << "Wrapper (std::string name, std::unique_ptr<Resource> resource)\n";
  }

  Wrapper(const Wrapper& other) : m_name(other.m_name), m_resource(std::make_unique<Resource>()) {
    std::cout << "Wrapper (const Wrapper& other)\n";
  }

  Wrapper(Wrapper&& other) {
    std::cout << "Wrapper (Wrapper&& other)\n";
  }

  Wrapper& operator=(const Wrapper& other) {
    std::cout << "Wrapper& operator=(const Wrapper& other)\n";
    return *this;
  }
	
  Wrapper& operator=(Wrapper&& other) {
    std::cout << "Wrapper& operator=(Wrapper&& other)\n";
    return *this;
  }
};


int main() {
  std::string aName = "bla";
  std::vector<Wrapper> v{};
  v.emplace_back(aName, std::make_unique<Resource>());
  // v.push_back(Wrapper(aName, std::make_unique<Resource>()));
}

/*
output with emplace_back:
Wrapper (std::string name, std::unique_ptr<Resource> resource)
~Wrapper()
*/

/*
output with push_back
Wrapper (std::string name, std::unique_ptr<Resource> resource)
Wrapper (Wrapper&& other)
~Wrapper()
~Wrapper()
*/
```

Our assumption was right, except that I forgot to note the extra destructor call on a moved-from object. While these extra calls would be often negligible, we have no reason not to use `emplace_back()`. It's not more difficult to write it and it's better for the performance.

With that, we've seen why we couldn't `{}`-initialize a `vector` of `Wrapper`s following the [rule of zero](https://en.cppreference.com/w/cpp/language/rule_of_three#Rule_of_zero), where `Wrapper` is a type that allocates on the heap.

Now let's move over to the other issue.

## The same issue with `initializer_list` once again

Actually, by answering the first question, we also learned why we cannot `{}`-initialize a `vector` of `unique_ptr`s. It's the problem of an `initializer_list` taken by value instead of reference and the fact that `unique_ptr`s cannot be copied.

Can we do anything to improve the situation?

We can extract the creation and population of the `vector` to a separate method so that we don't have the visual noise of `vector` insertions along with the rest of the code.

```cpp
std::vector<std::unique_ptr<Resource>> createResources() {
  std::vector<std::unique_ptr<Resource>> vec;
  vec.push_back(std::make_unique<Resource>());
  vec.push_back(std::make_unique<Resource>());
  vec.push_back(std::make_unique<Resource>());
  return vec;
}


int main() {
  std::vector<std::unique_ptr<Resource>> resources = createResources();
  // ...
}
```

But can we do something better?

We can make a better, generalized function that makes us a `vector` of `unique_ptr`s, but the idea behind is essentially the same: the pointers are added one by one after the construction of the vector.

[Let me borrow an implementation by Bartek](https://www.cppstories.com/2023/initializer_list_improvements/). I take this piece of code from [C++ Storties](https://www.cppstories.com/2023/initializer_list_improvements/) and I encourage you to read the whole article on `initializer_list`:

```cpp
template<typename T, typename... Args>
auto makeVector(Args&&... args) {
  std::vector<T> vec;
  vec.reserve(sizeof...(Args)); 
  (vec.emplace_back(std::forward<Args>(args)), ...);
  return vec;
}
```

Now we can update our original example.

```cpp
#include <memory>
#include <string>
#include <vector>

#include <iostream>

class Resource {
  // unimportant
};

struct Wrapper {
  std::string m_name;
  std::unique_ptr<Resource> m_resource;
  Wrapper() = default;
  ~Wrapper() = default;
  Wrapper(std::string name, std::unique_ptr<Resource> resource) : m_name(std::move(name)), m_resource(std::move(resource)) {}
	
  Wrapper(const Wrapper& other) : m_name(other.m_name), m_resource(std::make_unique<Resource>()) {}
    Wrapper(Wrapper&& other) = default;
	
  Wrapper& operator=(const Wrapper& other) {
    m_name = other.m_name;
    m_resource = std::make_unique<Resource>();
    return *this;
  }
	    
  Wrapper& operator=(Wrapper&& other) = default;
};

// this is from Bartek: https://www.cppstories.com/2023/initializer_list_improvements/
template<typename T, typename... Args>
auto makeVector(Args&&... args) {
  std::vector<T> vec;
  vec.reserve(sizeof...(Args)); 
  (vec.emplace_back(std::forward<Args>(args)), ...);
  return vec;
}

int main() {
  [[maybe_unused]] std::vector<std::unique_ptr<Wrapper>> v = makeVector<std::unique_ptr<Wrapper>>(
  std::make_unique<Wrapper>("keyboard", std::make_unique<Resource>()),
  std::make_unique<Wrapper>("mouse", std::make_unique<Resource>()),
  std::make_unique<Wrapper>("screen", std::make_unique<Resource>())
  );
  return 0;
}
```

## Conclusion

Today, I shared with you a problem I faced lately. I wanted to `{}`-initialize a `vector` of unique pointers, but it didn't work. A `std::vector` takes an `initializer_list` by value, so it makes a copy of it. Hence, the compilation will fail if you try to use an `initializer_list` with move-only types.

If you want to use the `{}`-initializer for a vector, you need to implement the move constructor. If that's not an option and you want to separate the creation of the `vector`, you have no other option than move the related code to a separate function.


## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
