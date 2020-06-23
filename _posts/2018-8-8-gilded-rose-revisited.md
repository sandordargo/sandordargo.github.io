---
layout: post
title: "Gilded Rose kata revisited"
date: 2018-8-8
category: dev
tags: [cpp, dojo, gilded rose, branch by abstraction]
excerpt_separator: <!--more-->
---
If you are into coding dojos and solving katas, you might have already tried the [Gilded Rose kata](https://github.com/emilybache/GildedRose-Refactoring-Kata) by [Emily Bache](https://twitter.com/emilybache?lang=en). 

In this kata, you are given some existing code that handles the quality and the number of days before expiration properties of the products in a store. The code handles almost everything in one single huge function. Unsurprisingly, the goal of the kata is to refactor the code. Besides, there is also a new functionality to implement. 
<!--more-->

I've done this kata a couple of times before, but recently when I did it again with my team, we took and we discussed a totally different approach and I want to share some of its aspects.

But first things first. How did I do it before?

Let's start with the testing aspect.

Either I just automated the execution and evaluation of the [characterization tests](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/master/cpp/GildedRoseTextTests.cc) or I implemented the unit tests. In the latter case, I read scrupulously the [requirements](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/master/README.md) and I added the unit tests one by one. If I found a bug in the implementation, I fixed it or documented it depending on the discussion I had with my partner. In my opinion, it's not evident what you should do in such a situation. Probably the buggy behaviour is acceptable because possibly your clients take that buggy output granted/by-design and you'd actually break their flow in case you fixed the bug you identified. Such cases happen to us in real life too, especially when we are maintaining long-lived products.

The approach I take for testing can have an effect on the way I refactor the code. When I use only the characterisation tests, usually I use the capabilities of my IDE for the refactoring. I extract till I drop and I rename as much as I can. Once the code is a little bit more readable, I start to do some manual refactorings as well.

If I implement unit tests one by one, I might be more adventurous with refactoring/reimplementing the small pieces of functionalities. From the very beginning.

How the code will be structured might highly depend on the choice of your language/IDE combination. For example with C++ and Eclipse, you cannot extract some code into a new class, whereas you can do it with Java and IntelliJ (maybe with Java and Eclipse too). In other terms, it's easier to end up with a more object-oriented code with Java than with C++ without thinking too much. (Is that a good thing? I leave that to you.)

On this occasion, to save some time, we decided to stay only with the characterization tests. Our main goal was to try [branching by abstraction](https://martinfowler.com/bliki/BranchByAbstraction.html).

The main idea behind this model is to have a deployable version of the code after each small step that can be either refactoring or implementing a new feature. Why is this so important? Because using this approach, one can perform big changes without maintaining a long-lived feature branch. You free yourself from merging troubles and what you are doing is transparent to your peers.

Let's see step by step how we implemented the [Gilded Rose kata](https://github.com/emilybache/GildedRose-Refactoring-Kata)!

# Step 1: [extracting the body of the of the for loop](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/65a26523958d1c6b7eb2f9e8d81cf80492a8adc8).

This step is quite evident. I also changed how the iteration happens, so instead of referring to the elements by their index, I changed to a range-based `for` loop - this step required to upgrade the C++ version to C++11.

# Step 2: [Implement the quality and sellIn behaviour for non-special items](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/ac069ef55e1a5cd5a0c512faad6b104ecc4bbc30).

And here it comes, the branching-by-abstraction. We introduce a big `if-else`.

```
if (item.name != "Ragnaroos" ...) {
  // freshly implemented behaviour
} else {
  // old code
}
```

In case the item is a non-special one, the new piece of code is used but in all other cases still, the old behaviour is executed.

# Step 3: [Move the updates to the Item class](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/ce61820212f68e536ef189675a253edd65c786d1)

As `quality` and `sellIn` are attributes of an item, it makes sense to maintain them in the `Item` object. At this point, we might be tempted to introduce methods such as `decreaseQuality` and `decreaseSellIn`, but it would mean a quite short-term dead-end, so better to stick with the more abstract `updateQuality` and `updateSellIn` names.

# Step 4: [Implement the behaviour for the special item of "Sulfuras, Hand of Ragnaros"](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/c63ed5e9a9406582b82f4833447b3ab074900885)

According to the specs, _Sulfuras_ does not age and its quality rests the same. There is nothing to do with their attributes! If you run forward, there is already a chance here to refactor, but it's not really needed at this moment. So the code is as simple as that:

```
if (item.name != "Sulfuras...") {
  
}
```

# Step 5: [Implement the behaviour for Aged Brie](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/f47e38300412b33ec0b3a3df1dfacf3b481f8578)

While the quality of normal items decreases over time, _Aged Brie_'s increases and not even with the same speed. This means we cannot simply reuse `Item::updateQuality`. At this point, we implemented the behaviour right there in the `processItem` method. If you have a deeper look, although the tests pass, the implementation is not completely in line with what the specs say. Or maybe the specs are not so well written. Who knows? This time, I decided to stay with the already existing behaviour.

This was the point when things started to get complicated.

For non-special items, the behaviour is completely encapsulated in the `Item` class. For _Sulfuras_ and _Aged Brie_, the behaviour is in the `GildedRose::processItem` function. It seems quite obvious that this is not optimal, and it'd be good to have all the different behaviours implemented in the `Item` class.

One option would be to make `Item` a base class with virtual `updateQuality` and `updateSellIn` methods, but I was not fond of the idea. It didn't seem like a small refactoring. Besides, I reminded myself of the [Liskov Substitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle). Whenever an `Item` is expected, I wouldn't be able to use an `AgedBrieItem` for example as `AgedBrieItem` doesn't extend but alters the default behaviour. Yet the biggest problem would have been that change of the instantiation. The burden of updating all the tests, and imagine if our clients are using the `Item` class...

My colleague who organized the dojo presented us another idea suited for this kind of problems. Hide the changing implementation details in another class, thus we don't have to transform Item into a common parent. We don't even have to change how the Items are instantiated. It sounded good enough for us. Here it comes.

# Step 6: [Extract the behaviour handling into an `Updater` class](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/32b9b235d242fee994cac6bf4f394fbc73a52dad)

So while Item is still is still instantiated the same way with a name, a quality and a sellIn date, it's internal structure changes. Yes, the size of your class changes and your clients will have to recompile, but I think this is less and less issue these days. On the other hand, they will not have to change their code, because you only modified your internal structure at this point.

In the constructor of the `Item` class, or in a method which is called from the constructor, based on the Item name an `Updater` will be created.

Then the `Item::updateQuality()` and `Item::updateSellIn()` will delegate the work to `Update` class' corresponding methods.

In order not to violate the Liskov principle, we shall not use inheritance. In this use case, derived classes would not extend the base class' behaviour they would simply alter it, which goes against our principles.

As in C++, there is no built-in concept for interfaces, I created an abstract base class that contains only pure virtual functions - apart from the constructor/destructor. Then I created the first three Updater classes, namely DefaultUpdater, RagnarosUpdater and AgedBrieUpdater.

```
class Updater {
 public:
  Updater(int& sellIn, int& quality) : _quality(quality), _sellIn(sellIn) {}
  virtual ~Updater() {};

  virtual void updateQuality() = 0;
  virtual void updateSellIn() = 0;

 protected:
  int& _quality;
  int& _sellIn;
};
```

I went through many iterations and commits before the Updater class actually reached this point and I had to tackle one serious bug that I'll cover in more details in another blog post.

# Step 7: [Create the Updater classes for the rest](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/48ae24e02ed93d587331eaff1f2dba63d5e91850)

At this point, I still had to implement two updater classes. One for the backstage passes and one for the Conjured items which is a new feature. At this point, these are only handwork exercises.

# Step 8: [Remove the Original branch of code](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/fcfa2ca3826f865c03bfea33808d6701c64add28)

You might have noticed that up until this step, my big if-else was just growing in `GildedRose::processItem` which was not necessary, but I didn't want to touch in. Instead, I remove it completely now. As such, the whole function will be only two lines long.

```
void GildedRose::processItem(Item& item)
{
  item.updateSellIn();
  item.updateQuality();
}
```

# Step 9: Any cleanups to do

We are done with the bigger part of the refactoring as well as with the implementation of the new feature. Let's look for other refactorings to do.

The `GildedRose` class seems quite fine, but in fact, I don't think we need `processItem`. It shouldn't know which two functions of an `Item` have to be invoked and it shouldn't know the order of the invocation either. `GildedRose::updateQuality` seems to be a very bad name.

Once [it was done](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/fcfa2ca3826f865c03bfea33808d6701c64add28), I decided to clean up the `GildedRose.h` in a sense that I moved every class definition to its own header and the implementation to the corresponding source files. Up to this point, it was convenient to work in one file, but it's time to move things where they belong to. It will give us the possibility to make some further refactorings, after we can use includes and forward declarations properly.

This step also required to [modify our Makefile, to include all the new files to the build](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/345e0e7ae173976af963dff31d9cfb7345e5782f).

Finally, I could [remove the instantiation of the `Updater` from the `Items` consturctor](https://github.com/sandordargo/GildedRose-Refactoring-Kata/commit/0f94efb095c8562fdc52574e3325181b15279599), and I moved it to a static factory method inside the `Updater` interface/abstract class.

I could see some other possibilities to refactor, but at one point, one has to stop. I stopped here.

## Takeaways

I've worked on the [Gilded Rose kata](https://github.com/emilybache/GildedRose-Refactoring-Kata) a couple of times, and even though it was a bit different every time, this was far the most interesting occasion.

To me the most interesting concepts were:

- Delegate to another class (hierarchy) the work, so that you don't have to make your client facing a new class hierarchy instead of the one single class he used to have. As such, I could keep the instantiation the same all the time. I didn't have to change the existing tests.

- I used the idea behind abstraction by branch. The new code was used for the parts I already finished refactoring/reimplement, while I didn't touch the old code at all. In the end, I could remove all the old code at once. This seems indeed quite same for implementing bigger migrations or to conduct massive refactorings.

I'd encourage you to do the [Gilded Rose kata](https://github.com/emilybache/GildedRose-Refactoring-Kata) and to document how it went.