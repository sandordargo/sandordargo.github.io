---
layout: post
title: "The decorator pattern and binary sizes"
date: 2023-3-8
category: dev
tags: [cpp, binarysizes, designpatterns, decorator]
excerpt_separator: <!--more-->
---
[In one of the previous articles on binary sizes](https://www.sandordargo.com/blog/2023/02/08/binary-sizes-and-virtual), we discussed how making a class polymorphic by using the `virtual` keyword affects the binary size. Turning a function into `virtual` has a substantial effect on the binary size, but adding more and more `virtual` methods to a class that already has at least one `virtual` function does not change that much.

To have an elaborate example I went to the publicly available code examples of [C++ Software Design](https://www.amazon.com/Software-Design-Principles-Patterns-High-Quality/dp/1098113160?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=e9b6f64671aac55ff52ecfd91e137d6e&camp=1789&creative=9325) by [Klaus Iglberger](https://coaches.xing.com/profile/Klaus_Iglberger/trainings). As I explained [here](https://www.sandordargo.com/blog/2022/12/31/8-best-books-in-2022), it's one of the best books I read in 2022. If you are interested in software design and C++, in my opinion, it's a must-read.

In the book, you can find different implementations of various design patterns. All the discussed design patterns are first presented through their classic implementation, usually based on polymorphism and then modern alternatives are also explained.

Sometimes, the modern implementations offer the same functionality, sometimes they are restricted for compile-time needs, but that's often enough for the needs.

In this and the next article, I want to go through two implementations of two design patterns and focus on their effects on binary sizes. Today, it's the decorator pattern on the plate.

## The decorator pattern

Let's start by quickly recap on what is the *decorator design pattern*. In the [Gang of Four book](https://amzn.to/2KWCkLN), it was listed as one of the structural design patterns. Some also refer to it as the *wrapper pattern*. Both names are good, as this pattern is about adding new behaviour to objects in a non-intrusive way. The decorator pattern places these objects into special wrapper objects responsible for attaching the new behaviour.

Imagine that you have an item which has both a name and a price and some other attributes. However, having a price is not enough. Depending on where you want to sell it, you have to apply different taxes. Not to mention that you might also want to apply some discounts.

Having the logic inside the class is not a great idea, and creating inheritance hierarchies does not scale as you add more and more taxes and discounts.

The *decorator pattern* provides a scalable solution.

## The classic solution

Klaus provided 3 solutions in his book. The first one is a classical solution based on runtime polymorphism. As I cannot compile templates with floating-point non-type arguments, I modified his example a bit.

```cpp
// taxed.h
#pragma once

#include "item.h"
#include <utility>

class Taxed
{
 public:
   Taxed( double taxRate, Item item )
      : item_( std::move(item) )
      , factor_( 1.0 + taxRate )
   {}

   Money price() const
   {
      return item_.price() * factor_;
   }

 private:
   Item item_;
   double factor_;
};

// money.h

#pragma once

#include <cmath>
#include <concepts>
#include <cstdint>
#include <ostream>

struct Money
{
   uint64_t value{};
};

template< typename T >
   requires std::is_arithmetic_v<T>
Money operator*( Money money, T factor )
{
   return Money{ static_cast<uint64_t>( money.value * factor ) };
}

constexpr Money operator+( Money lhs, Money rhs ) noexcept
{
   return Money{ lhs.value + rhs.value };
}

std::ostream& operator<<( std::ostream& os, Money money )
{
   return os << money.value;
}

// item.h

#pragma once

#include "money.h"

#include <memory>
#include <utility>

class Item
{
 public:
   template< typename T >
   Item( T item )
      : pimpl_( std::make_unique<Model<T>>( std::move(item) ) )
   {}

   Item( Item const& item ) : pimpl_( item.pimpl_->clone() ) {}

   Item& operator=( Item const& item )
   {
      pimpl_ = item.pimpl_->clone();
      return *this;
   }

   ~Item() = default;
   Item( Item&& ) = default;
   Item& operator=( Item&& item ) = default;

   Money price() const { return pimpl_->price(); }

 private:
   struct Concept
   {
      virtual ~Concept() = default;
      virtual Money price() const = 0;
      virtual std::unique_ptr<Concept> clone() const = 0;
   };

   template< typename T >
   struct Model : public Concept
   {
      explicit Model( T const& item ) : item_( item ) {}
      explicit Model( T&& item ) : item_( std::move(item) ) {}

      Money price() const override
      {
         return item_.price();
      }

      std::unique_ptr<Concept> clone() const override
      {
         return std::make_unique<Model<T>>(*this);
      }

      T item_;
   };

   std::unique_ptr<Concept> pimpl_;
};

// discounted,h

#pragma once

#include "item.h"
#include <utility>

class Discounted
{
 public:
   Discounted( double discount, Item item )
      : item_( std::move(item) )
      , factor_( 1.0 - discount )
   {}

   Money price() const
   {
      return item_.price() * factor_;
   }

 private:
   Item item_;
   double factor_;
};

// cpp_book.h

#pragma once

#include "money.h"

#include <string>
#include <utility>

class CppBook
{
 public:
   CppBook( std::string name, Money price )
      : name_{ std::move(name) }
      , price_{ price }
   {}

   std::string const& name() const { return name_; }
   Money price() const { return price_; }

 private:
   std::string name_;
   Money price_;
};

// conference_ticket.h

#pragma once

#include "money.h"

#include <string>
#include <utility>

class ConferenceTicket
{
 public:
   ConferenceTicket( std::string name, Money price )
      : name_{ std::move(name) }
      , price_{ price }
   {}

   std::string const& name() const { return name_; }
   Money price() const { return price_; }

 private:
   std::string name_;
   Money price_;
};

// main.cpp

#include "conference_ticket.h"
#include "cpp_book.h"
#include "discounted.h"
#include "taxed.h"
#include <cstdlib>

int main()
{
   // 20% discount, 15% tax: (499*0.8)*1.15 = 459.08
   Item item( Taxed( 0.15, Discounted(0.2, ConferenceTicket{ "Core C++", Money{499} } ) ) );
   Item item2( Taxed( 0.17, Discounted(0.2, ConferenceTicket{ "Core C++", Money{499} } ) ) );
   Item item3( Taxed( 0.18, Discounted(0.21, CppBook{ "Software Design", Money{499} } ) ) );


   Money const totalPrice = item.price();
   Money const totalPrice2 = item2.price();
   Money const totalPrice3 = item3.price();

   // ...

   return EXIT_SUCCESS;
}
```

## The modern solution

The second solution is a modern value-semantics-based one, providing a solution for compile-time decoration.

```cpp
// taxed.h

#pragma once

#include "money.h"
#include "priced_item.h"
#include <utility>

template< int taxRate, PricedItem Item >
class Taxed : private Item  // Using inheritance
{
 public:
   template< typename... Args >
   explicit Taxed( Args&&... args )
      : Item{ std::forward<Args>(args)... }
   {}

   Money price() const {
      return Item::price() * ( 1.0 + (taxRate/100) );
   }
};

// priced_item.h

#pragma once

#include "money.h"

template< typename T >
concept PricedItem =
   requires ( T item ) {
      { item.price() } -> std::same_as<Money>;
   };

// money.h

#pragma once

#include <cmath>
#include <concepts>
#include <cstdint>
#include <ostream>

struct Money
{
   uint64_t value{};
};

template< typename T >
   requires std::is_arithmetic_v<T>
Money operator*( Money money, T factor )
{
   return Money{ static_cast<uint64_t>( money.value * factor ) };
}

constexpr Money operator+( Money lhs, Money rhs ) noexcept
{
   return Money{ lhs.value + rhs.value };
}

std::ostream& operator<<( std::ostream& os, Money money )
{
   return os << money.value;
}

// discounted.h

#pragma once

#include "money.h"
#include "priced_item.h"
#include <utility>

template< int discount, PricedItem Item >
class Discounted  // Using composition
{
 public:
   template< typename... Args >
   explicit Discounted( Args&&... args )
      : item_{ std::forward<Args>(args)... }
   {}

   Money price() const {
      return item_.price() * ( 1.0 - (discount/100) );
   }

 private:
   Item item_;
};

// cpp_book.h

#pragma once

#include "money.h"

#include <string>
#include <utility>

class CppBook
{
 public:
   CppBook( std::string name, Money price )
      : name_{ std::move(name) }
      , price_{ price }
   {}

   std::string const& name() const { return name_; }
   Money price() const { return price_; }

 private:
   std::string name_;
   Money price_;
};

// main.cpp

#include "conference_ticket.h"
#include "cpp_book.h"
#include "discounted.h"
#include "taxed.h"

#include <cstdlib>



int main()
{
   // 20% discount, 15% tax: (499*0.8)*1.15 = 459.08
   Taxed<15,Discounted<20,ConferenceTicket>> item{ "Core C++", Money{499} };
   Taxed<16,Discounted<21,ConferenceTicket>> item2{ "Core C++", Money{499} };
   Taxed<17,Discounted<22,CppBook>> item3{ "Core C++", Money{499} };

   Money const totalPrice = item.price();  // Results in 459.08
   Money const totalPrice2 = item2.price();
   Money const totalPrice3 = item3.price();
      // ...

   return EXIT_SUCCESS;
}   
```

## Comparing the solutions

As Klaus explains in the book, the compile-time approach provides a faster run-time performance by a factor of 10.

That's a huge difference!

It's not very surprising though as there is no virtual dispatching, no run-time type resolution, and everything is known at compile-time. If that fits your needs, you should seriously consider the compile-time version.

But what about the executable sizes?

[As we saw](https://www.sandordargo.com/blog/2023/02/08/binary-sizes-and-virtual), declaring classes with `virtual` destructors has a price. The polymorphic solution is not just slower, but it also generates a bigger executable.

|   Version                        | Binary size at -O0 | Binary size at -O3 | Binary size at -Os | 
|----------------------------------|--------------|
|  classical decorator  | 76.1K          |   35.5K   | 36.1K 
| modern decorator  | 39.9K       | 33.6K   | 34.3K   


But...

There is a huge *but* here. The scope of the example is very limited in terms of different items. Meaning that while the size of the `virtual` solution is slightly bigger, we have to ask ourselves the question of how would it scale.

The runtime solution requires creating a new subclass for each different kind of item and that costs some bytes. We cannot talk about exact sizes as it depends on so many things. But in the `-Os/-O3` optimized version, a new `JavaBook` class (based on `CppBook`) added an extra 200 bytes. But it could easily be much more depending on the class itself.

On the other hand, the compile-time solution uses templates both for taking a discount, for a tax and for taking the item too. Each different invocation will generate a new class which also adds up to the size.

Based on what I can see, if you deal with the same few types of items, but with a variety of taxes and discounts, the virtual solution will scale better.

But if there are also lots of new item types needed, the compile-time solution is not that bad not even in terms of executable size. Based on my experiments, both solutions scaled similarly when I added new types of items.

But you have to measure so that you know for sure. And also let's not forget that a compile-time decorated is not always an option as you might need that runtime flexibility.

## Conclusion

Today, we compared classic and modern implementations of the decorator patterns in terms of binary sizes. The modern version of a decorator pattern is both faster and smaller than the classic implementation. But you have to keep in mind, that the `virtual` solution might scale better and that you might need the run-time flexibility of the classic pattern.

Next week, we'll look into the observer design pattern!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
