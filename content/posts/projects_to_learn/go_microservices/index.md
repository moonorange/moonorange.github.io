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

1. Microservices (gRPC): We’ll implement two microservices adhering to the Command Query Responsibility Segregation (CQRS) pattern:
Command Service: Responsible for executing write operations, handling commands, and modifying data.
Query Service: Retrieves data and responds to read queries.

2. Backend For Frontend (BFF): This layer acts as an intermediary between the frontend and the microservices. It handles frontend-specific logic, aggregating data from various microservices and presenting it to the client.

3. Kubernetes for Orchestration and Deployment: We’ll leverage Kubernetes to manage our microservices, ensuring scalability, resilience, and efficient resource utilization.

{{<figure src="./project_arch.png" alt="Project Architecture" width="100%">}}

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

`{PROJECT_ROOT/microservices/query_service/cmd/server/main.go`

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

# BFF

Backend For Frontend (BFF) is an architectural pattern where a dedicated backend service is created for each frontend application or client type. 

Its primary role is to act as an intermediary between the frontend and the backend services.

BFF aggregates data from multiple backend services and provides tailored APIs optimized for specific frontend requirements.

BFF decouples the frontend from the complexities of backend services. It allows frontend teams to evolve their applications independently without being tied to backend changes.

By reducing over-fetching and under-fetching of data, BFF improves performance and ensures a smoother user experience.

Speak to two microservices and return results to clients in this project.

# GraphQL

GraphQL is a query language for APIs and a runtime for executing those queries with existing data.

GraphQL enables clients to request only the data they need. Clients can specify the shape and structure of the response, avoiding unnecessary data retrieval.

Unlike traditional REST APIs, GraphQL provides a more flexible and efficient approach to data fetching. It allows clients to retrieve nested data in a single request.

# Implementing BFF

In practice, the BFF aggregates data from microservices and constructs GraphQL responses for clients. This ensures that frontend applications receive precisely the data they require, enhancing performance and maintainability.

We are going to use [gqlgen](https://github.com/99designs/gqlgen) to generate code automatically from GraphQL schema for this project.

Add github.com/99designs/gqlgen to your project's tools.go

`{PROJECT_ROOT}/bff/`

```sh
printf '// +build tools\npackage tools\nimport (_ "github.com/99designs/gqlgen"\n _ "github.com/99designs/gqlgen/graphql/introspection")' | gofmt > tools.go

go mod tidy
```

Initialize gqlgen config and generate models

```sh
go run github.com/99designs/gqlgen init

go mod tidy
```

Now you get the template of GraphQL server.

## Define Your GraphQL Schema

Update graph/schema.graphqls like below.

```graphqls
type Query {
  getTask(id: ID!): Task

  getTasksByTag(tag: String!): [Task]
}

type Mutation {
  createTask(input: NewTask!): Task!
}

type Task {
  Id: ID!
  Text: String!
  Tags: [String!]
  Attachments: [Attachment!]
}

input NewTask {
  Text: String!
  Tags: [String!]
  Attachments: [NewAttachment!]

}

scalar Time

type Attachment {
  Name: String!
  Date: Time
  Contents: String
}

input NewAttachment {
  Name: String!
  Date: Time
  Contents: String
}
```

I added a new model named Attachment which does not exist in microservices' model.

This model is for showing GraphQL benefits of fetching only necessary data.

Generate code from the schema

```sh
go run github.com/99designs/gqlgen generate
```

## Preventing over-fetching

Suppose we do not want to load the Attachment on the Task unless the user actually asked for it.

To archive that, replace the generated Task model with something slightly more realistic.

First let’s enable autobind, allowing gqlgen to use your custom models if it can find them rather than generating them.

We do this by uncommenting the autobind config line in gqlgen.yml:

```yaml
# gqlgen will search for any type names in the schema in these go packages
# if they match it will use them, otherwise it will generate them.
autobind:
 - "bff/graph/model"
```

And add Task fields resolver config in gqlgen.yml to generate resolver for Attachment field

```yaml
# This section declares type mapping between the GraphQL and go type systems
#
# The first line in each type will be used as defaults for resolver arguments and
# modelgen, the others will be allowed when binding to fields. Configure them to
# your liking
models:
  ID:
    model:
      - github.com/99designs/gqlgen/graphql.ID
      - github.com/99designs/gqlgen/graphql.Int
      - github.com/99designs/gqlgen/graphql.Int64
      - github.com/99designs/gqlgen/graphql.Int32
  Int:
    model:
      - github.com/99designs/gqlgen/graphql.Int
      - github.com/99designs/gqlgen/graphql.Int64
      - github.com/99designs/gqlgen/graphql.Int32
  Task:
    fields:
      Attachments:
```

Next, create a new file called graph/model/task.go

```go
package model

type Task struct {
	ID   string   `json:"Id"`
	Text string   `json:"Text"`
	Tags []string `json:"Tags,omitempty"`
	Attachment *Attachment `json:"Attachment,omitempty"`
}
```

You can see Attachments resolver was implemented.

```go
// Attachments is the resolver for the Attachments field.
// If assuming it fetches data from DB, it can reduce the unnecessary query to storage by implementing attachment resolver
func (r *taskResolver) Attachments(ctx context.Context, obj *model.Task) ([]*model.Attachment, error) {
	panic(fmt.Errorf("not implemented"))
```

## Implementing resolvers

Let's implement resolvers to get and create data via microservices.

`bff/graph/schema.resolvers.go`

```go
// GetTasksByTag is the resolver for the getTasksByTag field.
func (r *queryResolver) GetTasksByTag(ctx context.Context, tag string) ([]*model.Task, error) {
	// Access to query service
	tc := client.NewTaskServiceClient("http://localhost:8082")
	res, err := tc.ListTasksByTag(ctx, connect.NewRequest(&gen.ListTasksByTagRequest{
		TagName: "tag1",
	}))
	if err != nil {
		return nil, err
	}

	models := make([]*model.Task, len(res.Msg.Tasks))
	for i, t := range res.Msg.Tasks {
		models[i] = &model.Task{ID: string(t.Id), Text: t.Text, Tags: t.Tags}
	}
	return models, nil
```

Let's run all servers and check if bff server returns expected values

```sh
cd bff
go run cmd/server/main.go
```

```sh
cd microservices/query_service
go run cmd/server/main.go
```

```sh
cd microservices/command_service
go run cmd/server/main.go
```

Go to [localhost:8080](http://localhost:8080/) to interact with bff servers.

GraphQL query

```graphql
query{
  getTasksByTag(tag: "tag1") {
    Text
    Tags
    Attachments {
      Name
      Date
      Contents
    }
  }
}
```

{{<figure src="./graphql_playground.png" alt="GraphQL Playground" width="100%">}}


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

[Building GraphQL servers in golang](https://gqlgen.com/getting-started/)
