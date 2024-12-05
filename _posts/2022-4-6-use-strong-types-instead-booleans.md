---
layout: post
title: "Use strong types instead of bool parameters"
date: 2022-4-6
category: dev
tags: [cpp, bestpractices, strongtypes]
excerpt_separator: <!--more-->
---
There are some recurring themes in code reviews. Experienced reviewers often already have a template of comments somewhere for such recurring patterns. Sometimes only in the back of their minds, but often written somewhere. Probably they also have some reference materials that they refer to, they are [crucial parts of good code review comments](https://www.sandordargo.com/blog/2021/10/06/airy-code-reviews).
<!--more-->
By using references, you can delegate the question of credibility to someone else, to someone usually well-known to other developers too.

One of these recurring themes in code reviews that I perform is about whether we should accept using `bool`s as function parameters. And the reference material I use is a conference talk presented by Matt Godbolt, [Correct by Construction: APIs That Are Easy to Use and Hard to Misuse](https://www.youtube.com/watch?v=nLSm3Haxz0I) 

## Boolean parameters make your code difficult to understand

It is so simple to introduce `bool` function parameters. You might also think that it's a cheap yet good enough solution. Imagine that you work on a car rental system that has to retrieve locations where you pick up or drop off cars.

First, you write a simple function that takes the city code and the company code as parameters to identify locations, and also a date to make sure that the returned locations are open at the point in time you wish to pass.

```cpp
std::vector<Location> searchLocation(const std::string& iCityCode, const std::string& iCompanyCode, const Date& iDate);
```

Later, you figure out that there are locations where - at least at a given point in time - you cannot pick up a car, but you can drop it off.

Your `searchLocation` function has to take into account whether you look for a pick-up or a drop-off point.

How do you do that?

A tempting and way too often chosen implementation is to take this parameter as a boolean.

```cpp
std::vector<Location> searchLocation(const std::string& iCityCode, const std::string& iCompanyCode, const Date& iDate, bool iPickupOrDropoff);
```

What's the problem now?

The signature has not become particularly unreadable.

In fact, most of the problems with `bool` parameters are not in the signature, they are at call sites. Now you start seeing calls like that:

```cpp
auto locations = searchLocation(aCityCode, aCompanyCode, false);

// or 

auto locations = searchLocation(aCityCode, aCompanyCode, true);
```

Without looking up the definition of `searchLocation` you have no idea what that third argument, that `bool` stands for.

You might argue that you could do things that the situation better. You could rename the local variable storing the return value from `locations` to either `pickupLocations` or `dropoffLocations`. Indeed you could, but does it mean that you know what that `bool` stands for?

No, you don't know it. If you think about, you can assume it. 

But who wants to assume?

And what if you add more boolean parameters?

Someone realizes that the function should be able to search for locations that are accessible for handicapped people.


```cpp
auto locations = searchLocation(aCityCode, aCompanyCode, false, true);
```

Of course, we could continue adding more boolean parameters, but let's stop here.

Seriously.

What the hell do these booleans mean? What if we send `false` or maybe `true`? You don't know without jumping to the signature. While modern IDEs can help you with tooltips showing the function signature even the documentation - if available -, we cannot take that for granted, especially for C++ where this kind of tooling is still not at the level of other popular languages.

But we don't only have a problem with readability. With multiple parameters of the same type next to each other, we can also simply mix up the values.

## There are different solutions

I'm going to show you 3 different solutions, though I would really not go with the first one. It's more of an anti-solution.

### Adding code comments are not for the long run

A very simple way to make the situation better is to make some comments.

```cpp
auto locations = searchLocation(aCityCode, aCompanyCode, false /* iPickupDropoff */, true /* iAccessible */);
```

Even better if you break lines!

```cpp
auto locations = searchLocation(aCityCode, 
                                aCompanyCode,
                                false /* iPickupDropoff */,
                                true /* iAccessible */);
```

At least you know what each argument stands for.

Still, you have no guarantees that the comments are right and it's extra effort to add or maintain them. Someone will either forget to add such comments or when things change they might become invalid, yet nobody will update them.

In addition, `false /* iPickupDropoff */` still doesn't communicate clearly whether `false` has the meaning of a pick-up or a drop-off location...


### Use strong types, introduce some `enum`s!

A real solution is to replace the booleans with `enum`s. Let's declare an `enum` instead of each `bool` parameter!

```cpp
enum class LocationType {
  Pickup, Dropoff
};

enum class Accessible {
  Yes,
  No,
};
```

They could even have a base type of `bool`, but what if you realize later that you want to add more types? I kept the path open.

Now you can update both the functions and the function calls to use these enums.

```cpp
std::vector<Location> searchLocation(const std::string& iCityCode, const std::string& iCompanyCode, const Date& iDate, LocationType iLocationType, Accessible isAccessible);

// ...
auto locations = searchLocation(aCityCode, aCompanyCode, LocationType::Pickup, Accessible::Yes);

// ...

auto locations = searchLocation(aCityCode, aCompanyCode, LocationType::Dropoff, Accessible::No);
```

When you read these calls, you have no more doubts. At least not because of the last two arguments. You couldn't express your intentions in a cleaner way.

In the function definitions if you use some branching, you cannot simply type `if (iLocationType) { /* ...  */ }`, you have to explicitely compare it to the possible `enum` values, like `if (iLocationType == LocationType::Pickup) { /* ...  */ }`. I consider this as an advantage. It's so explicit that it leaves no questions about what goes on.

The flip side is that you need to type more not only in the function definition but actually everywhere. But I think that's a fair price for the gain in readability, therefore the gain in maintainability.

### Get rid of the need for those extra parameters

What if we could remove the need for those extra parameters?

Instead of having a function taking a `bool` parameter that represents whether you want to search for a pickup or dropoff location, we could have two functions with appropriate names.

```cpp

std::vector<Location> searchLocation(const std::string& iCityCode, const std::string& iCompanyCode, const Date& iDate, bool iPickupOrDropoff);

// vs

std::vector<Location> searchPickupLocation(const std::string& iCityCode, const std::string& iCompanyCode, const Date& iDate);

std::vector<Location> searchDropoffLocation(const std::string& iCityCode, const std::string& iCompanyCode, const Date& iDate);

```

While this solution is extremely readable, it only goes as far. It leaves two problems for us.

When you have multiple boolean parameters, what do you do? If you want to follow this technique, your API would grow exponentially.

Besides what do you do in the implementation? Will you duplicate the code? Will you use a `private` common function that takes a `bool`? Or you go for a class hierarchy where the base class would contain the common code and the derived classes would offer the customization points.

The latter one seems overkill in most situations. 

Using an internal interface that is based on boolean parameters is not much better than using it in an external API. You should respect your maintainers. You should make understanding code easy for them.

With `bool`s it's not possible. In the end, you should probably use some `enum`s.

## Conclusion

In this article, we saw how unwanted booleans can appear in our function signatures and how they can decrease the understandability and maintainability of our APIs.

We discussed two ways to make the situation better. Differentiating implementations and using better names are usually not a long-term solution as they can lead to the exponential growth of the API, but they can be good enough in certain situations.

Otherwise, we can introduce strong types, in this case, `enum`s in order to get rid of the unreadable `bool`s and improve readability once and for all.

For some other approaches and opinions, you might want to check out [C++ Stories](https://www.cppstories.com/2017/03/on-toggle-parameters/)

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!