---
title: Tree
date: '2022-04-08'
categories: ["DSA"]
tags: ["Tree", "Trie"]
---

# Tree

{{<figure src="./tree_datastructure.jpeg" alt="Tree" width="75%">}}

## Properties

- Every tree has a special node called the root node. The root node can be used to traverse every node of the tree. It is called root because the tree originated from root only.
- If a tree has N nodes(vertices), the number of edges is always one less than the number of nodes (i.e., N-1). If it has more than that, it's called a graph not a tree.
- Every child has only a single Parent but Parent can have multiple children.

## Terminology

- The `depth` of a node is the number of edges from the node to the tree's root node. A root node will have a depth of 0.
- The `height` of a node is the number of edges on the longest path from the node to a leaf. A leaf node will have a height of 0.

{{<figure src="./height_depth_tree.png" alt="Tree Height and Depth" width="75%">}}

## Types of Trees

### General Tree

A Tree without any constraint imposed on the hierarchy of the tree is called a general tree.

Each node can have an infinite number of children.

This is the super-set of all other types of trees.

### Binary Tree

Binary Tree is the type of tree in which each parent can have at most two children.

### Binary Search Tree

BST is an extension of BT with some added constraints.

In BST, the value of the left node must be smaller than the parent node, and the value of the right node must be greater than the parent node.

### AVL Tree

AVL Tree is a self-balancing binary tree.

In AVL tree, the heights of children's node differ at most 1, i.e., the valid balancing factor in AVL tree are 1, 0 and -1. When a new node added to a tree and tree becomes unbalanced, rotation is done to make a tree balanced.

![](https://www.thecrazyprogrammer.com/wp-content/uploads/2019/09/AVL-Tree.png?ezimgfmt=ng:webp/ngcb1)

### N-ary Tree

In N-ary Tree, the maximum number of children that a node can have is limited to N.

### Tries

It is commonly used to represent a dictionary for looking up words in a vocabulary

For example, when implementing a search bar with auto-completion or query suggestion, tries is needed to look up words quickly starting with the characters input by the user.

{{<figure src="./query_suggestion.png" alt="Query Suggestion" width="100%">}}

# Ref

[Types of Trees in Data Structure](https://www.thecrazyprogrammer.com/2019/09/types-of-trees-in-data-structure.html)

[MIT Lecture 6: AVL Trees, AVL Sort](https://www.youtube.com/watch?v=FNeL18KsWPc)

[Tries](https://albertauyeung.github.io/2020/06/15/python-trie.html/)
