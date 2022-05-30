---
title: 'Stack and Queue'
date: '2022-05-21'
categories: ["DSA"]
tags: ["Stack", "Queue", "English Article"]
---

## Queue

FIFO

```go
package queue

import "sync"

type node struct {
 data interface{}
 next *node
}

type Queue struct {
 head *node
 tail *node
 count int
 lock *sync.Mutex
}

func NewQueue() *Queue {
 return &Queue{lock: &sync.Mutex{}}
}

func (q *Queue) Len() int {
 q.lock.Lock()
 defer q.lock.Unlock()

 return q.count
}

func (q *Queue) Push(data interface{}) {
 q.lock.Lock()
 defer q.lock.Unlock()

 ele := &node{data: data}
 if q.head == nil {
  q.head = ele
  q.tail = ele
 } else {
  q.tail.next = ele
  q.tail = ele
 }
 q.count++
}

func (q *Queue) Poll() interface{} {
 q.lock.Lock()
 defer q.lock.Unlock()

 if q.head == nil {
  return nil
 }

 res := q.head
 q.head = res.next
 if q.head == nil {
  q.tail = nil
 }

 q.count--
 return res.data
}

func (q *Queue) Peek() interface{} {
 q.lock.Lock()
 defer q.lock.Unlock()

 if q.head == nil {
  return nil
 }
 return q.head.data
}
```

## Stack

LIFO

```go
package stack

import "sync"

type node struct {
 value interface{}
 next *node
}

type Stack struct {
 head *node
 count int
 lock *sync.Mutex
}

func NewStack() *Stack {
 return &Stack{lock: &sync.Mutex{}}
}

func (s *Stack) Len() int {
 s.lock.Lock()
 defer s.lock.Unlock()

 return s.count
}

func (s *Stack) Peek() interface{} {
 s.lock.Lock()
 defer s.lock.Unlock()

 if s.head == nil {
  return nil
 }
 return s.head.value
}

func (s *Stack) Pop() interface{} {
 s.lock.Lock()
 defer s.lock.Unlock()

 if s.head == nil {
  return nil
 }

 res := s.head
 s.head = res.next
 s.count--
 return res.value
}

func (s *Stack) Push(value interface{}) {
 s.lock.Lock()
 defer s.lock.Unlock()

 node := &node{value: value}
 if s.head == nil {
  s.head = node
 } else {
  node.next = s.head
  s.head = node
 }
 s.count++
}
```

## Circular Queue

Circular Queue is an extend version of regular queue where the last element is connected to the first element so that it makes a circle

Dequeueing elements from fixed sized regular queue makes empty space in memory. That's where circular queue was invented so that it reuses the empty memory as the rear has the pointer to the first element.$class

{{<figure src="https://www.simplilearn.com/ice9/free_resources_article_thumb/Circular_link_resolving_problem_of_MemoryWastage.png" alt="Circular Queue" width="100%">}}

## Reference

[Design Circular Queue](https://leetcode.com/explore/learn/card/queue-stack/228/first-in-first-out-data-structure/1337/)

[Circular Queue in Data Structure: Overview, Implementation Using Array and Linked List](https://www.simplilearn.com/tutorials/data-structure-tutorial/circular-queue-in-data-structure)
