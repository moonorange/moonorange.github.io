---
title: 'Implementation of Job Queue model using goroutine and channel'
date: '2024-04-11'
categories: ["Programming Language", "Go", "Concurrency"]
tags: ["Go", "English Article"]
---

## Introduction

In concurrent programming, managing tasks efficiently is crucial. One common pattern is the job queue model, where multiple tasks (jobs) are submitted to a queue and processed asynchronously by worker routines. In this blog post, we'll explore how to implement a job queue model in Go using goroutines and channels.

## Understanding the Job Queue Model

At its core, the job queue model consists of two main components:

- Job Queue: The job queue is a data structure that holds tasks awaiting execution. When a new task is submitted, it is added to the queue. Worker routines continuously monitor the queue for incoming tasks and execute them as they become available.

- Workers: Workers are concurrent routines responsible for executing tasks retrieved from the job queue. These routines are typically spawned when the application starts and continue running in the background, processing tasks as they arrive. By employing multiple workers, the system can handle a higher volume of tasks concurrently, improving overall efficiency and throughput.

## Implementation in Go

Let's dive into the implementation using Go's powerful concurrency primitives: goroutines and channels.

main.go

```go
package main

import (
	"context"
	"fmt"
	"os"
	"os/signal"
	"sync"
	"syscall"
)

const (
	totalJobs      int = 100
	maxNumWorkers      = 100
	jobsPerRoutine     = totalJobs / maxNumWorkers
)

func main() {
	// Create a cancellation context to allow graceful shutdown
	ctx, cancel := context.WithCancel(context.Background())

	// Create a channel to receive OS signals
	sigCh := make(chan os.Signal, 1)
	defer close(sigCh)

	// Notify sigCh channel
	signal.Notify(sigCh, syscall.SIGTERM, syscall.SIGTERM, syscall.SIGQUIT, syscall.SIGINT)
	go func() {
		// Wait until receiving the signal
		<-sigCh
		// Cancel the context to propagate cancellation through the context tree
		cancel()
	}()

	p := NewAPIWorker()
	d := NewDispatcher(p, 10, 100)
	// Start the dispatcher with the cancellation context
	go d.Start(ctx)

	// WaitGroup to wait for all goroutines to finish
	var wg sync.WaitGroup
	wg.Add(maxNumWorkers)
	concurrentEnqueue(d, &wg)

	// enqueue(d, 0, totalJobs)

	// Wait for all goroutines to finish
	wg.Wait()

	fmt.Println("All enqueue jobs completed.")

	// Wait for the dispatcher to finish processing all jobs
	d.Wait()
	fmt.Println("Finished!")
}

// enqueue enqueues jobs into the dispatcher within the specified range
func enqueue(d *Dispatcher, start, end int) {
	for i := start; i < end; i++ {
		payload := fmt.Sprintf("dummy_payload_%d", i)
		job := &Job{Payload: payload}
		d.Enqueue(job)
	}
}

// concurrentEnqueue enqueues jobs concurrently using multiple goroutines
func concurrentEnqueue(d *Dispatcher, wg *sync.WaitGroup) {
	for i := 0; i < maxNumWorkers; i++ {
		start := i * jobsPerRoutine
		end := start + jobsPerRoutine
		// Handle remaining jobs for the last goroutine
		if i == maxNumWorkers-1 {
			end = totalJobs
		}
		fmt.Printf("start: %d end: %d\n", start, end)
		// Launch a goroutine to enqueue jobs within the specified range
		go func() {
			defer wg.Done()
			enqueue(d, start, end)
		}()
	}
}
```

worker.go

