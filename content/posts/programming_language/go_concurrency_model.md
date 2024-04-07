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

In go you can use goroutine like below with go keyword.

```go
package main

import (
	"fmt"
)

func somefunc() {
    fmt.Println("called by goroutine")
}

func main() {
    go somefunc()
    // Wait somefunc not to exit main goroutine before executing somefunc
    time.Sleep(100 * time.Millisecond)
}
```

## Channels

A channel provides a way to connect between go routines, allowing them to communicate and synchronize with each other.

Example

```go
var chan int
c := make(chan int)
// Sending on a channel
c <- 1

// Receiving from a channel
val = <-c
```

Example with goroutine

```go
package main

import (
	"fmt"
	"time"
)

func someFunc(word string, c chan string) {
    time.Sleep(100 * time.Millisecond)
	c <- word
}

func main() {
	c := make(chan string)
	go someFunc("word", c)
    // It will wait until the channel sends some data
	fmt.Printf("from channel %s\n", <-c)
}
```

## Select



## References

[Google I/O 2012 - Go Concurrency Patterns
](https://www.youtube.com/watch?v=f6kdp27TYZs)

[Google I/O 2013 - Advanced Go Concurrency Patterns
](https://www.youtube.com/watch?v=QDDwwePbDtw)
