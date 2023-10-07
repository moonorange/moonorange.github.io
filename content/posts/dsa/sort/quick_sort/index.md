---
title: 'Implementation of Quick Sort in Python'
date: '2023-10-05'
categories: ["DSA"]
tags: ["Quick Sort", "Sort", "English Article", "Python"]
---


## Quick Sort

In sorting n objects, quick sort has an average performance of O(n log n) and a worst performance of O(n^2).

Most implementations of quick sort are not stable, but sort in place, meaning it does not require extra space.

## Implementation in Python

```python
# O(NLogN), space complexity O(n) in this case
def quick_sort(nums):
    if len(nums) <= 1:
        return nums

    pivot = nums[len(nums) // 2]
    left = [n for n in nums if n < pivot]
    mid = [n for n in nums if n == pivot]
    right = [n for n in nums if n > pivot]

    return quick_sort(left) + mid + quick_sort(right)

# Test cases

if __name__ == "__main__":
    # Test case 1: Sorting an empty list
    arr1 = []
    result1 = quick_sort(arr1)
    assert result1 == [], "Test case 1 failed"

    # Test case 2: Sorting a list with one element
    arr2 = [5]
    result2 = quick_sort(arr2)
    assert result2 == [5], "Test case 2 failed"

    # Test case 3: Sorting a list with multiple elements
    arr3 = [3, 6, 1, 8, 2, 4, 5, 7]
    result3 = quick_sort(arr3)
    assert result3 == [1, 2, 3, 4, 5, 6, 7, 8], "Test case 3 failed"

    # Test case 4: Sorting a list with duplicate elements
    arr4 = [3, 6, 1, 8, 2, 4, 5, 7, 1, 2]
    result4 = quick_sort(arr4)
    assert result4 == [1, 1, 2, 2, 3, 4, 5, 6, 7, 8], "Test case 4 failed"

    # Test case 5: Sorting a list with duplicate pivot elements
    arr5 = [3, 6, 1, 5, 4, 4, 5, 7, 1, 2]
    result5= quick_sort(arr5)
    assert result5 == [1, 1, 2, 3, 4, 4, 5, 5, 6, 7], "Test case 5 failed {0}".format(result5)

    print("All test cases passed!")
```
