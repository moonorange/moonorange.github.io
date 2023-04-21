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
e.g. MongoDB, CouchDB, and DynamoDB

### Key-Value databases

Key-Value databases store data as a key value pair.

Use cases include shopping carts, user preferences, and user profiles.
e.g. Redis and Memcached

### Wide-column stores(Columns-Oriented Databases)

Wide-column stores store data in table, rows, and dynamic columns.

Writing data requires a lot of costs as that triggers multiple write events on disk.

Use cases include analytics and IoT.
e.g. Cassandra and HBase

### Graph databases

Graph databases store data in nodes and edges. Nodes typically store information about people, places, and things, while edges store information about the relationships between the nodes.

It is optimized to capture and search the connections and relations between data elements, overcoming the overhead associated with JOINing multiple tables in SQL.

Use cases include fraud detection, social networks, and knowledge graphs.
e.g. Neo4j and JanusGraph
