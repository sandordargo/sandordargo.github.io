---
layout: post
title: "C++26 Reflection: All You Need To Know About std::meta::info"
date: 2025-X-X
category: dev
tags: [cpp, cpp26, reflection, syntax]
excerpt_separator: <!--more-->
---
In the previous parts of our C++ Reflection journey, we explored [the motivations for C++ Reflection]() and [the basics of the syntax](), as well as the essentials you need to know about [`std::meta::info`]().  

Now it's time to dig deeper into how to work with this type.

## Working with `std::meta::info`

We've already seen some short examples of operations on reflection values — instances of `std::meta::info`. Here's one:

```cpp
constexpr int i = 42, j = 42;

static_assert(^^i != ^^j);  // 'i' and 'j' are different entities.
static_assert(constant_of(^^i) == constant_of(^^j));    // Two equivalent values.
```
This isn't just an exception to the rule — it is the rule.

Instead of invoking member functions on a reflection value, we use **metafunctions**. These are free functions that operate on `std::meta::info` and return other reflection values, types, or constants.

Metafunctions are `consteval`, meaning they are evaluated only at compile time. In C++23, constant evaluation produces pure values without observable side effects, so the order of evaluation didn't matter.

However, some reflection metafunctions do have visible and desirable compile-time side effects. For example, `define_aggregate` can provide a class definition during compilation.

