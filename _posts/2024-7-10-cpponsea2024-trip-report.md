---
layout: post
title: "Trip report: C++ On Sea 2024"
date: 2024-7-10
category: books
tags: [cpp, conference, cpponsea, tripreport]
excerpt_separator: <!--more-->
---
Last week, between the 3rd and 5th of July, I had the privilege to attend and present at [C++ on Sea 2024](https://cpponsea.uk/) for the 5th time in a row!

I'm grateful that the organizers accepted me not simply as a speaker, but they granted me a double slot to deliver a half-day workshop about how to reduce the size of a binary. I'm also thankful for my management that they gave me the time to go to Folkestone and share our knowledge on binary size.

Last but not least, a great thanks goes to my wife, who took care of the kids alone this last week.

Let me share with you a few thoughts about the conference.

First, I'm going to write about the three talks that I liked the most during the 3 days, then I'm going to share 3 interesting ideas I heard about and then I'm going to share some personal impressions about the conference.

*I'll update this article with the links to the recordings once they become available.*

## My favourite talks

Last year, I wrote that I was pondering what makes a good talk for me and that I enjoyed more the talks that covered beginner topics in depth. I still feel the same way, but I'm not 100% sure if my selection represents that. There are a couple of presenters who due to their outstanding knowledge and wonderful presenter skills can captivate any audience full of C++ entusiastics.

The talks are ordered by their scheduled time.

### [Understanding The constexpr 2-Step: From Compile Time To Run Time](https://cpponsea.uk/2024/session/understanding-the-constexpr-2-step-from-compile-time-to-run-time) by Jason Turner

The conference had a very strong start. Right after the keynote by Dave Abrahams, 4 incredible speakers were on stage at the same time on the 4 different tracks. Jason Turner, Walter E Brown, Nico Josuttis, Mateusz Pusz...

![C++ On Sea schedule]({{ site.baseurl }}/assets/img/cpponsea2024-strong-start.png)

If a conference could have these people throughout the whole program, it would already be a strong conference. C++ On Sea proposed such a strong line-up that these people could be scheduled at the same time. It didn't make my decision easier, so I chose based on the topic, and I wanted to grow my knowledge on `constexpr` so I stayed in the `main()` room.

Let's talk about the talk.

C++20 brought us `constexpr` `std::vector` and `std::string`. Yet, [this simple piece of code doesn't compile](https://godbolt.org/z/zx1vEGzcr):

```cpp
#include <vector>

int main()
{
  constexpr std::vector<int> data{1,2,3,4,5};
}
```

*constexpr variable `data` must be initialized by a constant expression*...

In Jason's talk, we could learn about why it's that the case and how we can use those `constexpr` constructs. The key to answering that question is to understand that to instantiate objects that - might - allocate dynamic memory, *memory allocated at compile time must be freed at compile time*.

I don't want to go through the whole reasoning and examples of Jason, I simply share those two steps that he referred to in the title. For the rest, watch the recording once it's available:

1) Do all the dynamic storage stuff you want to at compile-time
2) Copy the dynamic storage stuff to static stuff at compile-time (and make sure you free the dynamic thing at compile-time)

While I don't want to give away all of his talk, I do want to share a compiler bug that Jason shared with us. If you use llvm and you want to write a `consteval` function, be prepared that you cannot have static local variables, the compiler removes them. Here is a minimal example taken from [the corresponding Github Issue](https://github.com/llvm/llvm-project/issues/82994):


```cpp
consteval const int* a() {
    static constexpr int b = 3;
    return &b;
}
int c() { return *a(); }
/*
GCC assembly:
c():
        mov     eax, 3
        ret

Clang assembly:
c():                                  # @c()
        xor     eax, eax
        ret
/*
```

Now you are aware. Let's see the other talks.

### [Core and other guidelines. The good, the bad, the... questionable?](https://cpponsea.uk/2024/session/core-and-other-guidelines-the-good-the-bad-the-questionable) by Arne Mertz

