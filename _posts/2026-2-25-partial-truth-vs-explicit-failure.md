---
layout: post
title: "Partial Truth vs Explicit Failure: Designing Honest System Responses"
date: 2026-2-25
category: dev
tags: [error handling, api design]
excerpt_separator: <!--more-->
---
Systems often face partial failure, yet our interfaces force us into a binary choice: success or error. Treating partial truth as full success hides uncertainty — and is often more dangerous than returning an explicit failure, even though reality usually lies somewhere between these two extremes.

Let's admit it: real-world systems rarely fail cleanly. There are many ways things can go *partially* wrong. Dependencies time out, caches go stale or fail to synchronize, subsystems lag behind or respond slower than expected.

Yet APIs tend to present a binary world. Not because it reflects reality, but because it is convenient. It is measurable. And, perhaps most importantly, it is familiar to all of us.

Let's get more specific.

## A partial state snapshot

Imagine an API that returns the state of a running service. Clients consume this state and make decisions based on the data they receive.

The active operation is known, but upcoming operations are based on the returned state. The state depends on several subsystems. What should happen if one of those subsystems is temporarily unavailable? Perhaps it is simply slow.

In such a situation, the service might still be able to produce a valid state usable to determine an upcoming operation — but not the optimal one. Not the one that would be produced if all subsystems responded successfully.

At this point, there are two choices in front of us:

- Return what is known and *generously* omit the rest.
- Return an error indicating that the state is incomplete.

A slower response does not fundamentally change the problem — it merely postpones the same decision. And combining both approaches often just shifts responsibility to the caller without making the uncertainty explicit.

## What each option really communicates

A successful response implicitly claims completeness. It does not reveal internal issues. It does not tell the caller that the response is suboptimal. It simply *pretends certainty*.

An error, on the other hand, admits that there was a problem. It communicates a limit: *this system does not know enough right now to answer safely*.

Formally, an API often only defines a data format. In practice, it defines much more. Sometimes it even specifies invariants explicitly (think about C++26 contracts), but there is always an informal part as well — usually documentation or conventions that establish a trust contract between producer and consumer.

That trust contract determines what callers believe they can safely assume.

## Where can "best effort" responses lead

When "best effort" responses are returned as successful ones, they can harm the system on multiple levels.

Downstream systems may make very confident — yet incorrect — assumptions. Their behavior might be suboptimal, but the degradation will be silent and inconsistent, because those systems are unaware that the data they received was incomplete.

Users are not the only ones affected. Observability also suffers, as partial failures are masked as successes. Monitoring stays green while correctness quietly erodes.

## When partial responses are the right choice

As I often say, life is not black or white. It is nuanced — and so is engineering. It would be wrong to claim that partial responses are always bad. If they were, this article would not exist.

Partial responses *can* be the right choice, but only if it is clearly indicated that something is missing.

Callers must always be able to distinguish between *missing* and *empty* data.

It is similar to being honest in an interview. Instead of trying to elegantly skip a question you cannot answer, you admit that you do not know — and then you share what you *do* know. That honesty builds trust.

In API responses, unknown states should be explicit.

On a lower level, think about the difference between `optional<bool>` or `optional<string>`. `false` or an empty string clearly means something different from `std::nullopt`. *No* and *I don't know* are not the same — even if humans sometimes try to disguise one as the other.

## Conclusion

Engineering is not only about the happy path and making successful responses as fast and accurate as possible. We must constantly think about what can go wrong — and how to respond when it does.

When a system fails only partially, it is tempting to compensate for that failure and return a successful "best effort" answer. Marking such a response as successful may feel helpful in the short term, but it quietly lays the groundwork for downstream problems.

This does not mean that partial responses are always wrong. Sometimes they are exactly the right choice. But when we choose them, we must be honest about what is missing and what is merely empty.

How do you deal with missing data in your responses?

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!

