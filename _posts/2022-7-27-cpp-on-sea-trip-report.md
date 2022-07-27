---
layout: post
title: "Trip report: C++ On Sea 2022"
date: 2022-7-27
category: dev
tags: [cpp, conference, cpponsea, tripreport]
excerpt_separator: <!--more-->
---
It was the first time for me to go abroad for a conference and the first time to travel to a C++ conference as a speaker. I was so excited! I'm grateful to the organizers, my employer and of course my family to make this happen.

My excitement was mostly positive, though with the current state of air travel in Europe I was also a bit worried about whether I can get there, and whether everybody can get there. In the end, things worked out better than I expected!

The conference was in Folkestone, UK. Right on the coast where the Eurostar train comes out of the tunnel. The venue had several balconies and when the weather was nice (almost all the time), one could even see the Northern coastline of France.

Folkestone is exactly how I imagine a typical English town in the countryside with its architecture and overly kind people who greet each other on the streets. It's so easy to feel at home in such a place.

![The view of Folkestone]({{ site.baseurl }}/assets/img/folkestone-2022.jpg "The view of Folkestone")
_The view of Folkestone_

In addition to the typical architecture, a part of Folkestone is also a bit more bourgeois neighbourhood. It seemed that the rich people used to come here if they wanted to spend time at the seaside.

This time it was C++ developers.

I mentioned several times on my blog that I'm essentially an introvert and discussing with people is often difficult for me. Something I'd often try to avoid. This time though I didn't feel like that at all. I enjoyed approaching people, talking to them and also being approached. As a speaker, it obviously happened more frequently than it would have happened other times.

I'm not sure why I felt better this time. Maybe because of the past COVIDious years? Maybe because I knew many of these people from Twitter, from online spaces, and from conferences and it gave me a kickstart? Who knows...

Speaking of a kickstart. I stayed in a hotel just across the street from the venue with several other speakers. When I got down the first morning to get some breakfast, I was not seated alone, but the waiter gave me a place at a table with a bunch of other speakers who I didn't know or at least not by face. The socialization started there quite early.

And despite I'm an introvert, I try to grab each opportunity to go on stage and present so that I can practice, I can get better at it. It was a no-brainer for me to submit a lightning talk. Due to [a recent very annoying bug](https://www.sandordargo.com/blog/2022/07/06/lifetime-extension-bughunt), I had a topic at my hands. It was an honour to go on the main stage at Folkestone and speak in front of so many smart people.

The second night we had a speakers' dinner with once again a great view of the sea and delicious food.

![The dinner]({{ site.baseurl }}/assets/img/dinner.jpg "The dinner")
_The dinner_

Due to the discussions at the tables, the room became quite noisy, so many of us continued to share some stories on the terrace. I know that later many continued at pubs, but I wanted to get to bed early because I had the first slot the next morning.

