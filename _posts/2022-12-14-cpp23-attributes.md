---
layout: post
title: "C++23: attributes"
date: 2022-12-14
category: books
tags: [cpp23, cpp, attributes, lambdas]
excerpt_separator: <!--more-->
---
C++11 introduced attributes. Even though the language itself only came with two (`[[noreturn]]` and `[[carries_dependency]]`), it also provided the unified standard syntax for *implementation-defined language extensions/. Ever since that version, each new release brought us some new standard attributes or changes in the ways we can use them.

The list of changes continues with C++23. Three papers affect attributes. Duplications will not be punished anymore, their usage with lambdas evolves and there is also a new standard attribute `[[assume]]`. Let's have a deeper look.

## DR: Allow Duplicate Attributes

So far the standard didn't allow that an attribute such as `[[nodiscard]]` appear more than once in an attribute list.

Wait? Isn't that a good thing? You might ask these questions, and that's exactly what I asked myself when I saw this DR proposal.

> When you see `DR` in a proposal title, it means that it's a remediation for a *"defect report"* that was filed against the standard.

Defining the same attribute more than once is not a good practice. It's something you shouldn't do as a developer. But according to the paper [P2156R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p2156r1.pdf), it can happen that a macro-based solution generates the same attribute more than once.

Specifying that attributes shouldn't appear more than once is a constraint for such a solution. At the same time, it doesn't make standardization easier. On the contrary, removing this constraint, makes the standard easier! So far, it was specified for each standard attribute that *"it shall appear at most once in each attribute list"*. From now on, they can and that constraint is removed from the text. 

## Attributes on lambdas

After reading this section title, you might ask whether it's already possible to use attributes with lambdas. The answer is yes, it is possible. The place for adding an attribute is in the lambda declarator, either before or after the parameter declaration clause, but always between the optional noexcept specifiers and trailing return types.

The attributes sequence belongs to the **type** of the corresponding function call operator. The paper [P2173R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2173r1.pdf) argues - and rightly so - that this should not necessarily be the case. Attributes should be allowed to belong to the function call operator.

After all, why couldn't the operator and with that almost always the lambda be `[[nodiscard]]`, `[[noreturn]]` or `[[deprecated]]`?

According to the proposed and accepted change, regardless of the other optional elements of a lambda expression, now we can declare the attribute sequence right after the lambda introducer (and its optional capture) or right after the template parameter list including its requires clause.

The proposed wording says that an attribute specifier sequence in a lambda declarator appertains to the type of the function call operator, but if it comes before the lambda declarator, then it belongs to the function call operator itself, not its type. You might say that you cannot attach a `[[nodiscard]]` attribute to a type, it wouldn't make sense. You're right, but don't think only about the standard attributes, even the proposal mentions vendor-defined ones.

With [this change](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2173r1.pdf), the following piece of code becomes valid, where the function call of the operator is `[[nodiscard]]`:

```cpp
auto lm = [][[nodiscard]]()->int { return 42; };
```

Meaning that this would emit a warning:

```cpp
auto lm = [][[nodiscard]]()->int { return 42; };
// ...
lm(); // warning: ignoring return value of 'main()::<lambda()>', declared with attribute 'nodiscard' [-Wunused-result]
```

Please note that GCC and Clang already implemented this behaviour, GCC already in version 9! 

## Attribute `[[assume]]`

The new attribute `[[assume]]` is the standardized version of some already existing compiler-specific attributes, such as `__assume()` (MSVC, ICC) and `__builtin_assume()` (Clang). With their help, the programmer can signal to the compiler that an expression will be true without the compiler having to evaluate it.

> On gcc, an alternative was not available, so probably that's why they are the first, who implemented the standardized `[[assume]]` in version 13. If you want to emulate the same functionality before you have to use `if(expr) {} else {__builtin_unreachable();}`.

As such, the compiler can generate more efficient, faster code. This can bring significant benefits in some high-performance, low-latency explications.

It's syntax is `[[assume(expr)]]`. It's worth noting that the expression is not evaluated. Which is a change compared to the gcc simulation, but is the same as how clang and msvc already worked.

The [accepted paper](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1774r8.pdf) brings us quite a few examples when and how to use `[[assume]]`. The first one is an example from audio processing, and it uses information that is commonly known by the developers, but it's not represented in the code:

```cpp
void limiter(float* data, size_t size) {
    [[assume(size > 0)]];
    [[assume(size % 32 == 0)]];
    
    for (size_t i = 0; i < size; ++i) {
        [[assume(std::isfinite(data[i])]];
        data[i] = std::clamp(data[i], -1.0f, 1.0f);
    }
}
```
I first overlooked the first assumption, but Thief in the comments and Julien in an e-mail gently pointed out my mistake. The first assumption is that the parameter `size` will never be zero, but it will always be a positive number!

The second example hints to the compiler that the passed in size will always be the multiple of 32 and with the last assumption, we even tell that the data is not *NaN* or *infinity*.

In another example, the copy constructor of a reference counting shared pointer got the assumption, that the refcount is already at least one. Makes sense, it's a copy constructor. This could also help the compiler to ignore some increments and decrements and avoid destroying the owned resource when the passed in smart pointer's lifecycle ends. Interestingly, it didn't lead to great optimizations on some of the compilers.

So I'd say, measure, measure and measure before you assume some gains.

On most compilers, this leads to a significant performance increase (for the details, please check paper [P1774R8](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1774r8.pdf).

But what if an assumption does not hold true?

It's important to emphasize that assumptions are not evaluated! They are not checked! Instead, the compiler assumes that the expression **would** evaluate to true and optimize accordingly. If the assumption does not hold, so if the assumption returns false, throws an exception or if it's UB, the behaviour will be undefined. Use it with care.

## Conclusion

C++23 brings 3 changes to attributes. The restriction that an attribute cannot be duplicated is removed now. Attributes can be used with lambdas in a way that it belongs to the function call operator from now on, not to the lambda itself. Last but least, we get a new standard attribute `[[assume]]` to ease compiler optimizations.

Do you often use attributes?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
