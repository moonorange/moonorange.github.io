---
title: 'Implementation of Merge Sort in Python'
date: '2023-10-05'
categories: ["DSA"]
tags: ["Merge Sort", "Sort", "English Article", "Python"]
---

## Merge Sort

In sorting n objects, merge sort has an average and worst-case performance of O(n log n).

Merge sort is a stable sort in the most implementations.
The most common implementation does not sort in place, meaning it requires extra space.


{{<figure src="./merge_sort_algorithm.png" alt="Merge sort diagram" width="75%">}}


## Implementation in Python

```python
from typing import List

# Python merge sort implementation

# Time complexity: O(NlogN), space complexity: O(N)

def merge_sort(nums: List[int]):
    # Base case
    if len(nums) <= 1:
        return nums

    mid = len(nums) // 2
    left, right = merge_sort(nums[:mid]), merge_sort(nums[mid:])
    return merge(left, right)

def merge(left: List[int], right: List[int]):
    i, j = 0, 0
    res = []
    # Compare the elements in the left and right list and append the smaller one to the result list
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            res.append(left[i])
            i += 1
        else:
            res.append(right[j])
            j += 1

    # add the rest of the list if any elements in the left remains
    if i < len(left):
        res.extend(left[i:])
    # add the rest of the list if any elements in the right remains
    if j < len(right):
        res.extend(right[j:])

    return res

# 1. Empty list

assert merge_sort([]) == []

# 2. One element list

assert merge_sort([1]) == [1]

# 3. Two element list

assert merge_sort([2, 1]) == [1, 2]

# 4. More than two element list

assert merge_sort([3, 2, 1]) == [1, 2, 3]

# 5. Duplicate elements

assert merge_sort([3, 2, 1, 2]) == [1, 2, 2, 3]

# 6. Negative elements

assert merge_sort([-3, -2, -1]) == [-3, -2, -1]

# 7. Mixed elements

assert merge_sort([-3, 2, -1]) == [-3, -1, 2]

# 8. More complex case

assert merge_sort([3, 2, 1, 2, -3, -2, -1]) == [-3, -2, -1, 1, 2, 2, 3]
print("All tests passed!")
```
