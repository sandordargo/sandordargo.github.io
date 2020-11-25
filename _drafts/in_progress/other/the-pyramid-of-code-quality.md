## What is sofware quality?

The first question raises when we talk about quality software is about its definition.

I remember we had a discussion with my team about two years ago about this question. There is only a few people in the department now from these people. Me and Christian were relatively new, him having arrived a few months before me and I accepted to join the team, because I knew whatever crap I can find in the code, he is a promise that they want to make things better.

And there was crap in the code.

While we were discussing about our engagement to the team, to the company, we had to discuss the question of how much our colleagues are dedicated to quality. Khm... It didn't use to be that team.

Let's stop for a second.

I don't think that they were bad people. They joined the team young, there was an awful management and they didn't have the courage yet to stand up for themselves and for their work.

That changed with having the manager changed.

### The client centric approach

Some argued that software quality is OK as long as it does what the client wants and the client is happy.

This is a very client-centric approach which might be valid. After all, the software exists for the clients.

But the question is, does te software do what the clients want if there are hundreds of tickets open? Can they be happy or they just used to accept such long queues?

But I think, client satisfaction must be part of the definition of quality.

### Thinking about the maintenance costs

Building a software costs money. Keeping it afloat costs even more in the long run. Probably even for stable high-quality ones too. Think about the different framework/library upgrades, security patches, changing regulations, keeping some people skilled on the system in case of troubles.

And in most cases there will be troubles.

All these costs money. Agile and shift-left taught us that the later we fix an issue, the more expensive it will be. One thing is that you must follow a full cycle or more, the other thing is that probably the people who built that part won't be around anymore. People will have to relearn things and here readability strikes back.

And we shall mention, Hyrum's law. Your users will take any exposed behaviour granted. Now let's suppose that you expose a buggy behaviour. It will be very costly to remove it.

So we can also say that the quality of a software is proportional to the ratio generated value/maintenance cost.
D=(Pv∗Vi)/(Ei+Em)


Pv: Probability of Value
Vi: Implementation value
Ei: Implementation Efforts
Em: Maintenance Efforts 

The Desirability of Implementation is directly proportional to the Probability of Value and the Potential Value of Implementation, and inversely proportional to the total effort, consisting of the Effort of Implementation plus the Effort of Maintenance.

### Reactivity

How fast can we adopt? How fast can we fix problems? How fast we can react to new customer needs?

These kind of things usually don't have a human bottle neck. There can be, but usually that's not the main problem. It's much more about the difficulty to understand the code, to understand where a fix should be mode, the high coupling. When you fix a problem somewhere and seeminglt completely unrelated things break in the system.

### Attractivity

There is - at least - one more thing to think about. How attractive your software is for developers. Do they ran away from your teams/codebase or you can have a fair selection among applicants.

While this is a consequence of many of the previous aspects and could be the topic of a talk itself, that's something that can also help measuring.

Of course, you have to think about this wisely. Just using the latest JS FW might make you attractive among some of the applications, but it doesn't increase the quality of your software.

Can we attarct ppl?
Attractivity will not help you increasing your software quality, but your software's quality will increase (or the lack will decrease) your attractivity.

So if you want to measure your attractivty as a function of quality, think about it a consequence, not as a cause.

## Conclusion

In this first section, we discussed about what is software quality and what kind of different meanings it can have. Due to the lack of time and space we didn't talk about direct measurements or detailed existing studies. We saw that sodtware that is considered high quality might not have high quality code. I personally find that it's not the right approach, because because a software that is easily maintainable that can adapt qiuickly to changes.

In the next episode, we will talk about the different layers adding up to software quality. Stay tuned.