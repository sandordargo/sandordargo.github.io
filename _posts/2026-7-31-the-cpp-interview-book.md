---
layout: post
title: "The C++ Interview Book is here"
date: 2026-7-31
category: books
tags: [cpp, career, books]
---
Five years ago I published *Daily C++ Interview* — a collection of questions and answers that grew out of an email course I ran for C++ developers preparing for interviews. It did better than I expected. Over a thousand copies sold and plenty of kind emails from readers who found it genuinely helpful while preparing for their next role. Over time, the feedback made one thing clear: the book deserved a proper second edition, not just a few patches.

This is that edition. I have renamed it *The C++ Interview Book* because the daily-email framing no longer fits what the book has become.

## What changed

The short answer: almost everything except the core idea.

The first edition had 150 questions, many of which asked the same thing from slightly different angles — a side effect of the one-question-per-day email format. I deduplicated ruthlessly. Where two questions covered the same concept, I kept the stronger version and added a cross-reference. The result is 173 questions — more than before, but with much less repetition.

I restructured the book around thirteen parts organised by topic rather than by the semi-arbitrary order of the original emails. Each part is self-contained enough to read on its own. If you need to brush up on move semantics the night before an interview, you can jump straight to Part 5 without working through four other parts first.

Every question now carries metadata — a difficulty tag (Junior, Mid, or Senior), topic tags, and cross-references to related questions. The difficulty tags are new and they help you calibrate where to spend your time.

I added a 100-day study schedule in Appendix A for readers who want the structured daily experience. It uses spaced repetition — you revisit material at increasing intervals — and covers all 173 questions in about three and a half months.

There is a new code-reading capstone in Part 13. Interviewers have always handed candidates a snippet and asked "what does this do?" or "what is wrong with this?" — but now that AI tools can generate plausible-looking C++, the ability to *read* code critically matters more than ever.

I also added coverage of C++23 and C++26 features — `std::expected`, `std::print`, deducing this, pack indexing, contracts, reflection, and more. These are distributed across the relevant topical parts rather than isolated in their own chapter.

## What AI changed about C++ interviews

I wrote about this in the *Introduction*, but the short version: rote recall of syntax has become less valuable. If you can look up the signature of `std::rotate` in two seconds during your normal work, an interviewer gains little by asking you to recite it. What they *do* want to know is whether you understand *why* you would reach for it, what the complexity implications are, and what happens to iterators when you call it.

The "what" has become cheaper. The "why" and "when" have become more expensive.

This book is built for that shift. Every question aims to give you not just the answer but the reasoning — the kind of understanding that survives follow-up questions and does not depend on having a tool open in another tab.

## Who this book is for

The same audience as before: C++ developers preparing for interviews, whether junior or senior. It is also useful as a reference if you want to deepen your understanding of a specific area of C++ without the interview framing.

If you bought the first edition, I think you will find this one worth the upgrade. It is not the same book with a new cover — it is a substantially different book that happens to share some of the same questions.

## Where to get it

The book is available now on [Leanpub](https://leanpub.com/cppinterviewbook/c/LAUNCH). You get DRM-free PDF, EPUB, and MOBI, plus free updates.

For the first two weeks the minimum price is **$9.99** (regular minimum is $14.99). If you have been thinking about it, this is the best time.

If you bought *Daily C++ Interview*, check your email — I have sent you a coupon for **$4.99** or [just this one](https://leanpub.com/cppinterviewbook/c/UPGRADE).You already supported this book once; I want the upgrade to feel like a no-brainer.

A Kindle edition will follow shortly.

## Thank you

To everyone who bought the first edition, sent feedback, pointed out mistakes, or told me the book helped them land a job — thank you. This edition exists because of you.

{% include connect-deeper.html %}
