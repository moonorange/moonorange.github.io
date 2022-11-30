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

Sorted String Tables and Log-Structured Merge Trees are still used in some database.

As a rule of thumb, LSM Trees are faster for writes but slower for read than B-Tree index.

### B-Trees

The B-Tree is the most widely used indexing structure.

It stores data in a balanced tree like image below, thus searching data can be done in O(logN).

{{<figure src="./b_tree_branches.png" alt="B-Tree" width="75%">}}

Each B-Tree parent page has references to child pages, and the number of references that a parent page can have is called `branching factor`, which is 6 in the case above but typically several hundred.

### Index

Primary index is index that resolves references to primary key that can identify each record.

Secondary index, whereas, is not necessary references to unique keys.

### In-Memory Database

In-memory database archives durability by writing data to disks.

Redis has weak durability as it writes to disks asynchronously.

The reason why in-memory database has advantages on performance is not because it doesn't need to read from disks, but because they don't need to encode data in a form that can be written to disks.
