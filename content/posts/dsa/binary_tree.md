---
title: 'Binary Tree'
date: '2021-05-13'
categories: ["DSA"]
tags: ["Tree", "Python", "English Article"]
---

# Overview

A tree is a frequently-used data structure to simulate a hierarchical tree structure.

Each node of the tree will have a root value and a list of references to other nodes that are called child nodes. From graph view, a tree can also be defined as a directed acyclic graph that has N nodes and N-1 edges.

A Binary Tree is one of the most typical tree structures. As the name suggests, a binary tree is a tree data structure in which each node has at most two children, which are referred to as the left child and the right child.

![binary tree](https://media.geeksforgeeks.org/wp-content/cdn-uploads/binary-tree-to-DLL.png)

A Binary Tree node contains following parts.

Data
Pointer to left child
Pointer to right child

```python
# Definition for a binary tree node.
class TreeNode:
	def __init__(self, val=0, left=None, right=None):
		self.val = val
		self.left = left
		self.right = right
```

![](https://media.geeksforgeeks.org/wp-content/cdn-uploads/2009/06/tree12.gif)

Depth First Traversals:

(a) Inorder (Left, Root, Right) : 4 2 5 1 3

(b) Preorder (Root, Left, Right) : 1 2 4 5 3

(c) Postorder (Left, Right, Root) : 4 5 2 3 1

Inorder Traversal Implementation
The other two depth first traversal implementation are almost the same.

```python
from binarytree import tree
from binary_tree import TreeNode
from typing import List


class Solution:
    # recursive solution
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        ans = []
        self.traverse(root, ans)
        return ans

    def traverse(self, root: TreeNode, ans: List[int]):
        if root:
            self.traverse(root.left, ans)
            ans.append(root.val)
            self.traverse(root.right, ans)

    # iterative solution
    def inorderTraversal2(self, root: TreeNode):
        res, stack = [], []
        while True:
            while root:
                stack.append(root)
                root = root.left
            if not stack:
                return res
            node = stack.pop()
            res.append(node.val)
            root = node.right
```

Breadth first traversals:

Level order traversal of a tree is breadth first traversal for the tree.

1 2 3 4 5

```python
# Given the root of a binary tree, return the level order traversal of its nodes' values. (i.e., from left to right, level by level).
from itertools import chain
def levelOrder(self, root: TreeNode) -> List[List[int]]:
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
def bfs(self, root: TreeNode) -> List[int]:
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

# Reference

[Binary Tree Data Structure](https://www.geeksforgeeks.org/binary-tree-data-structure/)

[Introduction to Data Structure Binary Tree](https://leetcode.com/explore/learn/card/data-structure-tree/134/traverse-a-tree/931/)
