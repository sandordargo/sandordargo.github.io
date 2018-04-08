---
layout: post
title: "Coders at work: Brendan Eich"
date: 2017-10-20
category: books
tags: [books, coders at work]
header: "The fourth guy interviewed in <a href=\"http://amzn.to/2wKEeVt\">Coders at Work: Reflections on the Craft of Programming</a> is yet another JavaScript guy. In fact, the JavaScript guy: <a href=\"https://brendaneich.com/\">Brendan Eich</a>. Here is a small reflection on this chapter."
---
At a certain point, this guy really reminded me to a former colleague of mine. Eich said that "a blue-collar language like Java shouldn't have a crazy generic system because blue-collar people can't figure out what the hell the syntax means with covariant, contravariant type constraints." This sounds extremely elitist.

That colleague of mine said that python doesn't need real private methods and fields, because it is for adults and adults can behave themselves. If you mark something as private even if the language will not ban you from using it, it doesn't matter. It was said so and if you abused the language, deal with the consequences.

So there should be a language for adults and for children. You are not given all the toys, you couldn't use them anyway. I don't really like it. There should be automated tests and serious code reviews so that you can safely learn to use those toys.

What I did like, on the other hand, is how he designs code. He is not fond of ivory-tower design and design patterns. It's important to know design patterns, but it's a bad thing to worship any and just wait for the nail you can hammer with it - and if you are waiting for a nail, everything will appear so. He does a lot of prototyping (no surprise that he created the most widely used prototype-based language), he writes some pseudo-code and then writes the code bottom up. I used to follow similar practices. I'm more with TDD nowadays but sometimes I combine these approaches.

Something which struck me, by the way, is that he talked so much about debugging and stepping through the code, but not a single word about automated tests. That's a bit hard to understand for me.

Regarding suggested books, he is not the first person to propose [Knuth's Art of Computer Programming](http://amzn.to/2xu5NaS), but the first to mention [Brian Kernighan's books](https://www.amazon.com/gp/search/ref=as_li_qf_sp_sr_il_tl?ie=UTF8&tag=mummysherpa-20&keywords=Brian Kernighan&index=aps&camp=1789&creative=9325&linkCode=xm2&linkId=415a78c9fbd3c764d8424dcfdb668276).

Not surprisingly he still likes coding. Thinking about trade-offs, what's worth to implement, what's not. What is small and simple enough, what is too theoretical. To find the sweet spots. He is definitely a passionate coder. Someone who can fulfill his own ideal of a principal engineer who leads by example, without the authority of a manager. We need such people, because "you don't have enough hours in the day or fingers."