Because of this, [P2996](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#constant-evaluation-order) introduced important but intuitive changes to constant evaluation rules:
- consteval blocks must result in a valid C++ program.
- They must be evaluated exactly once during translation.
- They must be evaluated in lexical order.

This makes reasoning about compile-time code more predictable, especially when reflection is involved.

## Error handling

But what if a metafunction call doesn't result in a valid program?

The proposal adopts `constexpr` exceptions for error handling during constant evaluation.

> I've discussed *constexpr exceptions* in more detail here: [C++26: Constexpr Exceptions](https://www.sandordargo.com/blog/2025/05/07/cpp26-constexpr-exceptions).

While runtime exceptions face criticism for performance or binary size concerns, `constexpr` exceptions are different — they never reach runtime. Any uncaught `constexpr` exception simply means the program fails to compile.

Interestingly, [P2996](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r12.html#error-handling-in-reflection) does not specify any standard metafunctions that throw such exceptions. Details came in a separate proposal, [P3560](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p3560r2.html). It details the exception type used and when metafunctions throw.

Any exception thrown by a `std::meta::*` type or function will be of type: `std::meta::exception`. Here is the interface:

```cpp
namespace std::meta {

class exception : public std::exception
{
public:
    consteval exception(u8string_view what,
                        info from,
                        source_location where = source_location::current());
    consteval exception(string_view what,
                        info from,
                        source_location where = source_location::current());

    constexpr char const* what() const noexcept override;
    consteval u8string_view u8what() const noexcept;
    consteval info from() const;
    consteval source_location where() const;
};

}
```

As you can see, it pretty much looks like a normal exception type - for consistency reasons it even inherits from `std::exception` -, except that it's all `consteval`
with the exception of `what` which is `constexpr`.

There is something important in there though. Look at the the `from()` member function returning `std::meta::info`. There is only one exception type in `std::meta`, there is no hierarchy, so you have to write only one `catch` block. Within that, you can use `from()` to handle different kind of issues:

```cpp
catch(const std::meta::exception & ex)
{
    if(x.from() == ^^identifier_of)
    {
        // handle errors originating from identifier_of
    }
    else if(x.from() == ^^members_of)
    {
        // handle errors originating from members_of
    }
    // ...
}
``` 

## The list of standard metafunctions

Here's a consolidated list of all the proposed metafunctions accepted into C++26 so far.
For reference, I've included comments indicating which paper introduced them.

```cpp
namespace std::meta {
  // P2996
  using info = decltype(^^::);

  template <typename R>
    concept reflection_range = /* see above */;

  // name and location
  consteval auto identifier_of(info r) -> string_view;
  consteval auto u8identifier_of(info r) -> u8string_view;

  consteval auto display_string_of(info r) -> string_view;
  consteval auto u8display_string_of(info r) -> u8string_view;

  consteval auto source_location_of(info r) -> source_location;

  // type queries
  consteval auto type_of(info r) -> info;
  consteval auto parent_of(info r) -> info;
  consteval auto dealias(info r) -> info;

  // object and constant queries
  consteval auto object_of(info r) -> info;
  consteval auto constant_of(info r) -> info;

  // template queries
  consteval auto template_of(info r) -> info;
  consteval auto template_arguments_of(info r) -> vector<info>;

  // member queries
  consteval auto members_of(info r) -> vector<info>;
  consteval auto bases_of(info type_class) -> vector<info>;
  consteval auto static_data_members_of(info type_class) -> vector<info>;
  consteval auto nonstatic_data_members_of(info type_class) -> vector<info>;
  consteval auto enumerators_of(info type_enum) -> vector<info>;

  // substitute
  template <reflection_range R = initializer_list<info>>
    consteval auto can_substitute(info templ, R&& args) -> bool;
  template <reflection_range R = initializer_list<info>>
    consteval auto substitute(info templ, R&& args) -> info;

  // reflect expression results
  template <typename T>
    consteval auto reflect_constant(const T& value) -> info;
  template <typename T>
    consteval auto reflect_object(T& value) -> info;
  template <typename T>
    consteval auto reflect_function(T& value) -> info;

  // extract
  template <typename T>
    consteval auto extract(info) -> T;

  // other type predicates (see the wording)
  consteval auto is_public(info r) -> bool;
  consteval auto is_protected(info r) -> bool;
  consteval auto is_private(info r) -> bool;
  consteval auto is_virtual(info r) -> bool;
  consteval auto is_pure_virtual(info r) -> bool;
  consteval auto is_override(info r) -> bool;
  consteval auto is_final(info r) -> bool;
  consteval auto is_deleted(info r) -> bool;
  consteval auto is_defaulted(info r) -> bool;
  consteval auto is_explicit(info r) -> bool;
  consteval auto is_noexcept(info r) -> bool;
  consteval auto is_bit_field(info r) -> bool;
  consteval auto is_enumerator(info r) -> bool;
  consteval auto is_const(info r) -> bool;
  consteval auto is_volatile(info r) -> bool;
  consteval auto is_mutable_member(info r) -> bool;
  consteval auto is_lvalue_reference_qualified(info r) -> bool;
  consteval auto is_rvalue_reference_qualified(info r) -> bool;
  consteval auto has_static_storage_duration(info r) -> bool;
  consteval auto has_thread_storage_duration(info r) -> bool;
  consteval auto has_automatic_storage_duration(info r) -> bool;
  consteval auto has_internal_linkage(info r) -> bool;
  consteval auto has_module_linkage(info r) -> bool;
  consteval auto has_external_linkage(info r) -> bool;
  consteval auto has_linkage(info r) -> bool;
  consteval auto is_class_member(info r) -> bool;
  consteval auto is_namespace_member(info r) -> bool;
  consteval auto is_nonstatic_data_member(info r) -> bool;
  consteval auto is_static_member(info r) -> bool;
  consteval auto is_base(info r) -> bool;
  consteval auto is_data_member_spec(info r) -> bool;
  consteval auto is_namespace(info r) -> bool;
  consteval auto is_function(info r) -> bool;
  consteval auto is_variable(info r) -> bool;
  consteval auto is_type(info r) -> bool;
  consteval auto is_type_alias(info r) -> bool;
  consteval auto is_namespace_alias(info r) -> bool;
  consteval auto is_complete_type(info r) -> bool;
  consteval auto is_enumerable_type(info r) -> bool;
  consteval auto is_template(info r) -> bool;
  consteval auto is_function_template(info r) -> bool;
  consteval auto is_variable_template(info r) -> bool;
  consteval auto is_class_template(info r) -> bool;
  consteval auto is_alias_template(info r) -> bool;
  consteval auto is_conversion_function_template(info r) -> bool;
  consteval auto is_operator_function_template(info r) -> bool;
  consteval auto is_literal_operator_template(info r) -> bool;
  consteval auto is_constructor_template(info r) -> bool;
  consteval auto is_concept(info r) -> bool;
  consteval auto is_structured_binding(info r) -> bool;
  consteval auto is_value(info r) -> bool;
  consteval auto is_object(info r) -> bool;
  consteval auto has_template_arguments(info r) -> bool;
  consteval auto has_default_member_initializer(info r) -> bool;

  consteval auto is_special_member_function(info r) -> bool;
  consteval auto is_conversion_function(info r) -> bool;
  consteval auto is_operator_function(info r) -> bool;
  consteval auto is_literal_operator(info r) -> bool;
  consteval auto is_constructor(info r) -> bool;
  consteval auto is_default_constructor(info r) -> bool;
  consteval auto is_copy_constructor(info r) -> bool;
  consteval auto is_move_constructor(info r) -> bool;
  consteval auto is_assignment(info r) -> bool;
  consteval auto is_copy_assignment(info r) -> bool;
  consteval auto is_move_assignment(info r) -> bool;
  consteval auto is_destructor(info r) -> bool;
  consteval auto is_user_provided(info r) -> bool;
  consteval auto is_user_declared(info r) -> bool;

  // define_aggregate
  struct data_member_options;
  consteval auto data_member_spec(info type_class,
                                  data_member_options options) -> info;
  template <reflection_range R = initializer_list<info>>
    consteval auto define_aggregate(info type_class, R&&) -> info;

  // data layout
  struct member_offset {
    ptrdiff_t bytes;
    ptrdiff_t bits;
    constexpr auto total_bits() const -> ptrdiff_t;
    auto operator<=>(member_offset const&) const = default;
  };

  consteval auto offset_of(info r) -> member_offset;
  consteval auto size_of(info r) -> size_t;
  consteval auto alignment_of(info r) -> size_t;
  consteval auto bit_size_of(info r) -> size_t;

  // P3394, annotations
  consteval bool is_annotation(info r);
  consteval vector<info> annotations_of(info item);
  consteval vector<info> annotations_of_with_type(info item, info type);

  // P3293, base class subobjects
  consteval bool has_inaccessible_subobjects(info r, access_context ctx);
  consteval vector<info> subobjects_of(info type, access_context ctx);

  // P3491, define_static_{string,object,array}
  template <ranges::input_range R>
  consteval info reflect_constant_string(R&& r);

  template <ranges::input_range R>
  consteval info reflect_constant_array(R&& r);

  // P3096, function parameter reflection
  consteval bool is_function_parameter(info r);
  consteval bool is_explicit_object_parameter(info r);
  consteval bool has_default_argument(info r);
  consteval bool has_ellipsis_parameter(info r);
  consteval vector<info> parameters_of(info r);
  consteval info variable_of(info r);
  consteval info return_type_of(info r);

}
```

## Conclusion

`std::meta::info` is at the heart of C++26 reflection — but it's only half the story. You can't do much with a reflection value unless you also know the right metafunctions to apply to it.

In this article, we saw:
- Why metafunctions are free functions, not member functions.
- How `consteval` evaluation order now matters for reflection.
- The role of compile-time exceptions in error handling.
- A complete reference list of standard metafunctions proposed for C++26.

In the next articles, we'll dive into these metafunctions group by group — starting with name and type queries — and see how they can be applied to real-world use cases like serialization, code generation, and compile-time validation.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!