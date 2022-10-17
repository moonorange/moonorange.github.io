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

Implementation in Python

```python
from graph import Graph
import sys

nodes = [
    "Reykjavik",
    "Oslo",
    "Moscow",
    "London",
    "Rome",
    "Berlin",
    "Belgrade",
    "Athens",
]

init_graph = {}
for node in nodes:
    init_graph[node] = {}
init_graph["Reykjavik"]["Oslo"] = 5
init_graph["Reykjavik"]["London"] = 4
init_graph["Oslo"]["Berlin"] = 1
init_graph["Oslo"]["Moscow"] = 3
init_graph["Moscow"]["Belgrade"] = 5
init_graph["Moscow"]["Athens"] = 4
init_graph["Athens"]["Belgrade"] = 1
init_graph["Rome"]["Berlin"] = 2
init_graph["Rome"]["Athens"] = 2

graph = Graph(nodes, init_graph)


def dijkstra_algorithm(graph, start_node):
    unvisited = set(graph.get_nodes())
    shortest_path = {}
    previous_nodes = {}

    inf = sys.maxsize
    # Initialize shortest path with max
    for node in unvisited:
        shortest_path[node] = inf
    # Let the starting node 0
    shortest_path[start_node] = 0

    # while there is any unvisited node
    while unvisited:
        current_min_node = None
        # Find the node with the lowest value
        # It will get the starting node as its value is 0 for the first time
        for node in unvisited:
            if current_min_node is None:
                current_min_node = node
            elif shortest_path[node] < shortest_path[current_min_node]:
                current_min_node = node

        # Get outgoing edges from the current min node
        neighbors = graph.get_outgoing_edges(current_min_node)
        for neighbor in neighbors:
            # shortest path to the current min node + distance from current min node to neighbor
            temp = shortest_path[current_min_node] + graph.value(
                current_min_node, neighbor
            )
            if temp < shortest_path[neighbor]:
                # If it's the shorter path, update the shortest path table
                shortest_path[neighbor] = temp
                # Update the previous node table as well
                previous_nodes[neighbor] = current_min_node
        # Once you have checked all neighbor for the current min node, remove current min node from unvisited set
        unvisited.remove(current_min_node)
    return previous_nodes, shortest_path
```

## References

[Dijkstra Algorithm - Single Source Shortest Path - Greedy Method](https://www.youtube.com/watch?v=XB4MIexjvY0)

[Graph Data Structure 4. Dijkstra’s Shortest Path Algorithm](https://www.youtube.com/watch?v=pVfj6mxhdMw)

[Implementing Dijkstra’s Algorithm in Python](https://www.udacity.com/blog/2021/10/implementing-dijkstras-algorithm-in-python.html)
