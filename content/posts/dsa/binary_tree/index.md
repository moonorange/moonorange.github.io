---
title: 'Binary Tree'
date: '2021-05-16'
categories: ["DSA"]
tags: ["Tree", "Python", "English Article"]
---

# Overview

A tree is a frequently-used data structure to simulate a hierarchical tree structure.

Each node of the tree will have a root value and a list of References to other nodes that are called child nodes. From graph view, a tree can also be defined as a directed acyclic graph that has N nodes and N-1 edges.

A Binary Tree is one of the most typical tree structures. As the name suggests, a binary tree is a tree data structure in which each node has at most two children, which are referred to as the left child and the right child.

![binary tree](https://media.geeksforgeeks.org/wp-content/cdn-uploads/binary-tree-to-DLL.png)

A Binary Tree node contains the following parts.

Data
Pointer to a left child
Pointer to a right child

```python
# Definition for a binary tree node.
class TreeNode:
	def __init__( val=0, left=None, right=None):
		val = val
		left = left
		right = right
```

## Types of binary tree

There are many types of binary tree

### Full Binary Tree

`All nodes except leaf nodes` have two children.

{{<figure src="./full_bt.png" alt="Full Binary tree" width="75%">}}

### Complete Binary Tree

All the levels are completely filled except possibly the last level and the last level has all keys `as left as possible.`

{{<figure src="./complete_bt.png" alt="Complete Binary tree" width="75%">}}

### Perfect Binary Tree

All the internal nodes have two children and the all leaf nodes are at the same level.

{{<figure src="./perfect_bt.png" alt="Perfect Binary tree" width="75%">}}

### Balanced Binary Tree

`The height of the tree is O(Log n)` where n is the number of nodes.

### A degenerate (or pathological) tree

Every internal node has `only one child.`

{{<figure src="./pathological_bt.png" alt="Pathological tree" width="75%">}}

## Depth First Traversals

{{<figure src="./bt_dfs_raversals.jpg" alt="DFS" width="75%">}}

### Implementation

```python
from binarytree import tree
from binary_tree import TreeNode
from typing import List

# recursive solution
def traverse(root: TreeNode) -> List[int]:
    ans = []
    traverse(root, ans)
    return ans

def inorder_traverse(root: TreeNode, ans: List[int]):
    if root:
        traverse(root.left, ans)
        ans.append(root.val)
        traverse(root.right, ans)

def preorder_travese(root: TreeNode, ans: List[int]):
    if root:
        ans.append(root.val)
        traverse(root.left, ans)
        traverse(root.right, ans)

def postorder_travese(root: TreeNode, ans: List[int]):
    if root:
        traverse(root.left, ans)
        traverse(root.right, ans)
        ans.append(root.val)
```

"Breadth First Traversals":

Level order traversal of a tree is breadth first traversal for the tree.

1 2 3 4 5

```python
# Given the root of a binary tree, return the level order traversal of its nodes' values. (i.e., from left to right, level by level).
from itertools import chain
def levelOrder(root: TreeNode) -> List[List[int]]:
    if not root:
        return []
    ans, same_lev = [], [root]
    while (same_lev):
        ans.append((map(lambda node: node.val, same_lev)))
        tmp = chain.from_iterable(map(lambda node: [node.left, node.right], same_lev))
        same_lev = [leaf for leaf in tmp if leaf]
    return ans


from binary_tree import TreeNode
from typing import List
import queue

# Implementation using queue
def bfs(root: TreeNode) -> List[int]:
    ans = []
    my_queue = queue.Queue()
    my_queue.put(root)

    while not my_queue.empty():
        root = my_queue.get()
        ans.append(root.val)

        if root.left:
            my_queue.put(root.left)

        if root.right:
            my_queue.put(root.right)
    return ans
```

## References

[Binary Tree Data Structure](https://www.geeksforgeeks.org/binary-tree-data-structure/)

[Introduction to Data Structure Binary Tree](https://leetcode.com/explore/learn/card/data-structure-tree/134/traverse-a-tree/931/)

[Binary Tree | Set 3 (Types of Binary Tree](<https://www.geeksforgeeks.org/binary-tree-set-3-types-of-binary-tree/)
