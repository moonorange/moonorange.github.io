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
    // Wait a while not to exit main goroutine before finishing another goroutine
    time.Sleep(100 * time.Millisecond)
}
```

## Channels

A channel provides a way to connect between go routines, allowing them to communicate and synchronize with each other.

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

Example2

```go
package main

import "fmt"

func join(s []string, c chan string) {
	joined := ""
	for _, v := range s {
		joined += v
	}
    // send joined string to a channel
	c <- joined
}

func main() {
	strings := []string{"a", "b", "c", "d", "e", "f"}
	c := make(chan string)
    // join the first half of the strings
	go join(strings[:len(strings)/2], c)
    // join the second half of the strings
	go join(strings[len(strings)/2:], c)
    // Receives from a channel, the order of data is unknown
	joined1, joined2 := <-c, <-c
	fmt.Println(joined1, joined2)
}

```

## Buffered Channel

If you provide the buffer length as the second argument, a channel can be buffered

When you send data to a buffered channel, the send operation will block only if the buffer is full

When you receive data from a buffered channel, the receive operation will block if the buffer is empty

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int, 3)
	ch <- 1
	ch <- 2
	go func() {
		time.Sleep(5 * time.Second)
		ch <- 3
	}()
	fmt.Println(<-ch)
	fmt.Println(<-ch)
    // Wait until a channel is not empty
	fmt.Println(<-ch)
}
```

## Select



## References

[Google I/O 2012 - Go Concurrency Patterns
](https://www.youtube.com/watch?v=f6kdp27TYZs)

[Google I/O 2013 - Advanced Go Concurrency Patterns
](https://www.youtube.com/watch?v=QDDwwePbDtw)
