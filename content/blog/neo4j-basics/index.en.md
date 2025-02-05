---
title: Neo4j Basics
weight: -60
draft: true
description: The basic info and operations of Neo4j Graph Database
slug: neo4j-basics
tags:
  - Neo4j
series:
  - Operation
series_order: 5
date: 2025-02-05
lastmod: 2025-02-05
authors:
  - Morethan
---
{{< lead >}}
The basic info and operations of Neo4j Graph Database
{{< /lead >}}

## Introduction
[Neo4j](https://github.com/neo4j/neo4j) is a high-performance graph database that stores data and the relationships between data in the form of graphs.

The specific form of the graph is a **labeled property graph**, and the query language used is `Cypher`.

## Labeled Property Graph
A **labeled property graph** is a specific type of graph:
- A node has one or more labels to define its type.
- Relationships and nodes are treated equally, holding the same level of importance.
- Each node and relationship possesses accessible properties.

## Cypher

> Cypher is a **declarative** query language that allows you to identify **patterns** in your data using an **ASCII-art style syntax** consisting of **brackets**, **dashes** and **arrows**.

### Patterns
A **pattern** in a graph database is a specific combination of nodes and relationships. For example, a person (Person) acting in a movie (Movie) is a pattern, represented in code as:

```cypher
(p:Person)-[r:ACT_IN]->(m:Movie)
```

Here, the parts enclosed in parentheses represent nodes, while the parts enclosed in square brackets represent relationships. In the node section, `p` and `m` are variables referring to the respective nodes, with `Person` and `Movie` being the labels of the nodes, connected by a colon. In the relationship section, `r` is the variable referring to the relationship, and `ACT_IN` is the specific type of relationship. Translated into natural language, this means: use `p` to refer to a node labeled `Person`, use `m` to refer to a node labeled `Movie`, and represent the relationship between them with `r`, which is of type `ACT_IN`.

### Data Reading
Data reading operations rely on **pattern matching**, using the keyword `MATCH`. This is equivalent to sending an instruction to the database to filter out only the **node-relationship pairs** that match the specified pattern, i.e., a collection of triples `(p, r, m)`.

```cypher
MATCH (p:Person)-[r:ACT_IN]->(m:Movie)
```

Similar to SQL, you can further filter using the `WHERE` keyword:

```cypher
MATCH (p:Person)-[r:ACT_IN]->(m:Movie)
WHERE p.name = 'Tom Hanks'
RETURN p, r, m
```

Here, `name` is a property of the node `p`, accessed using the `.` operator. The `RETURN` keyword marks the values to be returned.

Below is a more complex pattern matching command, where the `AS` keyword is used to set aliases:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)<-[r:ACTED_IN]-(p2:Person)
WHERE p.name = 'Tom Hanks'
RETURN p2.name AS actor, m.title AS movie, r.role AS role
```

This command filters out all actors who have acted in the same movie as Tom Hanks, returning their names, the movies they acted in, and the roles they played.

Sorting by a specific property and pagination are also supported:

```cypher
MATCH (m:Movie)
WHERE m.released IS NOT NULL
RETURN m.title AS title, m.url AS url, m.released AS released
ORDER BY released DESC LIMIT 5
```

This will filter out the 5 most recent movies.

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
Cypher keywords are case-insensitive; property names, variable names, and other identifiers are case-sensitive.
{{< /alert >}}
### Data Writing
Data writing uses the `MERGE` keyword, which means **merging** a node or relationship into the data graph.

- Merging nodes:
```cypher
MERGE (m:Movie {title: "Arthur the King"})
SET m.year = 2024
RETURN m
```

This command creates a new `Movie` node with the `title` property set to `"Arthur the King"` and the `year` property set to `2024`.

You might wonder why not write it like this:
```cypher
MERGE (m:Movie)
SET m.year = 2024, m.title = "Arthur the King"
RETURN m
```

The reason is that the content within the curly braces is used to check whether the **pattern** you want to create already exists, avoiding duplicate patterns.

- Merging relationships:
```cypher
MERGE (m:Movie {title: "Arthur the King"})
MERGE (u:User {name: "Adam"})
MERGE (u)-[r:RATED {rating: 5}]->(m)
RETURN u, r, m
```

