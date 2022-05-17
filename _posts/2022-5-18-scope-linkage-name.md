---
layout: post
title: "C++ basics: scopes, linkage, names"
date: 2022-5-18
category: books
tags: [cpp, beginners]
excerpt_separator: <!--more-->
---
First, I learnt C++ at university, but I better not count it. Then I first started to work with it 9 years ago. My employer booked a 5-day-training only for me. Those were good, generous times. But I think that the training was not that much of a success for several reasons.

I understood years later when I started to review our C++ training offerings that the instructor was below average. Just like my English and programming knowledge. Despite the fact that I had been using English for a long time - even for work - following a 5-day-long technical training delivered by a non-native instructor was a bit too difficult for me.

But I learned on the go.

More or less.

I still realize that sometimes I lack the correct understanding of some basic concepts. Whenever I have the realization, I consider writing an article on the topic. And I've been posting every week for the last 5 years.

Lately, I had a similar realization while I was reading [Beautiful C++](https://devreads.sandordargo.com/beautiful-cpp-by-kate-gregory-and-guy-davidson/). I'd still run into some issues if I had to explain what linkage is.

So let's discuss now a couple of things that the book brought up; the differences between name, linkage and scope.

## What is a name?

That seems a simple question, especially if you consider this piece of code.

```cpp
struct S {
  int m_num = 0;
};

int main() {
    [[maybe_unused]] S s{42};
}
```

What is a name? That's `s`, right? It's a name! Well. Right. But what is the difference between a name and an object?

That's probably still an easy one. What's the difference between your name and you?

Your name denotes you, but it's not you, it's not your physically existing body.

A name is just a handle, a reference to an object. 

This might seem philosophical. Still, it's important to make the distinction. Not only because the C++ standard does it, but because names and objects have different attributes.

Names have a scope and objects have storage durations.

Besides, not every object has a name, and not every name refers to an object. The latter one is obvious. For example functions and classes also have names but they are not objects.

Objects might not have names. Like temporaries. Look at this example.

```cpp
void foo(std::string s) {
  // ...
}

int main() {
  foo(std::string{"bar"});
}
```

`std::string{"bar"}` creates an object, but it doesn't have a name.

But let's get back to the question of scopes and store durations. We start with the latter.

## Storage duration

All objects have a [storage duration](https://en.cppreference.com/w/cpp/language/storage_duration). The storage duration of an object determines what rules to apply for its creation and destruction.

Often, people find it difficult to make a distinction between *storage duration* and *lifetime*. Lifetime is about the time when objects are usable and it's a runtime property of an object. The storage duration determines the minimum potential lifetime of the storage containing an object. This is determined by the construct that is used to create the object.

An object will always have one of the 4 following storage durations:
- automatic
- static
- dynamic
- thread

*Automatic* storage duration means that all the storage necessary for non-`static`, non-`extern`, non-thread-local local objects in a code block are allocated at the beginning of the block and deallocated at the end. This also shows how the storage duration can start earlier than the lifetime of an object. The storage is usually allocated sooner than the object is constructed.

In the case of *static* storage duration, the storage is allocated when the program begins and deallocated when the program ends. Variables with *static* storage duration have only one instance. Which objects have *static* storage duration? All that were declared with the `static` keyword! Besides, all of the objects that were declared at a namespace level or declared with the `extern` keyword.

*Dynamic* storage duration probably raises the least number of questions. Storage for such objects is allocated and deallocated upon request. Think about the dreaded `new`/`delete` pairs. Objects that use them have a *dynamic* storage duration.

Last but not least, we have to speak about *thread local* storage duration. The storage for such variables is allocated when the thread begins and deallocated when the thread ends. There is a different instance of the object in each thread. Only objects declared with the `thread_local` specifier have this kind of storage duration. `thead_local` can be combined with the `static` or `extern` keywords.

## Linkage

Now that we talked about names and storage durations, we can finally talk about linkage. You declare a name in a scope. But what happens if you declare another entity with the same name in another scope? Or in several other scopes? It depends on the (lack of) linkage that how many instances will be generated.

Till C++20, there were 3 different linkages, the fourth is a new one.

- no linkage
- internal linkage
- external linkage
- module linkage (introduced in C++20)

> *A translation unit is the basic unit of compilation in C++. It consists of the contents of a single source file, plus the contents of any header files directly or indirectly included by it, minus those lines that were ignored using conditional preprocessing statements.*
> 
> *A single translation unit can be compiled into an object file, library, or executable program.*

With *no linkage*, a name can be referred to only from the scope where it was created. Think about simple local variables declared in a block of code. They have no linkage, you cannot refer to them from an outer scope.

When a name has *internal linkage*, that name can be referred to from all scopes in the current translation unit. Static functions, variables, and their templated version, they all have internal linkage. Also, any names declared in an unnamed namespace have this level of linkage.

When a name has *external linkage*, it can be referred to from the scopes of another translation unit. This can go as far as using variables and functions from translation units that were written in another language. Enumerations, class names and their member functions and static data members, non-static templates and class templates, etc.

*Module linkage* was introduced in C++20. When a name has *module linkage*, it can only be referred to from the same module unit. This might mean another translation unit.

Note that this section aimed to show what kind of different linkages exist in C++. If you want to verify the full specs of what kind of names have what kind of linkage, [please read this page](https://en.cppreference.com/w/cpp/language/storage_duration#Linkage).

## Scope

Last but not least, let's talk about scopes. Scopes are collections of names referring to abstractions. Scopes are where a name is visible with an unqualified name lookup. This implies two things:
- names might be looked up in non-unqualified ways even outside of their scope
- the lifetime of an object might not end where the scope of its name ends

There are 6 different scopes we can talk about:

- block scope
- function parameter scope
- namespace scope
- class scope
- enumeration scope
- template parameter scope

A *block scope* is the most usual one. It starts with an opening brace and ends with a closing one. It's worth noting that they can be discontinuous when we use nested blocks.

```cpp
if (x.isValid) { // opens scope 1
  auto r = 42;
  auto z = Foo{};
  { // opens scope 2!

    auto r = z.something(); // this is a different r

  } // ends scope 2!
  // it's scope 1 again
  std::cout << r << '\n'; // r is 42 once again
} // ends scope 1
```

It's worth noting that in the nested block you can declare names that are used within the outer scope and as such, those become inaccessible (like `r`), but once the nested scope is closed we can refer to them again.

*Function parameter scope* is very similar to *block scope*. In terms of scopes, the function is the combination of the block and the function header. A *function-try-block* is similar, the end of the scope is the end of the last `catch` block. By the way, have you ever seen a *function-try-block*? The following piece of code is a valid function:

```cpp
float divide(float a, float b)
try {
  std::cout << "Dividing\n";
  return a / b;
} catch (...) {
  std::cout << "Dividing failed, was the denominator zero?\n";
}
```

The *namespace scope* starts where the namespace is declared and includes the rest of the namespace and all other namespace declarations with the same name. The top-level scope of a translation unit is also a *namespace scope*, that's the *global namespace scope*.

The *class scope* starts when a class starts being declared but does not end where the class declaration ends. It just pauses. You can resume it any time to define the declared elements. After resuming the scope, you can access entities of the class with different syntaxes (`.`, `->`, `::`).

The *enumeration scope* depends on the type of the enumeration. In any case, the scope starts at the beginning of the enumeration declaration. The scope of a scoped `enum` ends at the end of the declaration. But the scope of an unscoped `enum` ends at the end of the enclosing scope.

Last but not least, let's not forget about the *template parameter scope*. The scope of a template parameter name begins at the point of declaration and ends at the end of the smallest template declaration in which it was introduced. Such parameters can be used in subsequent template parameter declarations and also in the base class specifications.

## Conclusion

In this article, we discussed a couple of ideas that are often used when people talk about C++ or programming in general. Words that we might not dare to use in everyday conversations because we are not sure if we understand them well. Names, scopes, linkage and even translation units! Today we came a few steps closer to having a better view of them.

I highly recommended you read through the linked materials to have a deeper understanding!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
