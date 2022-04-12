---
layout: post
title: "My first work experience with C++20"
date: 2022-4-13
category: books
tags: [cpp, cpp20, ranges, concepts]
excerpt_separator: <!--more-->
---
I joined a new team recently. We have our own internal microservices as well as libraries. While for microservices we support one main branch, for libraries we do have to support at least three, in reality about five versions.

The different releases are using different toolchains supporting different versions of C++. Starting from C++11 we have all the versions up to C++20. While I had already been studying C++20 on my own, I didn't have to chance to use it in a real-world corporate context. In fact, not even C++17 - though it doesn't offer so many novelties.

In this small post, I'd like to reflect on our so called innovation week that I could spend on modernizing some our codebases.

## Not even C++11

Using a new version is not just *l'art pour l'art*. Using a new standard can and should simplify quite a bit your code, it should make the life of maintainers easier. Long years after introducing C++11 to our codebases, I barely found the use of range-based for loops. Okay, okay, range-based for loops do have an important bug, but I clearly doubt that it's the reason behind not having these readable loops.

Instead, I found many long constructs of iterators, or even the good old for loops with the use of an incremented index along with the subscription operator (`[]`).

And then I haven't even mentioned the lack of using smart pointers, [default member initalization](https://en.cppreference.com/w/cpp/language/data_members#Member_initialization), etc.


## Maps and sets now have contains

If you have to work with `std::map` or `std::set` or their unordered versions, you most probably know how cumbersome is to find out whether they have a certain item (as a key) or not. Using a `find()` and then comparing its result with the `end()` iterator is verbose, not very readable and not elegant.

