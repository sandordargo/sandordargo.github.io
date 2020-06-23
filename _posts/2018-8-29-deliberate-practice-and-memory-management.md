---
layout: post
title: "Deliberate practice and memory management"
date: 2018-8-29
category: dev
tags: [cpp, dojo, gilded rose, branch by abstraction]
excerpt_separator: <!--more-->
---
I have recently read the eye-opening book of [Cal Newport, So Good They Can't Ignore You](/blog/2018/08/22/so-good-they-cant-ignore-you). He emphasizes a lot on the importance of deliberate practice. I also decided to take a bit more seriously my practice sessions and I reorganized [how I spend my personal pomodoros](/blog/2018/02/28/setting-yourself-up-to-succeed) in the morning and at lunchtime to have more deliberate practice. I want to strech my limits. In C++, it's not so difficult.
<!--more-->

In [one of my articles](/blog/2018/08/08/gilded-rose-revisited), I've already written about a new approach I used while implementing the [Gilded Rose kata](https://github.com/emilybache/GildedRose-Refactoring-Kata).

Now, I want to go into details regarding one part of the refactoring, the part I struggled the most with.

At that point, I've already created and implemented an `Updater` interface, to manage the `sellIn` and `quality` properties of an `Item`. But I didn't like the solution, as it didn't update directly the corresponding properties of the `Item`, instead just those of the `Updater`. Right after, it copied back the values of the `Updater` to the `Item` class.

```
class Updater {
 public:
  Updater(int sellIn, int quality) : _quality(quality), _sellIn(sellIn) {}
  virtual ~Updater() {};

  virtual void updateQuality() = 0;
  virtual void updateSellIn() = 0;

// later these became protected
  int _quality;
  int _sellIn;
 };

// There were several updaters implementing this abstract class
// ...

};

class Item {     
public:
    string name;
    int sellIn;
    int quality;
    Updater* updater;

    Item(string name, int sellIn, int quality) : name(name), sellIn(sellIn), quality(quality)//, updater()
    {
      if (name == "Sulfuras, Hand of Ragnaros") {
        updater = new SulfurasUpdater(this->sellIn, this->quality);
      } 
      // else if ...

    }

    void updateSellIn() {
      updater->updateSellIn();
      this->sellIn = updater->sellIn; // This is so ugly!
    }

    void updateQuality() {
      updater->updateQuality();
      this->quality = updater->quality;
    }
};


```

What did I want to achieve instead and what were my constraints?

I wanted to update the attributes of the `Item` class from the `Updater`. My self-imposed constraint was that I didn't want to change even the tiniest way how we have to interact with an Item in the tests. Not because I'm lazy, but the way we interact with our object in our tests is the same way our users would interact with the objects. If it changes for me in the tests, obviously it would change for our users. As such changes can be costly for our imagined clients, we might lose them when we introduce some API changes. Such changes are not welcome.

My idea was that in the constructor of the `Item` I'd pass the address of the `sellIn` and `quality` variables to the `Updater` instead of their values. Then in the `Updater`, instead of the values, I'd store references, i.e. non-null pointers.

Sounds good?

It definitely did sound better to me than the existing solution, until I implemented it.

```
class Updater {
 public:
  Updater(int& sellIn, int& quality) : _quality(quality), _sellIn(sellIn) {}
  virtual ~Updater() {};

  virtual void updateQuality() = 0;
  virtual void updateSellIn() = 0;

// later these became protected
  int& _quality;
  int& _sellIn;
 };

//...

class Item {
  //...

  void updateSellIn() {
    updater->updateSellIn();
    // this->sellIn = updater->sellIn; // This line is removed now!
  }

  void updateQuality() {
    updater->updateQuality();
    // this->quality = updater->quality; // Just like this! Yay!
  }
};

```
It didn't work. The `quality` and `sellIn` attributes of the `Item` class were not updated. Okaaay... Well, not okay, not at all! I must have missed something, I thought. I read the code. It seemed fine. I read it again. And again. And again. Looking for that missing ampersand or something similarly trivial. I couldn't find it.

It was quite late in the evening. I said I leave it like that for that night, I'd have a look into it later. Then I went to the bathroom, but I kept the laptop still turned on. Just in case the solution will hit me right in the head. And guess what, while I was standing there I realized that the problem must not be that `Item.quality` and `Item.sellIn` gets copied, but most probably the whole `Item` class gets copied somewhere and in the test I try to assert the properties of the original instance, while I update something else. I wanted to run back right then, but I had to wait a bit.

When I had a look at my test and I knew I got it.

```
//GildedRoseTextTests.cc
int main()
{
  vector<Item> items;
  items.push_back(Item("+5 Dexterity Vest", 10, 20));
  items.push_back(Item("Aged Brie", 2, 0));
  // ...
  GildedRose app(items);
  // ...
  app.processItems();
}

```

I added some logs to make it sure and yes.

The address of an `Item` was different in the constructor and in when `updateQuality` or `updateSellIn` were called. I created an Item and when it was pushed back to items vector, it got copied. That's fine. But it got copied in a bad way, including the member references.

If not implemented (or not explicitly deleted starting from C++ 11), C++ will automatically implement the copy constructor and the assignment operator for you. Is that a good thing? It doesn't matter. What matters is that it will happen and sometimes that implementation will not work the way you would expect it. Like it happened in this case. 

What happened, in fact, is that a new copy of Item was created, a copy of the `sellIn` and the `updater` was created (at new addresses), but the reference to `sellIn` in the `updater` still pointed to the "old" sellIn of the copied object. So in fact `sellIn` was updated, but not the one we wanted.

The fix was easy, I just had to implement the copy constructor and the assignment operator:

```
Item& Item::operator=(const Item& i){
  this->name = i.name;
  this->quality = i.quality;
  this->sellIn = i.sellIn;
  this->updater = i.updater;
  return *this;
}


Item::Item(string name, int sellIn, int quality) : name(name), sellIn(sellIn), quality(quality)//, updater()
{
  updater = Updater::CreateUpdater(name, this->sellIn, this->quality);
}
```

I was more than happy to see the implementation I wanted to achieve finally working. After the copy, the `updater`'s reference also pointed to the new `sellIn`.

I also found two important takeaways:

1. Never forget about the copy constructor and the assignment operator.
2. C++ is a language that gives you a great power over how things should happen. And as you might know it well, with a great power, great responsibility also comes. Never forget that either.