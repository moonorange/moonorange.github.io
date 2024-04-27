---
title: 'Learn gRPC, GraphQL and Kubernetes by building Microservices: Part 1 - Building gRPC Microservices'
date: '2024-04-21'
categories: ["Projects", "Go", "Microservice", "gRPC", "Kubernetes", "GraphQL"]
tags: ["Go", "English Article", "Kubernetes"]
---

# Intro

This is the first post in a series about learning gRPC, GraphQL and Kubernetes by building Microservices in Go.

In this series of posts, we will embark on a journey to implement a Backend For Frontend (BFF) and microservices architecture utilizing gRPC and GraphQL, and deploy services locally by Kubernetes.

This project serves as an educational endeavor to understand the microservices communication and the benefits of using gRPC, GraphQL, and Kubernetes.

Here is a list of posts in the series:

- [Part 1 - gRPC Microservices](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part1)
- [Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)
- [Part 3 - Deploy services by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

# Project Structure(Overview)

Our project revolves around three essential components:

## Microservices (gRPC):

We’ll implement two microservices adhering to the Command Query Responsibility Segregation (CQRS) pattern:

Command Service: Responsible for executing write operations, handling commands, and modifying data.

Query Service: Retrieves data and responds to read queries.

## Backend For Frontend (BFF):

This layer acts as an intermediary between the frontend and the microservices.

It handles frontend-specific logic, aggregating data from various microservices and presenting it to the client.

## Kubernetes for Deployment:

We’ll leverage Kubernetes to deploy our microservices.

{{<figure src="./project_arch.png" alt="Project Architecture" width="80%" height="80%">}}

Explore the full code [here](https://github.com/moonorange/go_programs/tree/main/microservices_tutorial)

Entire Project structure is like below

```sh
tree .
.
├── Makefile
├── README.md
├── bff
│   ├── Dockerfile
│   ├── client
│   │   └── task_client.go
│   ├── cmd
│   │   └── server
│   │       └── main.go
│   ├── go.mod
│   ├── go.sum
│   ├── gqlgen.yml
│   ├── graph
│   │   ├── generated.go
│   │   ├── model
│   │   │   ├── models_gen.go
│   │   │   └── task.go
│   │   ├── resolver.go
│   │   ├── schema.graphqls
│   │   └── schema.resolvers.go
│   └── tools.go
├── go.mod
├── go.sum
├── k8s
│   ├── bff.yaml
│   ├── command-service.yaml
│   ├── microservice
│   │   ├── Chart.yaml
│   │   ├── charts
│   │   ├── templates
│   │   │   ├── NOTES.txt
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── tests
│   │   │       └── test-connection.yaml
│   │   └── values.yaml
│   └── query-service.yaml
├── microservices
│   ├── command_service
│   │   ├── Dockerfile
│   │   ├── cmd
│   │   │   └── server
│   │   │       └── main.go
│   │   ├── go.mod
│   │   └── go.sum
│   └── query_service
│       ├── Dockerfile
│       ├── cmd
│       │   └── server
│       │       └── main.go
│       ├── go.mod
│       └── go.sum
└── protogo
    ├── buf.gen.yaml
    ├── gen
    │   ├── genconnect
    │   │   └── task.connect.go
    │   └── task.pb.go
    ├── go.mod
    ├── go.sum
    └── proto
        ├── buf.yaml
        └── task.proto
```

# Microservices

The CQRS(Command and Query Responsibility Segregation) pattern separates read and write responsibilities within our system.

While it’s not necessary and over-engineering for small projects, understanding it can enhance your architectural knowledge.

We will use gRPC for communication between services.

## Understanding gRPC

gRPC, developed by Google, is an open-source remote procedure call (RPC) framework. It offers the following features:

- Interface Definition with Protocol Buffers: gRPC uses Protocol Buffers as its interface definition language. This ensures clear communication between services.
- HTTP/2 Transport: gRPC utilizes HTTP/2, providing efficient, high-performance communication.
- Remote Method Invocation: Services can invoke methods on remote servers as if they were local function calls.

In summary, gRPC simplifies communication between distributed systems, making it an excellent choice for microservices architectures.

## Protocol Buffers (protobuf)

We’ll define our service interfaces and messages using [Protocol Buffers (protobuf)](https://protobuf.dev/).

These language-agnostic schemas allow us to precisely specify the structure of our data.

## Defining protobuf

Let's kickstart by defining Protocol Buffer for our project:

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

After defining protobuf, we can automatically generate code from protobuf.

We'll use [buf](https://github.com/bufbuild/buf) for managing protobuf in our project.

Refer to the official document to leann how to install and use buf.

Generate Go code from protobuf:

`{PROJECT_ROOT}/proto_go/`

```sh
cd proto
buf generate proto
```

You can see codes for both gRPC server and client were generated.

## Implementing gRPC server

Let's dive into implementing gRPC servers using auto-generated code.

As I explained, we will implement two gRPC servers, Command Service and Query Service.

We'll begin by creating two gRPC servers: Command Service and Query Service. The following code illustrates setting up a gRPC server:

`{PROJECT_ROOT/microservices/query_service/cmd/server/main.go`

```go
const (
	defaultPort = "8082"
	defaultHost = "localhost"
)

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = defaultPort
	}
	host := os.Getenv("QUERY_SERVICE_HOST")
	if host == "" {
		host = defaultHost
	}

	mux := http.NewServeMux()
	path, handler := genconnect.NewTaskServiceHandler(&taskServer{})
	mux.Handle(path, handler)
	logrus.Println("... Listening on", host)

	eg := errgroup.Group{}
	// Start the gRPC server
	eg.Go(func() error { return http.ListenAndServe(":"+port, h2c.NewHandler(mux, &http2.Server{})) })
	logrus.Printf("Query service is running on host %s", host)

	err := eg.Wait()
	if err != nil {
		logrus.Fatal("failed to serve: ", err)
	}
}
```

`{PROJECT_ROOT/microservices/command_service/cmd/server/main.go`

```go
const (
	defaultPort = "8082"
	defaultHost = "localhost"
)

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = defaultPort
	}
	host := os.Getenv("COMMAND_SERVICE_HOST")
	if host == "" {
		host = defaultHost
	}

	mux := http.NewServeMux()
	path, handler := genconnect.NewTaskServiceHandler(&taskServer{})
	mux.Handle(path, handler)
	logrus.Println("... Listening on", host)

	eg := errgroup.Group{}
	// Start the gRPC server
	eg.Go(func() error { return http.ListenAndServe(":"+port, h2c.NewHandler(mux, &http2.Server{})) })
	logrus.Printf("Command service is running on host %s", host)

	err := eg.Wait()
	if err != nil {
		logrus.Fatal("failed to serve: ", err)
	}
}
```

### Implementing TaskService

Next, we'll implement methods within the TaskService interface for both Query and Command Services.

Here's a snippet demonstrating the implementation:

For simplicity, we're returning mock data instead of interacting with a real database.

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

Let's implement `TaskService.CreateTask` in command service:

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

You can verify if the servers return expected result by buf curl

Start the query service:

```sh
cd microservices/query_service
go run cmd/server/main.go
```

Start the command service:

```sh
cd microservices/command_service
go run cmd/server/main.go
```

Curl API:

ListTasks in the QueryService:

`{PROJECT_ROOT}/proto_go/`

```sh
buf curl \
  --schema proto \
  --data '' \
  http://localhost:8082/task.TaskService/ListTasksByTag
```

CreateTask in CommandService:

```sh
buf curl \
  --schema proto \
  --data '{"text": "task", "tags": ["tag1", "tag2"]}' \
  http://localhost:8081/task.TaskService/CreateTask
```

# Summary

In this post, we have dived into the fundamentals of gRPC and Protocol Buffers.

Also, we have implemented two microservices: Query Service and Command Service.

In part 2 we will explore GraphQL and construct our BFF layer.

[Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)

# References

[Try the Buf CLI](https://buf.build/docs/tutorials/getting-started-with-buf-cli#resolve-go-dependencies)
