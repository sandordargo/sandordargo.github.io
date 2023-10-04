---
layout: post
title: "Trip report: Dev Talks Cluj 2023"
date: 2023-10-4
category: dev
tags: [cpp, cpp20, cleancode, integercomparison]
excerpt_separator: <!--more-->
---
The last week of September I had the honour to share my thoughts in the heart of Transylvania about clean code and software quality at [DevTalks Cluj](https://www.devtalks.ro/cluj). Over the last few years, DevTalks became a successful series, this was the 6th DevTalks conference in Cluj. I'm grateful to the organizers for having me and for everyone to turn the event into such a welcoming place.

I must mention that at a certain level, this was a personal milestone for me. A few months earlier I went to [C++ On Sea](https://www.sandordargo.com/blog/2023/07/05/trip-report-cpp-on-sea-2023) and told my management that I wouldn't submit any more papers this year as I didn't want to exploit their generosity with the time they granted me to travel and attend the conference.

So what did I do at DevTalks then?

Let me delve into the personal milestone part. This was the first time that I didn't submit a paper at a conference, but the organizers reached out to invite me.

I know that it's not only about me, the fact that I work for Spotify, a company that has an incredibly strong brand, helped a lot. Yet, I'm both happy and honoured.

## The conference in the Silicon Valley of Romania

Cluj (or Kolozsvár in Hungarian and Klausenburg in German) is a bulging student city with a growing IT sector. It has 9 universities with the biggest in the country among them, the multilingual Babes-Bolyai University with almost 50,000 students. I talked to several students from there who shared that the city has an official population of about 290,000 people, but during the school years, the real population is roughly double. You can feel it when you walk in the streets, there are so many smart-looking young people everywhere. As one of the taxi drivers who is also experimenting with programming, Cluj has become the Silicon Valley of Romania.

The conference was held at the Cluj Innovation Park which is located in the hills surrounding the city. There were more than 2000 attendees following the two tracks at the *main* and *programming* stages and also in the expo area discussing with the numerous sponsors.

I usually don't write about the sponsors of a conference. I know that without them a conference wouldn't be possible and I'm grateful for their support, but I think they get their fair share of appearance and I don't want to be unfair to mention only specific ones. This time though I make an exception. [Adapta Robotics](https://www.adaptarobotics.com/solution/testing-based-on-visual-elements/) impressed many of us.

They brought a robot that is *"designed to reproduce, as accurately as possible, human interaction with a device, leading to effectiveness in recognizing any on-screen elements, button pressing independent of their position on the device, and the executions of more complex gestures such as pinch, swipe, rotate"*. Indeed, we were watching the robot playing a game on a phone, while the representatives of the company explained to us how to program the robot.

![Adapta's Robot playing a game]({{ site.baseurl }}/assets/img/adapta-devtalks-cluj-2023.jpeg)
_Adapta's Robot playing a game_

## Three points to highlight about the conference

Let me highlight a couple of thoughts that stuck with me.

### The talented people

Let me not start with any talk. Quality presentations are essential, but the attendees are more important. There were more than 2,000 of them, they were welcoming and curious.

Conferences as a speaker can serve two purposes apart from sharing your knowledge. One option is to attend all the talks and learn as much as possible and then share what you learned with your colleagues. The other option is to take part in as many conversations as possible. This second option lets you grow your network, learn from what others have to say and also talk with them and give back as much as possible.

As I mentioned earlier, Cluj is a vibrating student city and I also had the feeling at the conference that students gave the majority of the participants. I might be wrong about that though, I consider even myself young whereas I'm closer and closer to 40. After attending some talks, I decided to stay outside for some time and discuss with people, learning about them, what they are interested in and sharing some insights about how my career grew and what they should focus on if they want to be successful at their jobs. I met some extremely talented and determined people.

### Software Engineering at Google: What's the Secret?

One talk that I found particularly important was presented by Ignacio Blanco, Senior Staff Engineer at Google. It was a high-level overview of roles and responsibilities at Google, but I think they would apply to most of the BigTech companies. This talk was mostly interesting for junior developers, yet I must mention it because I think the quality of the talk deserves it and this is such an important topic that is often so difficult to learn about at your workplace.

![Ignacio Blanco speaking about the secret of Google (Photo by DevTalks Romania)]({{ site.baseurl }}/assets/img/ignacio-blanco-devtalks-cluj-2023.jpg)
_Ignacio Blanco speaking about the secret of Google (Photo by DevTalks Romania_

I remember how much I appreciated it when about 5 years ago one of my previous senior managers showed me a public to employees, yet well-hidden internal document with the exact roles and responsibilities and made me understand how I can get ahead with my career.

Ignacio also shared some thoughts about shift-left and the importance of early testing. One cannot emphasize how much money and time you can save for your company if you manage to catch some bugs at the earliest possible. The earlier a developer learns about the responsibility of writing tests, the better it is for everyone.

### The future of software development by Gary Crawford

I had the chance to share a ride from the hotel to Cluj Innovation Park with Gary. His was one of the talks I waited for the most. I was a bit afraid though that it would be about how much we'll use LLM-fueled AI bots to write code.

He reassured me already in the cab that his talk was not about that, he'd only briefly mention it, though AI would still be an essential element.

What I didn't know though is that Gary would serve more the Spotify brand than I did as a Spotify presenter. Certain people even asked me if we were colleagues, both of us working for Spotify. No, but our product clearly inspired Gary's presentation.

![Gary Crawford speaking about the future of software development]({{ site.baseurl }}/assets/img/gary-crawford-devtalks-cluj-2023.jpg)
_Gary Crawford speaking about the future of software development_

So the future of of software development is about providing users a more personalized experience. That's something that Spotify has been clearly spot on for a long time. Even before I would have been even just considering joining the band, I appreciated how much Spotify helped me broaden the range of music I listen to while still paying attention to the genres I like. With the expanding AI usage and the introduction of AI DJ, we clearly moved to the next level which is apparently highly appreciated by our users.

If you approach the question of AI from another angle, this also means moving towards run-time intelligence from build-time. Even though one might argue that it's just a matter of perspective. Most developers nowadays use AI to write production code or at least scripts faster and automate the automatable. Either through asking ChatGPT to write code for a more-or-less well-described problem or just by using an AI-fueled coding tool predicting what you'd write. This is compile-time AI usage (even though from the tools' perspective it was already runtime). According to Gary, the future is in using AI at runtime (from a user perspective).

## So what about clean code?

Finally, let me share a couple of thoughts about my talk which was the last one on the programming stage. I never talked in front of so many people, but luckily it didn't give me stage fright. Though the first minutes I was quite a bit stressed due to some technical issues, I managed to overcome them also with the quick help of the organizers and then things went on smoothly.

![Me ending my talk about clean code]({{ site.baseurl }}/assets/img/me-at-devtalks-cluj-2023.jpeg)
_Me ending my talk about clean code_

This was not the first time, I gave a talk with this title, so obviously I know my talk each time better, but I always keep modifying it a bit based on my deepened understanding of the topic and the feedback I got. For this time, I made sure even more clearly that the talk was not about Uncle Bob's clean code, but about "code that is easy to read and easy to understand". After explaining the different meanings of quality and in particular software quality, I highlighted the importance of not only doing the right thing but also doing things right. According to some studies, that's even more important than doing the right thing.

![The Alignment Trap by Allan Kelly]({{ site.baseurl }}/assets/img/doing-the-thing-right-or-doing-the-right-thing.png)
_The Alignment Trap by Allan Kelly_

These parts required very little modification. On the other hand, I slightly revamped and simplified the part in which I explain how I see the tasks of a developer, what exactly I mean by communication which is probably the most important task of a senior engineer, even if we ignore communication through code. I made sure that my end message about our roles and responsibilities of passing on the right values is simple and clear.

More than ever people connected to me after the talk which felt great, even though it's pure math. There were probably almost a thousand people in the room!

One of the outcomes was that right on stage, I realized what should be the next topic I'd talk about! So be ready for a proposal about clean code and performance next year.

If you are interested you [get the slides here]({{ site.baseurl }}/assets/presentations/WhyCleanCodeIsNotTheNorm-DevTalksCluj2023.pdf).

## Conclusion

The 6th edition of DevTalks Cluj was a delight to me. Not only because I was invited there but because I feel that this was the conference that I would have wanted to attend at the beginning of my career. Such presentations would have pushed me in the right direction early on, earlier than it happened to me.

There were tons of presentations aiming to give industry knowledge to less experienced developers that are usually difficult to get at school. But don't get me wrong, the conference had something relevant for developers at any stage of their careers. I'm probably overwhelmed by certain aspects because I missed such events so much.

It was also the biggest conference in terms of attendance I've ever been part of. It was nice to see that there were so many young people keen to learn and I wish them the best in leading the growth of the IT Space in Cluj and in other cities.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!