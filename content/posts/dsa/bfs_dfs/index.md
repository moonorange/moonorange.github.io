---
title: 'BFS and DFS'
date: '2022-06-19'
categories: ["DSA"]
tags: ["English Article", "BFS", "DFS"]
---

# BFS Templates

Write a template in a python-ish pseudo-code for bfs


{{<figure src="https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F182963%2F0d83198d-9e0a-0fb9-ddff-f7b64f1673b7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&w=1400&fit=max&s=5515e347dc5d77374a2c577794b1eaa1" alt="Queue and BFS" width="100%">}}

```python

from collections import deque
def bfs(root, target):
    step = 0
    # Enqueue root node in queue
    q = deque([root])
    # Set to store visited nodes
    visited = set(q)

    while q:
        size = len(q)
        # Iterated the nodes which are already in the queue
        for _ in range(size):
            # Pop the first node in the queue
            curr = q.popleft
            if curr == target: return step
            # Search all adjacent nodes of current node and append them to queue
            for neighbor in (the adjacent nodes of curr):
                if neighbor in visited continue:

                q.append(neighbor)
                visited.add(neighbor)
        # Increment step for each search of the node
        step+=1

    # Return -1 if there is no target node
    return -1
```

# Reference

[BFS (幅優先探索) 超入門！ 〜 キューを鮮やかに使いこなす 〜](https://qiita.com/drken/items/996d80bcae64649a6580#1-4-bfs-%E3%81%AE%E8%A8%88%E7%AE%97%E9%87%8F)

[5.1 Graph Traversals - BFS & DFS -Breadth First Search and Depth First Search](https://www.youtube.com/watch?v=pcKY4hjDrxk)