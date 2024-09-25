---
layout: post
title: "What is software performance?"
date: 2024-X-X
category: dev
tags: [cpp, binarysizes, staticlinking, dynamiclinking]
excerpt_separator: <!--more-->
---
Though the main topic of this blog is not clean code or *Clean Code*, we discuss from time to time patterns that will lead to cleaner, more understandable, more maintainable code.

Sometimes we even talk about optimization. Though its about [optimizing for binary size]() and not for runtime performance - what we usually mean by optimization.

There are claims that clean code is the enemy of performant software. It's certainly true that clean code is not optimized code, at least not optimized for performance. Some go further saying that clean code makes discards decades of hardware improvement.

We are not going to discuss that claim today. We are going to answer a more fundamental question.

*What is software performance after all? What are its different aspects?*

Performance is a non-functional characteristic of a software. It's almost always a non-functional requirement as well. Even if it's not specifically mentioned, a software that is too slow - in a given context - is unusable and useless. In that sense, performance is a non-functional requirement that ensures that the software can achieve its functional requirements in a reasonable timespan.

We can think about software performance as how efficiently the software can accomplish its tasks. But it's still nothing specific. There are many several ways to describe and measure performance and efficiency.

- **Response time** is the amount of time it takes for the software to respond to a user's request. This is crucial for user satisfaction, especially in interactive applications like websites or mobile apps. Many companies measure how many users abandon their product in case of a slight slowdown in their software.
- **Throughput** is the amount of work the software can handle within a given time frame. This is particularly important in systems that need to process a large number of transactions or requests, such as servers or databases.
- **Scalability** is the software's ability to maintain performance levels as the workload increases. A scalable system can handle more users, data, or transactions without a significant drop in performance.
- **Resource utilization** is about how efficiently the software uses system resources like CPU, memory, disk space, and network bandwidth. Efficient resource utilization helps ensure that the software runs smoothly without overloading the system.
- **Stability** is the software's ability to perform consistently over time without crashes, memory leaks, or other issues. Stability is essential for ensuring reliable and continuous operation.
- **Capacity** is the maximum load or number of users the software can support before performance degrades to an unacceptable level.

If we think about these aspects, we can see that one barely exists about the other. They are often highly related. Is it possible to achieve a high throughput without a scalable sysytem? Without high resource utilization? It is, but it's unlikely. Also high throghput and capacity is related, although there are differences.

We have to think in systems, we have to think about software performance as a whole, but these are different aspects that help us think about it. At the same time, they also help us define different aspects we might want to focus on.

## Conclustion

Software performance is a non-functional requirement ensuring that the software meets its functional requirements and users get a good experience while the software remains relatively easy to operate. Response time, throughput, scalability, resource utilisation, stability and capacity are important aspects of software perfoamnce which are highly related, yet they provide different ways to analyse and improve software performance depending on the requirements.

## Connect deeper

If you liked this article, please 
- hit on the like button,  
- [subscribe to my newsletter](http://eepurl.com/gvcv1j) 
- and let's connect on [Twitter](https://twitter.com/SandorDargo)!
