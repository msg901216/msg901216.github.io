---
layout:     post
title:      "cypher学习笔记"
subtitle:   "cypher for neo4j notes"
date:       2018-12-03
author:     "msg"
header-img: "img/posts/02.jpg"
header-mask: 0.3
catalog:    true

tags:
    - neo4j
    - cypher
    - 学习
    - 转载
---



### About Cypher

Cypher is a declarative, SQL-inspired language for describing patterns in graphs visually using an ascii-art syntax.

It allows us to state what we want to select, insert, update or delete from our graph data without requiring us to describe exactly how to do it.

![cypher pattern](img/posts/cypher_pattern_simple.png)

### Nodes

Cypher uses ASCII-Art to represent patterns. We surround nodes with parentheses which look like circles, e.g. **(node)**. If we later want to refer to the node, we’ll give it a variable like **(p)** for person or **(t)** for thing. In real-world queries, we’ll probably use longer, more expressive variable names like **(person)** or **(thing)**. If the node is not relevant to your question, you can also use empty parentheses **()**.

We might use a pattern like **(person:Person)-->(thing:Thing)** so we can refer to them later, for example, to access properties like **person.name** and **thing.quality**.

The more general structure is:

```cypher
MATCH (node:Label) RETURN node.property

MATCH (node1:Label1)-->(node2:Label2)
WHERE node1.propertyA = {value}
RETURN node2.propertyA, node2.propertyB
```

### Relationships

To fully utilize the power of our graph database we want to express more complex patterns between our nodes. Relationships are basically an arrow **-->** between two nodes. Additional information can be placed in square brackets inside of the arrow.

This can be

* relationship-types like ```-[:KNOWS|:LIKE]->```

* a variable name ```-[rel:KNOWS]->``` before the colon

* additional properties ```-[{since:2010}]->```

* structural information for paths of variable length ```-[:KNOWS*..4]->```

To access information about a relationship, we can assign it a variable, for later reference. It is placed in front of the colon ```-[rel:KNOWS]->``` or stands alone ```-[rel]->```.

General Syntax:
```
MATCH (n1:Label1)-[rel:TYPE]->(n2:Label2)
WHERE rel.property > {value}
RETURN rel.property, type(rel)
```

### Patterns

Nodes and relationship expressions are the building blocks for more complex patterns. Patterns can be written continuously or separated with commas. You can refer to variables declared earlier or introduce new ones.

* friend-of-a-friend ```(user)-[:KNOWS]-(friend)-[:KNOWS]-(foaf)```

* shortest path: ```path = shortestPath( (user)-[:KNOWS*..5]-(other) )```

* collaborative filtering ```(user)-[:PURCHASED]->(product)<-[:PURCHASED]-()-[:PURCHASED]->(otherProduct)```

* tree navigation ```(root)<-[:PARENT*]-(leaf:Category)-[:ITEM]->(data:Product)```