With C++20 we can replace all that with [`contains`](https://en.cppreference.com/w/cpp/container/unordered_set/contains)!

```cpp
std::map<std::string, int> myMap;
// ...

//before C++20
if (myMap.find(aKey) != myMap.end()) {
    // the map contains a key
    // ...
}

// with C++20
if (myMap.contains(aKey)) {
    // ...
}
```

Of course, if you need an iterator to that item, you'll still need to use `find`, but `contains` will simplify your code in lots of cases.

## Iterate over maps with structured bindings

I often saw that people created an iterator outside the loop because the type is very long, then in the first lines of the loop body they took references to the key and value of the given `map` element.

```cpp
std::map<std::string, SomeLengthClassName>::const_iterator aIt;

for (aIt = myMap.begin(); aIt != myMap.end(); ++aIt)
{
    const std::string& aKey = aIt->first;
    const SomeLengthClassName& aValue = aIt->second;
    // ...
}
```

With C++17, we can use [structured bindings](https://en.cppreference.com/w/cpp/language/structured_binding) and we can get rid of these complex loops including the manual creation of those references.

```cpp
for (const auto& [aPhase, aValue]: myMap)
{
    // ...
}
```

That's shorter and way more readable.

But what you should you do when you only need the key or the value?

## Ranges and what is missing

But there is more than that we can do with C++20 when we don't use the keys or the values!

Continuing the idea of structured bindings, when you don't need one of the key-value pair, with C++17 you used to simple name the not needed one as an `_`. With C++20 ranges there are these possibilities instead!

```cpp
std::map<std::string, int> myMap { {"one", 1}, {"two", 2}, {"three", 3} };
for (auto aIt = myMap.begin(); aIt != myMap.end(); ++aIt)
{
    std::cout << aIt->second << '\n';
}


for (auto const& aValue: std::views::values(myMap))    
// or...
for (auto const& aKey: std::views::keys(myMap))

```

That's already more readable and we haven't even tried to use the "pipe-syntax" that must be a kind of a satisfaction for programmers working on Linux.

```cpp
for (auto const& aValue: myMap | std::views::keys) {
       std::cout << aValue << '\n';
}
```

This pipe-syntax shows best it's potential when we chain multiple algorithms, views, etc. together and instead of building layers around the initial range we can simply read from the left to the right and quickly understand what goes on. This is all possible as functions in the `ranges` and `views` namespace do not take a pair of iterators but the containers directly. More on that in another article.

Is there a difference in performance between the good old way, the loop with structured bindings and with ranges/views?

I did some analyzes on quick bench and I found no difference between the C++17 and C++20 way of iterating over keys or values, but they are both a bit faster than dealing manually with the iterators.

Not surprisingly, I didn't find many usages of [standard algorithms](https://www.sandordargo.com/blog/2019/01/30/stl-algos-intro). But when I did I could almost always replace them with the range version, meaning that I don't have to pass the begin and end iterators anymore just the container - which is treated as a whole as a range.

I already showed how ranges could help me simplify loops to iterate over the keys of map or how I could replace simple standard algorithms with standard range-based algorithms.

```cpp
std::copy(myVec.begin(), myVec.end(), std::back_inserter(results));

// The above example would become
std::copy(myVec, std::back_inserter(results));
```

At a first glance, it seems that there is a small performance penalty on the ranges version. Something I have to analyze further. It's definitely not significant in applications were most time is lost in database and network class, but maybe it's too much in other cases.

In any case, the increase in readability might justify a bit of loss in CPU time. It depends on your situation.

I found ranges the best when I wanted to replace full for loops. Let me share an example with you.

```cpp
bool Configuration::warnOnMissingData(const Date& iCheckDate)
{
    bool aWasAWarningIssued(false);

    Date aLastValidDate;
    std::vector<ImportantData>::const_iterator aDataIterator;
    for (aDataIterator = _data.begin(); aDataIterator != _data.end(); ++aDataIterator)
    {
        aLastValidDate = aDataIterator->second->getLastDate();
        if (aLastValidDate < iCheckDate)
        {
            LOG_ERROR(aDataIterator->second);
            aWasAWarningIssued = true;
        }
    }

    return aWasAWarningIssued;
}
```

That loop was never great. Like why do we keep looping after the first matching condition? Because of logging maybe? It's not a great explanation. Even C++11 had great options for simplifying the above loop. But it's hard to find time to change working code. But when you do, don't be shy. Make sure that the code is tested and refactor it according to your best knowledge.

```cpp
bool Configuration::warnOnMissingDataKeys(const Date& iCheckDate)
{
    auto isDataLastDateOlderThan = [&iCheckDate](const auto& aData) {
            if (aData == nullptr) {
                    return false;
            }
            return aData->getLastDate() < iCheckDate;
        };
    const auto& aPotentialMatch = std::ranges::find_if(
            _data,
            isDataLastDateOlderThan,
            &std::vector<ImportantData>::value_type::second
    );
    if (aPotentialMatch == _data.end()) { return false; }
    LOG_ERROR(aPotentialMatch->first);
    return true;
}
```

With this refactoring, we could introduce an algorithm instead of a raw loop and we could give a name even to the condition. We only lost some logging which was probably not even meant.

## Concepts for templates

Last but not least, I followed the [T.10 core guideline](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t10-specify-concepts-for-all-template-arguments)'s recommendation of not having bare template parameters. Each of them is constrained by some concepts now. Sometimes I only used a standard concept, but often I had to create the our own concepts first.

How did I come of with these new concepts?

I had a deep look into the templates to see how they use their template parameters. With that, I understood what API we have to require from any type. Then I also had a look into each instantiation to see if I can find a pattern. Often I realized that the API I require is the API defined by an abstract base class which each template argument type used as a base.

Nowing this fact let you decide whether I wanted to describe once more the interface or just require that the incoming parameters are implementing that base class, that interface. Ultimately I might even think about removing the base class if it is just for an interface, turn it into a concept and make sure that the used-to-be child class satisfies that base constraint. With that I'd basically introduce duck-typing, but I'd remove some virtual tables and pointers and runtime interface in general.

But let's come back to the creation of concepts. Only when I had a couple of rounds of this investigation I could focus on coming up with a good name for the concept. I found this part the most difficult one. Should I use a noun or an adjective? I'm not all set on that question. So far I used nouns that seemed to read slightly better. What do you think?

## Conclusion

In this article I shared my first experience with C++20 and production code. I didn't only introduce C++20 features, 
in some cases, C++17 suffice - bear in mind structures mindings. C++20 introduced some great library features like `contains` for maps and sets, but also new the `ranges` library and concepts. All this require some learning, but they can greatly simplify your code.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
