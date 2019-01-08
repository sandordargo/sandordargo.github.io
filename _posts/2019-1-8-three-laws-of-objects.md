---
layout: post
title: "The Three Laws of Objects"
date: 2019-1-8
category: dev
tags: [ood, oop, design, clean-code]
excerpt_separator: <!--more-->
---
Three laws of objects. Sounds catchy enough? To me, it did. I read about these laws in [Ken Pugh's Prefactoring](https://amzn.to/2VoCO0k).

If you sense a not too much-hidden reference to Asimov's [Three Laws of Robotics](https://en.wikipedia.org/wiki/Three_Laws_of_Robotics) from [I, Robot](https://amzn.to/2Rp2sD8), it's not a coincidence. It's the author's purpose.

Enough, Sandor, show me the laws - you might say and you'd be right. There you go:
<!--more-->

* An object shall do what its methods say it does
* An object shall do no harm
* An object shall notify its user if it is unable to perform a requested operation

Let's expand them one by one.

## An object shall do what its methods say it does

Let's break this law even down into two paragraphs.

### A Rose by Any Other Name Is Not a Rose

For each concept in the system, you must create a clearly defined name. One clearly defined name. Stick to it.

The name is an important part of the method itself. Meaning that if you change the name, but keep the code intact, the method becomes another one. A new name will hold at least a slightly different meaning, so conceptually the method changes. Even worse if you start using both the old and the new names at the same time.

Would the famous painting of Magritte be the same piece of art if it was called "This is a pipe"? 


![The Treachery of Images from https://knowyourmeme.com/memes/this-is-not-a-pipe-parodies]({{ site.baseurl }}/assets/img/magritte-this-is-not-a-pipe.jpg "The Treachery of Images from https://knowyourmeme.com/memes/this-is-not-a-pipe-parodies")

### Principle of Least Surprises

So we have a clearly defined name for a given concept that is used by a given object/method. It is also important that the code does what the name reasonably indicates.

It is necessary that the method doesn't do anything else. It should not cause us some unwanted surprises. In the book, the example of `delete` and `remove` is used. Let's say you store references to some objects in an array. Should a `remove` operation delete the referenced object itself? Or should a `delete` operation remove the reference from the list?

Good question.

Given a name, the method should do what you would expect by that name. Your expectations might be based on plain English or on some company dictionary, the most important is to stay consistent.

But surprises might still arise. What if your function is not [idempotent](https://en.wikipedia.org/wiki/Idempotence)? Meaning that you call it twice in a row with the same list of arguments and it doesn't return the same value. Would it be surprising for you? Unless the underlying data source doesn't change, it most probably would. But let's say, [it depends](https://dev.to/thorstenhirsch/poll-do-you-know-what-idempotent-means-4759). Just like some unwanted side effects that change the state of your object, even though it's supposed to be a getter. Apparently, functional programming is a possible answer to guarantee at least the least possible surprises. 

Regarding the name, keep in mind that a [good names depends](http://arlobelshee.com/good-naming-is-a-process-not-a-single-step/) on you. You are responsible to name things well in the code you deliver.

## An object shall do no harm

The second law seems to be an obvious next step after the Principle of Least Surprises. Unless a function/object harming itself is something that you'd expect to have. What is considered harm anyway?

[Prefactoring](https://amzn.to/2VoCO0k) takes an object creating widgets as an example. In that program, for each widget creation, the configuration from a file had to be read. But the object responsible for creating those widgets didn't close the configuration file as soon as they were not needed only much later at destruction time. The running OS could open 20 files simultaneously. 

As such, no more than 20 widgets could be created. The object creating widgets was harmful as it kept locked unnecessary resources.

In C++, it's even more easy to create some harmful objects. Think about objects leaving memory leaks after themselves. They are harmful. How much? That depends on the size of the leak. If you are using raw pointers when ownership has to be handled, it's kind of guaranteed that you'll eventually find yourself fixing memory leaks. Want to learn a bit more about memory leaks? [Check out this Wikipedia page first](https://en.wikipedia.org/wiki/Memory_leak). Ray, I mean [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization) should be a great friend of yours while you're learning the different ways of tackling memory leaks.

If you work with a language where you don't directly control object destructions - so you depend on a garbage collector -, you have to pay explicit attention to release resources manually.

But even if you use a language like C++, you have to think twice, when some resources can be released. The sooner, the better.

## An object shall notify its user if it is unable to perform a requested operation

According to the Third Law of Objects, if an object faces troubles, it shall never stay silent, it must speak! It might print to the logs, it can return some error codes or it can throw exceptions. As you wish, or better to say as it makes sense given the circumstances.

The point behind this law is that if the object fails silently or does a reparative action without reporting it, then debugging will become very difficult. Let's say that you have a function that should append a new element to list, but only if such an element is not part of the list yet. As an example, let say you have a big garage and you a function called park car. You have your Ferrari already in the garage, but then you call the method again with your Ferrari:

```
garage.park(myFerrari);
```

If the method does nothing or it replaces the previous instance of your Ferrari with this one, most probably you just mask an error which will become more daunting later on. But when the error will come back to you, you won't have all the logs that could help you investigate easier.

Just like in management, report issues as soon as they arise in software too.

## Conclusion

In this article, we have read about the Three Laws of Objects, as Ken Pugh described them in [his book called Prefactoring](https://amzn.to/2VoCO0k). According to these laws, objects should act as their name suggest and their behaviour shouldn't cause surprises. While they do as they say, they shall leave no harm nor in the system nor in the neighbouring objects. Last but not least, objects should report their difficulties, so you are aware of problems as early as they arise.

Happy coding and reading!