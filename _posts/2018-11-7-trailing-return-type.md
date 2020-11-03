---
layout: post
title: "How do you declare a function in C++?"
date: "2018-11-7"
category: dev
tags: [cpp, tutorial, clean code]
excerpt_separator: <!--more-->
---
Beginning of this year I came back to a C++ developer position and we are making or final steps towards complete a migration to (among others) C++11 and I decided to level up my knowledge. It's almost like discovering a new language which is, by the way, a lot more pleasant than C++98.

One of the things that made my eyes open was how function declarations evolved.
<!--more-->

If you've been around for a long time in the C++ ecosystem, probably you'd reply something similar to this:

```
int getElement(const std::vector<int>& container, int index) const;
```

But if you started only lately or if you're experienced with newer versions of C++ (>=C++11), you might have another answer, like:

```
auto getElement(const std::vector<int>& container, int index) const -> int;
```

I'm sure you noticed the differences:
* Instead of starting with `int` as a return type, we used the `auto` keyword
* We added `int` as a return type after an arrow (`->`). 

The return type is coming after the function name and the list of parameters and function qualifiers!

Why is this interesting for us? You might say that this makes no sense it just makes the code less readable. I think it's a matter of style, but I tend to agree. It definitely makes the code longer without any benefits added.

So why has this trailing return type been added? How can we use it?

### Omitting the scope

Even though we saw that by using trailing return types our code became longer, that's not necessarily true in all cases.

Let's have a look at our class that represents wines.

```
class Wine {
 public:
 enum WineType { WHITE, RED, ROSE, ORANGE };
 void setWineType(WineType wine_type);
 WineType getWineType() const;

 //... 

 private:
  WineType _wine_type;
}
```

If you wonder what is Orange wine, it's not made of orange. You can find more details [here](https://vinepair.com/articles/orange-wine-guide/).

Now let's check the implementations.

The setter's look quite obvious, does it?

```
void Wine::setWineType(WineType wine_type) {
  _wine_type = wine_type;
}
```

On the other hand, our first approach for the getter might not work:

```
WineType Wine::getWineType() {
  return _wine_type;
}
```

The above code will just not compile, because WineType is unknown to the compiler. It looks for it in the global scope. You have to explicitly declare that it's part of the Wine class.

```
Wine::WineType Wine::getWineType() {
  return _wine_type;
}
```

It seems like a duplication, but it's necessary. Necessary, yet avoidable since trailing return type declarations are available. Have a look at this:

```
auto Wine::getWineType() -> WineType {
  return _wine_type;
}
```
At the beginning of the line, the compiler couldn't know the scope, hence we had to write `Wine::WineType`, but when we declare the return type at the end the compiler already knows what we are in the scope of `Wine`, so we don't have to repeat that information.

Depending on your scope's name, you might spare some characters, but at least you don't have to duplicate the class name.

This is nice, but do you think that the ISO CPP committee would have introduced a change just in order not to duplicate the scope? I don't think so, but who knows. What's for sure that there are other uses of trailing type declaration.

### Use trailing type declaration in templates with `decltype`

Probably a more compelling reason to use trailing return type declaration is the case when the return type of a function template depends on the argument types.

Let's see the good old example:

```
template<class L, class R>
auto multiply(L const& lhs, R const& rhs) -> decltype(lhs * rhs) {
  return lhs * rhs;
}
```

It is possible to create such a function template by using `std::declval`, but it's getting so lengthy and unreadable that I don't even put here. Look it up, if you want to have a bad sleep.

On the other hand, it's even simpler in C++14 where the scope of return type deduction was extended:

```
template<class L, class R>
auto multiply(L const& lhs, R const& rhs) {
  return lhs * rhs;
}
```

### Conclusion

You saw that using trailing return type declaration can help you not repeating the scope for normal functions, and for template functions in C++11, it makes easier to declare return types that depend on the template parameters than it was before.

Should you use it in each every case? Do you have to always use it? No. But I don't say that you shouldn't use it all the time. It's a matter of style. Do as you wish and be consistent. Either use it all the time or just when it actually brings a positive value. But don't do it half-half.

What's the most important is that you the new syntax, you know it exists and knows how to use it. For me, this has been completely new until recently when I started to read [Effective Modern C++](https://amzn.to/2Rbh5pI) by [Scott Meyers](https://www.aristeia.com/). I'm also recommending [Fluent{C++}](https://www.fluentcpp.com/) as a source to learn about this very rich language.