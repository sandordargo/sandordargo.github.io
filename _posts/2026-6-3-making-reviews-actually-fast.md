---
layout: post
title: "Keeping Code Reviews From Dragging"
date: 2026-6-3
category: dev
tags: [code-review, career, watercooler]
excerpt_separator: <!--more-->
---
You know the feeling. You open a pull request on Monday morning. You ping the reviewer(s). You go to lunch. You come back. Nothing. You context-switch to something else. On Wednesday, the reviewer finally leaves a comment — a single one, on a minor detail. You fix it. You wait again. By Friday, the PR is still open, the branch is conflicting with master, and you've forgotten half of what ~~the agent~~ you wrote.

I've been on both sides of that scenario, like most of us. More times than I'd like to admit...

When I gave my talk on code reviews at Meeting C++, [one of the attendees wrote a thoughtful reaction afterwards](https://guillaumedua.github.io/publications/2026/03/13/meeting-cpp-2025-trip-report/). He agreed with most of what I said, but pointed out that I had under-served one important part: the real-world problem of reviews dragging. I had said that PRs should be reviewed promptly and shouldn't linger for days — but I hadn't said much about how to make that happen, especially when junior contributors, unclear guidelines, or just plain over-stretched teams turn each review into a multi-day ping-pong match. A 60-minute talk has its limit, but it's certainly worth talking about this problem.

## The Cost Nobody Measures

Slow reviews don't just slow down delivery. They quietly tax everything around them.
The author loses context. By the time the first round of feedback arrives, they've moved on mentally — and now they have to swap back in, find the right state of mind, and try to remember why they made the decisions they made. That swap costs more than people realize.

The reviewer loses context too. Coming back to a PR three days after the author wrote it means re-reading the description, reloading the change, and re-deriving the intent. The second pass through a PR is almost always shallower than the first.

The team loses momentum. A PR that sits open for a week eats merge conflicts, blocks dependent work, and sends a quiet signal that this is just how things are around here. The longer it stays open, the more the next PR copies its pace.

And then there's the new dynamic nobody had to think about a few years ago: AI changed the supply side. Generating a PR is now cheap. Reviewing one isn't. The team that used to produce five PRs a day is producing twelve, and external contributors send another handful on top. Yet, the review capacity hasn't moved. If your reviews were just barely keeping up before, you're underwater now — and the queue grows faster than you can drain it.

And the manager — the one watching the delivery dashboard — concludes that reviews are slowing us down — which is half true. A review queue that drags slows us down. A review queue that moves doesn't.

## What Actually Helps

I don't think there's a silver bullet. But there are a few patterns that I've seen work — across teams, across companies, across languages. None of them are revolutionary. They're just the things that the fast teams actually do.

### Review First, Write Second

This is the single biggest lever, and it's also the hardest one to pull. The idea is simple: when you sit down to work, the first thing you do is clear your review queue. Only then do you start writing new code.

Most teams operate in the opposite mode. You start the day writing your own thing, and you'll get to reviews *"when you have a moment"*. The problem is that the moment doesn't come. By the time you remember the open PRs in your queue, half a day has passed and the author has already context-switched away.

The team that reviews first ships faster than the team that writes first.

It's not intuitive, but it works. Every minute spent unblocking a teammate's PR pays back several minutes of their productivity. And the moment a team adopts this norm, the average PR lifetime drops noticeably.

This only works though when the team agrees. If you're trying to enforce it alone, while the rest of your team writes first, you're going to burn out before the culture shifts. Start by getting buy-in. Then you do what you agreed on, you show the way. Lead devs, this is on you — your team copies what you do, not what you say.

### Have a Playbook for When a PR Drags

Sometimes a PR drags despite everyone's best intentions. Here's the playbook I try to follow:

- Day 1: Reviewer leaves a first pass within the working day. Even a partial review. Even just "I'm on it, will come back this afternoon" or just a 👀 emoji.  The author needs to know the PR has been seen.
- Day 2: If the PR is still going back and forth, the author and reviewer have a 10-minute call. Written ping-pong is a smell. Two or three rounds of comments on the same thread usually means there's a misunderstanding that comments can't fix.
- Day 3: Escalate to the team. Is this PR too big? Is the scope unclear? Are the requirements off? Bring it up at standup. The whole team should know there's a PR stuck.
- Day 4 and beyond: Something is wrong with the process, not the PR. Close it, redesign it, or split it.

A PR that lives longer than three days is rarely a code problem.

That last point matters. When a review takes a week, it's almost never because the code is hard. It's because the requirements are unclear, the PR is too big, the author and reviewer are on different pages, or someone is avoiding a conversation. Reviews are the symptom; the root cause always lies deeper.

### Break the Ping-Pong Cycle Early

Some PRs don't drag because nobody's looking — they drag because the same comments keep coming back. The reviewer flags something, the author addresses it, the reviewer flags something adjacent, the author addresses that, and round and round we go.

This pattern is especially common with junior contributors, or with anyone who's still learning the team's unwritten rules. And when it happens, the worst thing you can do is leave another comment.

Here's the rule I try to apply: three round-trips on the same PR? Stop reviewing. Start talking.

A 15-minute pairing session beats six more review rounds. You'll resolve the misunderstanding in five minutes, learn what the author was actually trying to do, and probably discover that the comments you've been leaving aren't really about the code at all — they're about a shared mental model that hasn't been built yet.

And if the same patterns keep coming up with the same contributor? That's a coaching opportunity, not a code review opportunity. Turn the recurring feedback into a real conversation: *"I keep flagging this; let me show you why, and let's talk about how to spot it yourself next time."* That conversation saves dozens of future review rounds.

If you're explaining the same thing twice, the review isn't the right format anymore.

## AI Changed the Math

Most of what I've written above applies whether or not your team uses AI. But there are a few patterns that have only really shown up in the last couple of years, and they deserve naming.

### AI-authored Code Needs a Different Review

When the author of a PR didn't write every line — when they accepted suggestions, refactored a Copilot-generated function, or asked a model to *"make this cleaner"* — the usual review questions don't quite fit. *"Why did you do it this way?"* used to be a nudge; now it's a real question, and the honest answer is sometimes *"I'm not sure, the model suggested it."* That's not a moral failing. It's a sign that the author needs to slow down and verify before asking for review, and that the reviewer needs to spend more time on intent than on implementation.

The shortcut I use: if the author can't defend a choice, the code isn't ready. That applies whether the choice came from the model, from a tutorial they half-remembered, or from a senior engineer they didn't fully understand. The author doesn't have to have written every line — but they have to own every line.

### AI Reviewers Are Part of the Queue, Not Separate from It

Most teams now have a bot reviewing every PR. And most of what those bots produce is noise: nits the team doesn't care about, style suggestions that contradict the codebase's conventions, *"potential issues"** that aren't issues.

The danger isn't the noise itself — it's what the noise does to the reviewer. A reviewer who learns to skim past 70% of comments will eventually skim past the 30% that mattered. Alert fatigue is real, and bots are the fastest way to manufacture it.

The fix is to treat bot comments the way you'd treat any unreliable reviewer: triage first, then engage. If your bot is producing more noise than signal, tune it, mute it, or turn it off entirely until you can. A reviewer that comments on everything is a reviewer nobody reads.

### The Human Comments Matter More, Not Less

Counter-intuitively, the more AI participates in code review, the more important the human comments become — because the human comments are the ones with context, judgment, and the willingness to ask questions a bot can't. If you find yourself writing the kind of comment a linter could have written, you're spending your review energy in the wrong place.

## The Conversation With Your Manager

There's one more piece worth saying out loud, because it's the part lead devs have to deal with most often.

Many managers and stakeholders still view code reviews as something that slows down delivery. They see the queue, they see the latency, and they conclude that the team would ship more if it reviewed less. As a lead, part of your job is to help them understand why this work is essential — and that's not always easy.

The truth is that the value of thorough reviews, like the value of good tests, careful error handling, or solid design, mostly only becomes obvious when things go wrong. Nobody throws you a parade for the production incident that didn't happen. But the codebase that didn't slide into a state where every change costs three times what it should — that codebase is the one that pays you back, quietly, for years.

If you find yourself having to make this case, I'd suggest framing it the way the commenter on my talk did: thorough reviews prevent the codebase from sliding toward a point of no return, where changes become prohibitively expensive — or nearly impossible. That framing tends to land better than *"reviews catch bugs"*, because it speaks to a cost trajectory rather than a single line item.

## Conclusion

Code reviews don't have to slow you down. They slow you down when they drag — when the queue builds up, when PRs live for a week, when the same comments keep bouncing back and forth. The fix isn't to do fewer reviews. The fix is to do them faster, more deliberately, and with a willingness to switch formats when the review format itself isn't working anymore.

If you only remember three moves, let them be these:

| Pattern | Move |
|---|---|
| New code waiting to be written | Review the queue first |
| PR dragging past day 2 | Have a call, not more comments |
| Same feedback for the third time | Switch from review to coaching |

And keep an eye on the bots. A noisy reviewer — human or not — trains everyone to stop reading.

None of these are revolutionary. But together they make the difference between a team where reviews are a small, predictable cost and a team where reviews are the bottleneck everyone complains about. You can't fix slow reviews with a Slack bot — but you can change how your team thinks about reviewing, and the moment that shifts, everything downstream of it gets faster.

If you've found patterns that work on your team, I'd love to hear them.

{% include connect-deeper.html %}
