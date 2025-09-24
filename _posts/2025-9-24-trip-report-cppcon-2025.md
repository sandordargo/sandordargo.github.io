---
layout: post
title: "Trip report: CppCon 2025"
date: 2025-9-24
category: books
tags: [cpp, conference, cppcon, tripreport]
excerpt_separator: <!--more-->
---
A dream came true. My C++ conference journey started with CppCon. Well, almost. Back in 2019, my senior manager told me I would travel to the USA for a week to attend CppCon. In the end, cost-cutting measures changed the plan, and I was sent instead to a one-day conference within my country — CPPP. [It was a wonderful event](https://www.sandordargo.com/blog/2019/06/26/travel-report-cppp) and formative in two ways:  
- I realized that I, too, could stand on stage, share my experience, and help others grow.  
- I also realized it's better to carve my own path to conferences than to rely on a company sending me.  
In short, I learned that it's better to be invited than to be sent.  

Of course, speaking at a conference is never only about speaking. It's at least as much about attending, listening, and learning from others. For that, I'm deeply grateful — both to the organizers for putting together such a remarkable event and to my family for making it possible for me to be there.  

In this post, I'll share:  
- Thoughts on the overall conference experience.  
- Highlights from talks and ideas that resonated with me.  
- A few words about my own talk.  

*I'll update this article with links to the recordings as soon as they become available.*  

## CppCon, a conference on a different scale

CppCon is simply bigger than any other C++ conference I've attended. A massive venue packed with people and sponsors. I logged more than 10,000 steps on the very first day — without ever leaving the resort or going to the gym.

The whole experience felt like it was on another scale compared to European conferences (which I also love). But then again, that's often the impression when you see something American from a European perspective, isn't it?

I never would have imagined a C++ conference where a live band plays while Bjarne Stroustrup himself makes final checks before stepping on stage to deliver the opening keynote. Absolutely rocks.

> ![Bjarne is ready to rock at CppCon2025]({{ site.baseurl }}/assets/img/bjarne-rock-cppcon2025.jpg "Bjarne is ready to rock at CppCon2025")
> _Bjarne is ready to rock at CppCon2025_

One area where I think European conferences often do better is catering. Probably it's a matter of scale, but I really appreciate not having to think about where and what to eat. At CppCon, many sponsors hosted invitation-only lunches and dinners, which I found both interesting and unusual.

That leads me to a funny story about language differences. Living in France, for me *entrée* clearly means a starter course — it's the entry to a longer meal. In the US, however, it turns out *entrée* is a fancy way of saying "main course". So at my first seated dinner at Aurora, I served myself only a little of the *entrée*, assuming I'd need room for the main course. As you can guess, I unintentionally ran a calorie deficit that day.

Another thought on the venue: I hardly met anyone - especially from Europe - who liked it. I understand why, but I tend to disagree. Yes, it's in the middle of nowhere, and visiting Downtown Denver center during the conference is out of the question. But the schedule is so packed — with regular talks, open-content sessions, fireside chats, lightning talks, and sponsored meals — you're often booked from 7 AM to 10 PM. How could you even think about doing anything else?

One more difference compared to Europe: people here seemed a bit harsher. I saw many more attendees leaving in the middle of talks than I usually do at European events — thankfully not many left mine. I don't know if it was just coincidence or a cultural difference.

All in all, it was an incredible experience to finally be at CppCon.

## My favourite talks

I usually share my three favourite talks and three favourite ideas from a conference. Since most conferences I attend last three days, that number fits nicely. But CppCon — excluding pre- and post-conference workshops — runs for five full days. So this time, I picked five talks, one per day. They're listed here in the chronological order of their presentation.

On the very first evening, I had a great discussion with [Peter Muldoon](https://cppcon2025.sched.com/speaker/petetheladd) about what makes a talk truly exceptional. We came up with three essential components:  
- The talk must share interesting ideas (they don't even have to be technical).  
- The presenter must deliver those ideas in an engaging way (something that's often missing!).  
- The talk must energize you so much that once you're back at work, you want to try them out immediately.  

The talks I've chosen to share below satisfy at least two of these three criteria. (Sadly, I can't return to work and experiment with C++26 contracts or reflection just yet..)  

### The Joy of C++26 Contracts (and Some Myth-Conceptions) by Herb Sutter

I wasn't sure which talk to attend, but I chose Herb's. It's always a safe bet — he's an excellent speaker and knows more about C++ than most. Plus, I wanted to deepen my understanding of contracts.

I already had a general idea of what contracts are and why they're useful, but the description of Herb's talk promised something more — best practices and deeper insights for something that's not even shipped yet. And indeed, I walked away with far more than I expected. Here are six key takeaways to watch out for:

- **Don't use contracts as program logic asserts.** Otherwise, you end up duplicating work.  
- **Don't write contract assertions with side effects.** You can't predict how many times a check will be executed. (Fortunately, most such mistakes are caught at compile time.)  
- **Avoid splitting compound conditions.** Keep them together and benefit from short-circuiting.  
- **Use throwing violation handlers judiciously.** Throwing during an assertion failure can be dangerous — you must ensure no other exception is being handled and no stack unwinding is in progress.
- **Understand how build modes interact.** Some enforce assertions, some ignore them, and others offer more nuanced options. They might be combined over translation units. Know which one you're in.  
- **Be aware of the limitations of the minimal viable product arriving in C++26.** This is only the beginning, not the full vision of contracts.  

### Alex Stepanov, Generic Programming and the STL by Jon Kalb

Jon Kalb, the conference chair, gave an open-content talk at 7:15 AM. Despite the early hour, dozens of people showed up for a historical session on the father of the Standard Template Library. Let me stress that *historical* here is not a negative label. Quite the opposite: understanding history makes it easier to grasp the present. The same applies to programming languages and libraries — it's much easier to understand how something works if we know when, how, and why it was designed.

When the STL was created in the 1980s and later submitted for standardization in the early 1990s, Object-Oriented Programming was the dominant paradigm. Committee members struggled to fully appreciate the enormous contribution Stepanov made by designing a library of generic algorithms that allowed code reuse across problems without rewrites — and without sacrificing performance.

The STL is built around four main components:  
- **Algorithms**: function templates that typically encapsulate loops.  
- **Containers**: structures that store the data algorithms operate on.  
- **Iterators**: the bridge between algorithms and containers.  
- **Callables**: functions or function-like objects that customize algorithm behavior.  

While many of us tend to see containers as the most useful or fundamental component, Stepanov considered them mere tools to store data. For him, the true foundation were always **algorithms**.

Jon also shared some lessons learned from the STL design. Some are obvious — like the confusing choice of names between `clear` and `empty`. Others are less well-known. For example: if `unordered_*` containers are hash-based, why aren't they simply called `hash_*`? The answer is partly political, partly practical. Back in the pre-C++98 days, the committee couldn't resolve all disagreements, so they left them out of the first standard. By the time they were eventually standardized, different companies had already implemented their own versions — sometimes even inside the `std` namespace. Choosing one over another would have broken existing code and upset vendors. The compromise was to adopt another name.

### How to Tame Packs, `std::tuple`, and the Wily `std::integer_sequence` by Andrei Alexandrescu

Andrei Alexandrescu, author of [*Modern C++ Design: Generic Programming and Design Patterns Applied*](https://amzn.to/46FqeAl), is always among the best speakers at C++ conferences. I'm not sure he'd appreciate this comparison, but if there were a competition for *technical stand-up comedy*, he'd be a strong contender.

Of course, I wouldn't highlight his talk if it were only entertaining. It was also technically deep and complex — better described as *multi-threaded*. He explored in detail how to work effectively with template parameter packs and tuples. Summarizing the full content would be impossible in a few paragraphs, so I'll just recommend watching the recording when it becomes available. Still, I want to share two points that stuck with me.

First, about that “multi-threaded” aspect: while explaining template metaprogramming, Andrei was also weaving in a parallel discussion on the use of LLMs. Some attendees might have felt reassured when he said we shouldn't rely on chatbots for programming. But his actual point was subtler: don't use chatbots, use **coding agents**.

Second, he reminded us why *"back to basics"* talks remain so important. While AI can master syntax and basic usage, humans still crave learning from humans, and real interaction matters. As he put it, coding agents are like *"interns high on sugar, caffeine, and cocaine at the same time"* — they can produce a lot, but their work always needs reviewing. Without a solid grasp of the basics, we simply won't be able to do that.

### Mastering the Code Review Process by Peter Muldoon

Unlike previous years, this time there were several talks focused on code reviews — a topic close to me as well, since it's what I'll be speaking about at [Meeting C++](https://meetingcpp.com/mcpp/schedule/talkview.php?th=c700fa9b9bd66880219928e7626e9848520b1fa9) this year. While many presenters covered the technical side of code reviews, Peter took a different angle.

As usual, he focused on the engineering principles behind the practice: the process of giving and receiving reviews, and how it ties back to the ultimate purpose of software engineering — **delivering business value**.

Code reviews support that goal by improving code quality, ensuring correctness, and educating engineers. But to achieve this, they must be **timely and relevant**. Peter emphasized that reviews should be prioritized: it's often more important to review and merge code than to write new code. And while it's easy to nitpick over small details, reviews should always keep the big picture in mind.

> ![Peter Muldoon at CppCon2025]({{ site.baseurl }}/assets/img/muldoon_at_cppcon2025.jpg "Peter Muldoon at CppCon2025")  
> _Peter Muldoon at CppCon2025_


Peter also shared some practical advice:  
- **As an author**: review your code yourself first. Fix warnings, ensure tests pass, and make life easy for reviewers. Use a clean title and description, highlight key points, and keep your changes small.  
- **As a reviewer**: review the code, not the author. Avoid destructive comments, and clearly label your feedback. Is it a **blocker** that must be fixed? A **consideration** for improvement? A **nitpick** that's just your opinion? It could also be a genuine **question** or even a simple **kudos**.  

A great talk with clear, actionable takeaways. Kudos, Peter!

### Cache Me Maybe: Using Caches to Improve Performance in Production Code! by Michelle Fae D'Souza

Michelle's talk was both fun and highly practical — she kept the suspense alive until the end with the question: *cache or not to cache?* And the answer, of course, depends.

She walked us through how modern CPUs and memory hierarchies work, showing how much performance we can leave on the table if we ignore cache behavior. For example, replacing an `unordered_map` with a `vector` (and using indices as keys) can deliver up to a **10x speedup** thanks to spatial locality: vectors use contiguous memory, while hash maps scatter their data, leading to frequent cache misses.

Michelle also explored the idea of *cache friendliness*:  
- How to lay out variables for better memory usage.  
- The difference between an array of structs and a struct of arrays.  
- Why declaration order matters—keeping variables close to where they're used improves locality and avoids cache pollution.  

She even touched on some lower-level details. We learned about **RDTSC**, how CPUs perform out-of-order execution, and why relying on prefetching isn't always the magic bullet. She showed how you can use memory fences to enforce ordering: any load or store after the fence will really happen after it. Both CPUs and compilers can reorder instructions, so understanding this interplay is crucial.

A great reminder that sometimes the biggest optimizations don't come from clever algorithms, but simply from arranging data in a way that works with the hardware rather than against it.

## My favourite ideas

Now let me share five interesting ideas from five different talks.

### Computing power vs. human expectations (Bjarne Stroustrup)

In my own talk, I pointed out that the cost of computing power is dropping on a logarithmic scale, while the cost of engineers continues to rise — albeit only linearly. Bjarne’s keynote made me reflect on that again.

He emphasized that although computing power keeps getting cheaper and faster, there's one thing moving even faster: *our expectations*. Every year we dream up more ambitious software, with more features and more complexity. As a result, performance still matters just as much as ever — not because hardware is lacking, but because our expectations have grown to outpace it.

### The best usages of `std::move` (Steve Downey)

Steve gave a fascinating talk on `std::optional<T&>`, which finally made it into the standard nine years after `std::optional<T>` was introduced. I'll save a full deep dive into this new feature for a separate article.

Here, I want to highlight one memorable insight from his talk: *“the best `move`s are the ones we don't have to write.”*

Every time we explicitly write `std::move`, we're explaining something to the compiler. But in most cases, the compiler already knows better than we do. This doesn't mean `std::move` is bad — far from it — but it does mean we should pause before sprinkling it everywhere. Often, the smartest `move` is letting the compiler handle things for us.

### Would you consider running away? (John Lakos)

Earlier this year at [C++ on Sea](https://www.sandordargo.com/blog/2025/07/02/cpponsea-trip-report), I was eager to attend John Lakos's talk. Unfortunately, I had to skip it at the last minute — since I was giving my own talk at the same time. At CppCon, though, I finally had the chance to hear him speak on *What C++ Needs to Be Safe*.

One thought in particular stuck with me:

> *“If C++32 was as safe as Rust, would you consider running away?”*

> ![John Lakos at CppCon2025]({{ site.baseurl }}/assets/img/lakos_at_cppcon2025.jpg "John Lakos at CppCon2025")  
> _John Lakos at CppCon2025_

For most of us, I think the answer is no. And there's a lot of ongoing work to make that future possible. John himself is investing a tremendous amount of effort into safety, and with features like contracts and the introduction of erroneous behavior, the language is steadily moving in that direction.  

Thank you, John — and thank you to everyone pushing C++ toward a safer future.

### C++ might be the new *lingua franca* of programming languages (Herb Sutter)

Herb often uses historical analogies in his talks, and as a history lover myself, I truly appreciate that. This time, he spoke about what held great empires together. Roads and commerce were of course important, but a common — often second — language was always crucial. For the Neo-Assyrian, Neo-Babylonian and Achaemenid Empires, that language was Aramaic. Later, for the Roman and Byzantine Empires, it was Koine Greek or Latin. Without a shared language, no empire could endure for centuries.

Programming languages also have their own *lingua franca* for interoperability: C. For decades, it has served as the universal glue for foreign function interfaces. But it comes at a cost—an unsafe mix of raw pointers, unencapsulated structs, and manual lifetime management.

Herb argued that with **C++ static reflection**, we may have the chance to move beyond this. C++ could become the new lingua franca, offering the same universality as C but at a higher level of abstraction and with far greater safety for cross-language communication.

### It Takes a Village (Matt Godbolt)

Whether at conferences, in online chat rooms, or in the comment sections of blogs — it's all part of **our** community. And building that community takes effort. It's not the job or destiny of just a few individuals. It truly takes a village.

If we want a vibrant community, we can't just be passive members. We each have to contribute in our own way: write, speak, share feedback, and support others. That's how the C++ community grows and thrives.

## My own talk

Finally, let me share my own contributions to the conference. On a small negative note, I didn't give a lightning talk this time. I usually try to, but I hadn't prepared in advance and by the end of each day in Colorado I was simply too knackered. Note to self: prepare lightning talks ahead of time, as I usually do.

So, how did my talk go — and what was it about in the first place?

Once again, I came to a C++ conference to talk about clean code and performance without showing a single line of code or a performance benchmark on the slides. My main message was simple: **prioritize maintainability first**. It took some courage to say this one day after [Vittorio Romeo delivered a keynote](https://cppcon2025.sched.com/event/27bNZ/more-speed-simplicity-practical-data-oriented-design-in-c++) arguing that we should think about performance first. In my view, that's rarely the case. If you're writing a program for the long term, maintainability has to come first.

> ![Me at CppCon]({{ site.baseurl }}/assets/img/me_at_cppcon2025.jpg "Me at CppCon")  
> _Me at CppCon_

Clean code is really just an optimization for readability. And optimization is always about trade-offs. While clean code may not yield the absolute fastest possible program, most of us don’t need those last drops of performance. There are usually more pressing concerns: maybe binary size, maybe portability, but more often than not, **readability**. Especially in large organizations, where people constantly come and go, readability becomes the default priority.

It's also worth remembering that the cost of CPU cycles continues to fall, while the cost of developers keeps rising. That's why it makes sense to optimize for developer productivity, not for CPU time. And clean, readable code directly contributes to productivity.

> ![Growing salaries, decreasing CPU costs]({{ site.baseurl }}/assets/img/cpu-cost-developer-salaries.png "Growing salaries, decreasing CPU costs")  
> _Growing salaries, decreasing CPU costs_

Of course, sometimes optimization is necessary — but do it wisely. Chances are, you'll never need the kind of low-level wizardry [that the Quake developers once pulled off](https://www.youtube.com/watch?v=p8u_k2LIZyo). In most real-world cases, focusing on higher-level optimizations — like reducing database queries or cutting down on network calls — will give you a solution that's "fast enough".

## Conclusion

CppCon 2025 was everything I had hoped for and more. The scale, the talks, the people — it all reminded me why conferences matter so much. They are not just about learning new techniques or hearing the latest on upcoming C++ features, but also about the famous *hallway track*: connecting with others, sharing ideas, and recharging our enthusiasm for the craft.

I walked away with new insights into C++26 contracts, fresh perspectives on generic programming, some serious food for thought about code reviews and even cache behavior. I also left with a stronger sense of community — the feeling that we are all contributing, in big or small ways, to the language and ecosystem we are part of.

Most importantly, I came back energized. Conferences like CppCon are not just about what you learn during the week, but about how you take those lessons back home and put them into practice. I'm grateful to the organizers, the speakers, and everyone I had the chance to meet and talk to. And I'm already looking forward to the next opportunity to gather with this amazing community.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
