---
layout: post
title: "What can you use for visualizing a graph?"
date: 2017-8-24
category: dev
tags: [ui, graphs, javascript]
header: "As I <a href=\"/blog/2017/07/11/meet-neo4j-again\">wrote recently</a> I have in mind a pet project using graph databases with a dataset about Hungarian wines. I'm quite comfortable with the backend part such as handling the database and manipulating the the data I retrieve, but I cannot say the same about the frontend part."
---
So I had to explore what technology I could use for visualizing my data. When I was building my [family tree](https://github.com/sandordargo/family-tree) application I used [sigma.js](http://sigmajs.org/) but looking back at it, I didn't find it very intuitive. Maybe because of my poor JS skills. Anyway, I decided to look around what is available to use. I started my exploration from [Neo4j's data visualization guide](https://neo4j.com/developer/guide-data-visualization/). 

My first choice was [Alchemy.js](http://graphalchemist.github.io/Alchemy/#/). At a first glance the code examples seemed quite easy to understand and I liked that there is some documentation. And in the end if it is listed on Neo4j's site, can it be that bad? Well. For me it did not work out. I faced the first problems when I wanted to include in my `html` files the library with a URL. The URL was broken. Okay... So I downloaded the library and copied the source code into my project. Not a big deal.

After some research it turned out that I had to include something else. If I remember well, it was [d3.js](https://d3js.org/), nothing more. But I'm not if it was the only library.

Finally I managed to plot a nice graph with some nodes representing Hungarian wine regions and sub-regions. It was nice.

The next set of problems found me when I wanted to add some interactivity to the graph. Long story short, the project seems a bit abandoned and the documentation is outdated. The code doesn't do what the docs say and even the API changed. I kept quite some time hacking the source code in javascript - do you remember that I don't know javascript? - and finally I concluded that this is just not going to work out.

However at the dependencies I met with [d3.js](https://d3js.org/) which was also mentioned in the [guide](https://neo4j.com/developer/guide-data-visualization/) and at some other places. It was also mentioned that it'd be quite difficult to use for a newbie as me as you have to take care of a lot of configurations manually.

It's said that taking the upper road is always beneficial. So I took it. Within less time and with less frustration I reached the same point as with Alchemy before. But it was mostly copy-pasting some parts from here and there and welding them together. Still - or more hence - there are some parts which are not working well. Such as gravity and scaling. Maybe they are not that important at the first iteration. But still...

So I decided to learn [d3.js v4](https://d3js.org/) better. However I don't want to spend weeks on reading books and learning javascript from scrach. The same time, I have been reading John Sonmez's book called [Soft Skills:The software developer's life manual](http://amzn.to/2wK0lLI). In it he lists some learning techniques which helped him to produce a lot of educational material quite fast. I decided to give a try to his techniques. I'm not sure if I could briefly conclude them as he is selling them in a form of a course and a book as well, so I rather not go into details.

The main idea is that at any point you learn the least necessary you need in order to make something work and then you start using it. Now I'm learning about data bindings and putting it into place. Then I move to stylings and transformations. In a few weeks I will get into a position where I will be in control of my UI and not the examples I find on the internet.

I will also post my findings about [d3.js](https://d3js.org/) and I will blog more about [Neo4j](https://neo4j.com/) as I plan to deepen my knowledge in it. Stay tuned.
