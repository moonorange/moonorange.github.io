---
title: 'Learn gRPC, Go and Kubernetes by building Microservices'
date: '2024-04-20'
categories: ["Projects", "Go", "Microservice", "gRPC", "Kubernetes"]
tags: ["Go", "English Article", "Kubernetes"]
---

# Intro

In this article, we embark on a journey to implement a Backend For Frontend (BFF) and microservices architecture utilizing gRPC and GraphQL. 

This project serves as an educational endeavor to understand the intricacies of microservices communication and the benefits of using gRPC and GraphQL.

# Project Structure(Overview)

Our project revolves around three essential components:

1. Backend For Frontend (BFF): This layer acts as an intermediary between the frontend and the microservices. It handles frontend-specific logic, aggregating data from various microservices and presenting it to the client.

2. Microservices (gRPC): We’ll implement two microservices adhering to the Command Query Responsibility Segregation (CQRS) pattern:
Command Service: Responsible for executing write operations, handling commands, and modifying data.
Query Service: Retrieves data and responds to read queries.

3. Kubernetes for Orchestration and Deployment: We’ll leverage Kubernetes to manage our microservices, ensuring scalability, resilience, and efficient resource utilization.

# Microservices

The CQRS pattern separates read and write responsibilities within our system.

While it’s not necessary for small projects, understanding it can enhance your architectural knowledge.

We will use gRPC for communication between services.

## gRPC

gRPC, developed by Google, is an open-source remote procedure call (RPC) framework. It offers the following features:

- Interface Definition with Protocol Buffers: gRPC uses Protocol Buffers as its interface definition language. This ensures clear communication between services.
- HTTP/2 Transport: gRPC utilizes HTTP/2, providing efficient, high-performance communication.
- Remote Method Invocation: Services can invoke methods on remote servers as if they were local function calls.

In summary, gRPC simplifies communication between distributed systems, making it an excellent choice for microservices architectures.

## Protocol Buffers (protobuf)

We’ll define our service interfaces and messages using [Protocol Buffers (protobuf)](https://protobuf.dev/).

These language-agnostic schemas allow us to precisely specify the structure of our data.

Define protobuf

This is a part of the protobuf

```proto
syntax = "proto3";

import "google/protobuf/empty.proto";

// Define a directory to put auto generated codes
option go_package = "gen";

package task;

service TaskService {
  rpc CreateTask(CreateTaskRequest) returns (CreateTaskResponse);
  rpc GetTask(GetTaskRequest) returns (GetTaskResponse);
}

message Task {
  int32 id = 1;
  string text = 2;
  repeated string tags = 3;
  int64 due = 4;
}

message CreateTaskRequest {
  string text = 1;
  repeated string tags = 2;
  int64 due = 3;
}

message CreateTaskResponse {
  Task task = 1;
}

message GetTaskRequest {
  string task_id = 1;
}

message GetTaskResponse {
  Task task = 1;
}
```

After defining protobuf, we can generate code automatically from protobuf.

I will use [buf](https://github.com/bufbuild/buf) to manage protobuf in this project

Refer to the official document to leann how to install and use buf.

Generate Go code from protobuf

```sh
cd proto
buf generate proto
```

## Implementing gRPC server

Let's implement gRPC server

As I explained, we will implement two gRPC servers, Command Service and Query Service.

### Run gRPC server

The following code will creates a gRPC server and run it locally.

It's the same in both services.

```go
package main

import (
	"fmt"
	"net"
	"os"
	"os/signal"
	"syscall"

	"github.com/sirupsen/logrus"
	"google.golang.org/grpc"
)

func main() {
	// Create a listener on TCP port
	port := 8081
	listener, err := net.Listen("tcp", fmt.Sprintf(":%d", port))
	if err != nil {
		logrus.Fatal("failed to listen: ", err)
	}

	// Create a gRPC server
	s := grpc.NewServer()

	// Listen for OS signals to stop the server when receiving a cancel signal
	sigChan := make(chan os.Signal, 1)
	signal.Notify(sigChan, os.Interrupt, syscall.SIGINT, syscall.SIGTERM)

	// Start the gRPC server
	go func() {
		if err := s.Serve(listener); err != nil {
			logrus.Fatal("failed to serve: ", err)
		}
		logrus.Printf("Command service is running on port %d", port)
	}()

	// Wait for a signal to stop the server
	sig := <-sigChan
	logrus.Printf("Command service is shutting down: %s", sig)

	// Stop the gRPC server gracefully
	s.GracefulStop()
}
```

### Read and Write data into files

# BFF

Backend For Frontend (BFF) is an architectural pattern where a dedicated backend service is created for each frontend application or client type. 

Its primary role is to act as an intermediary between the frontend and the backend services.

BFF aggregates data from multiple backend services and provides tailored APIs optimized for specific frontend requirements.

BFF decouples the frontend from the complexities of backend services. It allows frontend teams to evolve their applications independently without being tied to backend changes.

By reducing over-fetching and under-fetching of data, BFF improves performance and ensures a smoother user experience.

Speak to two microservices and return results to clients in the project.

# GraphQL

GraphQL is a query language for APIs and a runtime for executing those queries with existing data. 

GraphQL enables clients to request only the data they need. Clients can specify the shape and structure of the response, avoiding unnecessary data retrieval.

Purpose: Unlike traditional REST APIs, GraphQL provides a more flexible and efficient approach to data fetching. It allows clients to retrieve nested data in a single request.

# Implementing BFF

In practice, the BFF aggregates data from microservices and constructs GraphQL responses for clients. This ensures that frontend applications receive precisely the data they require, enhancing performance and maintainability.

# Orchestrate services by Kubernetes

## Understanding Kubernetes

Kubernetes is an open-source container orchestration platform. Its purpose is to automate deployment, scaling, and management of containerized applications.

Kubernetes abstracts away the underlying infrastructure complexity. It automates tasks like container scheduling, scaling, and load balancing.

By simplifying deployment and management, Kubernetes allows developers to focus on building and deploying applications without worrying about infrastructure details.

## Deploying Services Locally with Minikube

## What is Minikube?

Minikube is a powerful tool that allows us to run a Kubernetes cluster locally. 

It’s particularly useful for development and testing purposes.

Minikube creates a lightweight, single-node Kubernetes cluster on your local machine.

With Minikube, you can simulate a production-like environment without the need for a full-scale Kubernetes cluster.

## Testing the Project Locally

We’ll leverage Minikube to test our project.

# Summary

In this article, we’ve explored the Backend For Frontend (BFF) pattern, microservices, and gRPC. Additionally, we’ve touched upon Kubernetes and Minikube for orchestration and local testing.

While there are many other concepts related to microservices, such as Distributed Transactions, I hope this article has provided you with a foundational understanding of the microservices ecosystem.

# References

[REST Servers in Go: Part 7 - GraphQL](https://eli.thegreenplace.net/2021/rest-servers-in-go-part-7-graphql/)

