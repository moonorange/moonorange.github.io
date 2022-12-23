---
title: 'Designing Data-Intensive Applications'
date: '2022-11-26'
categories: ["Software Design"]
tags: ["Distributed System", "DB", "English Article"]

---

## What's this article about

This is my notes of a book called `Designing Data-Intensive Applications`.

## Chapter3

Storage and Retrieval

### SSTables and LSM Trees

SSTables(Sorted String Tables) a file which contains a set of arbitrary and sorted key-value pairs inside.

{{<figure src="./SSTables.png" alt="SSTables" width="100%">}}
<https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/>

SSTables and Log-Structured Merge Trees are still used in some database.

As a rule of thumb, LSM Trees are faster for writes but slower for read than B-Tree index.

### B-Trees

The B-Tree is the most widely used indexing structure.

It stores data in a balanced tree like image below, thus searching data can be done in O(logN).

{{<figure src="./b_tree_branches.png" alt="B-Tree" width="75%">}}

Each B-Tree parent page has references to child pages, and the number of references that a parent page can have is called `branching factor`, which is 6 in the case above but typically several hundred.

### Index

`Primary index` is index that resolves references to primary key that can identify each record.

`Secondary index`, whereas, is not necessary references to unique keys.

### In-Memory Database

In-memory database archives durability by writing data to disks.

Redis has `weak durability` as it writes to disks asynchronously.

The reason why in-memory database has advantages on performance is not because it doesn't need to read from disks, but because they don't need to encode data in a form that can be written to disks.

### OLTP and OLAP

`OLTP` is online transaction processing, and `OLAP` is online analytic processing.

The difference is not clear-cut, but here are some main characteristics of these two.

#### OLTP

- Main Read Pattern: Small number of records per query, fetched by key
- Main Write Pattern: Random-Access, low-latency writes from user input
- Primary used by: End user
- What data represents: Latest state of data
- Dataset Size: Gigabytes to Terabytes

#### OLAP

- Main Read Pattern: Aggregate over large number of records
- Main Write Pattern: Bulk import or event stream
- Primary used by: Internal analyst
- What data represents: History of events
- Dataset Size: Terabytes to Petabytes


## 4. Encoding and Evaluation. Page 111

### Formats for Encoding data

The translation from the `in-memory representation` to a `byte sequence` is called encoding(a.k.a marshalling or serialization). The reverse is called decoding, parsing, unmarshalling, or deserialization

#### gRPC(compatibility)

Forward Compatibility: If new code reads data from the old protobuf including new fields, it just ignores new fields. So the forward compatibility remains.

Backward Compatibility: If you add a new required field, then old code cannot read from the old protobuf. So you cannot add required fields afterwards

#### Message Broker and Actor model(P137)

`Message Broker`

One message sends a message to a named queue or topic, and broker ensures that the message is delivered to at least one consumers or subscribers to that topic or queue. There can be many producers and consumers on the same topic.

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
