---
layout: post
title: "I went to RivieraDEV 2018"
date: 2018-6-1
category: dev
tags: [rivieradev, conference, speaker]
excerpt_separator: <!--more-->
---
_[RivieraDEV](http://rivieradev.fr/) is a conference organized by developers and for developers._ That's how the organizers advertised the event and you can actually feel it. Especially when you consider that some of the organizers are speakers too. RivieraDEV will be 10 years old next year, for me this was the second one that I attended.

<!--more-->

Last year I applied for one of the free tickets, my employer, Amadeus offers for their employees. Just like I did this year. Luckily, this time I had to give back the ticket I got from Amadeus as I have been accepted by RivieraDEV as a [speaker](http://rivieradev.fr/orateur/363) at the event.

I submitted a few propositions and finally, I was accepted for the "We are not just devs" track to speak about [how we organize our holidays](https://docs.google.com/presentation/d/1U6Vu-SR4tT1aaMe2eLYb3PxDAMhQPoQat7kiPHekArk/edit#slide=id.p). It was a super experience for me. I had to give a 30 seconds speech in front of a few hundred people trying to convince them to attend my session later at the day and then I had my 50 minutes workshop.

You might think that it's easy to go on stage and try to persuade the audience in 30 seconds. After gaining some experience, I'm sure it is easier. But I think if you take it seriously, it needs a lot of thinking and practising. I was happy with my "act". I kept calm and fitted in the 30 seconds. I said exactly the same things I wanted to say. Truth is that I practised it for more than an hour while walking home from the speakers' dinner took place the previous night and even that morning driving to the conference.

[My presentation](https://docs.google.com/presentation/d/1U6Vu-SR4tT1aaMe2eLYb3PxDAMhQPoQat7kiPHekArk/edit#slide=id.p) went quite well. Even though I ran a bit out of time and I shared some thoughts earlier than I originally wanted, I could pay attention to many important aspects of a presentation. I kept calm. I didn't speak fast. I made a lot of eye contact and I asked some questions. I didn't dance on the stage, but I kept changing positions. Those few months of being part of the [B.E.A.T. Club](https://www.toastmasters.org/Find-a-Club/02137369-02137369) before I left Budapest taught me a lot. Still, practice is needed, my presentation skills are a bit rusty.

![Truffles at RivieraDEV]({{ site.baseurl }}/assets/img/riviera-dev-2018-work-hard-play-hard.jpg)

All-in-all, I'm extremely happy that I had this chance and I'm looking for some more feedback. So if you were there, please share your thoughts whether in comments or in private messages. Thank you.

First time this year, there was a deep-dive day prepended to the event. There were 7 tracks with 7 conferences/hands-on sesssions in the morning and 7 in the afternoon. The following four days there were about 70 sessions in 4 different tracks.

I will not summarize all of them and I will not write at all about the talks I didn't like. But I'll highlight - quite - a few that I particularly liked. I'll also translate the original titles if they were in French.

## Your first microservice in Go

[The presenter](https://twitter.com/sebastienfriess) from [SFEIR](https://www.sfeir.com/) was well prepared. He gave us the slides, the [GitHub repo](https://github.com/Sfeir/handsongo) with the examples, plus all the environment on pen drives. The fundamentals of [Golang](https://golang.org/) was quite briefly explained. Probably this is just enough for already polyglot developers like me. "Our first micro-service" was maybe a little bit of exaggeration, as most of the code was already written and in each class, we only had a few `ToDos` to complete in order to make everything work. I still think this was a good idea. Instead of making this hands-on a typing exercise, we could spend some time on discussing some above-junior level characteristics of Go.

## Come and explore the GRAND stack: GraphQL, React, Apollo and Neo4j

I couldn't attend this one but got the [slides](http://sim51.github.io/presentations/grand-workshop/) from the presenter! As I am really interested in [Neo4j](https://neo4j.com/) and I am preparing to become a [certified Neo4j developer](https://neo4j.com/graphacademy/neo4j-certification/), I definitely wanted to get at least the slides.

## The surprise of the chef

[Quentin Adam](https://twitter.com/waxzce), the CEO of [CleverCloud](https://www.clever-cloud.com/) in his keynote was speaking about how we should transform technical debt into technical investment. He claims that there is no technical debt, but you only make technology investments with different levels of return. It's definitely an interesting vision in my opinion, but I couldn't fully agree with him.

Indeed, in some situations, we can handle tackling debt or letting it in as an investment. I already wrote about this in [my article about the Design Stamina hypothesis](/blog/2018/03/06/design-stamina-hipothesis). Laying down the architecture, thinking about how your application will evolve can be handled as an investment, I agree. But I see so much crap packaged as "trade-off between quality and speed" that nobody should have ever done after 1 or 2 years of professional development that I'm convinced that there is a lot of debt out there which is nothing but debt and the results of ignorance, lack of knowledge and smart processes.

## A chatbot in 5 steps with DialogFlow

This was an interesting presentation about how to design some basic chatbots without basically any programming experience using an online tool, called [DialogFlow](https://dialogflow.com/). No magic involved. The discussed topics were relatively basic, we didn't discuss context management for example, but for an introduction to chatbots, it was useful.

## Discover the cuisine around the truffle

As I said, there was a track called _We are not just devs_. The organizers managed to get a chef, [Sara Tatouille](http://www.sarahtatouille.com/), and a truffle master, [Adam Edicoffer](http://www.truffes-domainedargens.fr/), to make us a small presentation and they even cooked for us some really simple but delicious tiny courses with the truffle. 

![Truffles at RivieraDEV]({{ site.baseurl }}/assets/img/riviera-dev-2018-truffle-1.jpg)

## Lock Picking

Florian Loitsch is a regular presenter at RivieraDev and each year his workshop on lock picking is one of the most popular ones. Just to be clear, we are speaking about physical locks. He brought quite a few and the necessary tools to try opening them.

In the row, I was sitting (4 people), I was the only one who managed to open one, but it was just a matter of luck because I couldn't reproduce it. So it's not so easy, but according to Florian and a colleague of mine, it'd take just a little bit of practice. I'm not so happy about this. It means that locks are not worth much. With a specific tool, he also managed to open a Kensington key in just a few seconds...

It was also interesting to hear that in certain countries it's illegal to carry these kinds of tools even if you don't have the intention of breaking into someone's place. Every hobbyist lock picker should keep that in mind!


## Teaching Programming To Children Using Minecraft On Openshift/Kkubernetes

This was an entertaining, yet useful presentation about how to develop custom plugins with ScratchX for Minecraft. These guys really tried to remove all the burdens of CLI "magic" to make programming entertaining for kids. [Here you can watch the presentation](www.youtube.com/watch?v=tKZr3OpnSZk) and you can find even more information at [https://www.learn.study/](https://www.learn.study/).

## Flutter - Mobile Apps, The Easy Way

[Flutter is a new SDK from Google]() to build native iOS and Android apps using the dart language. This talk was informative and shared a lot of examples through live coding. It really made me feel like trying this new SDK to build some apps.

In fact, I've already started. Keep tuned.

I have to emphasize that it was not just interesting but the presenter made a good job. He spoke well, he was clear and he even tackled with ease the difficult task of coding while speaking. Well done!


## Fire & Fury: Java Flight Recorder & Flamegraphs

[Flamegrapher](https://github.com/flamegrapher/flamegrapher) is an open-source tool that allows you to generate flamegraphs out of the [Java Flight Recorder](http://openjdk.java.net/jeps/328) recordings for methods on CPU, locks, exceptions and allocations.

[Here you can get the presentation](https://www.slideshare.net/secret/mpuAbsPAD7SM2D) of two very talented colleagues of my mine, Leonardo Freitas Gomes and Nened Bogojevic.

## Scalable Cloud Ide With Eclipse Che And Openshift

I was a bit hesitant about including this presentation in my summary. I think the presenter didn't have enough time upfront to master the tool he was presenting, but [Eclipse Che](https://www.eclipse.org/che/) seems to have some potential so that at least I wanted to mention it. [Eclipse Che](https://www.eclipse.org/che/) is an online IDE that runs in your browser and you can easily work on a Github repo that is linked with an online Openshift instance. It might give you a very convenient way of working on your cloud application.

I also think it's important to mention that Che refers to the [Ukranian city of Cherkasy](https://en.wikipedia.org/wiki/Cherkasy) where most of the development is done.

## How To Start Learning An Instrument By Yourself

This was again a workshop that you might not expect at a developer conference. [David Auk](https://www.linkedin.com/in/david-auk-41aa9422/) explained us some of the basics of music theory and how he started to learn at least 3 different instruments during a year. He didn't just present but was keen to play a bit for us to show where he is and let us to try his electric guitars, bass guitar and violin.

If you have ever learnt music, you know that it's practically impossible to become a master in so short of time without prior experience, but David showed us that if you are willing to dedicate some time on a daily basis to learn to play an instrument, you can already actually learn enough that is enough to entertain both yourself and some others at small parties or at campfires.

Thank you, David! Your presentation was really inspiring!

## Conclusion

I'm really grateful to both the organizers and to my employer that I could attend and speak at RivieraDEV. I hope it will not be the last time.

It was a great experience and I learnt about a couple of frameworks and concepts that I would not have heard about or I just simply would not have tried them. And truffles of course!
