---
layout: post
title: "C++ Best Practices by Jason Turner"
date: 2021-6-5
category: books
tags: [books, watercooler, cpp, bestpractices]
excerpt_separator: <!--more-->
---
[This is a book](https://leanpub.com/cppbestpractices) that I've been waiting for to finally read for a long time. I could have bought it, but I decided that it'll be the first book that I buy from the royalties I earned with [How to use const in C++](https://leanpub.com/cppconst).  
<!--more-->

My hard-earned money was well invested. Though I was a little bit surprised in the beginning and I was not completely convinced that it was worth the money.

I attended the talk of [Jason at C++Now](https://cppnow2021.sched.com/event/hhlq) where among others he talked about his journey of writing this book. He shared his experience with [Leanpub](https://leanpub.com/) and the reactions he and his book received.

Leanpub has a 45-day money-back guarantee meaning that you can read any book for free. I say for free because 45 days is enough for most of the books and at the same time, they cannot ask you to return a PDF copy... Despite this generous money-back guarantee, only a few people asked for a refund and their complaints were always about the length of the book. It's 130 pages and it's listed on the page of the book. Anyone can see it as Jason said.

![C++ Best Practices on 130 pages]({{ site.baseurl }}/assets/img/turner-130-pages.png "C++ Best Practices on 130 pages")

That's right. Anyone can see that number, yet I also had the same idea when I finally bought my (digital) copy a few days before I *"went"* to his talk. Maybe I didn't pay attention to the number of pages at all. But the number of pages is not everything. I found that even those pages have plenty of whitespace on them.

It's not added in purpose to pump up the number of the pages, it's just a consequence of the structure. Around 45 tips in 50 chapters including listings, section headings etc.

I was a bit puzzled.

And then an idea struck me. It came in form of a story. Probably you know the story of the expert who has been called to fix a big broken machine in the factory that nobody could fix. He looks at it, examines it for a few minutes, then he replaces a $2 screw. Everyone is amazed and even more when he charges $10,000.

When the factory manager indignantly asks how can he ask for $10,000 for a few minutes of work and a $2 piece, the expert said that you don't pay for the time it took him to fix, but for the years he learnt how to fix it so easily.

In the case of this book, you also don't pay for the pages. You pay for the wisdom, the experience, the guiding.

This guy knows what he talks about. Probably he also knows about the Pareto principle. He knows exactly what matters the most.

And he listed those items, cutting out all the rest. He doesn't have to [apologize that he didn't have time to write a short book so he wrote a long one](https://quoteinvestigator.com/2012/04/28/shorter-letter/).

With his experience and reputation, Jason Turner doesn't have to write long books just to make them "thick" enough.

And here comes the interesting part which some might consider a weak point of the book. I'd say it's challenging and motivating.

You'll find relatively few and short explanations directly in the book. Instead, it gives you several exercises, some instructions and lots of references. Instead of giving you know knowledge on a silver spoon, the author decided to show you where the find it. If you prefer, we might say that he teaches the reader to fish, instead of giving us the fish.

It depends on you if you like this approach. Nevertheless, a book of 130 pages that is easy to read and you can finish it in a half afternoon, might easily give you months of research and exercises.

## Some recommendations

That's about the book in general, let's see a couple of examples of the recommendations he lists.

## On constness

[Lefticus](https://github.com/lefticus) dedicates two chapters to the importance of using `const` and `constexpr`, but it is mentioned in multiple places after.

His point is that everything that is known at compile-time should be declared as `constexpr` and the rest should be `const` whenever possible.

These changes make the developer think about the lifetime of objects and it also communicates some meaning, some intentions to the reader.

If you're looking for more details on constness, check out my book on [How to use `const` in C++](https://leanpub.com/cppconst/).

## Prefer `auto` in many cases

The author shares that he is not a follower of the [Almost Always Auto "movement"](https://herbsutter.com/2013/08/12/gotw-94-solution-aaa-style-almost-always-auto/) that was propagated by Herb Sutter, but he does think that `auto` should be preferred in many cases.

The reason behind this is that often you should not be concerned by the type of something, such as the return type of `std::count`. 

By using `auto`, we can spare unnecessary conversions and even data loss!

Besides, with `auto` it's easier to write generic code. C++11 made a big step towards that, but with a better type deduction and generic lambdas, C++14 made an extra leap towards this direction. By the way, the author also suggests skipping C++11 and go directly to C++14 if you haven't migrated yet from old C++.

## Beware of undefined behaviour 

Undefined behaviour (UB) is something we should avoid as it's dangerous. As such it appears in the book in a couple of places.

One recommendation of Jason is to treat warnings as errors and to use different sanitisers, such as UBSan and ASan. They will point out most of the UB.

But that's not everything. He mentions a form of UB that I didn't know about before and I hadn't seen. Checking for `this` to be a `nullptr` is UB.

```cpp
int Class::member() {
  if (this == nullptr) {
    // removed by the compiler, it would be UB
    // if this were ever null
    return 42;
  } else {
    return 0;
  }
}
```

It's impossible for the check to ever fail, compilers nowadays remove this check, yet this is technically UB. I don't see any reason to write such code, but I look around in the codebases I have access to and... I don't want to continue that phrase... If you find any, just remove that code.

## Conclusion

If you're afraid of too lengthy books, but you also don't want something shallow, if you are ready to delve yourself into further research and experimentation, [this is your book](https://leanpub.com/cppbestpractices).

Jason Turner is probably among the most known C++ developers nowadays with [iconic talks](https://www.youtube.com/watch?v=zBkNBP00wJE) and a [popular YouTube channel](https://www.youtube.com/channel/UCxHAlbZQNFU2LgEtiqd2Maw) and this book is the distilled version of what he learned about C++ development during the last 15 years. Most of us have definitely a lot to learn from his experience, but it doesn't come for free.

I don't mean [the $10 that is the initial price](https://leanpub.com/cppbestpractices), but the work you have to put in. Take it the other way, he doesn't sell you dogmas and believes, he shares his best practices and asks you to do your research and decisions. It's the best way to grow.