---
layout: post
title: "When to use const in C++? Part IV: parameters"
date: 2020-11-25
category: dev
tags: [cpp, tutorial, const]
excerpt_separator: <!--more-->
---
_Just make everything `const` that you can! That's the bare minimum you could do for your compiler!_

This is a piece of advice, many _senior_ developers tend to repeat to juniors, while so often even the preaching ones - we - fail to follow this rule.
<!--more-->

In this series of articles, we'll discuss about:
In this series of articles, we discuss about:
- [`const` functions](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` local variables](https://www.sandordargo.com/blog/2020/11/04/when-use-const-1-functions-local-variables)
- [`const` member variables](https://www.sandordargo.com/blog/2020/11/11/when-use-const-2-member-variables)
- [`const` return types](https://www.sandordargo.com/blog/2020/11/18/when-use-const-3-return-types)
- [`const` parameters](https://www.sandordargo.com/blog/2020/11/25/when-use-const-4-parameters)

We already covered the last but one topic. Today we finish by stating when we should use const parameters. Let's differentiate between plain old data types and classes.

## Const POD parameters

In this section, we talk about the primitive data types, such as bools, ints, floats, chars and alike.

Should they be taken as const parameters?

They should not be passed as const references or pointers. It's inefficient. These data types can be accessed with one memory read if passed by value. On the other hand, if you pass them by reference/pointer, first the address of the variable will be read and then by dereferencing it, the value. That's 2 memory reads instead of one.

We shall not take a POD by `const&`.

But should we take them simply by const?

As always, it depends.

If we don't plan to modify its value, yes we should. For better readability, for the compiler and for the future.

```cpp

void setFoo(const int foo) {
  this->m_foo = foo;
}
```

I know this seems like overkill, but it doesn't hurt, it's explicit and you don't know how the method would grow in the future. Maybe there will be some additional checks done, exception handling, and so on.

And if it's not marked as const, maybe someone will accidentally change its value and cause some subtle errors.

If you mark `foo` const, you make this scenario impossible.

What's the worst thing can happen? You'll actually have to remove the const qualifier, but you'll do that intentionally.

On the other hand, if you have to modify the parameter, don't mark it as const.

From time to time, you can see the following pattern:

```cpp
void doSomething(const int foo) {
// ...
int foo2 = foo;
foo2++;
// ...
}
```

Don't do this. There is no reason to take a `const` value if you plan to modify it. One more variable on the stack in vain, on more assignment without any reason. Simply take it by value.

```cpp
void doSomething(int foo) {
// ...
foo++;
// ...
}
```

So we don't take PODs by `const&` and we only mark them `const` when we don't want to modify them.

## Const object parameters

For objects, there is another rule of thumb. If we'd take a class by value as a parameter, it'd mean that we would make a copy of them. In general, copying an object is more expensive, then just passing a reference around.

So the rule to follow is not to take an object by value, but by `const&` to avoid the copy.

Obviously, if you want to modify the original object, then you only take it by reference and omit the const.

You might take an object by value if you know you'd have to make a copy of it.

```cpp
void doSomething(const ClassA& foo) {
// ...
ClassA foo2 = foo;
foo2.modify();
// ...
}
```

In that case, just simply take it by value. We can spare the cost of passing around a reference and the mental cost of declaring another variable and calling the copy constructor.

Although it's worth to note that, if you are accustomed to taking objects by `const&` you might have done some extra thinking whether passing by value was on purpose or by mistake.

So the balance of extra mental efforts is questionable.

```cpp
void doSomething(ClassA foo) {
// ...
foo.modify();
// ...
}
```

You should also note that there are objects where making the copy is less expensive or on a comparison to the cost of passing a reference about. It's the case for [Small String Optimization]() or for `std::string_view`. This is beyond the scope of this article.

For objects, we can say that by default we should take them by `const reference` and if we plan to locally modify them, then we can consider taking them by value. But never by `const` value, which would force a copy but not let us modify the object.

## Conclusion

In this series, we saw when and how to use the `const` qualifier for functions, for return values, local and member variables and finally today for function parameters.

For function parameters, the rule is different for plain old data types and for objects. We tend to take primitive data types by value, and objects by `const&`.

In case, you liked the article, give it a like a subscribe to my [newsletter]().