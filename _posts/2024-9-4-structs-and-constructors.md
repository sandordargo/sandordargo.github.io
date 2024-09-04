---
layout: post
title: "Structs and constructors"
date: 2024-9-4
category: dev
tags: [cpp, structs, binarysize, constructors]
excerpt_separator: <!--more-->
---
Today, we are going to talk about when and why structs should have constructors if they should have them at all. We are also going to see once again that generic best practices and best practices to reduce binary size do not always go hand in hand.

I had some time to dedicate to cleaning up code. I remembered that recently I saw some structs like this:

```cpp
SomeSimpleStuct data;
data.a_number = 42;
data.a_character = 'a';
return data;
```

I didn't like this code. Even though the struct didn't have any constructors, even though not all the members were set, with the help of [designated initializers](https://en.cppreference.com/w/cpp/language/aggregate_initialization#Designated_initializers) it's so easy to get rid of the initialize then modify anti-pattern [Connor Hoekstra talked about a couple of years ago at the Italian C++ conference](https://www.youtube.com/live/CjHgL5EQdcY?si=0QEQc_uRH9gsb9io).

```cpp
return SomeSimpleStuct{.a_number = 42, .a_character = 'a'};
```

As a result, you get code that still tells you what is initialized, code which is more efficient without making it less readable and in addition you don't have to think about why the object is created before its members are set.

Next, I checked some code where I did see some non-default constructors, probably because people wanted to avoid to previous pattern.

```cpp
struct PileOfData {
  PileOfData() = default;
  explicit PileOfData(const std::string &type, const std::string &identifier = "")
      : type(type), identifier(identifier) {}


  std::string type;
  std::string identifier;
};

// Somewhere else...
someVar.data = PileOfData("foo", "bar")
// Yet another place
PileOfData data("foobar");
```

I didn't like this pattern either, because the members are public and we could initialize them directly. It's true that it was not enough to simply remove the constructors, we had to modify the syntax. We could either use designated initializers which come along with [aggregate initialization](https://www.sandordargo.com/blog/2024/04/17/initializations-part-2#aggregate-initialization) with its braces or if we didn't need the member names in the initializer expressions, we could use simply braces.

```cpp
struct PileOfData {
  std::string type;
  std::string identifier;
};

// Somewhere else...
someVar.data = PileOfData{.type = "foo", .identifier = "bar"}
// Yet another place
PileOfData data{"foobar"};
```

So this shows that if you still haven't upgraded to C++20, you can still use aggregate initialization without having to initialize all the members. If we provide `n` members in the initializer list, the first `n` member (counting from the top) will be initialized.

So unless the constructors are not initializing the first members of the struct (but they skip some in between), they didn't make much sense from the beginning assuming that the code was not written to a standard earlier than C++11.

Then I continued looking for similar code and I found this.

```cpp
struct AnotherPileOfData {
  std::string label;
  std::string remark;
  int quantity;

  AnotherPileOfData(const std::string& label, int quantity);
  AnotherPileOfData(const Blob& blob);

  DECLARE_DEFAULT_MEMBERS(AnotherPileOfData)
};

```

Oh, a macro. There is even a corresponding implementation file, with other macro and constructor definitions.

```cpp
AnotherPileOfData::AnotherPileOfData(const std::string& label, int quantity):
 label(label), quantity(quantity) {}

AnotherPileOfData::AnotherPileOfData(const Blob& blob): label(blob.label), remark(blob.text), quantity(blob.quantity) {}

DEFINE_DEFAULT_MEMBERS(AnotherPileOfData)
```

That's clearly not how most like to see their structs which should be simple. So I immediately started removing the fluff but I didn't merge the changes. I didn't even post a code review. Can you guess why?

Binary size was significantly impacted. Removing all the user-declared special member functions gave a few extra KBs for widely used classes. The reason is essentially inlining. With defaulted special member functions, each compilation unit where `AnotherPileOfData` is used, gets a copy of the special member functions' code. In other words, they are inlined. With the used-provided versions, they are not inlined, but you get simple function calls.

I must say that if we didn't care so much about binary sizes, I'd still go ahead and remove the macro and the constructors from `AnotherPileOfData`. There would be only one constructor to keep in a certain format and that's the one taking the blob. I'd probably turn it into a static builder function or a free function [as it was suggested by Jonathan Boccara on FluentC++](https://www.fluentcpp.com/2018/06/15/should-structs-have-constructors-in-cpp/).

```cpp
AnotherPileOfData makeAnotherPileOfData(const Blob& blob) {
  return AnotherPileOfData{blob.label, blob.text, blob.quantity};
}
```

## Conclusion

In my opinion, a `struct` barely needs a constructor. With the combination of aggregate initialization, designated initializers and the right order of members, you can easily get rid of constructors in a `struct`. In those rare cases, when you'd still need them you can replace it with a free function keeping your `struct` as simple as possible.

At the same time, it's worth noting that sometimes a `struct` has constructors and special member functions to limit binary sizes. [By moving special member functions to implementation files, we can limit inlining and thus decrease binary sizes.](https://www.sandordargo.com/blog/2023/02/01/special-functions-and-binary-sizes)

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!