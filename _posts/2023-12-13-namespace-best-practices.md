---
layout: post
title: "How to use your namespaces to their best"
date: 2023-12-13
category: dev
tags: [cpp, cpp23, namespaces, codeorganization]
excerpt_separator: <!--more-->
---
Today, we are not going to discuss any novelty of the language. Instead, we are going to discuss something old, something that we take for granted and probably something we don't even think about a lot. We are going to discuss namespaces, what they are and what are the related best practices, including *some* of the recommendations of the [Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines).

## What are namespaces in the first place?

In C++, a `namespace` is a mechanism for organizing and encapsulating functions, variables, and types. Namespaces help prevent naming conflicts and make it easier to manage and maintain large codebases. They allow you to define a scope within which you can declare identifiers (such as variables, functions, and classes) without worrying about naming collisions with identifiers in other parts of your code.

Just like in the different classes, you can use methods or variables with the same name, in different namespaces, you can reuse the same symbol names.

```cpp
#include <iostream>
#include <string>

namespace foo {
    constexpr int magic_number = 42; 
}

namespace bar {
    constexpr float magic_number = 3.14;
}


int main() {
    std::cout << foo::magic_number << '\n';
    std::cout << bar::magic_number << '\n';
}

```

There is no problem with having `magic_number` declared and defined in both namespaces, those are two different variables. We can use them either by fully qualifying their name or by importing their symbols from their enclosing namespaces. More on that later.

### Nested namespaces

Namespaces can also be nested and as you can see in the below example, since C++17 we have a shortcut to declare nested namespaces with the help of `::` so that we can save some typing.

```cpp
#include <iostream>
#include <string>

namespace foo {
    namespace consts {
        constexpr int magic_number = 42; 
    }
}

namespace bar::consts {
    constexpr float magic_number = 3.14;
}


int main() {
    std::cout << foo::consts::magic_number << '\n';
    std::cout << bar::consts::magic_number << '\n';
}
```

### A rarely used namespace feature: `inline`

`inline` is an optional keyword that we can use alongside the `namespace` keyword. If a `namespace` is marked `inline`, then its members will be treated in many situations as if they are part of the enclosing namespace. Both the below calls are correct.

```cpp
#include <iostream>
#include <string>

namespace foo {
    inline namespace consts {
        constexpr int magic_number = 42; 
    }
}

int main() {
    std::cout << foo::consts::magic_number << '\n';
    std::cout << foo::magic_number << '\n';
}
```

The `inline` attribute of a namespace is transitive, so if within an `inline namespace` there is another one, you can use the most nested member with the outermost namespace. As you can see, using nested `inline` namespaces can lead to quite a mess.

```cpp
#include <iostream>
#include <string>

namespace foo {
    inline namespace bar {
        inline namespace consts {
            constexpr int magic_number = 42; 
        }
    }
}

int main() {
    std::cout << foo::bar::consts::magic_number << '\n';
    std::cout << foo::bar::magic_number << '\n';
    std::cout << foo::consts::magic_number << '\n';
    std::cout << foo::magic_number << '\n';
}
```

At least the compiler is aware of it and it doesn't let us turn the situation into a run-time nightmare:

```cpp
#include <iostream>
#include <string>

namespace foo {
    inline namespace bar {
        inline namespace consts {
            constexpr int magic_number = 42; 
        }
        constexpr float magic_number = 3.14; 
    }
}

int main() {
    std::cout << foo::bar::consts::magic_number << '\n';
    std::cout << foo::bar::magic_number << '\n';
    std::cout << foo::consts::magic_number << '\n';
    std::cout << foo::magic_number << '\n';
}
/*
<source>: In function 'int main()':
<source>:15:28: error: reference to 'magic_number' is ambiguous
   15 |     std::cout << foo::bar::magic_number << '\n';
      |                            ^~~~~~~~~~~~
...
<source>:17:23: error: reference to 'magic_number' is ambiguous
   17 |     std::cout << foo::magic_number << '\n';
      |                       ^~~~~~~~~~~~
*/
```

Without the `inline` modifier, our code is perfectly valid:

```cpp
#include <iostream>
#include <string>

namespace foo {
    namespace bar {
        namespace consts {
            constexpr int magic_number = 42; 
        }
        constexpr float magic_number = 3.14; 
    }
}

int main() {
    std::cout << foo::bar::consts::magic_number << '\n';
    std::cout << foo::bar::magic_number << '\n';
    // ERROR: without inlining, we cannot use such "shortcuts"
    // std::cout << foo::consts::magic_number << '\n'; 
    // std::cout << foo::magic_number << '\n';
}
/*
42
3.14
*/
```

