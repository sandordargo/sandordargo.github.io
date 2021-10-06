---
layout: post
title: "Bring some fresh AIR and write effective code review comments"
date: 2021-10-6
category: dev
tags: [codereview, teaching, knowledgesharing]
excerpt_separator: <!--more-->
---
A few years ago, I shared some [guidelines about how not to ruin the team mojo with code reviews](https://www.sandordargo.com/blog/2018/03/28/codereview-guidelines), what practices should the different involved people follow to avoid feeling bad about each other, yet to fulfil the aims of a code review.
<!--more-->

It's time to talk about how to write comments that won't be neglected but will be taken into account. These practices will not work all the time, but they will help someone with a higher chance. Personally, I haven't worked with anyone who'd reject ideas based on gender, skin or whether you call a *pain au chocolat* a *chocolatine*, I'm sure there are such people out there. It's enough to "open the internet" to see examples from whatever groups.

Jerks are jerks.

Yet, as a stoic, you shouldn't care about them. At least you should try to care less.

If my comment doesn't achieve its purpose, the question I should ask myself is what I should have done better and why it bothers me if my point was neglected.

I think that a good code review comment consists of 3 parts:

- Action
- Information
- Reference

Adding these three items and staying polite will make sure that there is no smell of resentment in the AIR, at least not because of code reviews.

Let's take a very simple, short comment and I'll show you how to transform it into something AIRy.

*(Not all comments need these 3 elements. Sometimes you should really just ask for some info or highlight a typo.)*

**Code:**
```cpp
void doStuff(const std::unique_ptr<Widet>& iWidgetToUpdate) {
  // ...
}
```
**Comment:** *"Why don't you take it by value?"*

## Add a clear **action**

There are many problems with the above comment. One might interpret it as a bit passive-aggressive and besides it definitely lacks a clear action.

If you want the author to perform an action, ask for it.

So many relationships are broken or much worse than they could be simply because we fail to communicate what we want. You might expect that someone else does something - while you should not expect anything from others... - yet the other fails to get the message. Or simply he doesn't want to do it if it's not explicitly asked for.

So let's transform the above comment into something *actionable*:

> "Please, take the smart pointer parameter by value instead of a const&."

Note the word *"please"*. It makes wonders, yet we forget to use it so often.

## Add information

If I received the above comment, I would ask for an explanation. Okay, but why? I think that's the right attitude as someone whose code is being reviewed. But maybe the author is too shy to ask back or simply is too overwhelmed with the comments or with work, maybe with life in general.

As a reviewer, you should not be waiting and rubbing your palms until the author asks back. Don't have such expectations and don't be surprised if the author
- ignores your comment
- asks someone else as well

Even if you're a *C++ guru* or even a *C++ maven*, don't expect others to comply with your comments without giving more information.

So how would a better comment look like?

> "Please, take the smart pointer parameter by value instead of a const&.
> If you pass them by reference, you don't pass or share ownership. If you don't want to deal with ownership, prefer passing a raw pointer or a reference. If you do want to share the ownership, you must pass by value." 

Are we good enough already?

Not yet.

## Add references

Even if you're a major authority in the given domain, you shouldn't expect people to take your comments blindly without any proof.

The above comment is already not bad, but it lacks any proof or reference. If I see such a comment, I either look for a reference myself if I have the time, or I might ask back to include some references, or I ask another also experienced colleague. After all, if two people say the same, maybe it's worth taking the advice.

In fact, if I don't share references, I'd expect the others to ask back or to ask for confirmation from others. But once again, having expectations towards others' behaviour is not wise.

Either way, you cannot blame people who look for confirmation even if it hurts your ego.

Let's say get a big stack of cash and it was just counted in front of you. will you just take it, or will you count it? Of course, you will count it. You will count it until you two get the same result twice.

So to complete the AIRy code review, let's add some references.

> "Please, take the smart pointer parameter by value instead of a const&.
> If you pass them by reference, you don't pass or share ownership. If you don't want to deal with ownership, prefer passing a raw pointer or a reference. If you do want to share the ownership, you must pass by value.
> For more details, pleaser refer to
> - [GotW #91 Solution: Smart Pointer Parameters](https://herbsutter.com/2013/06/05/gotw-91-solution-smart-pointer-parameters/)"
> - [const and smart pointers](https://www.sandordargo.com/blog/2021/07/21/const-and-smart-pointers)"

Oh by the way, if you add only one reference, probably it shouldn't be your article, unless... No, there is no unless.

## Conclusion

Imagine that you live in a world where code review comments always bring some fresh AIR: they bring *action*, *information* and *reference*.

Sometimes such a world might seem far away, but you can make it closer. Start using this technique to comment on non-basic mistakes, non-typos.

With such comments, you will not simply ask for a change, but you'll teach, probably also learn by reviewing why you ask for something and you'll even share some resources that later others can use.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
