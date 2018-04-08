---
layout: post
title: "Cypher tutorial: the MATCH keyword"
date: 2017-10-25
category: dev
tags: [database, graphs, cypher, neo4j]
header: "After <a href=\"/blog/2017/10/11/cypher-create\">having discussed what one can do using CREATE and slightly mentioning MATCH</a>, let's explore what that MATCH can do for you in Cypher"
---
Luckily on a basic level both `CREATE` and `MATCH` are quite self-explanatory. So with `MATCH` you'll be able to look for certain elements in your database. In short, you can match patterns. Often you will you use `MATCH` together with `WHERE`, but not necessarily as I will show you. 

Let's see quickly what are the corresponding keywords in SQL. As usual there is no exact match, but you can think about `FROM` and `WHERE` if you think SQL. But while in SQL you heavily rely on `WHERE`, in Cypher only `MATCH` can do the work for you in a lot of cases.

When we want to explore the usage of `MATCH`, we also have to understand another two keywords: `WHERE` and `RETURN`. Later I will dedicate an own post for each of them, now let's just quickly skim them through.

#### __`WHERE`__

This keyword cannot stand on its own. It goes with any of the following clauses: `MATCH`, `OPTIONAL MATCH`, `START` or `WITH`. This also means that it can be used in different ways. Now I'm just going to focus on its basic usage with `MATCH` and then in another article we'll cover the rest.

`WHERE` is to add constraints to your patterns defined in `MATCH`. It these terms it works pretty similar to SQL. You can use use the usual boolean operators (`AND`, `OR`, `XOR` and `NOT`), you can use it for string matching, quantified comparisons, existence checks, etc. Here is a short (and really non-exhaustive) list of  examples:


```
WHERE region.name = 'Pannon' AND (grape.name = 'Merlot' OR grape.name = 'Cabernet franc')
```

```
WHERE subregion.size > 500
```

```
WHERE grape.name STARTS WITH 'Cabernet'
```

```
WHERE grape.name IN ['Muscat ottonel', 'Cabernet sauvignon']
```

These are only the simplest expressions, more advanced examples will be part of the post dedicated to the `WHERE` clause.


#### __`RETURN`__

It is like `SELECT` in SQL. But while in SQL you start with `SELECT`, in Cypher you end with `RETURN`. You define what you want to return from your query. In its simplest form, you can return everything: `RETURN *`. Most of the times, you will return nodes and/or relationships:

```
MATCH (wineRegion:WineRegion)
RETURN wineRegion
```

```
MATCH ()-[relationship:GROWS_AT]->()
RETURN relationship
```

```
MATCH (grape:Grape)-[relationship:GROWS_AT]->(subregion:SubRegion)
RETURN grape, relationship, subregion
```

As you can see if you want to return multiple elements, it's really easy, you can just list the elements separated by commas.

If you don't want to return the whole node, just a single property, it's also really easy, you specify the name of the property after the node/relationship name separated by a dot:

```
MATCH (wineRegion:WineRegion)
RETURN wineRegion.name
```

That's enough for now. Let's talk about the `MATCH` keyword.

#### Get all nodes

Let's start it easy. Let's return all nodes.
```
MATCH (n)
RETURN n
```
![Match N]({{ site.baseurl }}/assets/img/match_n.png)

It is that simple. Just match every node. You might expect to get a bunch of lonely nodes on your console, but in that case, you're mistaken. All the relationships are retrieved too. I have to be more precise with this latter statement. When you retrieve nodes all the direct relationships will be retrieved which are between those nodes.

You'll see quickly what I mean. But first, have a look at a sub-graph that we will heavily use today. You can see all the wine subregions contained in a region called _Eger_ and all the grapes which grow in these subregions.

![Eger sub-graph]({{ site.baseurl }}/assets/img/eger-sub-graph.png)

#### Get nodes by name

