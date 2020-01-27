Recently I've attended a C++ 11 Performance and Optimization training by [Hubert Matthews](https://www.linkedin.com/in/hubert-matthews-b1916/). Among other achievements he's actively participating in the UK C++ panel, meaning that he's working on the standardization process. Even though I agree with [Donald Knuth]() when said that "premature optimization is the root of all evil", I must be sure that he didn't mean that we should write O(n2) or even worse code in the first place. In other words, write something clean, take into account the basic performance relevant knowledge of your chosen language, then mesure your software, Mesure it a lot. Know what performance you need in terms of speed, in terms of memory consumption, etc. If you achieved that level, stop. If not, know your possibilities, go to the next level, then stop.

[put here some diagram, about the workflow]

This post is the opening part of a series about some of the topics, techinques I learnt about at Hubert's training. Just to emphasize, even though some the techniques are language independent, but most of them are about C++.

hard to speak about performant c++. some people focus on memort, for gamers is something else

You'll be likely to read posts about what kind of operations you have to start eliminating and what are those that you can leave there, even better to say what are those you should turn the former into.
The importance of data layout
(
data layout can help the compiler a lot.
everything might be calculated at compile time
)
Move semanticss - rule 3/5/0
Why not to use new
Review on STL
What know - what to read
write about small string optimization
Avoid threading


Performance requirements are most of time relative. Are they?

Depending on what kind of application you are working on 


Domain knowledge is more important than any compiler optimization


if you work on a small data set and you know it'll be small, don't care about big O...



don't put empty descrtuctors to your class. it cannot be moved

vector::emplace_back, better than move

rule 3/5/0



malloc is slow because get me memory that is not used == not in the cache == have to go to RAM, slow
cache misses!


STl

use a vector with reserve, it's fast
array is still even faster
cause in vector there is a check of size, even with reserve

problematic reallocation factor of two

set insertion with hint!

move operations should be always noexcept, otherwise they can throw

i guess we really have to encapsulate data containers so that we can change them easier. they have so different characteristics



write about books



string view is useful to pull out only some of the characters
zero copy, zero allocation, noexcept, constexpr

make decisions based on your << operational profile >>

maintenace wise.... easy to break performance. you have to setup performacnce tests

no objects too low level



with http2 you can have multiple requests without multiple connections

do not use volatile variables except for
memory-mapped device I/O

keyword alignas()... cacheline?

having more cores can be make us go slower due to false sharing

write about anonymous namespaces to hide symbols

concepts in c++ (c++20?)

