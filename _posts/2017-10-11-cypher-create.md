---
layout: post
title: "Cypher tutorial: the CREATE keyword"
date: 2017-10-11
category: dev
tags: [database, graphs, cypher, neo4j]
header: "In the first article of <a href=\"/blog/2017/10/04/cypher-introduction\">my series about Cypher</a> I introduce you the CREATE keyword."
---
The aim of this keyword is to add nodes or relationships to your graph database. As you are going to see you can add one simple node without any useful data, but you can also add multiple well-characterized nodes with relationships connecting them together. Let's dive deep! 

## Create one node
The simplest query I could imagine is this one:
```
CREATE ()
```

This creates a new node without any labels or properties. It is a pure node with only its technical ID. Probably not too useful in real life, but still this is the origo. It helps us to see that in a Cypher query a node is defined between parentheses.

### Create a node with one or more labels
Now if you want to create a new node with a label, you write the label after a colon, just like this:
```
CREATE (:CABERNET_SAUVIGNON)
```

As I mentioned in the [introduction](/blog/2017/10/04/cypher-introduction) you can attach multiple labels to a node:
```
CREATE (:CABERNET_SAUVIGNON:RED)
```

### Create a node with a label and properties
If you also want to attach some properties to a node you can do it within the parentheses, after the labels in a JSON-style:
```
CREATE (:WINE {name:"Cabernet Sauvignon", fruity: false})
```

Tagging a node with a label is not mandatory, but if you want to attach a label, you have to write it before the properties.

### Create two nodes
Just separate your nodes by a comma. The CREATE keyword has to be used only once.

```
CREATE (:WINE {name:"Cabernet Sauvignon", fruity: false}), (:WINE {name:"Merlot", fruity: true})
```

## Create new relationships
You've just seen how easy it is to create new nodes. Now we'll see how easy it is to create two nodes with a relationship between them.

### Create new relationship between new nodes
```
CREATE (:GRAPE {name:"Cabernet Sauvignon"})-[:GROWS_AT]->(:SUBREGION {name:"Villány"})
```

Whereas nodes are declared between parentheses, relationships are signaled by brackets. A relationship just as a node can have as many properties as you want - again, declare them in JSON style. However while a node can have multiple labels, a relationship can have only one type. But you can add multiple relationships with different types between the same two nodes.

### Create new relationships between existing nodes
Adding a new relationship to nodes that already exist is, of course, possible, but it requires the usage of the `MATCH` keyword. As `MATCH` is not part of this article, I won't go into details about it. I could relate Cypher's `MATCH` to SQL's `FROM` and `WHERE` keywords.

```
MATCH (cabernet:Grape {name: 'Cabernet sauvignon'}), (villany:SUBREGION {name:"Villány"})
CREATE (cabernet)-[relationship_type:GROWS_AT]->(villany)
```

Here you can see something you haven't seen before. I stared nodes and relationships with a name. You can name them and reuse them, i.e. you can have variables. If I create a node and I name it to cabernet, I can reuse the node in the same line without giving the full description again:

```
CREATE (cabernet:Grape {name: 'Cabernet sauvignon'}),
(villany:SUBREGION {name:"Villány"}),
(cabernet)-[relationship_type:GROWS_AT]->(villany)
```

In fact, if you use the same declaration twice (without giving a variable name to the new entity), two nodes/relationships would be created with the same attributes, but with a different id.

## Conclusion

Today we've seen how to use the CREATE keyword in order to create nodes and/or relationships. Soon we will discover other keywords, such as `MATCH` and `RETURN`. 

## Call to action
If you are interested in graph databases, go and get one. Start experimenting with for example Neo4j. It's easy to [install it](https://neo4j.com/docs/operations-manual/current/installation/), but you can also play with it [online](http://console.neo4j.org/).

If you are interested what's coming, follow me on Twitter. 