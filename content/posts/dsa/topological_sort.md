---
title: 'Topological Sort'
date: '2022-09-25'
categories: ["DSA"]
tags: ["Graph", "English Article"]
---

## What is Topological Sort?

The topological sort is an ordering of nodes such that every node appears before all the nodes it points to.

The canonical application of the algorism is dependency resolution.

## Coding problem using Topological Sort

```python
class Solution:
    def findOrder(self, numCourses: int, prerequisites: List[List[int]]) -> List[int]:
        graph = defaultdict(list)
        for crs, prereq in prerequisites:
            graph[crs].append(prereq)
        visited, cycle = set(), set()
        def dfs(crs):
            # if there is a cycle, return False
            if crs in cycle:
                return False
            # if it's already traversed
            if crs in visited:
                return True
            # add a node before traversing to check a cycle
            cycle.add(crs)
            for pre in graph[crs]:
                if not dfs(pre):
                    return False
            # remove the node from a cycle check set after traversing
            cycle.remove(crs)
            # check the node visited after traversing
            visited.add(crs)
            # add the node to a result after traversing
            result.append(crs)
            return True
        for c in range(numCourses):
            # return [] if there is a cycle
            if not dfs(c):
                return []
        return result
```

## References

[Topological Sort Graph Algorithm](https://www.youtube.com/watch?v=ddTC4Zovtbc)