In my 11 years of C++ journey, I never directly used `inline namespace`s and up until recently I haven't even encountered them. I found them while I was looking for a solution to some kind of a versioning issue and apparently, this feature is mostly used for library versioning.

One can store the latest version of an API behind an `inline` nested namespace. Those who are on the latest version, they don't have to use a version number. But those, who for some reason cannot migrate, they have the option to explicitly use an older version.

```cpp
#include <iostream>
#include <string>

namespace foo {
    namespace v1 {
        constexpr int magic_number = 42; 
    }
    inline namespace v2 {
        constexpr float magic_number = 3.14; 
    }
}

int main() {
    // using the latest version
    std::cout << foo::magic_number << '\n';
    std::cout << foo::v2::magic_number << '\n'; // this also works

    // using an older version
    std::cout << foo::v1::magic_number << '\n';
}
/*
3.14
3.14
42
*/
```

You can find more details about this feature [on Jonathan's blog post called Inline Namespaces 101](https://www.foonathan.net/2018/11/inline-namespaces/).

Now let's move on to discuss best practices.

## General C++ namespace best practices

Using a namespace in general to organize a codebase is already a best practice. With that in mind, we should minimize the use of the global namespace. Allowing code to reside in the global namespace can lead to naming conflicts and make code maintenance more difficult than it should be. Instead, encapsulate your code in (well-)named namespaces.

### What names to use?

First of all, use meaningful and descriptive names for namespaces. Well-chosen namespace names make the code easier to understand and easier to maintain.

Then use a consistent style. A style that matches the rest of the project. If you are using something new, I recommend writing the namespace names in *snake_case* as that's exactly what the standard library does (e.g. `std::literals::string_literals`) and that's what the most widely used style guide, the [Google Style Guide](https://google.github.io/styleguide/cppguide.html#Namespace_Names) recommends as well.

Try to avoid using abbreviations, unless they are evident. But more often than not even if you think that an abbreviation is straightforward, probably it's not.

### Use `using` declarations as much as you need, but only in .cpp files

The use of *namespace using directives* (e.g. `using namespace std;`) must be avoided in header files as [they extend the surrounding namespace with the namespace being “used”](https://godbolt.org/z/T9zsh3zha). This can lead to naming conflicts and ambiguous lookups.

```cpp
#include <iostream>
#include <string>

namespace foo {
    using namespace std;  // #F
    void func(string s) { // #D
        cout << s << '\n';
    }
}

void func(std::string s) {. // #B
    std::cout << "global func " << s << '\n';
}

namespace bar {
    using namespace foo; // #C 

    void fun() {
        string g{"oh no"}; // #E
        func(g);  // #A
    }
}

int main() {
    bar::fun();
}
```

If you look at the above example, first it will fail at *#A*, because calling `func(std::string)` is ambiguous. It can be the global `func` (*#B*), but after having the `use namespace foo;` using namespace directive at *#C*, it could be `foo::func` as well (*#D*).

If we decide to remove `using namespace foo` at *#C*, we will face another compilation error, even earlier. `string g{"oh no}"` doesn't compile anymore because by not using `foo`, we are not using `std` either (*#F*)...

Though using *namespace using directives* is not so much of a problem in a `.cpp` file, it’s still better to avoid it as it can make it difficult to figure out for the readers where certain symbols come from.

Instead of *using directives*, use *using declarations* (e.g., `using namespace foo::const_member::magic_number;`) to bring specific elements from a namespace into the current scope. This reduces ambiguity and clearly specifies which items you're using. Still, only use this in `.cpp` files as just like using directives, they introduce the used namespace into the surrounding namespace and that might be propagated.

If you still do it in the header, limit the scope and never use it in the global namespaces.

What I write here is a bit stricter than what the Core Guidelines say at [SF.6](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#sf6-use-using-namespace-directives-for-transition-for-foundation-libraries-such-as-std-or-within-a-local-scope-only) and [SF.7](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#sf6-use-using-namespace-directives-for-transition-for-foundation-libraries-such-as-std-or-within-a-local-scope-only) as it mostly protects the global namespace, but I think it makes sense to go further and protect our user-defined namespaces as well - and that's what we did in almost every project that I worked on.

### Avoid using namespace aliases, especially in header files

It’s tempting to use alias namespaces (`namespace foo::bar::baz = fbb;`) in order to simplify long or complex names, but do so sparingly. While it can make code more concise, overusing aliases can lead to ambiguity and confusion.

Like using directives and using declarations, aliases must also be avoided in header files, except in explicitly marked internal-only namespaces, because anything imported into a namespace in a header file becomes part of the public API exported by that file.

If alias namespaces are used in header files, we can easily end up with the same problem as namespaces try to solve: naming conflicts and ambiguity. If multiple namespaces or alias names are similar or if multiple aliases bring in the same names, it can lead to compilation errors and difficulties in determining the source of the referenced name.

Besides, namespace aliases can also hinder code maintainability. When alias names are cryptic or excessively used, it becomes challenging for developers to understand where identifiers are defined, which namespace they belong to, and the relationships between different parts of the code. Moreover, when the structure of namespaces changes, alias names may become misleading or incorrect. If you rely heavily on alias names, adapting to changes in the underlying namespaces can be cumbersome.

### Always fully qualify namespaces in preprocessor macros

While preprocessor macros seemingly might live within namespaces, that’s just a mirage. Macros are being replaced before the compilation and the preprocessor knows nothing about namespaces.

```cpp
#include <iostream>

namespace foo {

    #define MM(X) (X * magic_number)

    constexpr int magic_number = 42;

    int bar(int x) { return MM(x);}
}

int main() {
    std::cout << foo::bar(42) << '\n';
    std::cout << MM(42) << '\n'; // error: 'magic_number' was not declared in this scope;
}
```

In the above example, `MM` is seemingly part of the namespace `foo` and therefore it should always know about `foo::magic_number`, but that's not the case. If `MM` is used outside of `foo`, it won't work.

This means that to avoid potential issues with name lookups if you really want to use a macro, always fully qualify symbols in macros if they are public.

```cpp
#define MM(X) (X * ::foo::magic_number)
```

At the same time, we’d highly encourage anyone [not using macros given that they are error-prone](https://arne-mertz.de/2019/03/macro-evil/) and with modern C++ techniques, there is barely any need for them.

Oh, and I hope you didn't even have the idea, but don't use macros to create namespace aliases.

```cpp
#define NS MyNamespace  // Avoid using macros for this purpose
```

### Use internal linkage only in source files ([SF.21](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#sf21-dont-use-an-unnamed-anonymous-namespace-in-a-header), [SF.22](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#sf22-use-an-unnamed-anonymous-namespace-for-all-internalnon-exported-entities))

When you have functions or variables that are not supposed to be used outside of a source file, give them internal linkage either by placing them in unnamed namespaces or [by declaring them `static`](https://www.sandordargo.com/blog/2021/07/07/2-ways-to-use-static-with-functions-cpp).

Don’t use these constructs in header files. There are various good reasons for avoiding that:
Internal linkage is often used for implementation details. Let’s not expose implementation details via header files.

The [C++ One Definition Rule](https://en.wikipedia.org/wiki/One_Definition_Rule) states that an entity should have only one definition in the entire program. When you include a header file with internal linkage in multiple source files, each translation unit gets its own version of the internal linkage entity. This can lead to violations of the ODR and result in linker errors.


### Avoid deep dependencies and use them together with physical separation of source files

While organizing your code with namespaces is definitely a good idea, it matters how you do it. Just like with directories. If you end up having a too deep directory structure, it will become difficult to navigate, difficult to use and share paths. It's similar with namespaces. Don't have too deep nestings, names will be too long, your code will be difficult to read either because of the two long fully qualified names or because of the *aliases* and *using declarations* you'll need to keep the code short.

Based on experience, not having more than 3 levels of nested namespaces is a good solution.

You might be tempted to follow your directory structure with the namespaces. I find it a good idea, even if they don't follow exactly the same logic.

If you organize your codebase around libraries and your namespaces help you identify the usage of your libraries, then dependency management becomes a little bit easier. Whenever you have to "open up" a foreign namespace (foreign to your current scope), you know that you're dealing with a dependency that will both make your compilation longer and probably that will make your code recompile more frequently. So you can think about whether you really need that dependency or if there is a smarter way. But that's beyond the scope of this article.

What we should remember at this point is that use not too many nested namespaces and directories to organize your code and if they follow the organization of your deliverables, they will help you manage your dependencies.

## Conclusion

In this article, we learned about namespaces. They help organise code in the first place and to have further options, there are also inline namespaces that really come in handy for versioning APIs. 

There are many best practices to keep in mind when it comes to using namespaces and related features. A big chunk of best practices is about avoiding namespace pollution. We should avoid using namespace declarations and even directives in the header as they are extending the enclosing namespaces.

At the same time, we should not forget that namespaces are not the only way to organize code and we often use a physical separation as well by placing C++ code files into different directories. These different approaches should be used together in a coherent way to keep your codebase tidy both from a physical (file organization) and logical (namespace organization) standpoint.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

