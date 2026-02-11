---
layout: post
title: "Deferred member initialization"
date: 2026-2-11
category: dev
tags: [cpp, const, optional, initialization]
excerpt_separator: <!--more-->
---
At Meeting C++ 2025, I had an interesting discussion with another attendee. Here's the problem we talked about:

> *There is a class controlling a piece of hardware to which several other hardware modules could be installed. To manage the available modules, we want to pass a mapping to this class where the map would contain hardware IDs and corresponding module names.*
>
> *A perfect fit for a `map<int, string>`!*
>
> *Since the available modules would never change at runtime, the map should ideally be `const`.*
>
> *However, due to some hard constraints, the map cannot be initialized at construction time. It can happen later through an `init` function.*

Let's explore a few possible solutions—and I’d love to hear your thoughts too.

## A well-encapsulated non-`const` member

You might argue that there's no need for `_available_modules` to be `const`. As long as it's `private`, not exposed through getters, and not modified after initialization, you could rely on discipline and convention. After all, as the owner of the `MyHardwareController` class, you'll ensure no code will ever modify it again.

```cpp
// https://godbolt.org/z/Kbx55nn73
#include <iostream>
#include <map>
#include <optional>
#include <string>

std::map<int, std::string> list_available_modules() {
    return { {1, "widget"}, {2, "gadget"}, {42, "bar"} };
}

class MyHardwareController {
   public:
    void init() { _available_modules = list_available_modules(); }

    std::optional<std::string> get_module_name(int id) const {
        if (_available_modules.contains(id)) {
            return _available_modules.at(id);
        }
        return std::nullopt;
    }

   private:
    std::map<int, std::string> _available_modules;
};

int main() {
    MyHardwareController bar;
    bar.init();

    for (int id : {1, 2, 3, 42}) {
        std::cout << "Module " << id << " is named "
                  << bar.get_module_name(id).value_or("<unknown>") << ".\n";
    }

    return 0;
}
```

To make it safer, you can ensure that `init()` cannot be called twice — either by returning early, signaling an error, or throwing an exception. While some embedded environments avoid exceptions, [it's still worth reconsidering whether that restriction truly applies](https://www.youtube.com/watch?v=bY2FlayomlE).

```cpp
// https://godbolt.org/z/549qqrj9c
class MyHardwareController {
   public:
    void init() { 
        if (_already_initialized) {
            throw std::logic_error{"Object already initialized"};
        }
        _available_modules = list_available_modules(); 
        _already_initialized = true;
    }

    // ... rest of the class

   private:
    bool _already_initialized {false};
    std::map<int, std::string> _available_modules;
};
```

The downside? Nothing in the type system explicitly communicates that `_available_modules` should never be modified after initialization. It's a convention, not an enforced guarantee.

## Use an `optional<const map>`

Instead of a plain, mutable map, we can use an `optional<const map>`. What does this express to the reader?

It says: *there might or might not be a map yet — but once it exists, it cannot be modified*.

That's a strong and useful semantic. You can still replace the entire map using `std::optional::emplace()`, but the underlying data remains immutable.

```cpp
// https://godbolt.org/z/hcjsvKnGd
class MyHardwareController {
   public:
    void init() { 
        if (_already_initialized) {
            throw std::logic_error{"Object already initialized"};
        }
        _available_modules.emplace(list_available_modules()); 
        _already_initialized = true;
    }

    std::optional<std::string> get_module_name(int id) const {
        if (_available_modules->contains(id)) {
            return _available_modules->at(id);
        }
        return std::nullopt;
    }

   private:
    bool _already_initialized {false};
    std::optional<std::map<int, std::string>> _available_modules;
};
```

This solution is already quite robust — it enforces immutability once initialized, yet still allows deferred setup. For many cases, it's more than good enough.

## Have a registry class

But can we communicate our intent even better?

Let's encapsulate the map in a small helper class, `ModuleRegistry`, responsible for enforcing single initialization and `const` access. This separation makes the ownership and lifecycle of the data explicit.

```cpp
class ModuleRegistry {
   public:
    void set_once(std::map<int, std::string> m) {
        if (modules) {
            throw std::logic_error("Modules already initialized");
        }
        modules = std::move(m);
    }

    bool is_initialized() const { return modules.has_value(); }
    const std::map<int, std::string>& get_modules() const {
        if (!modules) {
            throw std::logic_error("Modules not initialized yet");
        }
        return modules.value();
    }

   private:
    std::optional<std::map<int, std::string>> modules;
};
```

Then, our controller simply delegates initialization and access:

```cpp
class MyHardwareController {
   public:
    void init() { 
        if (_module_registry.is_initialized()) {
            throw std::logic_error{"Module registry already initialized"};
        }
        _module_registry.set_once(list_available_modules()); 
    }

    std::optional<std::string> get_module_name(int id) const {
        if (_module_registry.is_initialized() && _module_registry.get_modules().contains(id)) {
            return _module_registry.get_modules().at(id);
        }

        return std::nullopt;
    }

   private:
    ModuleRegistry _module_registry;
};
```

In this setup, `ModuleRegistry` fully encapsulates the deferred initialization logic. It cannot be partially modified or reused incorrectly. You can even `delete` assignment operators to make replacement impossible — [just remember the rule of five](https://www.sandordargo.com/blog/2024/07/31/rule-of-5-once-again).

[See the full example on Compiler Explorer.](https://godbolt.org/z/nhb9nedvf)

## Conclusion

Deferred initialization is a tricky balance between expressiveness, safety, and practicality.
If runtime constraints prevent you from initializing const data at construction time, there are still clean ways to express your intent:

- A **private** non-`const` member works but relies on discipline.
- An `optional<const T>` clearly communicates immutability after initialization.
- A dedicated **registry or wrapper** class adds even stronger guarantees and separation of concerns.

In the end, the best solution depends on your context. But the key takeaway is this:
**even when constraints prevent you from using `const` directly, you can still design for immutability**.

That mindset — of expressing intent through types - is one of the most powerful tools in modern C++.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

