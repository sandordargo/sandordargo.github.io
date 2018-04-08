---
layout: post
title: "Graphs in everyday life"
date: 2017-9-20
category: other
tags: [graphs]
header: "Let's see how graphs appear around us in our everyday life. Start with a recap what a graph is."
---

![Test Data new wine services]({{ site.baseurl }}/assets/img/graphs-in-everyday-life-excel-chart.png)

No! That's a chart, not a graph! Charts are not graphs, even though for some odd reason many people tend to call them so.

In mathematics and more specifiaclly in graph theory a graph is a structure of related elements. The elements are called nodes (or points) and the relationships between them are called edges (or vertices).

Edges can be directed or undirected. Imagine such a structure with nodes and edges about a corporation. If Jim is a team member and his boss is Joe, we can imagine a relationship between them `MANAGES`. This direction is tipically a directed one. It's only Joe who manages Jim, not the other way around.

![Joe manages Jim]({{ site.baseurl }}/assets/img/graphs-in-everyday-life-joe-manages-jim.svg)

Now imagine a city which is composed of several parts separated by water and those parts are connected by bridges. If those bridges allow bidirectional traffic, we can consider those bridges as undirected or bidirectional relationships.

If graphs and this image of a city divided by a river and having some islands and bridges rings a bell, it's not a coincidence. It's the place where graph theory was born in 1736. The place is Königsberg (Prussia), today it is called Kaliningrad (Russia). The Swiss polymath Leonhard Euler was asked to advise a walk through Königsberg that would cross each bridge only once. You might know he didn't find a solution - and luckily he didn't end up on gallows...

![Konigsberg]({{ site.baseurl }}/assets/img/konigsberg.svg)


He represented the different parts of the city as nodes and the bridges as edges and analysed the problem which I won't do here, but you might want to check out what an [Eulerian path](https://en.wikipedia.org/wiki/Eulerian_path) is. This is already enough to understand that one of our daily graph usages is about representing and analyzing topologies and another is routing, or path finding. In another article, I'll give a better grasp on them.

Speaking about topology, let's not forget maps! You might have liked as a child how colourful they are. You might have even enumerated the different colours. And on most of the maps you haven't seen too many different ones. For simpler maps 3 colours are enough, but 4 is enough for all the maps according to the [four color theorem](https://en.wikipedia.org/wiki/Four_color_theorem). To be fair many political and history maps use more colors but not because of a necessity, but for other reasons just as expressivity or aesthetics.

Topology is something which appears in many different sciences for different goals. Besides the above usages, it's also important in chemistry and physics for example to visualize atomic structures and analyze them. In biology you might use graphs to represent migration paths, breeding patterns, or spread of species or diseases. Mentioning spreading, let's not forget about sociology where graph theory is widely used to explore rumor spreading or to mesaure actor's message. Of course graphs are great to represent social networks.

If you found this article through Google search, graph theory was involved there again. Google's [PageRank](https://en.wikipedia.org/wiki/PageRank) algorithm is also based on graphs where websites are nodes and hyperlinks are edges.

If you are managing the security system of a museum you'll use graph theory in order to decide how many [guards](https://en.wikipedia.org/wiki/Art_gallery_problem) you need to have an eye on every room. In fact this is interesting for you whatever security system you are responsible for.

You must have heard about the [Panama papers](https://en.wikipedia.org/wiki/Panama_Papers) that include tons of financial and attorney-client infromation for hundreds of thousands of offshore entities. In other words millions of documents describing complex networks of financial moves including many alleged crimes and frauds. Guess what! All these data were turned into a graph and analyzed as such looking for a better understanding of relationships. Let's not forget that graph theory is useful even in fraud detection and crime investigation!

Open your eyes, graphs are everywhere! It's worth learning about them! Keep with me, [follow me on Twitter](https://twitter.com/SandorDargo) if you want to be part of this journey!

