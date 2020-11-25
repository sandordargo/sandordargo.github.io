---
layout: post
title: "When to use const in C++? Part III: return types"
date: 2020-11-18
category: dev
tags: [cpp, tutorial, const]
excerpt_separator: <!--more-->
---
_Just make everything `const` that you can! That's the bare minimum you could do for your compiler!_

This is a piece of advice, many _senior_ developers tend to repeat to juniors, while so often even the preaching ones - we - fail to follow this rule.
<!--more-->

In this series of articles, we discuss about:
- [`const` functions](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` local variables](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` member variables](https://www.sandordargo.com/blog/2020/11/11/when-use-const-2-member-variables)
- [`const` return types](https://www.sandordargo.com/blog/2020/11/18/when-use-const-3-return-types)
- [`const` parameters](https://www.sandordargo.com/blog/2020/11/25/when-use-const-4-parameters)

In the last episodes, we covered the first three topics, `const` functions and `const` local variables, then `const` member variables and today we are covering return types.

What kind of variables can a function return? It can return values, references and pointers. And all of these can be const. Let's have a look at each of them.

## Returning const objects by value

If you're really enthusiastic about turning everything into const and that's your first time to do so, you might start converting signatures like `std::string getName() const` into `const std::string getName() const`. The only problem is that most probably it won't make so much sense.

Why is that?

Putting `const` somewhere shows the reader (and the compiler of course) that something should **not** be modified. When we return something by value it means that a copy will be made for the caller. Okay, you might have heard about [copy elision](https://en.cppreference.com/w/cpp/language/copy_elision) and its special form, return value optimization (RVO), but essentially we are still on the same page. The caller gets his own copy.

Does it make sense to make that own copy `const`?

Imagine that you buy a house but you cannot modify it? While there can be special cases, in general, you want your house to be your castle. Similarly, you want your copy to really be your object and you want to be able to do with it just whatever as an owner of it.

It doesn't make sense and it's misleading to return by value a const object.

Not just misleading, but probably even hurting you. 

Even hurting? How can it be?

Let's say you have this code:

```cpp
class SgWithMove{/**/};

SgWithMove foo() {/**/}
int main() {
SgWithMove o;
o = foo();
}
```

By using a debugger or by adding some logging to your special functions, you can see that RVO was perfectly applied and there was a move operation taking place when `foo()`s return value was assigned to `o`.

Now let's add that infamous `const` to the return type.

```cpp
class SgWithMove{/**/};

SgWithMove foo() {/**/}
const SgWithMove bar() {/**/}
int main() {
SgWithMove o;
o = bar();
}
```

Following up with the debugger we can see that we didn't benefit from a move, but actually, we made a copy.

We are returning a `const SgWithMove` and that is something we cannot pass as `SgWithMove&&` as it would discard the const qualifier. (A move would alter the object being moved) Instead, the copy assignment (`const SgWithMove&`) is called and we just made another copy.

_Please note that there are important books advocating for returning user-defined types by const value. They were right in their own age, but since then C++ went through a lot of changes and this piece of advice became obsolete._

## Returning const references

What about returning const references? Sometimes we can see this from very enthusiastic, but - hopefully - not so experienced developers that they return const references, just to be symmetric with the well-known rule of taking const reference arguments for objects.

So what is the problem?

Maybe nothing, maybe you'll have a [dangling reference](https://en.wikipedia.org/wiki/Dangling_pointer). The problem is with returning const references is that the returned object has to outlive the caller. Or at least it has to live as long.

```cpp
void f() {
  MyObject o;
  const auto& aRef = o.getSomethingConstRef();
  aRef.doSomething(); // will this work?
}
```

Will that call work? It depends. If `MyObject::getSomethingConstRef()` returns a const reference of a local variable it will not work. It is because that local variable gets destroyed immediately once we get out of the scope of the function.

```cpp
const T& MyObject::getSomethingConstRef() {
  T ret;
  // ...
  return ret; // ret gets destroyed right after, the returned reference points at its ashes
}
```

This is what is called a dangling reference.

On the other hand, if we return a reference to a member of `MyObject`, there is no problem in our above example.

```cpp
class MyObject 
{ 
public:
  // ...
  const T& getSomethingConstRef() {
    return m_t; // m_t lives as long as our MyObject instance is alive
  }
private:
  T m_t;
};
```

It's worth to note that outside of `f()` we wouldn't be able to use `aRef` as the instance of `MyObject` gets destroyed at the end of the function `f()`.

So shall we return const references?

As so often the answer is _it depends_. So definitely not automatically and by habit. We should return constant references only when are sure that the referenced object will be still available by the time we want to reference it.

At the same time:

__Never return locally initialized variables by reference!__

## Return const pointers

Pointers are similar to references in a sense that the pointed object must be alive at least as long as the caller wants to use it. You can return the address of a member variable if you know that the object will not get destroyed as long as the caller wants the returned address. What is important to emphasize once again is that we can never return a pointer to a locally initialized variable.

But even that is not so self-evident. Let's step back a little bit. 

What do we return when we return a pointer?

We return a memory address. The address can be of anything. Technically it can be a random place, it can be a null pointer or it can be the address of an object. (OK, a random place can be the address of a valid object, but it can be simply garbage. After all, it's random.)

Even if we talk about an object that was declared in the scope of the enclosing function, that object could have been declared either on the stack or on the heap.

If it was declared on the stack (no `new`), it means that it will be automatically destroyed when we leave the enclosing function.

If the object was created on the heap (with `new`), that's not a problem anymore, the object will be alive, but you have to manage its lifetime. Except if you return a smart pointer, but that's beyond the scope of this article.

So we have to make sure that we don't return a dangling pointer, but after that, does it make sense to return a const pointer?

- `int * const func () const`

The function is constant, and the returned pointer is constant but the data we point at can be modified. However, I see no point in returning a const pointer because the ultimate function call will be an rvalue, and rvalues of non-class type cannot be const, meaning that const will be ignored anyway

- `const int* func () const`

This is a useful thing. The pointed data cannot be modified.

- `const int * const func() const`

Semantically this is almost the same as the previous option. The data we point at cannot be modified. On the other hand, the constness of the pointer itself will be ignored. 

So does it make sense to return a `const` pointer? It depends on what is `const`. If the constness refers to the pointed object, yes it does. If you try to make the pointer `itself` const, it doesn't make sense as it will be ignored.

## Conclusion

Today, we learned about `const` return types. We saw that simply returning `const` values don't make much sense and - counterintuitively - it might hurt the performance. Returning a `const` reference is even dangerous and might lead to segmentation faults.

Const pointers are bit more varied topics, you don't face many dangers but constness there can be ignored.

Stay tuned, next time we'll learn about `const` parameters.