Once again, it was a tough call to decide whether I should attend [Peter Muldoon's talk on dependency injection](https://cpponsea.uk/2024/session/dependency-injection-in-cpp-a-practical-guide) or Arne's on guidelines. As with my team, we recently had some discussions on some of the core guidelines, I decided to attend Arne's talk.

![Arne Mertz at C++ On See - photo by C++ On Sea]({{ site.baseurl }}/assets/img/cpponsea2024-arne.jpeg)

Arne worked on a dozen different projects as a consultant during the last 9 years. Based on what he's seen, he shared how he's seen guidelines misused.

In the standard, or in the core guidelines - at least in the titles - every word is important. If you skip one, or a half sentence, it will mean something completely different.

Arne brought some examples of guidelines set in different companies that were most often based on the core guidelines or some other companies' (in)famous policies, but they were misinterpreted. For example, in one place, they had this guideline:

> *"Define or delete all copy, move, and destructor functions"*

Does it remind you of the rule of 5? It should. But they forgot to add that you should only define all these special member functions if you have to define at least one of them...

Another company declared that you should not use exceptions. There was no good rationale behind it, except that Google famously banned it. Google also said that if they started over, they would probably do it differently.

When it comes to adopting guidelines, it's important to put things into context. A guideline that makes sense in a certain context might be even harmful in a different environment.

There is also the question of whether rules are guidelines. The secret is in the name. Guidelines are guidelines. Sometimes you might go against them. At the same time, if you do so, it's better to document why. Otherwise, you'll make people waste too much time to figure that out. Worse, they might even undo a good decision.

Also, keep your style guide short. At the same time, automate as many checks as possible with the right tooling. A subject that I'll mention in later sections of this trip report.

### [There is no Silver Bullet](https://cpponsea.uk/2024/session/there-is-no-silver-bullet) by Klaus Iglberger

Klaus had the task of keeping the audience engaged at the end of Friday afternoon with his closing keynote. I think I'm not alone in saying that he fulfilled his job with his talk, *"There is no silver bullet"*.

Most of us can agree that 13 years after the release of C++11, using the term modern C++ is probably not the best. [Ivan Čukić](https://www.sandordargo.com/blog/2020/05/28/functional-programming-in-cpp) came up with the term progressive C++ that Klaus likes too. We'll see if that term sticks.

Klaus used an anonymous comment on one of his earlier talks to give a structure for this presentation. According to the commenter, *"object-oriented programming and especially its theory is overestimated. ... C++ always had templates, and now also has std::variant, which makes most of the use of inheritance unnecessary."* Heck, even [Jon Kalb said at CppCon 2019](https://www.youtube.com/watch?v=x1xkb7Cfo6w) that *"object-oriented programming is not what the cool kids are doing in C++. They are doing things at compile time, functional programming, ... Object-oriented programming, this is so 90s"*.

So Klaus went on with the good old shapes example and implemented it in various ways including the old school OO way, with variants, with templates and compared them.

He indeed managed to get a nice speedup, but it's not all black and white. While it's easy to add new operations to the variant solution, it's relatively difficult to add new types. With the OO solution, it's the other way around. Moreover, due to the reversed dependencies, the `std::variant` solution is an architectural disaster.

It doesn't mean that it doesn't have a place and that it cannot be used in certain situations. It simply means that different solutions have different pros and cons.

These solutions can even be combined into a value-based object-oriented solution, which is still not a silver bullet but can be well used in high levels of architecture.

Learn about it by watching the recording once it's released!

## My favourite ideas

Now let me share three interesting ideas from four different talks.

### *"We all write bad code sometimes"* - [Jan Baart](https://cpponsea.uk/2024/session/who-needs-unit-tests-anyway-modernizing-legacy-code-with-0pc-code-coverage)

Jan Baart gave a very useful talk about code modernization and unit testing. Talks like this are useful for several kinds of audiences. For junior developers, they gave actionable tasks and for seniors a reminder. A reminder of what is important and what message we have to share and distribute.

While modernizing and adding unit tests to legacy code is important, the most important message of Jan was about humility.

> *"No blaming, we all write bad code sometimes."*

We don't have to condemn others because a piece of code is bad. Probably they did their best writing it. In the past, I wrote much worse code than today and hopefully in the future, I'll write better than nowadays. It's probably true for you as well. Besides, we can simply have worse days.

Don't blame others for bad code. Help them grow.

(*Let's leave aside the question of someone not even trying to do a good job. That's a different problem to be dealt by management.*)

### You should write tests by [Robert Leahy](https://cpponsea.uk/2024/session/fantastic-bugs-and-how-to-test-them)

In my opinion, Robert Leahy was among the best presenters at C++ On Sea 2024. His points were clear and he was exceptionally energetic on stage.

His point was clear. We should write tests. Even for components or bugs that seem to be too small, simple or trivial to be tested. He brought several examples from his code to support his points. Indeed a one-liner function calling `std::min` can have a bug in it, so it's worth adding tests.

Even though I would argue with Robert whether some of his tests are actually unit tests, there is nothing to argue with his main message. Write tests, not only because of delivered wisdom, but because it levers up your output, and improves your design and code.

### Write your own `clang-tidy` checks by [Mike Crow](https://cpponsea.uk/2024/session/building-on-clang-tidy-to-move-from-printf-style-to-stdprint-style-logging-and-beyond) and [Svyatoslav Feldsherov](https://cpponsea.uk/2024/session/pets-cattle-and-automatic-operations-with-code)

There were two talks about writing your own `clang-tidy` checks or even refactoring actions. One by Mike Corw and another by Svyatoslav Feldsherov. I liked the message of both of these talks.

`clang-tidy` and the Abstract Syntax Tree (AST) looks sometimes a bit too... abstract.. to most of us. And if we look at some sample code we'd have to write, it doesn't make things any better. These talks bring these tasks a little bit closer to everyone and they clearly tell us that even though it's not the simplest thing, it's not rocket science either.

Until the recordings are available, it's worth looking into [AST Matcher Reference](https://clang.llvm.org/docs/LibASTMatchersReference.html).

## Personal impressions

Finally, let me talk about some more personal feelings about the conference.

### Starts to feel like home

The first two times, I attended C++ On Sea online, but this was already the third occasion to come in person. It starts to feel a bit like home. I don't need a map for basically anything, I know where to find what at the conference or at the town. I have my favourite places.

More importantly, with more and more people, we greet each other with a great smile. Organizers, speakers and attendees included. This feels right and just makes me even more appreciate the social aspects of a conference. I meet some of these people more than my colleagues.

It also made me realize that so far C++ On Sea is the only C++ conference where I gave a talk in person. This might change in a few months though.

### New ideas keep coming

It's nice to see that organizers pay close attention to details. If something doesn't work as expected one day, they improve it for the next day. And the venue staff members are also partners in this.

There were two novelties this year that I want to mention.

Wednesday evening there was a movie night hosted by Walter E Brown with some shorter or longer clips about the history of computer sciences. The organizers also provided pizza for everyone who wanted to stay around so that we couldn't claim that we needed to find dinner somewhere else. Sadly, I couldn't stay until the end as I had my sessions right the next morning. Nevertheless, I liked what I saw.

There was a fun buzzword bingo for some C++ books. The idea was that we had C++ buzzwords on a paper such as `const_cast`, `ADL`, `void`, etc. just to name a few. At each session, we could tick two of them that we heard and the first few people who got 5 words in a single row or column could get a book. Although I didn't intend to take the book as I already have it, I liked the idea and played along.

### My talks

On the second morning, I had two consecutive slots to deliver a half-day workshop about how to keep your binaries small. During the pandemic, I already delivered a half-day workshop online, but obviously, that was a vastly different experience. 

Last year, I left my clicker at home. This year I had it, but for some strange reason, it stopped working properly. So in the end, I was very static on stage, as I had to stay close to my laptop and its space button. Apart from that, it went quite well.

![Me at C++ On See - photo by C++ On Sea]({{ site.baseurl }}/assets/img/cpponsea2024-sandor.jpeg)

The first part of the first session was about binary formats. I was afraid that it would be too boring for most, but as it turned out many appreciated it and the great majority of people turned up for the second session as well.

Overall, I received good feedback from some attendees and also some follow-up questions, such as how dynamic linkage affects binary sizes.

I had a second commitment as well. I am someone who feels obligated to share a lightning talk if there is a possibility. I feel obligated to go on stage and practice whenever there is an opportunity. So did I, to speak about [whether engineering teams really resemble sports teams](https://www.sandordargo.com/blog/2024/05/22/are-we-a-sports-team). I ran a few seconds over time, but I could still finish what I wanted to share.

## Conclusion

C++ On Sea was a great experience in 2024 as well as [during the previous years](https://www.sandordargo.com/tags/cpponsea/). Three days packed with great presentations about various topics, including performance, tooling, design and best practices.

The best we can do is to spread the word so that maybe even more people join next year and also to just share what we learned.

I hope to be back in Folkestone in 2025.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
