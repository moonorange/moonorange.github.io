---
title: 'Union-Find'
date: '2023-08-19'
categories: ["DSA"]
tags: ["Union-Find", "Python", "English Article"]
---

## Union-Find

Union-Find, aka Disjoint Set, is a rooted tree data structure that can efficiently classifies elements into categories.
By utilizing this data structure, it becomes possible to rapidly determine whether two elements belong to the same group, as well as to swiftly merge two groups.

## Implementation in Python

```python
class UnionFind:
    def __init__(self, n):
        # Initialize all parents as -1, which means all nodes are root nodes
        self.parent = [-1] * n
        # Rank holds a weight to determine which tree has more weight
        # The tree with a smaller rank should be placed under the tree with a greater rank to keep the height of the merged tree smaller
        self.rank = [0] * n
        # Size holds the number of nodes in a tree
        self.size = [1] * n

    def find_root(self, x):
        if self.parent[x] == -1:
            return x
        else:
            # Make all nodes' parents the root(Path Compression)
            # This speeds up future root lookups by reducing the path length
            self.parent[x] = self.find_root(self.parent[x])
            return self.parent[x]

    # Check if x and y belong to the same root
    def has_same_root(self, x, y):
        return self.root(x) == self.root(y)

    def union(self, x, y) -> bool:
        rx, ry = self.find_root(x), self.find_root(y)
        # If two elements have the same root meaning they are in the same group
        # Do nothing
        if rx == ry:
            return False

        # Make the tree with the greater rank the parent(Union by rank)
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx

        # Increment the rank of the parent
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1

        # Increment the size of the parent
        self.size[rx] += self.size[ry]
        return True

    def get_size(self, x):
        return self.size[self.find_root(x)]
```

## Reference

[1.12 Disjoint Sets Data Structure - Weighted Union and Collapsing Find](https://www.youtube.com/watch?v=wU6udHRIkcc)