```go
package main

import (
	"fmt"
	"io"
	"math/rand"
	"net/http"
	"net/http/httptest"
	"strings"
	"time"
)

// APIWorker sends post requests
type APIWorker struct{}

func NewAPIWorker() *APIWorker {
	return &APIWorker{}
}

// Work waits for a few seconds and print a received URL.
func (p *APIWorker) Work(job *Job) {
	t := time.NewTimer(time.Duration(rand.Intn(5)) * time.Second)
	defer t.Stop()
	<-t.C

	server := mockServer()
	// Make a fake HTTP request to the mock server
	resp, err := http.Post(server.URL, "application/json", strings.NewReader(fmt.Sprintf(`{"key": %s}`, job.Payload)))
	if err != nil {
		// Output error logs so that failed payload can be retried
		fmt.Printf("Error ocurred with payload %s: %+v", job.Payload, err)
		return
	}
	defer resp.Body.Close()
}

func mockServer() *httptest.Server {
	// Create a new HTTP test server
	server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Read the request body
		body, err := io.ReadAll(r.Body)
		if err != nil {
			http.Error(w, "Failed to read request body", http.StatusInternalServerError)
			return
		}
		defer r.Body.Close()

		// Print the request body (fake processing)
		fmt.Println("Received POST request with body:", string(body))
	}))
	return server
}
```

dispatcher.go

```go
package main

import (
	"context"
	"sync"
)

// Job represents an interface of a job that can be enqueued into a dispatcher.
type Job struct {
	Payload string
}

type Worker interface {
	Work(j *Job) // Work method defines the behavior of processing a job.
}

// Dispatcher represents a job dispatcher.
type Dispatcher struct {
	workerPool chan struct{}  // Semaphore for limiting concurrent worker goroutines.
	jobQueue   chan *Job      // Channel for queuing incoming jobs.
	worker     Worker         // Worker interface for processing jobs.
	globalWg   sync.WaitGroup // WaitGroup for tracking the termination of the dispatcher.
}

// NewDispatcher creates a new instance of a job dispatcher with the given parameters.
func NewDispatcher(worker Worker, maxWorkers int, maxQueueSize int) *Dispatcher {
	return &Dispatcher{
		workerPool: make(chan struct{}, maxWorkers), // Buffered channel acting as a workerPool. Use empty struct to minimize the memory allocation
		jobQueue:   make(chan *Job, maxQueueSize),
		worker:     worker,
	}
}

// Start initiates the dispatcher to begin processing jobs.
// The dispatcher stops when it receives a value from `ctx.Done`.
func (d *Dispatcher) Start(ctx context.Context) {
	// Increment the wait group counter to indicate that the dispatcher has started processing jobs.
	d.globalWg.Add(1)

	var wg sync.WaitGroup

	// Main loop for processing jobs.
	for {
		select {
		case <-ctx.Done():
			// Block until all currently processing jobs have finished.
			wg.Wait()
			// When the loop exits (due to context cancellation), stop decrements the wait group counter to indicate that the dispatcher has stopped processing jobs
			d.globalWg.Done()
			return
		case job := <-d.jobQueue:
			// Increment the local wait group to track the processing of this job.
			wg.Add(1)
			// Push to the workerPool to control the number of concurrent workers.
			d.workerPool <- struct{}{}
			// Process the job concurrently.
			go func(job *Job) {
				defer wg.Done()
				// After the job finishes, release the slot in the workerPool.
				defer func() { <-d.workerPool }()
				d.worker.Work(job)
			}(job)
		}
	}
}

// Wait blocks until the dispatcher stops.
func (d *Dispatcher) Wait() {
	d.globalWg.Wait()
}

// Enqueue puts a job into the queue.
// If the number of enqueued jobs has already reached the maximum size,
// This will block until space becomes available in the queue to accept a new job.
func (d *Dispatcher) Enqueue(job *Job) {
	d.jobQueue <- job
}
```



## Conclusion

Implementing a job queue model using goroutines and channels in Go provides a simple yet powerful way to manage and execute tasks concurrently.

By leveraging Go's concurrency primitives, we can build efficient and scalable systems that handle large workloads effectively.

This implementation serves as a foundation for building various concurrent applications, such as web servers, background job processors, and more.

In summary, understanding and mastering concurrency concepts in Go opens up a world of possibilities for building high-performance and resilient software systems.
