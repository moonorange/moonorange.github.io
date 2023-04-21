---
title: 'Designing Data-Intensive Applications'
date: '2022-11-26'
categories: ["Software Design"]
tags: ["Distributed System", "DB", "English Article"]

---

## What's this article about

These are my notes on a book called `Designing Data-Intensive Applications`.

### SSTables and LSM Trees(P.76)

SSTables (Sorted String Tables) are files that contain a set of arbitrary and sorted key-value pairs.

{{<figure src="./SSTables.png" alt="SSTables" width="100%">}}
<https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/>

SSTables and Log-Structured Merge Trees are still used in some database.

As a rule of thumb, LSM Trees are faster for writes but slower for reads than B-Tree indexes.

### B-Trees

The B-Tree is the most widely used indexing structure.

It stores data in a balanced tree, like the image below. Thus, searching data can be done in O(logN).

{{<figure src="./b_tree_branches.png" alt="B-Tree" width="75%">}}

Each B-Tree parent page has references to child pages, and the number of references that a parent page can have is called the `branching factor`, which is 6 in the case above but typically several hundred.

### Index

A `Primary index` is an index that resolves references to a primary key that can identify each record.

A `Secondary index`, on the other hand, does not necessarily reference unique keys.

### In-Memory Databases

In-memory databases achieve durability by writing data to disks.

Redis has `weak durability` as it writes to disks asynchronously.

The reason why in-memory databases have advantages in performance is not because they don't need to read from disks, but because they don't need to encode data in a form that can be written to disks.

### OLTP and OLAP

`OLTP` stands for online transaction processing, and `OLAP` stands for online analytic processing.

The difference is not clear-cut, but here are some main characteristics of these two.

#### OLTP

- Main Read Pattern: Small number of records per query, fetched by key
- Main Write Pattern: Random-Access, low-latency writes from user input
- Primary used by: End user
- What data represents: Latest state of data
- Dataset Size: Gigabytes to Terabytes

#### OLAP

- Main Read Pattern: Aggregate over a large number of records
- Main Write Pattern: Bulk import or event stream
- Primary used by: Internal analyst
- What data represents: History of events
- Dataset Size: Terabytes to Petabytes

## 4. Encoding and Evaluation. Page 111

### Formats for Encoding data

`Encoding`, which refers to the conversion of `in-memory data` to `a byte sequence`, can also be called marshalling, serialization, or serialization. `Decoding`, on the other hand, is the reverse process, also known as parsing, unmarshalling, or deserialization.

#### gRPC(compatibility)

Regarding compatibility in gRPC, if new code reads data from an old protobuf that includes new fields, the new fields are simply ignored, thus maintaining forward compatibility.

However, adding a new required field would prevent old code from reading the old protobuf, so required fields cannot be added afterward.

#### Message Broker and Actor model(P.137)

`Message Broker`

When a message is sent to a named queue or topic, the broker ensures that it is delivered to at least one consumer or subscriber to that topic or queue.

Many producers and consumers can exist on the same topic.

Producer and Consumer Pattern Ref.(Japanese)
<https://www.hyuki.com/dp/dpinfo.html#ProducerConsumer>

English
<https://cloud.google.com/pubsub/docs/overview>

`Actor Model`

Location transparency is the ability to access objects without the knowledge of their location.

Actor model is a programming model for concurrency in a single process.

Each actor typically represents one entity or client and usually has local state.

It communicates with other actors by sending and receiving asynchronous messages.

Ref.
<https://www.youtube.com/watch?v=ELwEdb_pD0k>

## Chapter 5 Replication

## Failover

When the `leader(master or primary)` node failed, one of the `followers(read replicas, slaves, secondaries, or hot standbys)` is promoted to be a new leader, and this process is called `failover`.

Failover is fraught with things that can go wrong.

### Consistency with systems outside the database(github incident)

Discarding failed writes of the leader is dangerous especially when it has to coordinate with other storage systems outside the database.

There is an incident because of the leader crushing in Github.

Due to a leader crash in Github, an outdated follower became the new leader, resulting in a primary key conflict. The database employed an auto-incrementing primary key, which the new leader used despite it already being in use by the old reader. This caused inconsistencies between the data stored in Redis and the database.

### Split brain

When two nodes in a distributed system both consider themselves to be the leader, it is referred to as a "split brain" scenario.

This situation can cause data corruption and loss.

## Chapter 6 Partitioning(P.199)

A partition with a disproportionately high load is commonly referred to as a `hot spot.`

Hash partitioning is a popular technique used to distribute data evenly among partitions, but it can make range queries less efficient because data will be distributed regardless their similarity in contents.

Cassandra achieves a compromise of the above trade-off.

This can be achieved using compound primary keys, where only the first part of the key is used to determine a partition, and the other columns are used as a concatenated index for sorting the data.

- `Key range partitioning` involves sorting keys and assigning partitions to them. It has the advantage of sorting but carries a risk of creating hot spots.

- `Hash partitioning` involves applying a hash function to each key. It has the advantage of evenly distributing data into partitions but can lead to inefficient range queries.

## Chapter 7 Transactions
