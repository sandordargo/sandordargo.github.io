---
layout: post
title: "Refactor Knuth's Prime Printer"
date: 2017-1-17
category: dev
tags: [dojo, primes, knuth]
header: "During this week's coding dojo we had the exercise of refactoring Knuth's prime printer. If you have seen episode Functions from Uncle Bob's Clean Code videos, this might have rung the bell."
---


If not, imagine some nasty long main function with wonderful variable names like `CC`, `RR`, `j` and `jPrime` for example. I hope you don't like these names. [Here](https://github.com/sandordargo/KnuthPrimeGenerator/tree/original_code) is the starting code in Java and [here in C++](https://github.com/sandordargo/KnuthPrimeGeneratorCpp). 

Just for the record, I took the Java code from Uncle Bob's video and quickly translated it to C++ for those who prefer that language during dojos.

There were five of us, two people working in pair on the C++ code and three of us working on the Java code base. The Java guys not exactly in pair programming, but we constantly checked what the other's doing. Usually we do pair programming during these events, once we tried mob programming. But this time two guys also wanted to experiment a bit with it IntelliJ.

Everyone went into more or less the same direction. We isolated the prime generator code from the number printer part. Then most of us focused to clean up the number printer part. In my opinion it makes sense. That's something which potentially more reusable and the algorithm is quite understandable. And quite easy to clean up.

The prime generator part is a bit more tricky. The variable names are even worse than in the printer part and you'd better know the algorithm prior figuring out all the variable names. If you don't know the algorithm you might think that there are some unnecessary parts. Not really. Still you can extract a few functions. [Here](https://github.com/sandordargo/KnuthPrimeGenerator) is where I got by the end of our lunch break.