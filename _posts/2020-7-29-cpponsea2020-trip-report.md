---
layout: post
title: "Virtual Trip Report: C++ On Sea 2020"
date: 2020-7-29
category: dev
tags: [tripreport, cpp, conference, techtalks, ]
excerpt_separator: <!--more-->
---
Last week, I "went" to the [C++ On Sea 2020](https://cpponsea.uk/2020/schedule/), which was my second C++ conference, after [CPPP 2019](http://sandordargo.com/blog/2019/06/26/travel-report-cppp). I put went between quotes because as you might have guessed due to the [Coronavirus](http://sandordargo.com/blog/2020/04/08/gratitude-for-covid19), the organizers had to make a choice. They could either cancel the whole event or move it online.
<!--more-->

While quite a few events were canceled this year, in this case, luckily, the organizers with the lead of [Phil Nash](https://twitter.com/phil_nash), decided to go on with their work and created an awesome online conference.

It was three days of C++ with history, with practical advice, with legacy code, with new features, with production code, with katas on three different tracks, and even with live music coming from the US right before the closing keynote.

## The keynotes

Even if I just take into account the keynotes, it would be very difficult to choose which one I liked the most.

On the first morning, Walter E Brown shared his [retrospective on the evolution of computer sciences](https://www.youtube.com/watch?v=sIbERoa1Wj4), going back thousands of years right until nowadays. Even if one might say that the topic was not very practical, I think it's important. In the last months, I showed you books about [humanity's](http://sandordargo.com/blog/2020/06/17/sapiens-by-harari) and [computer science's history](http://sandordargo.com/blog/2020/07/08/code-the-hidden-language). So Walter's keynote perfectly fit into my reads and it was a very good reminder. Don't forget Churchill's words: 

>“A nation that forgets its history has no future”

On the second day, we had a [very technical keynote by Nico Josuttis](https://www.youtube.com/watch?v=elFil2VhlH8) basically on [`std::jthread`](https://en.cppreference.com/w/cpp/thread/jthread). It was really practical and detailed on what problems [`std::thread`](https://en.cppreference.com/w/cpp/thread/thread) has and how in the Committee they were working on fixing these issues with the introduction of [`std::jthread`](https://en.cppreference.com/w/cpp/thread/jthread) where `j` is apparently not for _Josuttis_ - who as a non-concurrent-programming expert led the workgroup - but for _joinable_. For me, it was a bit difficult to follow as I'm not working in a multi-threading environment, but it was enjoyable and I do know now that we should all use `std::jthread`s instead of `std::thread`s.

The event was closed with [the remarkable keynote of Herb Sutter](https://www.youtube.com/watch?v=BF3qw1ObUyo). And saying that it closed the conference is completely true without the slightest exaggeration. By the official program, there was supposed to be a wrap-up after, but due to some technical difficulties, we lost Herb for a good 20 minutes, which Phil used for the wrap up before we got Herb back. Then he continued where we lost him and delivered a great talk.

Why certain things fail and seemingly very similar initiatives, products succeed? This was the topic of his keynote. Identifying those - not so - tiny differences that can help us succeed. I have to tell you that Herb is an excellent presenter, many things that I learned at various presentation skills training I could pinpoint in his talk. So obviously he organized his content around three main points:

* What is the value you propose?
* How easy it is to start using your product?
* How easy it is to start having benefits?

Just to very briefly summarize, you have far better chances if you solve an existing problem if your product removes existing pain from its potential users. If your new thing is available by design, like TypeScript wherever there is a JavaScript interpreter, you also have better chances. And if you can just insert one line here, one line there to the existing codebase so that you start having those tiny bits of benefit that your new thing is proposing, there is a fair possibility of a faster adoption.

I'd really recommend watching his talk from the beginning till the end to anyone who ever wanted to launch a product, an API, or just a new major version of a software to watch his talk.

## The talks

Not counting the keynotes, there were 27 talks and it would be overwhelming to give an overview of all of them both for you and me. Anyways, you can watch them all [here](https://www.youtube.com/channel/UCAczr0j6ZuiVaiGFZ4qxApw/playlists).

In order to keep this report within a reasonable length, I'm going to pick 3 talks. One that I particularly liked, one that I found surprising and one that was entertaining.

### The one I particularly liked

The one I particularly liked is ["Correct by Construction: APIs That Are Easy to Use and Hard to Misuse"](https://www.youtube.com/watch?v=nLSm3Haxz0I) by the man behind the name behind [the website](https://godbolt.org/). Yes, that name is _Godbolt_. Matt shared some best practices for people delivering APIs. 

How many of us, developers, create APIs? 

Maybe 10%, 20%? 

Hell, no! All of us! 

A class's public interface is an API and will be used by your colleagues. Or maybe only by the future you. Or even your present self. 

These pieces of advice do matter.

From Matt's talk, we could learn about how strong typing helps to avoid expensive typos, and how replacing booleans with enums helps increasing the usability of your API. It was also really interesting to see [user defined literals](https://en.cppreference.com/w/cpp/language/user_literal) in action (such as `1000_dollars` or `100_qty`), that can further decrease the probability of typos and increase readability.

Often, when enums come into question, we soon end up handling switches. From Matt, I learned that it's better to avoid `default` cases because if you turn on most compiler warnings and you handle them as errors, the compiler will catch unhandled cases. In case (pun not intended), you have a `default` and later your enum is extended, the compiler will not remind you that you have to handle that new case. On the other hand, if you don't have a default, it will immediately signal you this problem.

The key is to be picky and handle warnings as errors, something Matt was advocating during the talk.

He mentioned a lot of other things, but I'd like to finish only with one that I'll later turn into an article here. Write fewer comments, but more expressive code. __Turn comments into actionable items, such as either compile- or runtime checks.__

If we wanted to summarize his talk in one sentence, it would be _let the compiler help you_.

### The surprising one

The presentation that I found quite surprising is ["Structured bindings uncovered"](https://www.youtube.com/watch?v=uZCvz-E1heA) by [Dawid Zalewski](https://www.linkedin.com/in/dawid-zalewski/?originalSubdomain=nl). So what is this about?

A structured binding declaration gives us the ability to declare multiple variables initialized from a tuple, pair or a struct/class. Here is an example:

```cpp
// from a container
std::array<double, 3> myArray = { 1.0, 2.0, 3.0 };  
auto [a, b, c] = myArray;

//from a pair
auto [a, b] = myPair; // binds myPair.first/second

// from a map, yes even this works!
std::map myMap {/* your data*/};
for (const auto & [k,v] : myMap) 
{  
    // k - key
    // v - value
} 
```

This is all nice and simple, even if there are some shortcomings compared to other languages, such as you can't ignore values. Though this is not everything, there is more depth on this topic. Dawid also presented what kind of helpers a class has to implement so that it can be used via structured bindings.

But when you all make it work and for some reason, you decide to have a look under the hood either by a debugger or by profiling, you'll figure out that what happens is not exactly what you - most probably - thought. Dawid's presentation covers all those oddities. By the end, you'll learn when to use structured bindings without moderation and when you should think about twice before you start introducing this feature of C++17.

### The entertaining talk

Last but not least, an entertaining talk! ["Lambda? You Keep Using that Letter"](https://www.youtube.com/watch?v=Bai1DTcCHVE) by [Kevlin Henney](https://twitter.com/KevlinHenney). Obviously, the talk is about lambdas. Not specifically in C++, but in programming languages in general. He covers the origin of [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus), the origin of the name term lambda in programming. As Kelvin said, this talk and the examples he prepared, were not for production usage. It was food for thought. And besides, he is an awesome presenter who hid quite a few puns in his talk. And there were even more puns in the chat window. Some of the attendees were so good in bad jokes, that they even deserved a punishment - pun intended.

It was both an educative and entertaining talk, I'm happy I chose his presentation.

## Conclusion

I could keep writing about C++ On Sea for so long. Or at least about the lightning talks where you could even learn how to KonMari your code or how to use the greek question mark to freak out your colleagues. And obviously you should watch [Sy Brand's cartoon](https://www.youtube.com/watch?v=pGO65OHo0EM). But I have to stop and let you [watch the talks](https://www.youtube.com/channel/UCAczr0j6ZuiVaiGFZ4qxApw/playlists). I really enjoyed C++ On Sea. And the talk I learned the most from was obviously mine. I spent so much time learning more about my topic, preparing for the P day and I do think that it paid off, even though it was not perfect. It never will be. I hope I'm not the only one who thought like that. You can check it out [here](https://www.youtube.com/watch?v=BEmAo6Fdg-Q).

So one last question. How did the online format work? I'm obviously a bit disappointed as someone who likes to travel, not to mention when all the fees are covered... :) There were some technical difficulties, but I think that the organizers did a great job in tackling those and these difficulties did not affect the enjoyability of the event.

Although I improved a lot in socializing, it's still a difficulty for me. The different platforms (_Remo, Streamyard, Discord_) used during the three days gave a lot of opportunities to make some connections, and for me maybe it was even easier than it would have been in real life. All in all, it was a great event, but I hope next time I'll be able to meet in real life the people I got to virtually know a little bit.

I'd like to thank the organizers' outstanding work preparing the conference and the opportunity they gave for me to present my topic and even a lightning talk.

See you soon!