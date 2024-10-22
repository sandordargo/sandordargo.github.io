I've been taking part of coding dojos either as a participant or a facilitator for the last more or less 10 years with some smaller gaps around reorganizations. Coding dojos helped me turn around my career completely. But that's not what I'm going to write about today. (some reference to the seniority trap)

As a facilitator I sometimes drive a session very closely, but sometimes I just sit and listen and let the others go in the direction they want until I join back again more actively to ask some questions about the decisions they made.

We recently solved the string calculator kata. At the dojos I facilitite, we follow TDD, we do the red-green-refactor cycle. First we write a test that doesn't pass, then we implement it with the least amount of logic and then we ask ourselves the question if we want to refactor the code anyhow. Sometimes there is nothing to do, sometimes people want to do some more work refactoring.

This time We already solved about half of the exercises and when the support for custom delimiters, and at that point, people really wanted to refacotr. I also wanted to refactor. I did this kata way too many times and I didn't want to push my ideas, I was listening what others do.

We had one big `add` function taking care of identifying the dlimiter, splitting the string and summing the different numebrs in it, sometimes with a bit of entangled logic.

And the others started discussing refactoring in which they wanted to eliminate string allocations. Let's ignore the fact that with small string optimization the vast majority of numbers will not need a heap allocation, everything will be on the stack. It'll be relatively fast. It's still some memory allocation that we might consider unnecessary.

But why would we want to perfrom micro optimizations at that point? Why would we want to improve performance? What is performance?

As we wrote a few weeks ago in [the different aspects of performance], *"Performance is a non-functional characteristic of a software. It's almost always a non-functional requirement as well. Even if it's not specifically mentioned, a software that is too slow - in a given context - is unusable and useless. In that sense, performance is a non-functional requirement that ensures that the software can achieve its functional requirements in a reasonable timespan."*

Considering that why would we want to improve the performance of the string calculator when it has a reasonable performance, we know nothing about the non-functional requirements, but we have a growing and more and more complex logic?

People said, well, because that's what C and C++ developers want to always do. Improve performance just for the sake of improvement.

While, I don't think we can generalize in such a way, I also think that's a wrong attitude and won't help your career unless you're in a niche where you must squeeze out even the last bit of performance.

In my experience, such attitude is often not the answer to performance problems.

I remember my very first project as a developer. Oh boy, it was so mismanaged. I was left alone I think for months to work on it and then people were surprised during the code review that it was shit. Nevertheless, it fulfilled it's functional requirements. But it was slow as a snail and suffered from a vaste memory leak.

There was another developer there who recently started out but he was clearly more gifted as a developer then me. He was freaked out and very qucikly claimed to find a culprit. It was 2 functions with the complexity of O(n^4). Yes, it was that bad. Probably it could have been replaced with a standard function, probably with a complexity around O(n * log n). I think I was also ridiculed a bit, but that's not the point. When he faced me with this bad algo, I clearly realized that yes, it's bad. I also knew that it cannot be the problem, but I didn't know how to prove it.

What did I do? I opened a notepad and started to jot down timestamps from our server logs to find out how long a serving a request took and how much time of it was about these two horrible functions. Not surbrusngly, they represent a tiny fraction of the time.

Our problems lied somewhere else.

As it turned out weeks later it was that we (most probably I) misused a third party library handling XMLs documents. A startup and a cleanup function should have been called when the service was started/finished, but they get called in the beginning and the end of each request. Recognizing that solved all our immediate problems.

Years later as a bit more experienced engineer, I had to develop a tool that helped migrating data from one database to another. It was not meant to be used for a long time but as it turned out they had quite some performance requirements.

Around those times I got familiarized with Clean Code practices and I decided that that piece of software will serve as a showcase (for me) what happens I follow all those rules in the book. (Since then my definitionof clean cdoe changed a lot). Well, the software was slow. At least slow compared to what was expected. By an order of magnitude.

I made this a bit faster here and there by optimizing the code on a low level, by changing some data structures, but it was very far from being enough.

In the end, I achieved the expected levels of performance and everyone was happy. I succeeded by employong two techniques. Instead of reading rows from the database to be migrated one by one, I read several rows at the same time and kept the data in memory until I had to use it

In order to insert the data into the new database, I couldn't access the database directly, but I had to use the CRUD services of a server. It turned out that the server offered some batch services where I could insert several rows at the same time.

These "tricks" made the magic. By reading and writing several rows at the same time, in other words, by limiting the number of DB and network calls I had to make, I could spped up the code by about an order of magnitude.

The last example I brought is about when at my previous eployer, we had to support a growth in traffic of about 15 times. We did some low level changes, but it was not about optimization per se. We removed undefined behaviour, we removed bugs that led to cracshes. Otherwise, we improved network settings, introduced in-memory caches instead of going to the database so many times, introduced more read-only views at the database and in general reduced the number of DB connections we opened.

These together were enough. You can raed more about it [here]().


## Conclusion

What did I want to say with all these little engineering stories? What I tried to say that in many realülife engineering problems, low level code optimization is not the answer. Real life engineering problems is rarely like leet code issues. If you have to interact over the network, oy you hvae a database to deal with, you ahve to ensure those are handled efficiently.

Everything else only comes after.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!