I was speaking about [strongly typed containers](https://cpponsea.uk/2022/sessions/strongly-typed-containers.html) and this time I was satisfied with my talk. Probably for the first time since started to present at conferences. I got some nice feedback and a very important remark about the inherited comparison operators, so I also learned something. [Check out slide 33 here](https://cpponsea.digital-medium.co.uk/wp-content/uploads/2022/07/COnSea2022-Strongly-typed-containers-C.pptx) for the details!

![Me during my talk, thanks for @hniemeye for the photo]({{ site.baseurl }}/assets/img/sandor-cpp-on-see.jpeg "Me at my talk, thanks for the photo, @hniemeye!")
_Me at my talk, thanks for the photo, @hniemeye!_

Now let's talk about other talks!

## Three talks I particularly liked

Let me share with you 3 talks that I particularly enjoyed during the event. Once the recordings are released on Youtube, I'll update the article with the links.

### [What do you mean by "Cache Friendly"? by Björn Fahller](https://cpponsea.uk/2022/sessions/what-do-you-mean-by-cache-friendly-online.html)

We hear often about cache hits and cache misses when we talk about performant code and performance-optimized (C++) code. I know so little about this topic that I definitely wanted to attend this talk. In addition, I met [Björn](https://twitter.com/bjorn_fahller) during breakfast after my arrival and I found him a very nice person who can explain things well.

I was not disappointed. He started with a personal story. He expected his code to be limited by latency, but it turned out that it was the CPU. His code spent most of its time in a certain `schedule_timer` function.

Very soon, he started to speak about object vs cache sizes. Why and how we should limit the sizes of our objects if we have a great bunch of them. As the presenter shared, *"doing more work can be faster than doing less"*.

Why is that?

Chasing pointers will almost always be a cache miss. Storing objects in a contiguous memory area and going through more objects to find something can be faster than just following pointers.

This concept was proved in his examples where [Björn](https://twitter.com/bjorn_fahller) optimized his initial code and tried using many different data structures and modified objects.

An important and not-so-surprising takeaway is that our predictions are not always right. If we deeply care about performance, we must continuously measure, measure and measure.

### [Six ways for implementing max: a walk through API design, dangling references and C++20 constraints by Amir Kirsh](https://cpponsea.uk/2022/sessions/six-ways-for-implementing-max-a-walk-through-api-design-dangling-references-and-cpp20-constraints.html)

I find it amusing that someone is always talking about how `std::max` is broken. Last year, it was [Walter E Brown talking about how its implementation is broken as `std::min` and `std::max` might return the very same values](https://www.youtube.com/watch?v=V1U6AXEgNbE).

[Amir](https://www.linkedin.com/in/kirshamir/) didn't talk about an implementation problem but more about a design issue. We cannot find the maximum of different types. For example, `std::max(5, 6.5)` will not compile because 5 is an `int` while 6.5 is a double. Of course, you can make it compile with a `static_cast`, but that you might consider ugly.

The speaker showed an implementation for `max` that could just take any two comparable types and return the maximum of them considering whether they were passed by value or reference.

Why do I mention this talk among the best ones?

First, it was truly interesting. But what I enjoyed the most was the ease with [Amir](https://www.linkedin.com/in/kirshamir/) was standing on stage and performing a live coding exercise. Of course, there were some issues, not everything worked at first, but he handled those situations well. And in addition, he made the session very interactive, there were lots of questions addressed to the audience and he often moved forward based on the answers. Bravo!

### [Midnote: For the Sake of Complexity by Kevlin Henney](https://cpponsea.uk/2022/sessions/midnote-for-the-sake-of-complexity.html)

[Kevlin](https://twitter.com/KevlinHenney)'s stage presence, his smile and the enthusiasm he talks with make it very difficult not to mention his talks anytime when you think about what you liked best.

This is the first time that I heard/saw him live and indeed it was a strong experience.

But what did he talk about?

Complexity!

He showed an image of a magnificent Swiss watch. It's the most complex watch ever made. And that increases its value!

![The world's most complex watch from https://newatlas.com/vacheron-constantin-57260-worlds-most-complicated-watch/39462]({{ site.baseurl }}/assets/img/complex-watch.png "The world's most complex watch, image from newatlas.com") 
_The world's most complex watch, image from newatlas.com_

Now imagine that you write an overly and selfishly complex piece of software.

Try to brag about its complexity!

While *"developers are drawn to complexity like moths to flame" (Neal Ford)*, our work is rather about maximizing the simplicity of software. We have to break down a big complex problem into small simple issues that we can solve.

We tend to generalize solutions where no generalization is needed. *"Oh, I will just add a strategy pattern here, some type erasure there and then it will work with the requirements of next year."* The problem is that nobody asks for that and most often and nobody will use or actually understand the code. We should care about general issues only when it's needed, otherwise aim for simplicity!

First let's build something complete, but simple.

Then add the clever parts. 

Use, before reuse.

## Three interesting ideas

As I usually do with trip reports, I don't only want to share some of my thoughts on entire talks, but sometimes I just want to transmit certain ideas I found particularly interesting.

### Having longer functions is sometimes the right

On my badge there was a quote:

> *"Your function is longer than 10 lines? [Extract till you drop.](https://sites.google.com/site/unclebobconsultingllc/one-thing-extract-till-you-drop)"*

I don't believe in extremes. Nothing is black and white. Although I do believe that in most situations, following strict rules is better than following no rules at all. It still doesn't make them true in every situation.

This quote - which is also in my corporate email signature - sparked many interesting discussions. You cannot spark discussion by saying that yeah, well, sometimes you should keep your functions relatively small...

The same idea was shared by [Arne Mertz](https://twitter.com/arne_mertz) in his talk about [Identifying common code smells](https://cpponsea.uk/2022/sessions/identifying-common-code-smells.html). Shorter functions are usually preferable. But not all the time.

But let's step back a bit.

Is a long function a problem?

No. It's just a code smell. [As Martin Folwer said](https://martinfowler.com/bliki/CodeSmell.html#:~:text=A%20code%20smell%20is%20a,me%20with%20my%20Refactoring%20book.), *a code smell is a "surface indication" that usually corresponds to a "deeper problem" in the system*.

In this case, the deeper problem is the violation of the [single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle).

But as the *usually* word implies, it's not always a problem.

In any case, it's impossible to name a number for the maximum function length. Is it 100 lines? 20? 10? A hundred seems too big number, but what about 10? Sometimes even that would be too long, but sometimes 20 is acceptable.

Often, there are some indicators that suggest factorizing a function, such as comments of code blocks.

```cpp
// Create the left paddle
sf::RectangleShape leftPaddle;
leftPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
leftPaddle.setOutlineThickness(3);
leftPaddle.setOutlineColor(sf::Color::Black);
leftPaddle.setFillColor(sf::Color(100, 100, 200));
leftPaddle.setOrigin(paddleSize / 2.f);

// Create the right paddle
sf::RectangleShape rightPaddle;
rightPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
rightPaddle.setOutlineThickness(3);
rightPaddle.setOutlineColor(sf::Color::Black);
rightPaddle.setFillColor(sf::Color(200, 100, 100));
rightPaddle.setOrigin(paddleSize / 2.f);

// Create the ball
sf::CircleShape ball;
ball.setRadius(ballRadius - 3);
ball.setOutlineThickness(2);
ball.setOutlineColor(sf::Color::Black);
ball.setFillColor(sf::Color::White);
ball.setOrigin({ballRadius / 2.f, ballRadius / 2.f});
```

In this case, it's evident that we should extract functions for creating the paddles and the ball. But imagine an algorithm, like the Sieve of Eratosthenes. [It will be longer than 10 lines](https://github.com/sandordargo/KnuthPrimeGeneratorCpp/blob/master/src/PrimePrinter.cpp#L30-L50).

Is that a problem?

No. The problem would be to break that entity down into incomplete, useless parts just for the sake of shortening it.

Don't follow rules blindly.

### Don't always pass input arguments by const reference

Victor Ciura's [C++ MythBuster](https://cpponsea.uk/2022/sessions/plenary-cpp-mythbusters.html) talk was very interesting and it's difficult to pick one myth from his talk, but here is one.

We all learned that we should pass non-POD input arguments by `const&`. And I still think that it's an easy way to follow, an easy rule of thumb that will be good enough in the majority of the cases.

At the same time, there is a new pattern emerged with the appearance of move semantics. When a class takes ownership of a variable you should consider taking the variable by value and moving it.

```cpp
class Widget {
    std::string id;
    std::string name;

public:
      Widget(std::string new_id, std::string new_name) : id(std::move(new_id)), name(std::move(new_name)) {

      }
};
```

Some are very uncomfortable with this. Taking a variable by value... One could spare a move operation if there were two overloads; one for `const&` and one for `&&`. But in the vast majority of the cases that doesn't really matter. One spared move operation is not worth polluting your API with an extra overload.

When a class should take ownership of input variables, think about the sink pattern and take them by value!

### Singleton is not a design pattern

[Klaus Igleberger](https://de.linkedin.com/in/klaus-iglberger-2133694), the main organizer of the [Munich C++ user group](https://www.meetup.com/en-AU/mucplusplus/) dedicated his talk to [the Singleton (anti)pattern](https://cpponsea.uk/2022/sessions/the-singleton-pattern-anti-pattern-or-solution.html). But what is the problem with it? Apart from that, it represents a global state...

The problem comes from a bad classification that also brings unmet expectations.

The Singleton was enumerated as a creational design pattern in the [Gang of Four Design Patterns book](https://amzn.to/36VKyO2). Its role is to ensure that only one instance of an object is created.

What do we expect from design patterns?

In general, we expect two things:
1. They should provide an abstraction
2. They should decrease dependencies

The Singleton pattern does not offer any of those. Therefore it's not a design, but an implementation pattern.

That observation gives way to combine it with other techniques and use it in a way that doesn't make the application more complex to test but actually helps simulate real-world relationships without making the software less testable.

For the rest, check out the talk!

## Ideas for improvement

I keep writing in all my trip reports, that mentioning only the good parts would be very unbalanced and you'd probably think that I do this because I was paid to do so. While it's true that as a speaker most of my expenses were covered, I still think that providing gentle, constructive feedback is useful and shouldn't hurt feelings. So, let me mention a couple of ideas.

The first lunch seemed a little bit chaotic. Like everyone else, the caterers also suffer from the lack of human resources. The situation greatly improved over the next two days. On the third day, they were a few minutes late which is not an issue, but I couldn't wait. I had to make a long phone call. I came back about 40 minutes later and most people finished feasting and there was still more than enough food for me. That's something I didn't expect after the first day, I wish I could improve as fast as the catering service adapted!

The only thing about food and refreshments that could have still improved a bit was the situation with the water.

I like that there was no bottled water all around. It's better to avoid all that plastic. At the same time a few jugs of water, not even in all the breaks, was clearly not enough. Probably some simple, but big bottled water dispensers would have been fine, or maybe just some duck-taped indications mentioning that the tap water is good to drink.

One last thing to mention. The sponsors were great. Besides financially supporting the event, some of them brought cool and useful swags (in particular [Roku](https://www.roku.com/) and [Optiver](https://www.optiver.com/)), and all of them were available for very interesting conversations. The only thing that saddened me was how some of them left the event. It's understandable if they cannot make it for the last afternoon, especially with the current air-traffic situation, but probably all of them could have avoided disassembling and packing up their stuff during the talks ongoing. That was a bit disturbing. But in any case, a big thanks to them.

## Conclusion

[C++ On Sea](https://cpponsea.uk/) was my first in-person C++ conference as a speaker and I truly enjoyed it. Even though even as a speaker and an attendee it felt like hard work and study, it was almost like being on a vacation. I met very nice people who knew only online or not-at-all. I listened to great talks and learnt a lot.

With this trip report, I hope I managed to give you back something from the vibe and I hope to see you at a future event!

And once again, a big thanks to all those who made this event happen!

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
