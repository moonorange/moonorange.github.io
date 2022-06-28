---
title: 'Random Notes for system design'
date: '2022-06-27'
categories: ["System Design"]
tags: ["English Article"]
---

# NoSQL database

NoSQL databases are non tabular databases.

Features
- Flexible schema
- Horizontal scaling
- Fast queries due to the data model
- Ease of use for developers

## Types

The main types are document, key-value, wide-column, and graph.

### Document databases

Document databases store data in documents similar to JSON objects.

Particular elements can be indexed for faster querying.

They have the flexibility to rework their document structures as needed.

Use cases include e-commerce platforms, trading platforms, and mobile app development.

### Key-Value databases

Key-Value databases store data as a key value pair.

Use cases include shopping carts, user preferences, and user profiles.

### Wide-column stores(Columns-Oriented Databases)

Wide-column stores store data in table, rows, and dynamic columns.

A Colum store is organized as a set of columns.

Columnar database can aggregate the value of a given column quickly

Use cases include analytics.

Writing data requires a lot of costs as that triggers multiple write events on disk.


### Graph databases

Graph databases store data in nodes and edges. Nodes typically store information about people, places, and things, while edges store information about the relationships between the nodes.

It is optimized to capture and search the connections and relations between data elements, overcoming the overhead associated with JOINing multiple tables in SQL.

Use cases include fraud detection, social networks, and knowledge graphs.