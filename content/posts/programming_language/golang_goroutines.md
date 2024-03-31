---
title: 'Write a Large Amount of Data to CSV File with Go (Golang)'
date: '2023-09-09'
categories: ["Programming Language", "Go"]
tags: ["Go", "English Article"]
---

## Introduction

Have you ever found yourself in a situation where you needed to prepare a substantial amount of data for testing your code? If so, you're not alone. Working with large datasets is a common requirement in software development, and Go provides some powerful tools to help you accomplish this efficiently.

In this blog post, I'll share my approach to writing a large amount of data into a CSV file using Go, and I'll demonstrate how to leverage Goroutines to make the process more efficient and faster.

## Writing Data Sequentially

Let's start by addressing the problem of writing data into a CSV file sequentially.

This is a straightforward approach where you iterate through your data and write it one record at a time.

```go
package main

import (
 "encoding/csv"
 "fmt"
 "io"
 "sync"
 "time"

 "github.com/brianvoe/gofakeit"
)

func main() {
    GenerateLargeCSV(10000000, "test")
}

// GenerateLargeCSV generates a CSV file with numRows rows
func GenerateLargeCSV(numRows int, fileName string) {
 err := os.Mkdir("data", 0777)
 if err != nil {
  if !errors.Is(err, os.ErrExist) {
   panic(err)
  }
 }
 file, err := os.Create(fmt.Sprintf("data/%s.csv", fileName))
 if err != nil {
  panic(err)
 }
 defer func() {
  err := file.Close()
  if err != nil {
   log.Printf("error occurred in file.Close() err: %+v", err)
  }
 }()

 writer := csv.NewWriter(file)
 for i := 0; i < numRows; i++ {
  row := generateFakeRow() // Replace with your data generation logic
  if err := writer.Write(row); err != nil {
   panic(err)
  }
//   Flush if it reaches batchSize
//   if i > 0 && i%batchSize == 0 {
//    writer.Flush()
//   }
 }
 writer.Flush()
 if writer.Error() != nil {
  panic(err)
 }
}
```

## Using Goroutines for Parallelism

We can leverage Goroutines. Goroutines allow for concurrent execution of tasks, which can utilize multi-core processors and might accelerate the process.

However, it's essential to note that the standard `csv.Writer` is not thread-safe. If multiple Goroutines attempt to write to the same file concurrently, it can lead to concurrency issues. Therefore, we'll customize the csv.Writer to ensure thread safety.

Here's an example of using Goroutines to write data to a CSV file concurrently in Go:

```go
// Parallelize CSV generation
func GenerateLargeCSVParallelToOneFile(numRows, numGoroutines int, fileName string) {
 err := os.Mkdir("data", 0777)
 if err != nil {
  if !errors.Is(err, os.ErrExist) {
   panic(err)
  }
 }
 file, err := os.Create(fmt.Sprintf("data/%s.csv", fileName))
 if err != nil {
  if !errors.Is(err, os.ErrExist) {
   panic(err)
  }
 }
 defer func() {
  err := file.Close()
  if err != nil {
   log.Printf("error occurred in file.Close() err: %+v", err)
  }
 }()

 var wg sync.WaitGroup
 // Add numGoroutines to the WaitGroup
 wg.Add(numGoroutines)

 // This CSVWriter is thread-safe
 writer, err := NewCSVWriter(file)
 if err != nil {
  panic(err)
 }
 for i := 0; i < numGoroutines; i++ {
  // Call GenerateLargeCSVWithLock in a goroutine for numGoroutines times
  go func(wg *sync.WaitGroup, i int, writer *CsvWriter) {
   GenerateLargeCSVWithLock(numRows, writer)
   // Decrement the WaitGroup counter after each goroutine finishes
   defer wg.Done()
  }(&wg, i, writer)
 }
 // Wait for all goroutines to finish
 wg.Wait()
 fmt.Printf("Done GenerateLargeCSVParallelToOneFile")
}

func GenerateLargeCSVWithLock(numRows int, writer *CsvWriter) {
 for i := 0; i < numRows; i++ {
  row := generateFakeRow()
  if err := writer.Write(row); err != nil {
   panic(err)
  }
//   Flush if it reaches batchSize
//   if i > 0 && i%batchSize == 0 {
//    writer.Flush()
//   }
 }
 writer.Flush()
 if writer.Error() != nil {
  panic(err)
 }
}

// thread-safe csv writer
type CsvWriter struct {
 mutex     *sync.Mutex
 csvWriter *csv.Writer
}

func NewCSVWriter(file io.Writer) (*CsvWriter, error) {
 w := csv.NewWriter(file)
 return &CsvWriter{csvWriter: w, mutex: &sync.Mutex{}}, nil
}

// lock and write
func (w *CsvWriter) Write(row []string) error {
 w.mutex.Lock()
 defer w.mutex.Unlock()
 err := w.csvWriter.Write(row)
 if err != nil {
  return err
 }
 return nil
}

// lock and flush
func (w *CsvWriter) Flush() {
 w.mutex.Lock()
 w.csvWriter.Flush()
 w.mutex.Unlock()
}
```

In this modified code, we've introduced the concept of thread-safe CSV writing using a custom CsvWriter type.

This ensures that multiple Goroutines can write to the same file without concurrency issues.

## Benchmarking

```md
goos: darwin
goarch: amd64
pkg: main/large_csv_generator
cpu: Intel(R) Core(TM) i9-9980HK CPU @ 2.40GHz
BenchmarkGenerateLargeCSV/totalNumRows=100000000-16                    1        109237932720 ns/op
Done GenerateLargeCSVParallelToOneFileBenchmarkGenerateLargeCSV/totalNumRows=100000000,numGoroutines=5-16                      1        104020250631 ns/op
Done GenerateLargeCSVParallelToOneFileBenchmarkGenerateLargeCSV/totalNumRows=100000000,numGoroutines=10-16                     1        103922320588 ns/op
Done GenerateLargeCSVParallelToOneFileBenchmarkGenerateLargeCSV/totalNumRows=100000000,numGoroutines=15-16                     1        105128871237 ns/op
Done GenerateLargeCSVParallelToOneFileBenchmarkGenerateLargeCSV/totalNumRows=100000000,numGoroutines=20-16                     1        107537896218 ns/op
PASS
ok      main/large_csv_generator        530.679s
```

## Conclusion

In conclusion, I have illustrated both sequential and parallel approaches for writing a substantial volume of data to CSV files. While the parallel implementation with Goroutines did not yield a speed improvement in this specific case, it's worth noting that Goroutines are a potent tool that, when employed effectively, can lead to substantial performance enhancements.

For the complete code of this example, please refer to the following link: GitHub Gist.
<https://gist.github.com/moonorange/d09b6a0cd7f265b9a8d1ee5ddadd18e0>
