---
layout: post
title: "Trip Report: CPPP 2021"
date: 2021-12-15
category: dev
tags: [cpp, conference, cppp, tripreport]
excerpt_separator: <!--more-->
---
December started with another fascinating C++ conference which was probably the last one for the year: CPPP 2021.
<!--more-->
The third *P* in the name represents the French touch in this conference, it stands for *Paris*. Sadly, this year, it was fully online due to well-known reasons, so whether a conference was American, Italian, English, German or French (the ones I went to) didn't make much difference in terms of catering ;)

Yet this doesn't remove anything from the values of the conferences, and I saw more and more efforts for trying to bring back conference chats, networking in between talks - more on that later.

If we look for another meaning for the 3 *P*s in *CPPP*, it's the 3 tracks of the conference:
- The *Progress* track dedicated to learning and reinforcing basic C++ knowledge and understanding - which went quite deep sometimes.
- The *Produce* track dedicated to sharing solutions to produce and maintain reliable software using C++.
- The *Push Forward* track dedicated to sharing new patterns and functionality of C++.

## On my performance

Human is a selfish beast and I'm pretty much human. Let me share some thoughts on my participation in CPPP.

[The first CPPP in 2019](https://cppp.fr/schedule2019/) was my very first C++ conference to attend. I went there and I saw some very engaging presentations. I was dreaming about once participating as a speaker.

Two years later, it became the reality! I could even share what I know in 2 talks and besides I signed up for a lightning talk.

So how did it go?

My presentation on [The Concepts of concepts](https://cppp.fr/schedule2021/#session-309) went really well. It was not the first time for me to present something similar, though the presentation keeps evolving based on my experience and on my knowledge.

Probably for the first time, I was really satisfied at the end when I turned off the streaming. I shared everything I wanted and I didn't feel that sometimes the words hadn't come.

I cannot feel the same way regarding [Parameterized tests with Gtest](https://cppp.fr/schedule2021/#session-107). I was facing two issues. The first one was my mood. I got some worrying news about a family member who got hospitalized. My son also didn't sleep very well, so obviously, neither we did.

The other problem was the way I prepared for this presentation. I cannot say that I was lazy, I was clearly not. I updated the article on this topic, I rewrote the repository containing the examples and I spent a lot of time polishing the slides.

I'm quite comfortable with this topic, I have explained it to my colleagues several times. I didn't feel the need to do practice sessions.

But practice sessions are not for learning about the topic. It's about memorizing the slides, how you built one idea on top of the other. So I should have done practice sessions to make the presentation smoother. And I added some extra slides 2 days before the presentation where I made a mistake in their order...

In any case, I hope the attendees found it useful and I learnt something once again.

## The 3 talks I loved the most

Now let me mention to you 3 talks that I particularly liked.

### C++'s Superpower by Matt Godbolt

CPPP 2021 opened in a very strong way, it all started with the keynote of [Matt *"sometimes verb"* Godbolt](https://cppp.fr/schedule2021/#session-3). His talk had 2 main parts. In the first one, he iterated over what might be considered the superpower of C++, what it is in his view and then he showcased it.

So first thing first. What is the superpower of C++?

You might think about ubiquity, meaning that C++ is there everywhere. In mobiles, cars, in whatever is critical, now even on the web with the spread of wasm.

You might think that the superpower is performance and given some later talks on the effect of C++ on CO2 emissions, I think this might be a real superpower, but it's not what Matt meant at this time.

The multi-paradigm approach makes C++ very versatile and we can write code in so many different ways, but still, it's not what Matt had in mind.

Not even a clear object lifetime, but that's also great.

No, for him it's the legacy support. You take a very old codebase and most likely it'll still work with modern compilers with a few things to fix maybe.

So in the second part, he explained how he took a codebase from his student years and updated it step by step to follow modern C++ practices and use the features available in C++17.

I think all the techniques, approaches he detailed will come in handy for everyone working on legacy code.

### The Performance Price of Virtual Functions by Ivica Bogosavljevic

[Ivica](https://twitter.com/i_bogosavljevic) delivered a very practical talk about something we heard so many myths about. The costs of virtual functions, something many of us are afraid of!

I'm in no way in the position to reiterate everything he said about jump destination caching or instruction cache eviction.

I'd rather just mention a couple of important points and let you watch the video.

Ivica shared that often the virtual functions' performance is not tested in relevant ways, not how they are used in real life. According to his measurements, big virtual functions have no relevant overheads compared to their non-virtual versions. Short functions do have a penalty of about 20% and it's mostly because virtual functions cannot be inlined, cannot be optimized that way. Long functions wouldn't be inlined anyway, so hence there is no problem there.

On the other hand, vectors of pointers perform much worse than vectors of objects because of all the heap allocations leading to cache misses during the iteration.

That can actually make your processing even 7 times slower which is quite significant. He proposed different solutions to avoid this problem, in particular using a variant combined with a visitor or having different vectors for the different types (no pointers!) and doing what is called type based processing.

In any case, one of his key messages was that if you have to optimize the code in terms of performance, always think about the hot code, the code that is frequently executed, otherwise, you won't achieve relevant results.

### The discussions!

I was hesitating whether I should detail [How I learned to stop worrying and love MISRA](https://cppp.fr/schedule2021/#session-210) by [Loïc Joly](https://www.linkedin.com/in/lo%C3%AFc-joly-1493391/) or the discussions we had. As you see, I went with the discussion, but I recommend you to watch the talk on MISRA - once it's available.

It was a bit unclear first in the agenda what the dark pink colour meant. 

![CPPP Pink schedule]({{ site.baseurl }}/assets/img/cppp-pink.png)

Then someone asked in the discord chat and it became clear. It was timeslots for discussions around dedicated topics.

I think that in Covid, conferences are struggling to provide an experience that justifies buying the tickets and not simply waiting a couple of weeks and watching the talks on Youtube. Talks that are often the same between the different conferences. 

One way - maybe the only way - of achieving this is through the discussions. What I particularly liked about CPPP's solution is that the topics were set in advance.

Probably because I'm an introvert, I barely join "lounges", "discussion rooms" with no topics. But when I see that there is a room dedicated for legacy code and I do have questions I want to ask others, I'll definitely join.

While I didn't always join the rooms during lunchtimes, I always joined the others and I actively participated in the discussions. That was a completely new conference experience for me, hence it made it to this list.

## Three catchy ideas

Now let me highlight 3 ideas from various presentations.

### Use C++ for a greener planet

Sometimes an image is worth more than a 1000 words.

![Carbon footprint]({{ site.baseurl }}/assets/img/carbon_footprint.png)

C++ is so much more performant than most of the other languages! Software written in C++ consumes way less than software written in PHP, Python, TS or Ruby.

It's not only C++, [C and Rust would perform similarly](https://thenewstack.io/which-programming-languages-use-the-least-electricity/?s=09). That's not the point.

The point is that writing software in these high-level languages out of "intellectual laziness" (*[thanks Marek!](https://twitter.com/mrkkrj/status/1467798371670925315)*) is damaging the planet.

I find this topic really interesting and I don't want it to take over the whole trip report, I'll elaborate on this later.

### Push- vs pull-based iterations

[Barry Revzin](https://twitter.com/BarryRevzin) delivered a very interesting keynote on iterators and ranges. He compared the design behind these concepts in different languages. He focused mainly on C++, D and Rust, but he covered a bit also Python and Java.

I'd like to highlight one topic, one concept that was completely new to me. The notation of and differences between push- and pull-based iterations.

I don't want to go into the very details, so in brief:

When you have an iterator that pushes elements out to you and you have to implement a function that you pass to this iterator, we talk about push-based iteration. The function basically consumes these values and they don't get outside.

On the other hand, when you have to pull elements out of this iterator that are exposed then, we talk about pull-based iteration.

Things are not black and white, as Barry's example shows a push-based iteration is often implemented by a pull-based one. Let me borrow and share his example:


```cpp
template <intput_iterator I, sentinel_for<I> S>
class cpp_stream {
  I first;
  S last;
public:
  using reference = iter_reference_t<I>;

  template<invocable<referece> F>
  void for_each(F f) {
    for(; first != last; ++first) { // pull based iteration
      invoke(f, *first); // item pulled from the iterator
    }
  }
};


template<Stream S>
void print_all(S stream) {
  stream.for_each([](auto&&){  // push based iteration, elements are kept inside
    fmt::print("{}\n", elem); 
  });
}
```

### YCombinator for recursive lambdas

Lambdas are not recursive. They cannot call themselves. It makes sense after all. A lambda is an anonymous function and has no name. You might save it a variable, but it's still not something that knows about itself.

And while probably you cannot come up with any good reason to have a lambda calling itself, it's still possible to achieve it.

[Sy Brand](https://twitter.com/TartanLlama) shared a story about how they thought to show the interviewer their smartness, but as you can imagine it was not how it was perceived. In any case, I borrow their code to share with you how to turn a lambda into recursive:

```cpp
#include <functional>

template<class Fun>
class y_combinator_result {
  Fun fun_;
public:
  template<class T>
  explicit y_combinator_result(T&& fun):
    fun_(std::forward<T>(fun)) {}

  template<class ...Args>
  decltype(auto) operator()(Args &&...args) {
    return fun_(std::ref(*this),
                std::forward<Args>(args)...);
  }
};

template<class Fun>
decltype(auto) y_combinator(Fun &&fun) {
  return y_combinator_result<std::decay_t<Fun>>(std::forward<Fun>(fun));
}
```

And how to use it?

Here is a simple example:

```cpp
auto gcd = y_combinator([](auto gcd, int a, int b) -> int {
  return b == 0 ? a : gcd(b, a % b);
});
std::cout << gcd(20, 30) << std::endl;
```

To go into details on how and why the y-combinator works is way beyond the scope of a trip report. If you want to learn more about it (you'll also find more references) [click here](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0200r0.html).

## Rooms for improvement?

I mentioned in my previous trip report on [Meeting C++](https://www.sandordargo.com/blog/2021/11/17/trip-rerport-meetingcpp-2021) that I don't believe that reports mentioning the not fully shiny parts are realistic. I know it might hurt people, though that's clearly not my point and I don't think I share these ideas in a hurtful way.

In this case, I'd mention two things. One is specific to CPPP, the other is a more common problem that I see.

I think the schedule page could be improved. At least some footnotes on the colour codes would ease the understanding of what yellow and pink mean without having to think about it. I think wouldn't be a big work.

Ideally, you wouldn't only have a button to see the whole schedule in Google calendar, but you'd have a button to add a specific talk to your calendar of choice (not only Google Calendar) and - if feasible - with a link to the live stream in it.

The other thing is not specific to CPPP. I went to 5 C++ conferences this year, and I feel more and more how difficult the job of the organizers is with the pandemic.

Before, it was easier to sell your conference saying that well, we target mostly French, Italian, German, English, etc. developers for the obvious reason of physical locality.

This is almost impossible now. The only thing that makes something localish is the time zone. The number of people willing to present seems very limited and let's face it, the talks are often very similar. I see people sharing the same talk (including me) 3-4 times. And while no two talks are the same (talks evolve and the presenters gain more and more experience), I think it's hard to sell tickets like that.

I see no solution for this, because 
- the number of people willing to present doesn't just grow by wishing for it
- the presenters most often prepare the talks on their personal time, they don't have the time to create 2-3 or more brand new different talks each year.

Hopefully, Covid ends soon and organizers will be in a better position to target their - local - audience.

## Conclusion

With going to and presenting at CPPP a dream came true, I finished a journey that I started in 2019. I could give something back, I could contribute and I proved to myself that I can understand something in a deeper way so that I can present it to my fellows. I know this is just the beginning.

As an attendee, I really enjoyed CPPP! By this time I learnt to enjoy online conferences. It was smooth and high-quality in every sense, I'd be happy to go back next year. Hopefully to Paris.

Till then, I encourage you to watch the videos - I'll update the article with the links, once they are available.

Thanks a lot, [Fred](https://twitter.com/FredTingaudDev), [Joël](https://github.com/jfalcou) and all the organizers for making this conference happen! 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

