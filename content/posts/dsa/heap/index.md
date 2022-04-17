---
title: 'Heap'
date: '2022-04-13'
categories: ["DSA"]
tags: ["Heap", "Python", "English Article"]
---

# Heap

Heap is a special Tree-based data structure in which tree is a complete binary tree.
Heaps are basically are binary trees with more properties and specifications and there are mainly two types of heaps.

Rules

- A heap must be a complete binary tree.
  - All of the levels of the tree must be completely filled except maybe the last one.
  - The last level has all keys as left as possible

## Types of heap

`Min Heap`

In min heap, all elements are smaller than their children. The root node will be the smallest element.

`Max Heap`

In max heap, all elements are bigger than their children. The root node will be the biggest element.

{{<figure src="./minmax_heap.png" alt="Query Suggestion" width="100%">}}

## Implementation

```python
# A Binary Heap is a Complete Binary Tree. A binary heap is typically represented as array. The representation is done as:

# The root element will be at Arr[0].
# Below table shows indexes of other nodes for the ith node, i.e., Arr[i]:
# Arr[(i-1)/2] Returns the parent_idx node
# Arr[(2*i)+1] Returns the left child node
# Arr[(2*i)+2] Returns the right child node

import sys

class MinHeap:
    def __init__(self, maxsize):
        self.maxsize = maxsize
        # Initialize the first value with the smallest value
        # With this first value, calculation to get parent_idx and children gets a little simpler
        self.heap_list = [0] * (maxsize + 1)
        self.heap_list[0] = -1 * sys.maxsize
        self.size = 0
        # Actual root index
        self.ROOT_IDX = 1

    def parent_idx(self, idx):
        return idx // 2

    def left_child_idx(self, idx):
        return 2 * idx

    def right_child_idx(self, idx):
        return (2 * idx) + 1

    def is_leaf(self, idx):
        # If left child index is bigger than self.size, it means there is no child as this is a complete binary tree
        return 2 * idx > self.size

    def swap(self, fi, si):
        self.heap_list[fi], self.heap_list[si] = self.heap_list[si], self.heap_list[fi]

    # Heapify the value in the given idx from top to bottom
    def min_heapify(self, idx):
        li = self.left_child_idx(idx)
        ri = self.right_child_idx(idx)
        curr = self.heap_list[idx]
        if not self.is_leaf(idx):
            l, r = self.heap_list[li], self.heap_list[ri]
            if l < curr or r < curr:
                if l < r:
                    self.swap(idx, li)
                    self.min_heapify(li)
                else:
                    self.swap(idx, ri)
                    self.min_heapify(ri)

 	# O(Log n)
    def insert(self, new_el):
        if self.size >= self.maxsize:
            print("Heap maxsize exceeded")
            return
        self.size += 1
        self.heap_list[self.size] = new_el
        tail_idx = self.size
        pi = self.parent_idx(tail_idx)
        # Heapify the inserted value from bottom to top while it's bigger than the parent
        while self.heap_list[tail_idx] < self.heap_list[pi]:
            self.swap(tail_idx, pi)
            tail_idx = pi
            pi = self.parent_idx(tail_idx)

	# O(Log n)
    def extract_min(self):
        if self.size < 1:
            print("Empty Heap")
            return
        min_ele = self.heap_list[self.ROOT_IDX]
        # Swap the root with the last element
        self.heap_list[self.ROOT_IDX] = self.heap_list[self.size]
        self.size -= 1
		# Heapify heap_list from top to bottom
        self.min_heapify(self.ROOT_IDX)
        return min_ele

	# O(1)
	def get_min(self):
        if self.size < 1:
            print("Empty Heap")
            return
        return self.heap_list[self.ROOT_IDX]
```

## Reference

[Heap Data Structure](https://www.geeksforgeeks.org/heap-data-structure/)
[Data Structures: Heaps](<https://www.youtube.com/watch?v=t0Cq6tVNRBA>)
[Learning to Love Heaps](https://medium.com/basecs/learning-to-love-heaps-cef2b273a238)
[Heap implementation in Python](https://www.educative.io/edpresso/heap-implementation-in-python)
