---
layout: post
title: "Travel report: CPPP 2019"
date: 2019-6-26
category: dev
tags: [trip report, travel report, cpp, conference]
excerpt_separator: <!--more-->
---
Right after I was returning from a three and a half weeks long road trip with destinations in 5 countries, then attending an advanced presentation skills workshop, I was finally heading to the airport to catch a plane to Paris in order to attend the very first [CPPP](https://cppp.fr/) conference.
<!--more-->

The first evening I had the chance to meet a friend of mine from university. Each time I have something to do in Paris, we try to meet each other and taste some craft beers while discussing what happened to us since the last time. Thank you [CPPP](https://cppp.fr/) and [my employer, Amadeus](https://jobs.amadeus.com/), you made this happen once more!

Next morning after having breakfast, I had nothing to do, but to go to the [venue of the conference](http://uicp.fr/?lang=fr#section7). It was located in a conference centre right next to the Eiffel tower. When we stepped out during breaks to have some fresh air, this is what we could saw. 

![Eiffel Tower From CPPP]({{ site.baseurl }}/assets/img/cppp-eiffel.jpg "Eiffel Tower from CPPP")

Needless to say, it was quite easy to spot the non-local attendants.

There were three tracks at the conference for the approximately 200 participants. One track purely in French dedicated for the beginners, the other two in English for more advanced topics.

## Emotional Code by [Kate Gregory](https://twitter.com/gregcons)

There were two hours dedicated for this presentation which I thought would be extremely long. [Kate](https://twitter.com/gregcons) finished speaking after 90 minutes leaving a lot of time for questions, which I highly appreciated.

In her presentation, Kate claimed that we - software developers, even the C++ ones! - are human beings with - wait for it - emotions inside! She's been reviewing code for long decades and of course, often she looked up and asked but WHY - and other questions...

![WTFs per minute at code reviews]({{ site.baseurl }}/assets/img/code-quality-measurement-wtfs-minute.png "WTFs per minute at code reviews")

After some time, she realized that this _why_ is worth answering because the response can reveal so many things about the author, the team, the circumstances. Often, the code hides the coder's negative emotions such as fear, arrogance, selfishness or laziness.

I'd like to elaborate on the last one.

Sometimes you might feel that the author of a piece of code was lazy and just committed whatever worked. But in terms of code produced, it's really easy to mix laziness with _crunch_.

When a team is in a constant crunch, the question often is _what is the minimum one can do_, simply because the poor developer has really no extra ten minutes to fix something because he wants to get home to see his kids before they go to bed and the whole cycle starts again.

Sometimes there are people who simply don't care and instead of delivering good code, they just snake... But that also reveals some other negative emotions towards the company and these negative emotions might lead to [quit the team](http://sandordargo.com/blog/2018/09/26/how-not-to-quit).

The goal of understanding the underlying emotions is to develop empathy and to try to remove any impediments. Many times the impediment is not the person, but - bad reactions given to - bad management, an offensive code reviewer, etc.

If we accept that emotions are involved in our code and we can recognize them, the next step is to turn them into positive ones, such as confidence, humility, generosity.

I don't want to go more into details, probably Kate's presentation would deserve a whole long article, but you can also [watch an earlier one](https://www.youtube.com/watch?v=uloVXmSHiSo).

## Improve your C++ with the algorithms of the STL by [Mathieu Ropert](https://twitter.com/MatRopert)

The next presentation I attended was about the [algorithms](https://en.cppreference.com/w/cpp/algorithm) that you can find in the C++ STL. In an hour it is difficult to have an overview of all the algorithms included in the standard library and that was not his goal. 

[Mathieu](https://twitter.com/MatRopert) presented a bit the history of the algorithms and explained why don't we have all/most of these algorithms as part of the containers' interface. Long story short, it would require a lot of code duplication, while keeping the algorithms elsewhere made it possible to use some generic implementations.

Before actually talking about some algorithms, Mathieu reminded us of the most important concepts of iterators.

In the remaining time, he decided to present us a couple of indispensables and some more interesting algorithms.

I would put `find` et al, `copy`, `transform` functions in the former group, while in the latter he put [Sean Parent](http://channel9.msdn.com/Events/GoingNative/2013/Cpp-Seasoning), I mean the `rotate` algorithm and some partitioning ones. One more important thing to remember was how to use and combine the algorithms of `erase` and `remove`. (Briefly, remove doesn't erase, so you have to wrap `remove` into `erase`).

In overall, even if I know many of the algorithms, I didn't come away with empty hands, but some more topics I want to discover better.

## Take a break

The 2 hours lunch break was a little bit longer to my taste, but at least it didn't just let us refill our tummies with some delicious bites, but we could also discuss a lot and even stroll around the neighbourhood of the Eiffel Tower. It changed a lot in a bad way since the last time I visited. You cannot freely walk under it, but there are big - but quite transparent - fences and you have to wait quite long queues after security control. It's a pity to see this at more and more places.

![Lunch break stroll at CPPP]({{ site.baseurl }}/assets/img/cppp-lunch-break.jpg "Lunch break stroll at CPPP")

## Adding a new clang-tidy check by the practice (Live coding) by [Jeremy Demeule](https://twitter.com/jeremydemeule)

After lunch, I started with the presentation I waited for the most. A live-coding session of how to implement custom checks in [clang-tidy](https://clang.llvm.org/extra/clang-tidy/). For those of you who don't know with clang-tidy is sort of a linter for C++. You can use it diagnose typical programming errors, style violations, bugs that can be deduced via static analysis, and better than that it can even fix these issues.

[Jeremy](https://twitter.com/jeremydemeule) took on a very difficult task. Presenting is not easy and live coding sessions are way more difficult and he chose to this format for a right after lunch timeslot. Very courageous. I think he did an extraordinary job in terms of delivering the code without blocks and failures. Though he succeeded by sacrificing a bit his dynamism and the contact with the audience.

I would argue that is easy to add new checks after this live coding session, but I find the idea really useful and the session eye-opener. Thank you, Jeremy, I'll find some time to dig deeper into this topic. I think this tool and idea can be used for [mutation testing](http://sandordargo.com/blog/2018/01/11/mutation-testing) in C++.

## The Anatomy of an Exploit by [Patricia Aas](https://twitter.com/pati_gallardo)

Among the next three sessions, as someone who is in charge of application security in the department, _The Anatomy of an Exploit_ was a must for me. I didn't know [Patricia Aas](https://twitter.com/pati_gallardo), but I quickly understood that she has a good reputation in the industry - well, she is someone who [presented as CppCon](https://www.youtube.com/watch?v=0S0QgQd75Sw) - and she is not only an expert of her subject, but she has all the skills to deliver good presentations.

A dynamic speech, with minimalistic slides, good examples about how the [weird machine](https://en.wikipedia.org/wiki/Weird_machine) works and how you can exploit it. What is the weird machine? It's _a computational artefact where additional code execution can happen outside the original specifications._ The most important point is that if you want to hack an application, you have to consider that the data you have is the program itself and the program you can execute is the data that you feed the program with. It's a big difference in the mindset that you have to identify with if you are interested in exploiting vulnerabilities.

Often I felt that even though I was trying to follow her, I felt lost. And at the end of the presentation, I realized that I must not have been alone. In fact, we, the audience failed Patricia. In my opinion, if there is no question at all at the end of the talk, rarely it means that it was way too clear - not the case here as it was a difficult topic -, sometime it means that it was too boring and people stopped listening to a long time ago - definitely not the case here -, or it can also mean that the people are completely lost because the topic is too complex for them. Probably this presentation would have fit better the most advanced _push forward_ track.

Anyway, I enjoyed Patricia's talk and made a few notes that I want to follow up on.

## Identifying Monoids: Exploiting Compositional Structure in Code by [Ben Deane](https://twitter.com/ben_deane)

For the last talk, I was hesitating a lot between _Quickly testing legacy code_ from the middle _Produce_ track and between Ben's in the advanced, _Push forward_ track.

I decided that I go to the latter, mostly because I wanted to attend one talk from that track too.

Ben proved that he deserved to speak in the big auditorium, I found him a fairly good speaker. His topic was interesting and he didn't miss to make it clear in the very beginning that monoids are not the same thing as monads. So what is a monoid? A monoid is an algebraic structure of three parts. 1) a set of values 2) an associative binary operation and 3) an identity element. I don't want to go into details, I will dedicate an own post to this topic. 

He brought a lot of examples of these structures. He said that the human brain can the best learn patterns if it sees a lot of occurrences. Well, we did.

Unfortunately, his point was a little bit unclear to me until the very end. I think as a speaker one of your obligations is to repeat your main messages as often as possible. I think the main point was that if you can identify monoids, you can also encapsulate them and treat them as units of your code, often using [STL algorithms](http://sandordargo.com/blog/2019/01/30/stl-algos-intro) which make programming easier and even runtime execution can be faster. Especially if you can transform one monoid into another. I'll definitely have to dig deeper before I write something utterly stupid.

## Conclusion

I found all the talks interesting and up to a certain level applicable to my job. Emotional code, algorithms and clang-tidy are definitely my cups of cake.

I liked the venue a lot. I mean, you step out and you see the Eiffel Tower and Paris is still wonderful wherever you can see the effects of [Haussmann](https://en.wikipedia.org/wiki/Georges-Eug%C3%A8ne_Haussmann). 

In terms of catering, I especially liked the infused waters and smoothies we could enjoy. The only strange thing was that during the last two breaks there were no drinks offered anymore. Not even water. But it's okay, the organizers also need some room for improvement for the next years! :)

In the past, I found it difficult to talk to strangers, but I work a lot on it and during my last two conferences speaking the new people much better than before. It's a good experience for me and it shouldn't have been very awkward to the those whom I spoke with.

![The Amadeus team at CPPP]({{ site.baseurl }}/assets/img/amadeus-cppp-2019.JPG "The Amadeus team at CPPP")

Overall, the conference great, even awesome, if one considers that this was the first edition only with a couple of months of preparation. Kudos! And thanks for the organization, [Fred Tingaud](https://twitter.com/FredTingaudDev) and [Joël Falcou](https://twitter.com/joel_f)!

One thing I specifically have to mention. The program was on time and almost all the presenters were on time. This is just something extraordinary. I really appreciated it. After all, I already left a team partly because all the meetings were seriously late and I failed to change it.

Next year, as the organizers will have more time, they will open a _Call For Papers_ and I hope I'll submit something that they will find appealing.
