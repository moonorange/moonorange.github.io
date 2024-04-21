---
title: 'Learn gRPC, GraphQL and Kubernetes by building Microservices: Part 1 - gRPC MicroServices'
date: '2024-04-19'
categories: ["Projects", "Go", "Microservice", "gRPC", "Kubernetes"]
tags: ["Go", "English Article", "Kubernetes"]
---

# Intro

This is the first post in a series about learning gRPC, GraphQL and Kubernetes by building Microservices in Go.

In this series of posts, we will embark on a journey to implement a Backend For Frontend (BFF) and microservices architecture utilizing gRPC and GraphQL.

This project serves as an educational endeavor to understand the intricacies of microservices communication and the benefits of using gRPC and GraphQL.

Here is a list of posts in the series:

[Part 1 - gRPC Microservices](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part1)
[Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)
[Part 3 - Orchestrating by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

# Project Structure(Overview)

Our project revolves around three essential components:

1. Microservices (gRPC): We’ll implement two microservices adhering to the Command Query Responsibility Segregation (CQRS) pattern:
Command Service: Responsible for executing write operations, handling commands, and modifying data.
Query Service: Retrieves data and responds to read queries.

2. Backend For Frontend (BFF): This layer acts as an intermediary between the frontend and the microservices. It handles frontend-specific logic, aggregating data from various microservices and presenting it to the client.

3. Kubernetes for Orchestration and Deployment: We’ll leverage Kubernetes to manage our microservices, ensuring scalability, resilience, and efficient resource utilization.

{{<figure src="./project_arch.png" alt="Project Architecture" width="80%" height="80%">}}

Full code is in [here](https://github.com/moonorange/go_programs/tree/main/microservices_tutorial)

# Microservices

The CQRS pattern separates read and write responsibilities within our system.

While it’s not necessary and over-engineering for small projects, understanding it can enhance your architectural knowledge.

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

## Defining protobuf

Let's start with defining Protocol Buffer.

`{PROJECT_ROOT}/proto_go/proto/task.proto`

```proto
syntax = "proto3";

import "google/protobuf/empty.proto";

// Define a directory to put auto generated codes
option go_package = "gen";

package task;

service TaskService {
  rpc CreateTask(CreateTaskRequest) returns (CreateTaskResponse);
  rpc GetTask(GetTaskRequest) returns (GetTaskResponse);
  rpc ListTasksByTag(ListTasksByTagRequest) returns (ListTasksByTagResponse);
}

message Task {
  int32 id = 1;
  string text = 2;
  repeated string tags = 3;
}

message CreateTaskRequest {
  string text = 1;
  repeated string tags = 2;
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

message ListTasksByTagRequest {
  string tag_name = 1;
}

message ListTasksByTagResponse {
   repeated Task tasks = 1;
}
```

After defining protobuf, we can generate code automatically from protobuf.

I will use [buf](https://github.com/bufbuild/buf) to manage protobuf in this project

Refer to the official document to leann how to install and use buf.

Generate Go code from protobuf

`{PROJECT_ROOT}/proto_go/`

```sh
cd proto
buf generate proto
```

You can see codes for both gRPC server and client were generated.

## Implementing gRPC server

Let's implement gRPC server using auto-generated code.

As I explained, we will implement two gRPC servers, Command Service and Query Service.

The following code will creates a gRPC server and run it locally.

It's almost the same in both services.

`{PROJECT_ROOT/microservices/query_service/cmd/server/main.go`

```go
func main() {
	const host = "localhost:8082"

	mux := http.NewServeMux()
	path, handler := genconnect.NewTaskServiceHandler(&taskServer{})
	mux.Handle(path, handler)
	logrus.Println("... Listening on", host)

	eg := errgroup.Group{}
	// Start the gRPC server
	eg.Go(func() error { return http.ListenAndServe(host, h2c.NewHandler(mux, &http2.Server{})) })
	logrus.Printf("Query service is running on host %s", host)

	err := eg.Wait()
	if err != nil {
		logrus.Fatal("failed to serve: ", err)
	}
}
```

`{PROJECT_ROOT/microservices/command_service/cmd/server/main.go`

```go
func main() {
	const host = "localhost:8081"

	mux := http.NewServeMux()
	path, handler := genconnect.NewTaskServiceHandler(&taskServer{})
	mux.Handle(path, handler)
	logrus.Println("... Listening on", host)

	eg := errgroup.Group{}
	// Start the gRPC server
	eg.Go(func() error { return http.ListenAndServe(host, h2c.NewHandler(mux, &http2.Server{})) })
	logrus.Printf("Command service is running on host %s", host)

	err := eg.Wait()
	if err != nil {
		logrus.Fatal("failed to serve: ", err)
	}
}
```

### Implementing TaskService

Let's implement `TaskService.ListTaskByTag` in query service.

In reality, you will use some sort of DB, but here just return values for simplicity.

`{PROJECT_ROOT}/microservices/query_service/cmd/server/main.go`

```go
type taskServer struct {
	genconnect.UnimplementedTaskServiceHandler
}

// Just return a list of tasks for simplicity
func (t *taskServer) ListTasksByTag(ctx context.Context, req *connect.Request[gen.ListTasksByTagRequest]) (*connect.Response[gen.ListTasksByTagResponse], error) {
	tasks := []*gen.Task{
		{
			Id:   1,
			Text: "This is a task",
			Tags: []string{req.Msg.TagName},
		},
		{
			Id:   2,
			Text: "This is a task",
			Tags: []string{req.Msg.TagName},
		},
	}
	return connect.NewResponse(&gen.ListTasksByTagResponse{Tasks: tasks}), nil
}
```

Let's implement `TaskService.CreateTask` in command service.

In reality, you will use some sort of DB, but here just return values for simplicity.

`{PROJECT_ROOT/microservices/command_service/cmd/server/main.go`

```go
// taskServer implements the TaskService API.
type taskServer struct {
	genconnect.UnimplementedTaskServiceHandler
}

// Just return a task for simplicity
func (t *taskServer) CreateTask(ctx context.Context, req *connect.Request[gen.CreateTaskRequest]) (*connect.Response[gen.CreateTaskResponse], error) {
	task := &gen.Task{
		Id:   1,
		Text: req.Msg.Text,
		Tags: req.Msg.Tags,
	}
	return connect.NewResponse(&gen.CreateTaskResponse{Task: task}), nil
}
```

You can check if the servers returns expected result by buf curl

Start the query service.

```sh
cd microservices/query_service
go run cmd/server/main.go
```

Start the command service.

```sh
cd microservices/command_service
go run cmd/server/main.go
```

Curl API

ListTasks in query service.

`{PROJECT_ROOT}/proto_go/`

```sh
buf curl \
  --schema proto \
  --data '' \
  http://localhost:8082/task.TaskService/ListTasksByTag
```

Create Tasks in command service

```sh
buf curl \
  --schema proto \
  --data '{"text": "task", "tags": ["tag1", "tag2"]}' \
  http://localhost:8081/task.TaskService/CreateTask
```

So far we have implemented two micro services, let's implement BFF using GraphQL next.


# Summary

We have learned about gRPC and Protocol Buffer and implemented two micro services, query service and command service in this post.

In part 2 we will implementing BFF speaking GraphQL.

# References

[REST Servers in Go: Part 7 - GraphQL](https://eli.thegreenplace.net/2021/rest-servers-in-go-part-7-graphql/)

[Building GraphQL servers in golang](https://gqlgen.com/getting-started/)
