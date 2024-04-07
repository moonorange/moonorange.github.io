---
title: 'Go Concurrency Model'
date: '2024-04-07'
categories: ["Programming Language", "Go", "Concurrency"]
tags: ["Go", "English Article"]
---

## Intro

This article aims to summarize what I have learnt about Go's concurrency model

## Concurrency and Parallelism

## Goroutines

goroutine is a cheap independently executing function, which has its own call stack.

goroutine is not a thread, a thread can have multiple goroutines.
However, for the point of understanding, it's not far off to think of it as a very cheap thread.

## Channels

A channel provides a way to connect between go routines, allowing them to communicate and synchronize with each other

## References

[Google I/O 2012 - Go Concurrency Patterns
](https://www.youtube.com/watch?v=f6kdp27TYZs)

[Google I/O 2013 - Advanced Go Concurrency Patterns
](https://www.youtube.com/watch?v=QDDwwePbDtw)
