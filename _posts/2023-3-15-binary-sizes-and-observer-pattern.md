---
layout: post
title: "The observer pattern and binary sizes"
date: 2023-3-15
category: dev
tags: [cpp, binarysizes, design patterns, observer]
excerpt_separator: <!--more-->
---
In the previous article on binary sizes, we discussed how the decorator pattern's classic and modern implementation fares in terms of binary size. We saw that the modern implementation had a smaller size, but the difference was not great and it's not always evident how it scales.

Today, let's talk about a different pattern, the observer. Once again, I borrow the examples from Klaus Igleberger's book, [C++ Software Design](https://www.amazon.com/Software-Design-Principles-Patterns-High-Quality/dp/1098113160?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=e9b6f64671aac55ff52ecfd91e137d6e&camp=1789&creative=9325).

## The Observer pattern

The observer pattern is thoroughly explained in the famous [GoF book](https://amzn.to/2KWCkLN). With the help of this pattern, you define a convenient way to notify objects about any event that happens to the observed object.

I borrowed Klaus' example, in which there are `Person` objects being observed for changes in their names, addresses, etc. How much more convenient it is get a notification instead of querying it all the time and tracking if the returned value means a change or not!

In his book, Klaus offers two solutions, a classic and the modern one.

## The classic observer

Let's have a look at the classic first.

```cpp
// observer.h
#pragma once


template< typename Subject, typename StateTag >
class Observer
{
 public:
   virtual ~Observer() = default;

   virtual void update( Subject const& subject, StateTag property ) = 0;
};

// name_observer.h
#pragma once

#include "observer.h"
#include "person.h"

class NameObserver : public Observer<Person,Person::StateChange>
{
 public:
   void update( Person const& person, Person::StateChange property ) override;
};


// name_observer.cpp
#include "name_observer.h"

void NameObserver::update( Person const& person, Person::StateChange property )
{
   if( property == Person::forenameChanged ||
       property == Person::surnameChanged )
   {
      // ... Respond to changed name
   }
}

// address_observer.h
#pragma once

#include "observer.h"
#include "person.h"

class AddressObserver : public Observer<Person,Person::StateChange>
{
 public:
   void update( Person const& person, Person::StateChange property ) override;
};

// address_observer.cpp
#include "address_observer.h"

void AddressObserver::update( Person const& person, Person::StateChange property )
{
   if( property == Person::addressChanged ) {
      // ... Respond to changed address
   }
}

// person.h
#pragma once

#include "observer.h"
#include <string>
#include <set>

class Person
{
 public:
   enum StateChange
   {
      forenameChanged,
      surnameChanged,
      addressChanged
   };

   using PersonObserver = Observer<Person,StateChange>;

   explicit Person( std::string forename, std::string surname )
      : forename_{ std::move(forename) }
      , surname_{ std::move(surname) }
   {}

   bool attach( PersonObserver* observer );
   bool detach( PersonObserver* observer );

   void notify( StateChange property );

   void forename( std::string newForename );
   void surname ( std::string newSurname );
   void address ( std::string newAddress );

   std::string const& forename() const { return forename_; }
   std::string const& surname () const { return surname_; }
   std::string const& address () const { return address_; }

 private:
   std::string forename_;
   std::string surname_;
   std::string address_;

   std::set<PersonObserver*> observers_;
};

// person.cpp
#include "person.h"

void Person::forename( std::string newForename )
{
   forename_ = std::move(newForename);
   notify( forenameChanged );
}

void Person::surname( std::string newSurname )
{
   surname_ = std::move(newSurname);
   notify( surnameChanged );
}

void Person::address( std::string newAddress )
{
   address_ = std::move(newAddress);
   notify( addressChanged );
}

bool Person::attach( PersonObserver* observer )
{
   auto [pos,success] = observers_.insert( observer );
   return success;
}

bool Person::detach( PersonObserver* observer )
{
   return ( observers_.erase( observer ) > 0U );
}

void Person::notify( StateChange property )
{
   // This formulation makes sure detach() operations
   // can be detected during the iteration
   for( auto iter=begin(observers_); iter!=end(observers_); )
   {
      auto const pos = iter++;
      (*pos)->update(*this,property);
   }
}

// main.cpp
#include "address_observer.h"
#include "name_observer.h"
#include "person.h"
#include <cstdlib>

int main()
{
   NameObserver nameObserver;
   AddressObserver addressObserver;

   Person homer( "Homer"     , "Simpson" );
   Person marge( "Marge"     , "Simpson" );
   Person monty( "Montgomery", "Burns"   );

   // Attaching observers
   homer.attach( &nameObserver );
   marge.attach( &addressObserver );
   monty.attach( &addressObserver );

   // Updating information on Homer Simpson
   homer.forename( "Homer Jay" );  // Adding his middle name

   // Updating information on Marge Simpson
   marge.address( "712 Red Bark Lane, Henderson, Clark County, Nevada 89011" );

   // Updating information on Montgomery Burns
   monty.address( "Springfield Nuclear Power Plant" );

   // Detaching observers
   homer.detach( &nameObserver );

   return EXIT_SUCCESS;
}
```

As you can see, the observers constitute an object hierarchy, but templates are also involved. Templates often (can) contribute to a bigger binary size. Having polymorphic classes both contribute to higher binary sizes and worse run-time performance.

## The modern observer

Now let's look at the much more slim modern version of this pattern.

```cpp
// observer.h
#pragma once

#include <functional>

template< typename Subject, typename StateTag >
class Observer
{
 public:
   // using OnUpdate = std::function<void(Subject const&,StateTag)>;
   // typedef void (*OnUpdate)(Subject const&,StateTag);
   using OnUpdate = void (*)(Subject const&,StateTag);
   // No virtual destructor necessary

   explicit Observer( OnUpdate onUpdate )
      : onUpdate_{ std::move(onUpdate) }
   {
      // Possibly respond on an invalid/empty std::function instance
   }

   // Non-virtual update function
   void update( Subject const& subject, StateTag property )
   {
      onUpdate_( subject, property );
   }

 private:
   OnUpdate onUpdate_;
};

// person.h
#pragma once

#include "observer.h"
#include <string>
#include <set>

class Person
{
 public:
   enum StateChange
   {
      forenameChanged,
      surnameChanged,
      addressChanged
   };

   using PersonObserver = Observer<Person,StateChange>;

   explicit Person( std::string forename, std::string surname )
      : forename_{ std::move(forename) }
      , surname_{ std::move(surname) }
   {}

   bool attach( PersonObserver* observer );
   bool detach( PersonObserver* observer );

   void notify( StateChange property );

   void forename( std::string newForename );
   void surname ( std::string newSurname );
   void address ( std::string newAddress );

   std::string const& forename() const { return forename_; }
   std::string const& surname () const { return surname_; }
   std::string const& address () const { return address_; }

 private:
   std::string forename_;
   std::string surname_;
   std::string address_;

   std::set<PersonObserver*> observers_;
};

// person.cpp
#include "person.h"

void Person::forename( std::string newForename )
{
   forename_ = std::move(newForename);
   notify( forenameChanged );
}

void Person::surname( std::string newSurname )
{
   surname_ = std::move(newSurname);
   notify( surnameChanged );
}

void Person::address( std::string newAddress )
{
   address_ = std::move(newAddress);
   notify( addressChanged );
}

bool Person::attach( PersonObserver* observer )
{
   auto [pos,success] = observers_.insert( observer );
   return success;
}

bool Person::detach( PersonObserver* observer )
{
   return ( observers_.erase( observer ) > 0U );
}

void Person::notify( StateChange property )
{
   // This formulation makes sure detach() operations
   // can be detected during the iteration
   for( auto iter=begin(observers_); iter!=end(observers_); )
   {
      auto const pos = iter++;
      (*pos)->update(*this,property);
   }
}

// main.cpp

#include "observer.h"
#include "person.h"
#include <cstdlib>

void propertyChanged( Person const& person, Person::StateChange property )
{
   if( property == Person::forenameChanged ||
       property == Person::surnameChanged )
   {
      // ... Respond to changed name
   }
}

int main()
{
   using PersonObserver = Observer<Person,Person::StateChange>;

   PersonObserver nameObserver( propertyChanged );

   PersonObserver addressObserver(
      [/*captured state*/]( Person const& person, Person::StateChange property ){
         if( property == Person::addressChanged )
         {
            // ... Respond to changed address
         }
      } );
   
   Person homer( "Homer"     , "Simpson" );
   Person marge( "Marge"     , "Simpson" );
   Person monty( "Montgomery", "Burns"   );

   // Attaching observers
   homer.attach( &nameObserver );
   marge.attach( &addressObserver );
   monty.attach( &addressObserver );


   // Updating information on Homer Simpson
   homer.forename( "Homer Jay" );  // Adding his middle name

   // Updating information on Marge Simpson
   marge.address( "712 Red Bark Lane, Henderson, Clark County, Nevada 89011" );

   // Updating information on Montgomery Burns
   monty.address( "Springfield Nuclear Power Plant" );

   // Detaching observers
   homer.detach( &nameObserver );

   return EXIT_SUCCESS;
}
```

## In comparison

The `Observer` uses the same template parameters, but instead of providing a `virtual` function as a customization point, it takes a `std::function` object. The `Person` object is the very same as in the classic example.

The modern version seems easier to implement, our assumption is that it's faster due to the lack of `virtual` functions, but what about the binary size?

### A surprising result

Interestingly, the modern version produces a binary that is about 10% bigger when optimized, otherwise, with `-O0` the difference is about 70%! Still, it compiled considerably faster than the classic version!

|   Version                        | Binary size | Compile time |
|----------------------------------|--------------|
|  classical observer -O0 | 85.5K          |   9.04s 
|  classical observer -O3 | 35.9K          |   9.84s 
|  classical observer -Os | 47.1K          |   9.91s 
| modern observer -O0 | 142K          |   7.56s 
| modern observer -O3 | 39.5K          |   8.34s 
| modern observer -Os | 40.9K          |   8.2s 

But what can be behind this size difference? And how do they scale?
                                                                                                    
Let's focus on the `-O3` version as most probably you'll use that for your releases! I used a new observer as simple as the above ones. Adding a new observer to the classic solution adds about 400 bytes. For the modern, we have two possibilities:
- adding a new observer with a lambda added an extra 1500 bytes. That scales up almost 4 times as fast as the classic solution!
- but implementing the new observer with a free function only added a hundred extra bytes which 4 times less as for the classic one.

### Turning the results around

At a first glance, it seems that if we want to shave off some of the binary, we should rely on free functions. Modifying the original example, to use only functions still left us with a 2k bigger binary than the classic solution.

Templates can significantly increase the size of the binary. And I realized that it's the case here too. `std::function` is a function template. While all our free function observers have the same type (`void (*)(Subject const&, StateTag)`), each lambda is compiled into a different object and therefore a new function template instantiation is necessary!

So what if we replace `std::function` with a good old function pointer?

> A `std::function` is a type erased wrapper around any kind of callable. A function pointer will take a free function or something that can be implicitly converted to it. Such as a lambda without a capture. `std::function` has quite some overhead, but it provides flexibility. Watch [this video on C++ Weekly](https://www.youtube.com/watch?v=aC-aAiS5Wuc) to learn more.

Let's replace one line in the Observer class:

```cpp
// using OnUpdate = std::function<void(Subject const&,StateTag)>;
using OnUpdate = void (*)(Subject const&,StateTag);
```
Now we have a smaller binary and even the lambdas are not problematic anymore! And we still have the compile-time advantage too!

|   Version                        | Binary size | Compile time |
|----------------------------------|--------------|
|  classical observer -O0 | 85.5K          |   9.04s 
|  classical observer -O3 | 35.9K          |   9.84s 
| modern observer with free functions -O0 | 84.4K          |   7.0s 
| modern observer with free functions -O0 | 35K          |   7.48s 
| modern observer with a free function and a lambda -O3 | 84.6K          |   7.1s 
| modern observer with a free function and a lambda -O3 | 35.1K          |   7.5s 

If you focus on the binary, you can win big by replacing `std::function` and some additional bytes by using free functions instead of lambdas.

## Conclusion

Today, we compared classic and modern implementations of the observer design patterns in terms of binary sizes. First, we saw that the classic version of the observer pattern was smaller and scaled sometimes better than the modern implementation.

Then we found that, if we don't need the features of `std::function`, but you can use a function pointer to take your observers, you can win big and end up with a nicely scaling implementation that is faster and smaller than the classic version.

Next time we'll look into binary sizes and templates as we already partially discussed this topic.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
