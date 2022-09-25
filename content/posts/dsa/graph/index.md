---
title: 'Intro to Graphs'
date: '2022-08-07'
categories: ["DSA"]
tags: ["Graph", "English Article"]
---

## What is a Graph?

A graph G is an ordered pair of a set V of vertices and a set of E of edges.

G can be defines as follows

$$G=\left(V,E\right)$$

ordered pair

$$(a, b) \neq \left(b,a\right) \text{where} \ a \neq b$$

{{<figure src="./graph.png" alt="Graph" width="100%">}}

## Graph terminology

There are two type of graph, the one is `directed graph` and the other is `undirected graph`.

Directed graphs contain ordered pairs of vertices while undirected graphs contain unordered pairs of vertices.

`Directed Graph`

{{<figure src="./directed_graph.png" alt="Directed Graph" width="100%">}}

`Undirected Graph`

{{<figure src="./undirected_graph.png" alt="Directed Graph" width="100%">}}

The number of edges connected to/from a node is called `degree`.

In directed graph, the number of edges to a node is called `indegree`, whereas the number of edges from a node is called `outdegree`.

The image below illustrates indegree(red color) and outdegree(blue color)

{{<figure src="./indegree_outdegree.png" alt="Directed Graph" width="100%">}}


`Weighted Graph` has weight/cost associated with every edge, while `Unweighted Graph` has the same weight in all edges.

Check [BFS and DFS](/posts/dsa/bfs_dfs/) for graph traversal

## Reference

[Data structures: Introduction to graphs](https://www.youtube.com/watch?v=gXgEDyodOJU)

[グラフ理論入門](https://dev.classmethod.jp/articles/graph-theory/)

[5.1 Graph Traversals - BFS & DFS -Breadth First Search and Depth First Search](https://www.youtube.com/watch?v=pcKY4hjDrxk)
