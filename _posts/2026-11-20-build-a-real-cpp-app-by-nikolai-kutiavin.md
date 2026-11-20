---
layout: post
title: "No More Helloworlds: Build a Real C++ App by Nikolai Kutiavin"
date: 2026-11-20
category: books
tags: [books, cpp, bookreview]
excerpt_separator: <!--more-->
---
I recently had the chance to read [No More Helloworlds: Build a Real C++ App by Nikolai Kutiavin](https://leanpub.com/nomorehelloworlds/c/sandor30). If you're new to C++, and maybe to software development in general, I recommend you pick it up. Before I tell you why, let me tell you what this book reminded me of.
<!--more-->

## How many languages should a C++ developer speak?

A few years ago at one of the conferences I attended, someone gave a lightning talk about how many languages a C++ programmer should speak. The point was that being a C++ developer is not only about writing C++. You need at least a bit of Bash and a bit of Python, and probably more.

And that's true. The C++ code you write will not magically turn into a library or an executable that you can ship. You have to think about how you compile it, link it, test it and deliver it. Most books gracefully skip those details. *No More Helloworlds* does not. It goes through all the aspects of shipping your code:
- writing your production code
- building a graphical user interface, in C++, for a C++ app
- testing your code with different kinds of tests (unit, integration, manual)
- building your code
- setting up a CI/CD pipeline

You might think that some of these topics don't belong in a C++ book, and I hear you. If you want to learn the language itself, this won't be the only book you need. Covering so much ground in full detail is simply not possible. But if you want to learn how software gets built, I think this is **one of** the practical books you should read. You'll learn enough about each of these topics to get started, and enough to know where to dig deeper on your own.

## Is this still valuable in the age of AI?

I think so. Yes, an AI could teach you about the tools and APIs the author covers. It could probably come up with good examples, too. You could also look everything up in the official documentation. But would you know *what* to look up? Would you have the knowledge required to judge whether you can trust what you got back? Probably not, and that's where the biggest value of this book lies: you get a curated list of what to learn, from an experienced developer.

## A few ideas that stuck with me

Experienced developers are not the target audience of this book, but even I found a couple of ideas that I particularly liked or that made me think.

### Two reasons to extract an interface

When discussing why you'd pull out an interface from a class, the author says that *"usually, an interface is created for one of two reasons"*:
- *"as an abstraction provided by the library for external use"*,
- *"as an abstraction of an external subsystem for the library"*.

If you think about it, testing fits into this picture too. Mocking works only because a dependency sits behind an interface. When you mock a database in your library's tests, that's the second reason. When your users mock your library in their tests, that's the first one. In the end, both are about hiding implementation details. The only difference is the direction: either you hide your own details from others, or you hide the details of others from yourself.

### Validate as much as you can at compile time

I also liked how much Nikolai emphasizes compile-time validation. Unit and integration tests are important, but it's just as important to validate as much as possible simply by parsing the code. Testing validates behaviour, static analysis validates shape.

The importance of the latter is only growing as AI takes over more and more of the code generation. I think one of our main tasks as AI-assisted developers is to provide better validation tools. But I digress.

### Software development is integrating change

The last lesson I want to mention, which is related to the previous one, is that software development is not only about writing new code. It's also about integrating change without breaking what already exists. Git and CI are the pillars that make this possible, and the book covers both of them.

## Conclusion

The biggest strength of [No More Helloworlds](https://leanpub.com/nomorehelloworlds/c/sandor30) is that it doesn't treat writing C++ code as something that happens in a bubble. It puts it where it belongs: as one part of how you *build a real C++ app*. You write code, you test it, you analyze and validate it, then you build and deploy it.

If you're at the start of your software development journey and you've decided to learn C++, [this book by Nikolai Kutiavin](https://leanpub.com/nomorehelloworlds/c/sandor30) is a good entry point in my opinion.

{% include connect-deeper.html %}
