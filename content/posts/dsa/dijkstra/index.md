---
title: 'Dijkstra’s Shortest Path Algorithm'
date: '2022-09-28'
categories: ["DSA"]
tags: ["Graph", "English Article"]
---

## What is Dijkstra’s Algorithm

Dijkstra's Algorithm is the algorithm to find the shortest path between any two vertices in a graph.

Dijkstra's Algorithm will find the shortest path from a given starting vertex to every other vertices in a graph.

{{<figure src="./dijkstra.png" alt="Dijkstra’s Algorithm" width="75%">}}

## Steps

Prepare a table to have the shortest distance from a starting vertex and previous vertex.

Initialize two list of visited and unvisited nodes

Let the distance of a start vertex from the start vertex 0.

Let the distance of all other vertices infinity

Repeat

- Visit the unvisited vertex with the smallest known distance from the start vertex
- Check its unvisited neighbors
- Calculate the distance of each neighbor from the start vertex
- If the calculated distance is smaller than the known distance on the table, update the shortest distance
- Update the previous vertex for each of the update distances
- Add the current vertex to the list of visited vertices

Until all vertices visited

## References

[Dijkstra Algorithm - Single Source Shortest Path - Greedy Method](https://www.youtube.com/watch?v=XB4MIexjvY0)

[Graph Data Structure 4. Dijkstra’s Shortest Path Algorithm](https://www.youtube.com/watch?v=pVfj6mxhdMw)
