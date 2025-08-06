---
layout: post
title: "Avoiding Undefined Behaviour with BoostTests and standard types"
date: 2025-8-6
category: dev
tags: [namespaces, undefinedbehaviour, boost, std]
excerpt_separator: <!--more-->
---
Maybe the title is a bit clickbaity — sorry for that. But I've seen many people unintentionally introduce undefined behaviour (UB) into their code through Boost unit tests. Sure, it's "only" in the unit tests, and it probably won't hurt. Still, undefined behaviour is best avoided wherever possible.

Let's first look at what causes this undefined behaviour, then why it happens so often, and finally how to fix it without compromising on safety or quality.

## Extending the namespace std is (usually) UB

You might not have come across this before, but adding declarations to `namespace std` — or any of its nested namespaces — is undefined behaviour in almost all circumstances.

There are rare exceptions. For example, specializing `std::hash` for your custom types is allowed. But adding new function definitions — like the one below — is definitely not:

```cpp
// Beware, this is Undefined Behaviour!
namespace std {

template <typename T>
std::ostream &operator<<(std::ostream &os, const optional<T> &value) {
  if (value.has_value()) {
    return os << "std::optional{" << value.value() << "}";
  }
  return os << "std::nullopt";
}

}  // namespace std
```

## Why would you add definitions to `std` - when using boost?

A compelling reason might be to make the unit tests build.

Take a minimal test example:

```cpp
BOOST_AUTO_TEST_CASE(FooBarTest) {
  // Test implementation for FooBar
  std::optional<std::string> expected_value = "foobar";
  std::optional<std::string> actual_value = "foobar";
  BOOST_TEST(expected_value == actual_value);
}
```

I know this test makes no sense, but it's enough for demonstration purposes.

It doesn't compile.

As is typical with C++ templates, the error message is long, but it starts like this:

```
external/boost/boost/test/tools/detail/print_helper.hpp:58:39: error: static assertion failed due to requirement 'boost::has_left_shift<std::ostream, std::optional<std::string>, boost::binary_op_detail::dont_care>::value': Type has to implement operator<< to be printable
   58 |             BOOST_STATIC_ASSERT_MSG( (boost::has_left_shift<std::ostream,T>::value),
```

In plain English: `std::optional<std::string>` must implement `operator<<` to be printable.

So you add the "innocent-looking" snippet from earlier that defines `operator<<` for `std::optional<T>`, your tests build and pass... and you've just stepped into undefined behaviour territory.

## What to do instead within boost boundaries?

Thankfully, there's a safe and Boost-compliant way to solve this issue without modifying your production code just to make the tests compile.

> Wrapping standard types and implementing `operator<<` for those wrappers could work, but it's intrusive, so I don't recommend it.

Instead of `namespace std`, specialize `print_log_value` in `namespace boost::test_tools::tt_detail`.

```cpp
namespace boost::test_tools::tt_detail {

template <typename T>
struct print_log_value<std::optional<T>> {
  void operator()(std::ostream &os, const std::optional<T> &value) {
    if (value.has_value()) {
      os << "std::optional{";
      print_log_value<T>()(os, *value);
      os << "}";
    } else {
      os << "std::nullopt";
    }
  }
};

}  // namespace boost::test_tools::tt_detail
```
This way, you make `std::optional` printable for Boost without violating the One Rule to Avoid: **don't mess with `namespace std`**.

You might also consider implementing the `boost_test_print_type` free function — but in my experience, I couldn't make it compile for standard types without putting the definition inside `namespace std`, which brings us back to UB land.

## Conclusion

Adding `operator<<` to `std::optional` might seem like a harmless fix to make your Boost tests work — but it's a kind of a trap. Modifying `namespace std` leads to undefined behaviour, even if things appear to work.

The right solution is to stay within Boost's own extension points, like `boost::test_tools::tt_detail::print_log_value`. It's just as effective, and it keeps your code standards-compliant and future-proof.

Always be mindful of what you're modifying — even in test code.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!