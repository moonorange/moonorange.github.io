---
title: 'Go Concurrency Model'
date: '2024-04-07'
categories: ["Programming Language", "Go", "Concurrency"]
tags: ["Go", "English Article"]
---

## Intro

This article aims to summarize the key concepts and principles of Go's concurrency model that I've learned.

To test your understanding and reinforce your knowledge, a quiz is available at the end of the article.

## Goroutines

A goroutine is a lightweight independently executing function with its own call stack.

While not equivalent to a thread, conceptually, it can be thought of as a very cheap thread.

Here's how you can use goroutines:

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
    // Usually you should use sync.WaitGroup to wait for goroutines to finish instead of time.Sleep
    time.Sleep(100 * time.Millisecond)
}
```

## Channels

Channels provide a way for goroutines to communicate and synchronize with each other.

Here are some examples:

Example 1:

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

Example 2:

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

### Buffered Channel

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
    // Wait until a channel is filled
	fmt.Println(<-ch)
}
```

### Range and Close

You can use range with channels to repeatedly receive values until the channel is closed

Closing a channel indicates that no more values will be sent, allowing the receiver to know when to stop waiting for new data.

Unlike files, you usually don't need to close channels.

Here's an example.

```go
package main

import (
	"fmt"
)

func loop(n int, c chan int) {
	for i := 0; i < n; i++ {
		c <- i
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go loop(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

## Select

The select statement allows a goroutine to wait until one of the cases is ready.

It randomly chooses one if multiple cases are ready.

Here's an example:

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func loop(ctx context.Context, c chan int) {
	for {
		select {
		// Quit if context got canceled
		case <-ctx.Done():
			fmt.Println("quit")
			return
		// Print values received from the channel
		case v := <-c:
			fmt.Println(v)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	c := make(chan int)

	go func() {
		for i := 0; i < 10; i++ {
			c <- i
		}
	}()
	go func() {
		time.Sleep(time.Second * 1)
		cancel()
	}()
	loop(ctx, c)
}
```

## Want to learn more?

Challenge yourself with quizzes available in this [repository](https://github.com/moonorange/go-quizzes/tree/main/advanced/concurrency/quizzes).

## References

[Google I/O 2012 - Go Concurrency Patterns
](https://www.youtube.com/watch?v=f6kdp27TYZs)

[Google I/O 2013 - Advanced Go Concurrency Patterns
](https://www.youtube.com/watch?v=QDDwwePbDtw)

[A Tour Of Go](https://go.dev/tour/concurrency/1)
