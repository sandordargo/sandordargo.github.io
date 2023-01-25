---
layout: post
title: "Deep vs shallow modules"
date: 2023-1-25
category: dev
tags: [books, softwaredesign, cleancode, watercooler]
excerpt_separator: <!--more-->
---
I keep telling that I don't post negative book reviews. It's because no matter how much I dislike a book, I can be pretty sure that the author made a huge effort writing it. I have absolutely no reason to post negative reviews.

I'm not afraid of giving constructive criticism in real life and at work when I'm personally involved. When it comes to books, there are just so many good ones I can focus on.

I won't share a negative book review either today, on the other hand, I'd like to share some ideas from a book that goes completely against how I see software development.

I recently read [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=5e8a182a2cccdd5aaeef828c6cb1ffb4&camp=1789&creative=9325) by [John Ousterhout](https://web.stanford.edu/~ouster/cgi-bin/home.php). The author studied at the best universities and is still teaching at Stanford. He received both the Grace Murray Hopper Award and the ACM Software System Award. He obviously has a deep knowledge and based on [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=5e8a182a2cccdd5aaeef828c6cb1ffb4&camp=1789&creative=9325) I must tell you that he knows how to write well.

He writes mostly about the opposite I've been learning during the last couple of years in the software craftsmanship movement. It doesn't mean that he doesn't have great points, because he has and it's a book worth reading.

Today, let's focus on one of the key ideas delivered by this book.

## Deep modules everywhere

Behind all the strange things, there is one notion, one idea.

Deep modules.

Let's treat the term module quite vaguely in this case. A component, a class, an interface, a method can all be considered a module.

According to Ousterhout, *"the best modules are those that provide powerful functionality yet have simple interfaces."*

Actually, I think it's a good point as long as we talk about software components and the public interfaces of whatever modules. 

## APIs should be hard to misuse

As the author wrote, you should not have to call many different methods to achieve a simple thing. And even more importantly, the most common cases should be no-brainers. The most common cases should be achieved without additional effort.

If you think about a constructor, the most common way of instantiating the class should require the least amount of parameters. Even for the more complex cases, APIs should be well-documented, and hard to misuse.

In any case, according to Ousterhout, you should not need to call many different methods in order to achieve a simple thing! As a counterexample, he mentions the Java way of reading serialized objects from a file. To achieve a simple thing, you need to manually instantiate 3 different classes and pass each one to the next. I completely agree with him on this.

At the same time, the author recommends that both public and non-public interfaces should be deep. Everything should provide something deep, complex and powerful. Small classes, short methods are not such things. 

This goes completely against what I learned during the last 10 years. Ousterhout advocates for what I considered evil and barely talks about something I considered one of the most important aspects of writing code...

## Testing seems difficult

The problem with deep modules, deep classes, and deep methods is that they are hard to test. They are designed to perform a lot of things. They probably instantiate lots of objects, they call some other APIs. And you cannot mock those, you cannot replace them with fakes! Or if you can, the APIs become so complex that it goes against what we already discussed.

As such, you can't really have a complete unit test suite. If you're lucky, you might be able to have some good integration or end-to-end tests, but no good unit tests.

On the other hand, if shallow methods are designed well, they will take their dependencies as parameters. They can be set up, and created in a way that unit testing becomes not only possible but even easy.

This one benefit already justifies to me why your methods should not be too deep and long. But it's not the only reason.

## Comments

In deep entities, you'll end up with huge blocks of code. Long methods that are hard to understand unless they are well commented. In [Clean Code](https://www.amazon.fr/dp/0132350882/ref=as_li_qf_asin_il_tl?ie=UTF8&linkCode=gs2&linkId=0e9c9c3d84461c60a88af599d18d9d04&creativeASIN=0132350882&tag=sandordargo07-21&creative=9325), we learned that in a long function there will blocks that are separated by empty lines. They are usually responsible for a specific task. These blocks usually also have a leading comment explaining what that block is responsible for.

These blocks serve as good indicators of what can be extracted as a method responsible for one thing and the comments give nice hints to name the new functions.

The usually cited problem is that comments are not well maintained so they cannot hold the truth. According to the author, we just need some discipline and reviews should ensure that these comments are well-maintained. 

I'm not that optimistic, but I agree that even maintaining function names need quite some discipline. We are lazy and we lack discipline. Yet, I think that we sooner update function names than comments lost in a huge function...

## The cognitive load

The topic of deep vs shallow modules reminds me of discussions I had with a former colleague. I have a deep respect towards that guy, a smart and hard-working C++ developer. I think he would probably agree with the author John Ousterhout on so many questions. We also agreed on many things, but we thought in very different ways about the optimal size of a method.

While reading code written by me, he always complained about the mental complexity of jumping into yet another function, yet into another class.

I heard him, I understood him, but I still don't agree. I don't think that short, well-named methods in the implementation increase the cognitive load. Why should I check the implementation of a well-named entity if I don't have to modify it, if I'm not looking for a bug in that area of code? I simply don't care and don't go in. If I don't need to understand the implemenation details of that piece of code. The same way as normally you don't check the implementation of the standard library and you don't check the assembly code generated. Sure, you could gain some deeper insight, but most often there is no need. 

In my opinion, exactly the contrary happens. Each well-named method or variable hides a bit of complexity and decreases the cognitive load.	

## Conclusion

In [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201?&_encoding=UTF8&tag=sandordargo-20&linkCode=ur2&linkId=5e8a182a2cccdd5aaeef828c6cb1ffb4&camp=1789&creative=9325) I learned about a different approach to software development. A different from what I learned for long years. The author, [John Ousterhout](https://web.stanford.edu/~ouster/cgi-bin/home.php) advocates for using deep modules instead of so-called shallow ones.

If we follow his recommendations, we'll end up having entities that are barely short, but they are always responsible for complex meaningful features. That also brings long blocks of code, comments instead well-named short functions and more difficult testing in practice.

While I don't completely agree with all his points, there is definitely one to consider. We should design our public APIs in a way so that the most common functionalities, the most common ways of using those functionalities should be dead simple to use.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
