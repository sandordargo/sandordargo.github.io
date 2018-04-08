---
layout: post
title: "Cypher tutorial: OPTIONAL MATCH"
date: 2017-11-22
category: dev
tags: [database, graphs, cypher, neo4j]
header: "After <a href=\"/blog/2017/10/25/cypher-match\">having discussed about Cyper's `MATCH` keyword</a>, let's talk a bit about its variant, the `OPTIONAL MATCH`."
---
I discovered `OPTIONAL MATCH` when I was preparing [the code kata of Test Driven Development with Neo4j](https://github.com/sandordargo/neo-wine-services).

First, let's have a look at how the test data look like.

![Test Data new wine services]({{ site.baseurl }}/assets/img/test-data-neo-wine-services.png)

You can see that there is one `WineRegion`, 3 `WineSubregions` connecting to the same `WineRegion` and there are `Grapes` as well. But not every `WineSubregion` has related `Grapes` - in our test database.

When I was developing the function to return a given WineSubregion with its parent region and with the grapes which are grown at that subregion, first I wrote something like this (instead of a variable I use a real subregion name here)

`MATCH (wr:WineRegion)-[:CONTAINS]->(wsr:WineSubRegion {name:"Mátra"})<-[:GROWS_AT]-(grape:Grape) RETURN wr, wsr, grape`

This query only worked if the subregion I was looking for had grapes associated with it. After I thought about it I realized that it must be about that missing relationship. So as a next step I added that I was looking for 0 or 1 instance of that relationship - that's what I thought at least.

`MATCH (wr:WineRegion)-[:CONTAINS]->(wsr:WineSubRegion {name:"Mátra"})<-[:GROWS_AT*0..1]-(grape:Grape) RETURN wr, wsr, grape`

Still no result for subregions without grapes grown there. Now I was truly surprised. I expected that if I look for potentially zero relationships then I should have results for any subregions.

In fact, it is so obvious this didn't work. If you put a number or a range after a relationship type, you don't define how many relationships there can be between the two nodes - why would you even do that - but you define a relationship length. I'll cover it in another article, but to give you an idea, you can define how many relationships of a certain type you have to hop through in order to reach another node. With a `:FRIEND` relationship in a social graph, it makes perfect sense. In my data model, it doesn't.

Then I found the `OPTIONAL MATCH` clause. Its name is really descriptive.

After the `MATCH` clause you describe all the nodes and relationships you are looking for and must be present. Then after the `OPTIONAL MATCH` you describe anything that is okay not to be there in the database. If the optional elements are not there, they will be represented as null in the results.

`MATCH (wr:WineRegion)-[:CONTAINS]->(wsr:WineSubRegion {name:"Mátra"}) OPTIONAL MATCH (wsr)<-[:GROWS_AT]-(grape:Grape) RETURN wr, wsr, grape`

Now you know how you can look for optional elements in a graph, next time we will talk about the `MERGE` keyword.