Let's assume that there are differently labeled nodes with the same name. Still, for some reason it wouldn't make sense the tag the same node with two different labels. There is such a case in our example graph related to wine regions.

_[Eger](https://en.wikipedia.org/wiki/Eger)_ which is a Hungarian city is also a name of a wine region and a wine subregion. If you want to get all the nodes with the same name without declaring the labels you are interested in, you must use the `WHERE` clause as you can see below. Putting it differently, you cannot write something like this: `MATCH (n:{name: 'Eger'})`. If you specify an attribute in the `MATCH` clause you must specify the label too.

```
MATCH (n)
 WHERE n.name = "Eger"
 RETURN n
```

#### Get nodes by label and name

You can can get the same result by executing this query:

```
MATCH (region:WineRegion{name:'Eger'}), (subregion:WineSubRegion{name:'Eger'})
RETURN region, subregion
```

In both cases, you could see that when you return these nodes, which are in a direct relationship with each other, the relationships are also shown on the graph returned on the Neo4j console. However, if you check other views, you can see that they are not returned, just shown as a sign of courtesy.

Now if you query two nodes which are in an indirect relationship with each other, like a grape and the region (there is a subregion in between), even on the graph view you won't see any relationship between them, you'll only see two lonely nodes.
```
MATCH (grape:Grape {name:"Cabernet franc"}), (region:WineRegion {name:"Eger"})
RETURN grape, region
```

#### Relationships with undefined directions

If you want to retrieve any direct relationship between some nodes, you can do it this way. You can see that there is no arrow pointing at any direction. Not even the relationship type is defined. We just named it as `rel` and returned it.

```
MATCH (region:WineRegion{name:'Eger'})-[rel]-(subregion:WineSubRegion)
RETURN region, subregion, rel
```

#### Relationships with defined directions

Now we write our query with the relationship pointing at the wine region. Executing the query one can see that there are no nodes or relationships retrieved. 

```
MATCH (region:WineRegion{name:'Eger'})<-[rel]-(subregion:WineSubRegion)
RETURN region, subregion, rel
```
Let's turn the direction of that relationship! Now you can see that we're getting the same results as before with the undirected relationships. In fact, the relationships in your graph are always directed, but the Cypher engine will look for both directions.

```
MATCH (region:WineRegion{name:'Eger'})-[rel]->(subregion:WineSubRegion)
RETURN region, subregion, rel
```

You can even specify the type of the relationship if you want to be more specific:

```
MATCH (region:WineRegion{name:'Eger'})-[rel:CONTAINS]->(subregion:WineSubRegion)
RETURN region, subregion, rel
```

Or you can even specify multiple relationship types to match:

```
MATCH (region:WineRegion{name:'Eger'})-[rel:CONTAINS|GROWS_AT]->(subregion:WineSubRegion)
RETURN region, subregion, rel
```

#### Getting the type of the relationship

Let's say you don't know exactly how some nodes are connected. As we saw in your query you don't have to indicate the relationship type. You can just get every relationship irrespective of their type or direction. You can retrieve their type instead of the relationships themselves. Or both! Just use the `type()` function.

```
MATCH (region:WineRegion{name:'Eger'})-[rel:CONTAINS]->(subregion:WineSubRegion)
RETURN region, subregion, rel, type(rel)
```

You can only use `type()` with relationships.

#### Get nodes or relationships by id

If you are building and executing your queries from an application, it's quite probable that you would identify your nodes and/or relationships with their technical IDs. No problem, you can easily retrieve them as such, using their id by using the `id()` function. You can easily match one single or a list of ids. It's very intuitive as you can see.

```
MATCH (n)
WHERE id(n) IN [0, 3, 5]
RETURN n
```
```
MATCH ()-[r]->()
WHERE id(r)= 0
RETURN r
````

We are far from finishing exploring the complete list of possibilities that `MATCH` provides, but that's it for today. I keep some topics about paths to an advanced article. Stay tuned!