---
title: 'Hash Set and Hash Table'
date: '2021-11-15'
categories: ["DSA"]
tags: ["Hash Set", "Hash Table", "Python", "English Article"]
---

# The Principle of Built-in Hash Table

- The key value can be any hashable type. A value that belongs to a hashable type has a hash code, and this code is used to get the bucket index.
- Each bucket contains an array to store all the values in the same bucket initially
- If there are too many values in the bucket, these values will be stored in the form of height-balanced BST so that look-up can be more efficient.


# Implementation

HashSet

```python
class MyHashSet:
    def hash(self, key):
        # Multiplicative hashing
        return ((key * 10000019) % (1 << 20)) >> 5

    def __init__(self):
        self.hashset = [[] for _ in range(1 << 15)]

    def add(self, key):
        idx = self.hash(key)
        if key not in self.hashset[idx]:
            self.hashset[idx].append(key)

    def remove(self, key):
        idx = self.hash(key)
        if key in self.hashset[idx]:
            self.hashset[idx].remove(key)

    def contains(self, key):
        idx = self.hash(key)
        return key in self.hashset[idx]
```

HashMap

```python
class ListNode:
    def __init__(self, key, value):
        self.pair = (key, value)
        self.next = None


class MyHashMap:
    def __init__(self):
        self.size = 997
        self.bc = [None] * self.size

    def put(self, key: int, value: int) -> None:
        idx = key % self.size
        if self.bc[idx] == None:
            self.bc[idx] = ListNode(key, value)
        else:
            cur = self.bc[idx]
            while True:
                if cur.pair[0] == key:
                    # update value for the same key
                    self.bc[idx] = ListNode(key, value)
                if cur.next == None:
                    break
                cur = cur.next
            cur.next = ListNode(key, value)

    def get(self, key: int) -> int:
        idx = key % self.size
        cur = self.bc[idx]
        while cur:
            if cur.pair[0] == key:
                return cur.pair[1]
            if cur.next == None:
                break
            cur = cur.next
        return -1

    def remove(self, key: int) -> None:
        idx = key % self.size
        cur = prev = self.bc[idx]
        if cur == None:
            return
        if cur.pair[0] == key:
            self.bc[idx] = cur.next
            return
        cur = cur.next
        while cur:
            if cur.pair[0] == key:
                prev.next = cur.next
                break
            else:
                prev, cur = cur, cur.next
```



# References

[Introduction to Data Structure
Hash Table](https://leetcode.com/explore/learn/card/hash-table/182/practical-applications/)
