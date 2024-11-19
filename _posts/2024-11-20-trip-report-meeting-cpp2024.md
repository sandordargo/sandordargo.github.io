---
layout: post
title: "Trip report: Meeting C++ 2024"
date: 2024-11-20
category: dev
tags: [tripreport, meetingcpp, cpp, watercooler]
excerpt_separator: <!--more-->
---
Last week, I got the chance to go to Berlin and participate in the [10th edition of Meeting C++](https://meetingcpp.com/2024/) which is as far as I know the biggest C++ conference in Europe. Considering both the online and onsite participants, there were about 500 of us, eager to learn more mostly but not only about C++.

The conference was held in the East side of Berlin, in the Vienna House by Wyndham Andel's Berlin Hotel offering some cool views of the city from its higher floors, especially from its sky bar. Not that we had a lot of time wandering in Berlin, the schedule was quite packed. Although the program only started around 9-10 AM depending on the day, the first two days the official program didn't end before 10 PM.

Given the bit cold and cloudy weather, I realized that Berlin in November is really perfect for a conference. People will not be tempted to escape the venue for long walks outside. It just feels better to be inside.

> Despite what I just wrote, the late autumn weather didn't block me from exploring Berlin when I had some time. On Friday, early morning, I could even visit the Spotify office, which is just above the Bud Spencer and Terrence Hill museum, big favourites of us Hungarians.
>
> On Sunday I had plenty of time and I visited more than I had hoped for! In Berlin, there are certain things we cannot forget. Actually, there are many things. Berlin is the mememto of totalitarian systems. Fascism and the Holocaust were mentioned by Peter Sommerlad in his keynote, but especially in Berlin, we shall not stop there. Tränenpalast (Palace of Tears) or the Museum at the Kulturbrauerei are museums where you can learn and remember what happened during the Cold War to the people living on the East side of the Iron Wall, particularly in Berlin.
>
> ![Heavy Metal in the DDR]({{ site.baseurl }}/assets/img/heavy-metal-ddr-berlin.jpg "Heavy Metal in the DDR")
> _Heavy Metal in the DDR_
>
> Besides the tears, I found some riffs as well. In Tränenpalast, I found a leaflet about the *Heavy Metal in the GDR* exhibition where you can discover some of the great heavy metal that oppressed musicians made in East Germany. [Here is a playlist about some great DDR Metal](https://open.spotify.com/playlist/2Veo36v4hbYJwuhsyye3ds?si=8c6365537e3243b2).
> 
> Let's get back to the conference.

As I usually do in my trip reports, I'll highlight my 3 favourite talks and 3 ideas before sharing some highlights of my talk.

## My three favourite talks

Meeting C++ featured more than 40 talks in 4 different tracks. It was a hard choice to decide which 3 talks to list here because there were many great ones. But this is my final selection in chronological order.

### Software Engineering Completeness: Knowing when You are Done and Why it Matters? by Peter Mouldon

Peter Mouldon is a presenter who I wanted to listen to in person for a long time. In addition, I also liked [the description of his talk](https://meetingcpp.com/2024/Talks/items/Software_Engineering_Completeness___Knowing_when_you_are_Done_and_Why_it_matters__.html). While it's good to learn about brand new or even future features, I also like when a talk is featuring some practical software engineering. Even if it has nothing to do directly with C++.

Peter's talk was like that. No C++, but full of practical advice.

*"Are you done yet?"* - a question we might hear so often from our bosses. But how can we know? Moreover, how can we know that once we are done, the software brings actual value?

Software only brings value when it's available, usable and reliable at the same time. Therefore huge and slow changes are not as useful as changes that are broken down into small pieces that have these three characteristics on their own.

Also, the smaller a change is, the easier it is to estimate it. And while estimation is often something we dislike, it's important for planning purposes and it will also become easier for us to know when we are done.

We don't only have to be done with a feature, but we also have to be prepared to maintain it, change it if needed, and support it over time. The different levels of done are defined in Peter's Software Engineering Completeness pyramid which he didn't fully cover in this 60 minutes, but hopefully, he'll be able to cover the rest next year.

### [The Beman project](https://www.bemanproject.org/mission) by David Sankel

Until this talk, I only saw David Sankel on recordings and his vivid style fascinated me. This time, I could finally listen to him in person. He was talking about the Beman project which I never heard of before. [Beman Dawes](https://www.bemanproject.org/2024/10/30/about-beman/) was one of the founders of the Boost project. Boost has often been considered the hallway to the standard library, but it also grew in many directions.

![The Beman project logo]({{ site.baseurl }}/assets/img/beman.png "The Beman project logo")
_The Beman project logo_

[The Beman project](https://www.bemanproject.org/mission) in some sense is about returning to the roots. It hosts implementations of proposals to the standard library. The main goal is to collect both implementation and user feedback before something gets standardized.

I also learned that not only highly experienced library authors are welcome to help out with the implementations, but anyone who wants to work on it and get better in C++. Contributors might even get some mentoring so they don't get stuck.

Sounds too good to be true? [Check it out!](https://www.bemanproject.org/mission)

### Peering forward - C++'s next decade by Herb Sutter

Herb's talks are always so energetic and positive. It's difficult not to love them. In this talk, he discussed the path ahead of C++. Important changes are being introduced to remove safety-related undefined behaviour, and if all goes well, finally we're going to get zero runtime-cost reflection which has the potential to significantly change how we code in C++.

It's important to mention and accept that not all the changes will happen overnight. Just like the capabilities of `constexpr` have been incrementally introduced, not everything that are related to reflections will be introduced with C++26. Some will only come in later standards.

> Did you think about how `constexpr` and compile-time programming are about safety as well? After all, [there is no undefined behaviour at compile-time](https://andreasfertig.blog/2023/06/constexpr-functions-optimization-vs-guarantee/).

It's interesting to see how performance-first options are replaced by safety-first choices in the world of C++. Priorities changed and that's probably alright, as long as we have the option to not pay for what don't use - which is the case!

An exciting thing I learned is the possibility to compile with `-ftrivial-auto-var-init=pattern` which automatically initializes all uninitialized local variables. Soon, it will be the default behaviour and be part of the standard.

Exciting times to come.

## My three favourite ideas

Now let me talk about 3 lovely ideas I learned about during the conference.

### Use concepts to replace CRTP (Klaus Iglberger)

Klaus gave two talks on the same day as he volunteered to provide a talk ([Design Patterns - The Most Common Misconceptions (2 of N)](https://meetingcpp.com/2024/Talks/items/Design_Patterns___The_Most_Common_Misconceptions__2_of_N_.html)) covering for someone else who couldn't make it. And I have to tell you, I liked his talk a lot. Especially its first part, where he was discussing the Curiously Returning Template Pattern (CRTP) and whether it's dead thanks to [deducing this](https://www.sandordargo.com/blog/2022/02/16/deducing-this-cpp23).

While you might think so, it's not so evident.

According to Klaus, we could even differentiate between two usages of CRTP and call them different patterns. One would be *static interface* and the other would be *mixin*. While the former creates a static family of types, the latter brings additional functionality from base classes without relying on runtime polymorphism.

In the case of static interfaces, we could try to replace the base class template with the mixture of class tagging and concepts.

```cpp
// CRTP version
#include <concepts>
#include <iostream>

template<typename Derived>
class Animal {
public:
    void make_sound() const {
        const Derived& underlying = static_cast<const Derived&>(*this);
        underlying.make_sound();
     }

};

class Sheep : public Animal<Sheep> {
    public:
    void make_sound() const { std::cout << "baa\n"; }
};

template<typename Derived>
void print(Animal<Derived> const& animal) {
    animal.make_sound();
}

int main() {
    Sheep sheep;
    print(sheep);
}

// non CRTP version

#include <concepts>
#include <iostream>

class AnimalTag {};

template<typename T>
concept Animal = 
    requires(T animal) { animal.make_sound();} &&
    std::derived_from<T, AnimalTag>;

template<Animal T>
void print(T const& animal) {
    animal.make_sound();
}

class Sheep : public AnimalTag {
    public:
    void make_sound() const { std::cout << "baa\n"; }
};

int main() {
    Sheep sheep;
    print(sheep);
}
```

In fact, you don't even need C++23, only C++20 for the new version. This thought sparked my interest because I suspect that besides making the core more readable, this might reduce binary size as well.

I'll try that sooner than later and go more into the details of what Kalus shared.

### Non-movable, non-copyable base classes (Sebastian Theophil)

Sebastian Theophil gave a talk about how C++20 and 23 change the way we write different types of classes. I'm not going into the details of the 4 different class types he mentioned (value, container, resource and singleton), I simply want to highlight one specific thing.

We probably want to make resource wrappers move-only (non-copyable) types and singletons non-moveable classes. How should we do that?

The well-known Hinnant table gives us a hint, it tells us what special member functions we should define or delete.

![The Hinnant table]({{ site.baseurl }}/assets/img/hinnant-table.jpg "The Hinnant table")
_The Hinnant table (source: https://howardhinnant.github.io/)_

But how to do this when we need it often?

Sebastian's scalable solution is to create two base classes, one called NonMovable, and one NonCopyable and inherit from those in your classes.

```cpp
class NonCopyable  {
	NonCopyable(NonCopyable&& other) = default;
	NonCopyable& operator=(NonCopyable&& other) = default;
};

class File : public NonCopyable {
public:
	File() = default;
	File(char const* file);

	File(File&& other) noexcept;
	File& operator=(File&& other) noexcept;
	~File();
private:
	FILE* fp = nullptr;
};


```

Even though I'm a bit more pedantic and I'd prefer implementations following the [rule of five](https://en.cppreference.com/w/cpp/language/rule_of_three), I find this very interesting idea which results in code clearly expressing intentions.

In a later article, we might compare it with other approaches.

### Agile was killed (Peter Sommerlad)

There was in point from Peter Sommerlad's closing keynote that really resonated with me. What he said about agile. Back in the time when he worked with the people creating agile, it was all about collaboration, collective code ownership, working together, improving together and figuring out together what works best.

Instead, thanks to the certifications and businesses built around them, it became a money-printing machine where people who often never had any meaningful IT experience want to tell you how to make software.

This killed Agile and so-called Agile-people very often don't follow [the Agile manifesto](https://agilemanifesto.org/) themselves.

## One more talk: The Aging Developer by Kate Gregory

I decided to mention one more talk. The Aging Developer by Kate Gregory. First of all, I loved the talk. I don't think it has much to do with software development, but it's an important talk. Hence I mention its own section. I think it could fit any conference where the audience is living a sedentary lifestyle. 

I only checked the title and the presenter and as I think Kate always gives great talks, I picked hers. I thought that she would speak about how to stay relevant, up-to-date as an aging developer in a world where our colleagues are often so young.

Instead, this talk was about something more important, our health.

She mentioned some aspects I didn't consider before. As the years pass by, you might not be able to drive in the dark and it means you cannot stay in the office late especially during winter in an environment where there is no great public transportation - like where I live. Also, if the toilets are on a different floor or simply far you might soon get too tired of it. Oh, I had that at my previous employer. I thought it was good to get a walk, I didn't think about what Kate said.

But to me, those are not the most important parts of the talk.

We sit and stare at our screens so much. We must stand up, get out and exercise. I personally really needed this talk. Of course, this was not new information, but it was a push that I needed. Now it's up to me to change my life a bit.

Oh and one more important message! If you are a grumpy young person now, don't expect to grow into a nice, generous old person. If you don't do anything, your traits will just become stronger. Change now. 

## My personal experience

As I mentioned this was my first time to attend in person Meeting C++. I also had the honour to give a talk. Even after a couple of times, it's scary to go on stage and talk to people who are often smarter and more experienced. This time it was especially scary. I went to a C++ conference to talk about clean code and performance without a single line of code or performance benchmark on the slides. Moreover, for the first time in my life, I got to present on the main stage!

Luckily, my stage fear went away after a few minutes, after I introduced myself and I think I gave a decent talk about whether clean code implies horrible performance. In brief, you should not worry about the performance implications of clean and readable code, until you prove so.

> ![Me at Meeting C++]({{ site.baseurl }}/assets/img/me-at-meetingcpp2024.jpeg "Me at Meeting C++")
> _Me at Meeting C++_

Clean code is just an optimization for readable code. Optimization is always about compromises. While it's true that clean code is not the fastest of all possible codes, most of us don't need the last bits of performance. We have more important aspects to worry about. Maybe it's binary size, maybe it's readability, maybe it's something else. In my opinion, readability is the most important by default. Many of us work in big corporations, where people come and go.

It's also worth considering that the cost of a CPU cycle keeps going down while developers are getting more expensive. Therefore it's worth optimizing for developer productivity rather than CPU cycles. And code readability, clean code, helps making us more productive.

> ![Growing salaries, decreasing CPU costs]({{ site.baseurl }}/assets/img/cpu-cost-developer-salaries.png "Growing salaries, decreasing CPU costs")
> _Growing salaries, decreasing CPU costs_

Even when you have to optimize, do it wisely. Most probably you won't need such low-level optimization [as the developers working on Quake needed](https://www.youtube.com/watch?v=p8u_k2LIZyo). In most cases, optimizing external requests, such as DB operations and network calls will leave you with a good enough solution.

But that's not all that I have to say about me being at Meeting C++.

It was tiring. In a good way. I'm so grateful for all the love, respect and gratitude I got. More people know my work than I would have ever thought and I'm so happy that people find useful what I write about. Which all started as and still is mostly about documenting what I learn.	

Though the talks were great, as in most cases, the best part of a conference is the hallway discussions. Even as an introvert. I met a lot of friends from earlier conferences and also many new faces with whom we had good exchanges.

I think my most memorable encounter was when three of us were discussing different tax systems and a guy joined us saying that he is interested in taxes! It turned out that he was [Dr. Ivan Čukić](https://cukic.co/), the author of [one of my favourite C++ books](https://www.sandordargo.com/blog/2020/05/28/functional-programming-in-cpp). 

## Conclusion

I absolutely loved the 10th edition of Meeting C++! So many great talks, so many inspiring ideas. Thanks to the 4 tracks, there was a wide range of topics available for the participants. We could learn about the newest features of C++, about best practices from already widely used versions, or even about language-agnostic best practices. I can't wait to try some of what I learned in production code.

Last, but not least, a great thanks to [Jens Weller](https://meetingcpp.com/2024/Speaker/items/Jens_Weller.html), the founder and organizer of Meeting C++ and to all the staff for making Meeting C++ such a great experience! Hope to see you next time! 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j)
