---
layout: post
title: "Trip report: C++ On Sea 2023"
date: 2023-7-5
category: books
tags: [cpp, conference, cpponsea, tripreport]
excerpt_separator: <!--more-->
---
Last week, between 27th and 30th June, I had the privilege to attend and present at [C++ on Sea 2023](https://cpponsea.uk/) for the 4th time in a row!

Despite having been accepted as a speaker, I was not sure if I would make it this year, as I changed jobs recently, but my management at Spotify was encouraging and supportive. They granted me the time so that I didn't have to use my vacation days. Also, I am grateful for my family, in particular for my wife, for taking care of the kids alone for the week.

Let me share with you a few thoughts about the conference.

First, I'm going to write about the three talks that I liked the most during the 3 days, then I'm going to share 3 interesting ideas I heard about and then I'll share some personal impressions about the conference.

*I'll update this article with the links to the recordings once they will become available.*

## My favourite talks

Over these few days, I was pondering a lot over what makes a talk good and enjoyable. What makes a presenter good, at least for me? While my thoughts are not crystal clear yet, I definitely enjoyed talks that covered "beginner" topics in depth. Another feeling I have is that good presenters limit the amount of knowledge they want to share so they have enough time to explain and they don't talk with the speed of Eminem.

### [Special member functions in C++ by Kris van Rens](https://cpponsea.uk/2023/sessions/special-member-functions-in-cpp.html) 

Kris' talk about special member functions in C++ is a good reminder of how difficult it can be to write a simple class in C++. Especially if you cannot follow the rule of 0. But do you know about the rule of 0? Or the rule of 5? Or the rule of four and a half? 🤯

First I was not sure if I want to mention this talk among my favourite ones. But when I already listed 2-3 favourite ideas from this talk, I realized that this in fact was one of my best picks.

Let's see those ideas.

Have you heard about the [Hinnant table](https://howardhinnant.github.io/classdecl.html)? The one below shows when you can or can not rely on the compiler to generate the special functions for you.

  
![The Hinnant table]({{ site.baseurl }}/assets/img/hinnant-table.jpg "The Hinnant table")
_The Hinnant table (source: https://howardhinnant.github.io/)_

Kris shared how you can memorize it easily. While the table has 42-48 fields (depending on whether you count the diagonal), you only need three rules in order to memorize it.

- When the user declares any other constructor then the default constructor is not declared
- When the user declares any copy or move operation or the destructor, then the move operations are not declared.
- When the user declares any move operation then the copy operations are deleted

Another idea I really appreciated was that we should test special functions. You might think that testing those are cumbersome. But not so! You don't necessarily want to test the internals of a copy constructor. You don't necessarily have to test if all the members are copied promptly. Maybe you want, but you don't have to go as far.

It's already a great step if you can ensure with the help of type traits (or [concepts]()) and `static_cast` that a given class satisfies certain characteristics.

```cpp
class X{};

static_assert(std::is_trivially_destructible<X>{});
static_assert(std::is_trivially_default_constructible<X>{});
static_assert(std::is_trivially_copy_constructible<X>{});
static_assert(std::is_trivially_copy_assignable<X>{});
static_assert(std::is_trivially_move_constructible<X>{});
static_assert(std::is_trivially_move_assignable<X>{});
```

Then even if you modify the class, you make sure that you don't lose its copyablity. Such tests might even enhance your understanding of [how certain types of members influence a class]().

While I think these also serve documentational reasons and they would look great in the header file along with the class declaration, probably it's wiser to put them along with the unit tests so that you don't make the compilation of the production code any longer.

One last thought! An explicit `=delete` is better than relying on that others know the Hinnant table just as well as you.

### [Typical C++, but why? by Björn Fahller](https://cpponsea.uk/2023/sessions/typical-cpp-but-why.html) 

Björn Fahller spoke at C++ On Sea 3 times this year! He volunteered to replace one of the speakers who sadly couldn't make it to the conference and he also did a lighting talk.

One of my favourite talks was his presentation about how to use C++'s type system effectively.

No, not because of the great images of jigsaw montages.

No, not because at the end he mentioned [my talk from last year as a valuable reference](https://www.youtube.com/watch?v=0cTOqwrvq94). But to be fair it really touched me. Especially that it was not because I was in the room, it was already mentioned on the references slide.

As [I also covered here](https://www.sandordargo.com/blog/2022/04/06/use-strong-types-instead-booleans) using several `bool` parameters is both difficult to read and dangerous. But it's not only about `bool`s. Adjacent parameters of the same type always have the danger of being swapped up.

Instead of relying on good eyes, you might want to rely on the compiler and use `enum`s and classes with descriptive names.

And as Björn said, instead of strong types don't use type aliases, a type alias is just a comment, nothing more.

An interesting idea he mentioned was how to deal with parameters when you have a bunch of them and many of them would be defaultable. In that case, use a `struct`, let the members have their default values declared in place and then take advantage of [C++20's designated initializers](https://www.cppstories.com/2021/designated-init-cpp20/)! 

What a nice idea!

### [C++ and Safety by Timur Doumler](https://cpponsea.uk/2023/sessions/cpp-and-safety.html)

The topic of safety often comes up in C++. It's been an important topic for long years, but the topic is even more prevalent since the NSA wrote that "exploitable software vulnerabilities are still frequently based on memory issues"  and recommended that "the private sector, academia and the U.S. Government use a memory-safe language when possible".

In his talk, Timur discussed the different forms of safety, and how they relate to correctness, he debunked some myths and shared his view if C++ is in trouble or not.

When it comes to safety, we can think about both functional and language safety. When we talk about C++, we rather talk about language safety. Language safety can be broken down into memory, thread, arithmetic and definition safety. Timur showed through a set of small and simple examples how much C++ lacks basically any aspect of language safety.

He also showed that even if you have language-safe programs, having a functionally safe, God forbid correct program is so difficult. In that sense, C++ is not the problem.

But otherwise, how much C++ is the problem? Why did the NSA explicitly target C++?

Those who complain most often speak about "C/C++". Anyone who speaks about "C/C++" shows how little they understand these programming languages. Those are two separate languages!

While it's true that almost 50% of the reported language vulnerabilities are coming from C, C++ is actually only the 6th on the list, behind languages such as PHP, Java and Javascript. Even Python.

![Language vulnerabilities by Timur Doumler]({{ site.baseurl }}/assets/img/language_vulnerabilities.png "Language vulnerabilities by Timur Doumler")
_Language vulnerabilities (source: https://timur.audio)_


C++ took on a long journey and is full of safer features and it's still getting further safer features. Will they be completely safe? No. Will C++ be ever fully safe? No, it's impossible. At the minimum, language safety would mean no undefined behaviour.

But we need undefined behaviour and we often have to make tradeoffs between safety and performance, portability or cost.

Yet, Timur showed the different strategies we could take to achieve the different kinds of language safety and also shared how viable these strategies would be.

Timur's conclusion is that C++ mostly has a PR issue. C++ isn't behind so many safety issues as many think. Yet, it's getting safer and many UBs have been or are being removed when it doesn't compromise performance and compatibility which are often the main reasons behind using C++.

There is no such thing as a safe code, languages call other languages and even so-called safe languages such as Rust have vulnerabilities. On the other hand, the C++ committee should probably be clearer on its strategy and also on how far we already came.

An interesting talk with full of easy-to-follow examples!

## My favourite ideas

Now let me share a few interesting ideas from three different talks.

### Jonathan Müller's favourite C++ question

Jonathan's talk would have probably been among my favourite ones if he got 30 minutes more to present the same content. My brain is too slow for interesting ideas coming so rapidly! :) He talked about C++ features that are either forgotten or undervalued.

He mentioned many interesting topics, and some I'll probably write about deeper in the coming months (I'll not forget to refer to his talk!), but here I want to mention only one thing.

His favourite C++ question [that was originally posted by Richard Smith](https://twitter.com/zygoloid/status/1107740875671498752?lang=en)

```cpp
// Assuming an LP64 / LLP64 system, what does this print?

short a = 1;
std::cout << sizeof(+a)["23456"] << std::endl;
``` 

So what is the output?

The answer requires quite a few steps. I don't want to go into an explanation in this post, I'd like you to think about it. Here are a few hints:
- What does unary plus do?
- What's the type of `"23456"`?
- What does `sizeof` return?
- What is a little-known characteristic of the built-in index operator?
- What's the precedence of operations in this expression?

If you are stuck and desperately looking for the answer, check it out [on Jonathan's site](https://www.jonathanmueller.dev/talk/cpp-features/). 

### Bryce Adelstein Lelbach thinks we often treat AI unfairly

In his endnote, Bryce talked about his experiments with ChatGPT and how it was helping him create a parallel algorithm.

Listening to him probably made many of us think that oh, okay, it's hard to use AI-assisted tools effectively, they are still dumb for this and they need too many iterations, too many rounds.

But at the end, Bryce reminded us that we are just unfair towards ChatGPT and other large language models.

Do we expect ourselves to write perfect code on the first run?

Not really, right?

If you post a pull request and someone asks you if there were any bugs in it, would you reply that yes, here they are?!

Not really, right?

ChatGPT et al. cannot write perfect code on the first or second run either, but it can analyze its own code more objectively than you or I could our own code. At the same time, it can iterate on code and write better and better solutions of the same problem.

So let's reconsider how we think about them.

### Dr. Allessandria Polizzi shared that boredom can also lead to burnout

Even at a conference dedicated to C++, you might find topics that are not necessarily about the language (such as my talk about clean code), or about software development. 

Dr. Allessandria Polizzi spoke about mental health. She shared what are the main risks leading to burnout and what best practices are available for us in case we want to do against it. Burnout is real and it doesn't simply happen to you, you can do against it.

There is one risk here, that I want to emphasize from her presentation. You might think how great it is when you have a low workload. I think that if you are conscious enough it's not so bad, but according to research, for most people low workload can lead to burnout faster than a high workload. Not only burnout but even *"boreout"* is real.

In my opinion, if you have a low workload, take advantage of it. Work on your own initiatives and invest time in learning to get even better at your job.

Nevertheless, it's important to know what are the different factors that can lead to burnout.

## Personal impressions

Finally, let me talk about some more personal feelings about the conference.

### C++ On Kaizen

I remember that even [last year](https://www.sandordargo.com/blog/2022/07/27/cpp-on-sea-trip-report) how much I appreciated the constant improvements at the conference. I think most of the complaints were about lack of water and long queueing times for lunch. The water problem was solved after the first day, and this year, also the queueing situation improved a lot. In different rooms, talks ended at different times right before lunch break so that not everybody went to eat at the same time. That was a great idea! But what matters more - to me - is the mindset of constant improvement.

### Hard to stay an introvert

By the end of the conference, I felt exhausted, but in a good way. I had inspiring discussions with so many people and I even met someone who went to the same high school as me and finished just one year earlier.

While I'm an introvert and I barely start conversations with strangers, I tried my best at the conference. And even when I didn't, as a speaker I often got approached by others.

I was on the phone with my wife and I told her that I got a baseball cap as a swag, though I'm not sure if I would ever use it. She reminded me that she wears such caps. "Oh I remember" - I said, "you even have the one signed by Charles Leclerc!" At that moment I realized that I could also get some autographs at the conference. Not on a cap, but on the conference T-Shirt which has the name of all the speakers!

By the end, I got a signature from almost anyone. And mine is just next to the signature of the Explorer of Compilers! How cool is that!

![The conference T-Shirt signed by most of the speakers]({{ site.baseurl }}/assets/img/cpponsea2023-tshirt.jpg "The conference T-Shirt signed by most of the speakers")
_The conference T-Shirt signed by most of the speakers_

### My two talks

This year, my topic(s) were not technical. I signed up for a lightning talk, where I shared my findings on how one can improve his or her job hunt experience. After all, I joined Spotify less than a year ago! Such an important topic for everyone!

Thursday afternoon I got an hour to speak about [why clean code is not the norm](https://cpponsea.uk/2023/sessions/why-clean-code-is-not-the-norm.html)! In particular about what clean code is, what it has to do with software quality and also how it is related to professional ethics.

At the end, there were some good and/or provoking questions and remarks. I was humbled by the ratio of other speakers in the audience and I received many great feedback. Even if we didn't agree on everything, my talk was thought-provoking and sparked many discussions.

## Conclusion

In this article, I covered one way that can help you get closer not only to attend but to speak at conferences. In my opinion, this is way better than just attending, because often (most of) your costs will be covered, you'll learn way more and build more connections. Not to mention that it's easier to convince your management to let you speak at a conference than to buy you a ticket and finance the trip.

C++ On Sea was once again an awesome experience! Great organization, a strong line-up and awesome attendees! I hope I can be back in Folkestone in 2024.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!