How do you prepare your commit?

Or I could also ask what git commands precede a git commit for you? If your answer is none and you just usually do a `git commit -am "some changes"` then this article is most probably not for you.

I love code reviews and I'm a bit sad if I don't get comments, because it means that people didn't think hard enough about it. On the other hand, I don't like to get comments about the obvious. Names not in line with our standards, constness, missing references, left in debug logs -  formatting is not an issue -, I try to avoid such comments.

How do I do that? I make my own code review before I commit?

I do a `git diff myFile`, then I do review what changed, I fix if I do something and I do it again.

Once I find everything acceptable, I do a `girt add myFile`.

I do this for small to medium-sized, not scripted changes.

Does that sound simple?

It is simple.

Let's go a bit lower level.

What do I do after I verified that myFile is in a stagable state?

I hit the up arrow, so I have `git diff myFile` on the line, with the cursor at the end.
Then I hit ctrl+A, so my cursor is at position 0.
Then I hit alt+F to move my cursor one word forward.
I hit right arrow to have my cursor at `d` of `diff`.
I type ad, so I have git `addiff myFile`
I move on more to the right, so my cursor is at _i_ of addiff.
Now I hit on the delete button three times. Et voila.

Is that easy? Yes. Is that simple? Well. It's boring, it's cumbersome.

Of course, there are other options. If you have the filname on the clipboard you can type git add and then you paste. That's also an option. It doesn't change much.

I was wondering how cool would it be if git diff would ask back if I want to stage the file.

After some more times going through the same processes, I decided to implement it. Or to be more exact, to implement an alias doing it.

I was not sure if I'd have to write a shell function or if I can create a git alias covering my use case.

I quickly learnt that




I do my own first round codr reviws

git diff myFile
git add myFile

Unless it's a scripted change

Is it tedious? Well.. How to make the secnond line out of the first..

ctrl+a///

I started to fucking hate.

