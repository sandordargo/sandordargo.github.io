---
layout: post
title: "Strong types for containers"
date: 2020-10-14
category: dev
tags: [cpp, tutorial, stl, strongtypes]
excerpt_separator: <!--more-->
---
Once again we were practising [Object Calisthenics](https://williamdurand.fr/2013/06/03/object-calisthenics/) during our weekly coding dojo. If you don't know what it is about, I'd advise you to check out [the rules](https://williamdurand.fr/2013/06/03/object-calisthenics/). You might not want to apply all of them for your production code, but at least some parts of the constraints could be extremely useful.
<!--more-->

The rules that are giving the biggest challenge are the ones prohibiting the use of primitive values and containers naked. It means that all numbers, booleans, even strings and all the containers must be wrapped into an object and by the way, you shall not use any getters. In other words, the rules say that one should use strong types that clearly represents the concepts you want to model. You won't use an `int` plain old type, but you'll rather introduce an `Age` class. You'll not use a simple `string` but rather `Name`. Or you'll not use a `vector` of `Players` but rather a `Team`.

This requires patience and practice. It is not as difficult as it might sound, but it definitely takes time to write all the boilerplate. On the other hand, in a few sessions for sure, you'll learn well how to override the different operators.

But let's not forget that we are humans and we tend to move towards the least resistance. We look for shortcuts, smart solutions so that we still comply with the rules.

Once someone had the idea of inheriting from `std::vector`. Next time someone else tried, and the last time everyone did. Did it work well? Actually, it was not so bad. We discovered some syntactic rules that we were not aware of - C++ 11 is still new... - but at the retrospective part, one of our junior hires said that it's not a good idea to inherit from a `vector`.

Why? - I asked. She couldn't reply more than _because some people said so on StackOverflow_. 

I think this is not a great argument even if those people are right. Anyone can share his or her dogmatic views on the internet presenting it as the one and only truth of life - not just in technology. Before taking something for granted, we'd better understand what is behind.

Here is my attempt to explain how to use strong types of containers in C++ what are the pros and cons of each approach. Feel free to share your ideas in the comments section.

## What is a strong type?

First, let's repeat what is a strong type. A strong type carries extra information, a [specific meaning through its name](https://www.fluentcpp.com/2016/12/08/strong-types-for-strong-interfaces/). While you can use booleans or strings everywhere, the only way they carry can carry meaning is the name of their instances.

If you look at this function signature, perhaps you think it's alright:

```cpp
Car::Car(unit32_t horsepower, unit32_t numberOfDoors, bool isAutomatic, bool isElectric);
```

It has relatively good names, so what is the issue?

Let's look at a possible instantiation.

```cpp
auto myCar{Car(96, 4, false, true)};
```

Yeah, what? God knows... And you if you take your time to actually look up the constructor and do the mind mapping. Some IDEs can help you visualizing parameter names, like if they were Python-style named parameters, but you shouldn't rely on that.

Of course you could name the variables as such:

```cpp
constexpr unit32_t horsepower = 96;
constexpr unit32_t numberOfDoors = 4;
constexpr bool isAutomatic = false;
constexpr bool isElectric = false;
auto myCar{Car(horsepower, numberOfDoors, isAutomatic, isElectric)};
```

Now you understand right away which variable represents what. You have to look a few lines upper to actually get the values, but everything is in sight. On the other hand, this requires will-power. Discipline. You cannot enforce it. Well, you can be a thorough code reviewer, but you won't catch every case and anyway, you won't be there all the type.

Strong typing is there to help you!

Imagine the signature as such:

```cpp
Car::Car(Horsepower hp, DoorsNumber numberOfDoors, Transmission transmission, Fuel fuel);
```

Now the previous instantiation could look like this:

```cpp
auto myCar = Car{Horsepower{98u}, DoorsNumber{4u}, Transmission::Automatic, Fuel::Gasoline};
```

This version is longer and more verbose than the original version - which was quite unreadable -, but much shorter than the one where introduced well named helpers for each parameter

So one advantage of strong typing is readability and one other is safety. It's much harder to mix up values. In the previous examples, you could have easily mixed up door numbers with performance, but by using strong typing, that would actually lead to a compilation error.

## Strongly typed containers

Now that we know what strong typing is about, let's see the different options to create a strongly typed container. We are going to start with the option we were experimenting at our coding dojo, the one that inspired this article.

### Inheriting from a vector

It's soo easy! You just publicly inherit from the `std::vector` and you either implement the constructors you'd need or you declare that you want to use the ones from the base class. This latter is even easier than the former.

Let's see an example:

```cpp
class Squad : public std::vector<Player> {
using std::vector<Player>::vector;
// ...
};
```

It's simple, it's readable, yet you'll find a lot of people at different forums who will tell you that this is the eighth deadly sin and if you are a serious developer you should avoid it at all costs.

Why do they say so?

There are two main arguments. One is that algorithms and containers are well-separated concerns in the STL. The other one is about the lack of virtual constructors.

But are these valid concerns?

They might be. It depends.

Let's start with the one about the lack of a virtual destructor. It seems more practical.

Indeed, the lack of a virtual destructor might lead to undefined behaviour and a memory leak. Both can be serious issues, but the undefined behaviour is worse because it can not just lead to crashes but even to difficult to detect memory corruption eventually leading to strange application behaviour.

But the lack of undefined behaviour doesn't lead to undefined behaviour and memory leak by default, you have to use your derived class in such a way.

If you delete an object through a pointer to a base class that has a non-virtual destructor, you have to face the consequences of undefined behaviour. Plus if the derived object introduces new member variables, you'll also have some nice memory leak. But again, that's the smaller problem.

On the other hand, this also means that those who rigidly oppose inheriting from `std::vector` - or from any class without a virtual destructor - because of undefined behaviour and memory leaks, are not right. 

If you know what you are doing, and you only use this inheritance to introduce a strongly typed vector, not to introduce polymorphic behaviour and additional states to your container, you are perfectly fine to use this technique. Simply, you have to respect the limitations, though probably this is not the best strategy to use in case of a public library. But more on that just in a second.

So the other main concern is that you might mix containers and algorithms in your new object. And it's bad because the creators of the STL said so. And so what? [Alexander Stepanov](https://en.wikipedia.org/wiki/Alexander_Stepanov) who originally designed the STL and the other who have been later contributed to it are smart people and there is a fair chance that they are better programmers than most of us. They designed functions, objects that are widely used in the C++ community. I think it's okay to say that they are used by everyone.

Most probably we are not working under such constraints, we are not preparing something for the whole C++ community. We are working on specific applications with very strict constraints. Our code will not be reused as such. Never. We don't work on generic libraries, we work on one-off business applications.

As long as we keep our code clean (whatever it means), it's perfectly fine to provide a non-generic solution.

As a conclusion, we can say that for application usage, inheriting from containers in order to provide strong typing is fine, as long as you don't start to play with polymorphism.

But we have other options to choose from.

## Creating an alias

We can create an alias either by using the `using` keyword or with the good old `typedef`. Essentially the next two statements are the same:

```cpp
using Team = std::vector<Player>;
typedef std::vector<Player> Team;
```

This is probably the simplest solution to get container types with descriptive type names. The only problem is that they are not so strong.

A `Team` in the above example is literally the same as a vector of Players. In other words, you can whatever list of players in where a `Team` is expected, it can even be a vector of players without a team. That's not a team, right?

So while this option requires the least amount of typing, it doesn't provide any safety, just a bit of extra readability.

Let's move to our next option.  

### Private inheritance

Instead of the original idea which was to use public inheritance, we can use private inheritance to get our strong type. [As discussed a few weeks ago](XXXXXXXXXXXXXX) with private inheritance, you'll only inherit the implementation from the base class, but not the API as it basically represents a `has-a` relationship instead of an `is-a` one.

This means that if you inherit privately from `std::vector` no functionality of the underlying container class will be exposed to the users of the new derived class.

Private inheritance eliminates the problem of a missing virtual destructor because it wouldn't even be possible to refer to the derived class with a base class pointer. That's how private inheritance works.

On the other hand, you'll have to type a lot as you'll have to expose manually the needed API of the base class. Depending on whether you use at least C++11 you might be able to use the `using` keyword. Here are the two ways to forward the calls, or in other words, to expose the API:

```cpp
class Team : private std::vector<Player> {
public:
 using std::vector<Player>::push_back;
 bool empty() const {
    return std::vector<Player>::empty();
 }
};
```

I strongly recommend the usage of the `using` keyword. It requires less typing and there are fewer opportunities to make mistakes, especially if you think about const correctness.

The necessity of manually exposing the underlying vector's API has a non-expected side effect. You'll actually expose only what you need and you'll have a leaner API.

### Composition

While using private inheritance has its pros we also have to keep in mind what [the C++ standard says about it](https://isocpp.org/wiki/faq/private-inheritance#priv-inherit-vs-compos):

> _Use composition when you can, private inheritance when you have to._

But do we __have__ to use private inheritance to have a strongly typed container?

The simple answer is no, we don't.

We can follow the good old _follow composition over inheritance rule_ and do something like this:

```cpp
class Team
{
public:
  
  Team() = default;

  std::vector<Person>::iterator begin() { return people.begin(); }
  std::vector<Person>::iterator end() { return people.end(); }
  std::vector<Person>::const_iterator begin() const { return people.begin(); }
  std::vector<Person>::const_iterator end() const { return people.end(); }
  std::vector<Person>::const_iterator cbegin() const { return people.cbegin(); }
  std::vector<Person>::const_iterator cend() const { return people.cend(); }

private:
  std::vector<Person> people;
};
```

You have to do almost the same as you'd with private inheritance pre C++11. It's a bit verbose and you have to pay a lot of attention to what should be const and what is not, but apart from it, there is no big difference.

What is a bit cumbersome is the long return type names everywhere.

Let's make it a bit simpler to read:

```cpp
class Team
{
  using Team_t = std::vector<Person>;
public:
  using iterator = std::vector<Person>::iterator;
  using const_iterator = std::vector<Person>::const_iterator;

  Team() = default;

  iterator begin() { return people.begin(); }
  iterator end() { return people.end(); }
  const_iterator begin() const { return people.begin(); }
  const_iterator end() const { return people.end(); }
  const_iterator cbegin() const { return people.cbegin(); }
  const_iterator cend() const { return people.cend(); }
  void push_back (const Person& person) {people.push_back(person);}

private:
  std::vector<Person> people;
};
```

We introduced a private alias for the container of persons also two public ones for the iterators. For the sake of the example, I also added implemented the push_back method.

Here is very simple example how you can `Team` now. Here is the full example. 
```cpp
#include <algorithm>
#include <iostream>
#include <vector>

class Person {
public:
    Person(std::string name) : _name(name) {}
    std::string _name{};
};

class Team
{
  // ...
};

int main() {
  
  Team team;
  team.push_back(Person{"Messi"});
  team.push_back(Person{"Suarez"});
  team.push_back(Person{"Griezmann"});
  
  
  
  std::cout << "team members are: ";
  for (const auto& player : team) {
    std::cout << ' ' << player._name;
  }
  std::cout << '\n';

  return 0;
}
```
## Conclusion

We briefly discussed how to create strongly typed collections in C++. It is not an exhaustive list, I didn't mention the [Curisouly Returning Template Pattern](https://www.fluentcpp.com/2017/05/23/strong-types-inheriting-functionalities-from-underlying/) for example, I didn't even mention the open-source libraries available.

Given the discussed options, I cannot say which is the best. As almost always in life, it depends. What is clear on the other hand that inheriting publicly from an STL container is not something from the devil as long as you understand what you do and you respect the rules.

Otherwise, if public inheritance is out of scope and a simple alias is not enough for your use-case, even though I prefer composition over inheritance, the possibility to use the `using` keyword pushes me a bit towards private inheritance.

Do you use strong types in your projects?