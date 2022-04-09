---
title: 'Linked List'
date: '2021-05-14'
categories: ["DSA"]
tags: ["Linked List", "Python", "English Article"]
---

# Introduction

The linked list is a linear data structure.

There are two types of linked lists, the singly linked list and the doubly linked list.

## Linked list

The linked list elements are not stored at a contiguous location; the elements are linked using pointers.

Advantages over arrays

1. Dynamic size
2. Ease of insertion/deletion (Need to move all elements after targeted element in array)

Implementation

```python
class Node(object):
    def __init__(self, val):
        self.val: int = val
        self.next: Node = None


class SinglyLinkedList:
    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.head: Node = None
        self.size: int = 0


    def get(self, index: int) -> int:
        """
        Get the value of the index-th node in the linked list. If the index is invalid, return -1.
        """
        if index < 0 or index >= self.size:
            return -1
        current = self.head
        for _ in range(index):
            current = current.next
        return current.val


    def addAtHead(self, val: int) -> None:
        """
        Add a node of value val before the first element of the linked list. After the insertion,           the new node will be the first node of the linked list.
        """
        self.addAtIndex(0, val)


    def addAtTail(self, val: int) -> None:
        """
        Append a node of value val to the last element of the linked list.
        """
        self.addAtIndex(self.size, val)


    def addAtIndex(self, index: int, val: int) -> None:
        """
        Add a node of value val before the index-th node in the linked list. If index equals to the         length of linked list, the node will be appended to the end of linked list. If index is             greater than the length, the node will not be inserted.
        """
        if index < 0 or index > self.size:
            return

        current = self.head
        new_node = Node(val)
        if index == 0:
            new_node.next = current
            self.head = new_node
        else:
            for _ in range(index - 1):
                current = current.next
            new_node.next = current.next
            current.next = new_node

        self.size += 1


    def deleteAtIndex(self, index: int) -> None:
        """
        Delete the index-th node in the linked list, if the index is valid.
        """
        if index < 0 or index >= self.size:
            return

        current = self.head
        if index == 0:
            self.head = current.next
        else:
            for _ in range(index - 1):
                current = current.next
            current.next = current.next.next
        self.size -= 1
```

## Doubly Linked List

Advantages over singly linked list

1. A DLL can be traversed in both forward and backward directions.
2. The delete operation in DLL is more efficient if a pointer to the node to be deleted is given.
3. We can quickly insert a new node before a given node.
   In a singly linked list, to delete a node, a pointer to the previous node is needed. To get this previous node, sometimes the list is traversed. In DLL, we can get the previous node using a previous pointer.

Disadvantages over singly linked list

1. Every node of DLL requires extra space for a previous pointer. It is possible to implement DLL with a single pointer though (See [this](https://www.geeksforgeeks.org/xor-linked-list-a-memory-efficient-doubly-linked-list-set-1/)).
2. All operations require an extra pointer previous to be maintained. For example, in insertion, we need to modify previous pointers together with next pointers. For example in following functions for insertions at different positions, we need 1 or 2 extra steps to set a previous pointer.

```python
class Node(object):
    def __init__(self, val):
  self.val: int = val
  self.prev: Node = None
  self.next: Node = None

class DoublyLinkedList:
    def __init__(self):
  """
  Initialize your data structure here.
  """
  self.head: Node = None
  self.tail: Node = None
  self.size: int = 0

    def get(self, index: int) -> int:
  """
  Get the value of the index-th node in the linked list. If the index is invalid, return -1.
  """
  if index < 0 or index >= self.size:
   return - 1
  current = self.head
  for _ in range(index):
   current = current.next
  return current.val

    def addAtHead(self, val: int) -> None:
        """
        Add a node of value val before the first element of the linked list. After the insertion,           the new node will be the first node of the linked list.
        """
        self.addAtIndex(0, val)


    def addAtTail(self, val: int) -> None:
        """
        Append a node of value val to the last element of the linked list.
        """
        self.addAtIndex(self.size, val)

    def addAtIndex(self, index: int, val: int) -> None:
        """
        Add a node of value val before the index-th node in the linked list. If index equals to the         length of linked list, the node will be appended to the end of linked list. If index is             greater than the length, the node will not be inserted.
        """
        if index < 0 or index > self.size:
            return

        current = self.head
        new_node = Node(val)
        if index == 0:
            new_node.next = current
            self.head = new_node
        else:
            for _ in range(index - 1):
                current = current.next
            new_node.next = current.next
            current.next = new_node

        self.size += 1

    def deleteAtIndex(self, index: int) -> None:
        """
        Delete the index-th node in the linked list, if the index is valid.
        """
        if index < 0 or index >= self.size:
            return

        current = self.head
        if index == 0:
            self.head = current.next
        else:
            for _ in range(index - 1):
                current = current.next
            current.next = current.next.next
        self.size -= 1
```

# Reference

[Linked List Data Structure](https://www.geeksforgeeks.org/data-structures/linked-list/)

[Introduction to Data Structure Linked List Singly Linked List](https://leetcode.com/explore/learn/card/linked-list/209/singly-linked-list/)
