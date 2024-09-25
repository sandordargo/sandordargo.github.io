---
layout: post
title: "So she we use static or dynamic linking?"
date: 2024-X-X
category: dev
tags: [cpp, builds, staticlinking, dynamiclinking]
excerpt_separator: <!--more-->
---
Last week, we were talking about static vs dynamic linking from a binary size point of view. Around the end of the article I wrote that I omitted other aspects.

Now let's talk briefly about some of those omitted aspects. 

For - quite - some time in the beginning of my career, I simply thought that dynamic linking is better. After all, if something is dynamic, it's better than being static, right?! Honestly, I didn't even give too much thought to this question.

The answer is not so simple.

We've already seen that in many scenrios, static linking will decrease the binary size. But that's not the only aspect where static linking might help you.

## Easier deployment with static libraries

With static linkage, deployment is easier. The reason is that you ship everything or at least almost everything that you need to run your executable. It'll work out of the box, maybe you need to have system libraries in place, but that's it.

On the other hand, with dynamic linking, you have to make sure that all the necessary libraries are also available in the right version on the system where you want to execute your binary.

## Slightly easier maintainability with dynamic libraries

While deployment is definitely easier with statically linked libraries, maintainability is another question. When you discover a bug in one of the linked libraries, with the statically linked version, you'll have to recompile and bundle up everything. Essentially, you have to ship the whole application again.

On the other hand, when you dynamically link libraries, you have to redeliver only the part that contains the bug. If it's in one of the libraries you use, it might be enough to deliver one library and not to care about the rest, not to recpmpile the whole application. 

At the same time, with dynamically linked libraries, you have to make sure that the upgraded libraries are compatible and safe to use. When you link libraries statically, it's easier to test that your application will be fine and the libraries don't contain anything malicious and they are compatible. With dynamically linked libraries, ensuring security and stability takes an extra effort.

## Performance

Anything that is performance-related is worth measuring. I'm not saying that you should take measurements from a blog post, on the contrary, you should measure performance in the environment you're deploying to, with the binaries you want to execute. You won't always observe the same effects that others wrote about, the effects that you expected.

With that being sad, executables which link binaries statically might have a performance advantage over the dynamically linked version. The reason behind is that it avoids the overhead of loading and resolving the library symbols at run time. That overhead includes both time for the actual loading and memory needed by the dynamic loader.

But again, it's worth measuring in your own environment, besides, maybe the performance penalty you have to pay is acceptable in your usecase.

## Extensibililthy 

This is going to be short and simple. If you need to use some plugins or extensions at runtime, you don't have a choice, you must go with dynamic linking.

## Conclusion

Last week we discussed static vs dynamic linking from a binary size perspective. This week, we listed the other aspects. We've seen that deployment is easier with static linking. Maintainability is a more complex question. When you use dynamic linking, you can fix some bugs only by updating a library version, but you have to take extra care to use the correct versions.

When it comes to performance, you'll probably be better off with static linkage, though you should measure whether or not it matters in your case.

Don't forget, when you have to use runtime extensions or plugins, dynamic linking is your only choice. 

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!