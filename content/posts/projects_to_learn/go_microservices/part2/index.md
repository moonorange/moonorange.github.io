---
title: 'Learn gRPC, GraphQL and Kubernetes by building Microservices in Go: Part 2 - GraphQL BFF'
date: '2024-04-22'
categories: ["Projects", "Go", "Microservice", "gRPC", "GraphQL", "Kubernetes", "GraphQL"]
tags: ["Go", "English Article", "Kubernetes"]
---

# Intro

This is the second post in a series about learning gRPC, GraphQL and Kubernetes by building Microservices in Go.
Here is a list of posts in the series:

- [Part 1 - gRPC Microservices](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part1)
- [Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)
- [Part 3 - Deploy services by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

Full code is in [here](https://github.com/moonorange/go_programs/tree/main/microservices_tutorial)

We have implemented gRPC servers in part 1.

In part 2, we will develop a BFF server that reads from and writes to these gRPC servers and communicates with clients using GraphQL.

BFF Directory structure:

```sh
tree .
.
├── bff // BFF talking to clients by GraphQL
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
```

# BFF

**Backend For Frontend (BFF)** is an architectural pattern where a dedicated backend service is created for each frontend application or client type.

Its primary role is to act as an intermediary between the frontend and the backend services.

BFF aggregates data from multiple backend services and provides tailored APIs optimized for specific frontend requirements.

BFF decouples the frontend from the complexities of backend services. It allows frontend teams to evolve their applications independently without being tied to backend changes.

By reducing over-fetching and under-fetching of data, BFF improves performance and ensures a smoother user experience.

In our project, the BFF speaks to two backend services and provides APIs for frontend.

# GraphQL

GraphQL is a query language for APIs and a runtime for executing those queries with existing data.

GraphQL enables clients to request only the data they need. Clients can specify the shape and structure of the response, avoiding unnecessary data retrieval.

Unlike traditional REST APIs, GraphQL provides a more flexible and efficient approach to data fetching. It allows clients to retrieve nested data in a single request.

# Implementing BFF

We'll use [gqlgen](https://github.com/99designs/gqlgen) to generate code automatically from our GraphQL schema.

First, add github.com/99designs/gqlgen to your project's tools.go:

`{PROJECT_ROOT}/bff/`

```sh
printf '// +build tools\npackage tools\nimport (_ "github.com/99designs/gqlgen"\n _ "github.com/99designs/gqlgen/graphql/introspection")' | gofmt > tools.go

go mod tidy
```

Next, initialize gqlgen config and generate models:

```sh
go run github.com/99designs/gqlgen init

go mod tidy
```

You get the template of GraphQL server.

## Define Your GraphQL Schema

Now, let's define our GraphQL schema in graph/schema.graphqls.

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

Generate code from the schema:

```sh
go run github.com/99designs/gqlgen generate
```

## Preventing over-fetching

Suppose we do not want to load the Attachment on the Task unless the user actually asked for it.

To archive that, we will add the custom Task model.

First let’s enable autobind, allowing gqlgen to use our custom models.

We can do this by uncommenting the autobind config line in gqlgen.yml:

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

This will be called only when users asked for Attachments field.

If assuming it fetches data from DB, it can reduce the unnecessary query to storage by implementing attachment resolver

```go
// Attachments is the resolver for the Attachments field.
// If assuming it fetches data from DB, it can reduce the unnecessary query to storage by implementing attachment resolver
func (r *taskResolver) Attachments(ctx context.Context, obj *model.Task) ([]*model.Attachment, error) {
	panic(fmt.Errorf("not implemented"))
}
```

## Implementing resolvers

We'll implement clients and resolvers to interact with microservices and retrieve or create data.

This resolver will communicate with our backend services to fulfill GraphQL queries and mutations.

`bff/client/task_client.go`

```go
package client

import (
	"net/http"
	"os"

	"connectrpc.com/connect"
	"github.com/moonorange/gomicroservice/protogo/gen/genconnect"
	"github.com/sirupsen/logrus"
)

var (
	queryClient   genconnect.TaskServiceClient
	commandClient genconnect.TaskServiceClient
)

func NewQueryServiceClient() genconnect.TaskServiceClient {
	queryHost := os.Getenv("QUERY_SERVICE_HOST")
	logrus.Info("queryHost: ", queryHost)
	if queryHost == "" {
		logrus.Fatal("empty QUERY_SERVICE_HOST")
	}
	// Set up a connection to the server.
	// Create a gRPC client using the connect.WithGRPC() option
	if queryClient != nil {
		return queryClient
	}
	queryClient = genconnect.NewTaskServiceClient(
		http.DefaultClient,
		"http://"+queryHost,
		connect.WithGRPC(),
	)

	return queryClient
}

func NewCommandServiceClient() genconnect.TaskServiceClient {
	commandHost := os.Getenv("COMMAND_SERVICE_HOST")
	logrus.Info("commandHost: ", commandHost)
	if commandHost == "" {
		logrus.Fatal("empty COMMAND_SERVICE_HOST")
	}
	if commandClient != nil {
		return commandClient
	}
	// Set up a connection to the server.
	// Create a gRPC client using the connect.WithGRPC() option
	commandClient = genconnect.NewTaskServiceClient(
		http.DefaultClient,
		"http://"+commandHost,
		connect.WithGRPC(),
	)

	return commandClient
}
```

`bff/graph/schema.resolvers.go`

```go
func init() {
  queryClient = client.NewQueryServiceClient()
  commandClient = client.NewCommandServiceClient()
}

// GetTask is the resolver for the getTask field.
func (r *queryResolver) GetTask(ctx context.Context, id string) (*model.Task, error) {
	// Access to query service
	res, err := queryClient.GetTask(ctx, connect.NewRequest(&gen.GetTaskRequest{
		TaskId: "1",
	}))
	if err != nil {
		return nil, err
	}
	return &model.Task{
		ID:   string(res.Msg.Task.Id),
		Text: res.Msg.Task.Text,
		Tags: res.Msg.Task.Tags}, nil
}
```

## Running the servers

With our BFF and microservices in place, it's time to run the servers and test our GraphQL endpoints:

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

Use the provided GraphQL queries and mutations to interact with the BFF server and verify that it returns the expected results.

**GetTasksByTag:**

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

**CreateTask:**

```graphql
mutation {
  createTask(input: {
    Text: "My new task"
    Tags: ["important", "work"]
    Attachments: [
      {
        Name: "document.pdf"
        Date: "2023-05-01T10:00:00Z"
        Contents: "base64-encoded-file-contents"
      }
    ]
  }) {
    Id
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

# Summary

In this post, we've explored the concept of BFF, implemented a GraphQL BFF service in Go, and learned how to optimize data fetching with GraphQL.

Check for the next part, where we'll deploy our microservices with Kubernetes.

 [Part 3 - Deploy services by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

# References

[REST Servers in Go: Part 7 - GraphQL](https://eli.thegreenplace.net/2021/rest-servers-in-go-part-7-graphql/)

[Building GraphQL servers in golang](https://gqlgen.com/getting-started/)
