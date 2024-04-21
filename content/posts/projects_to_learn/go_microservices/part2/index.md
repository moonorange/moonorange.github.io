---
title: 'Learn gRPC, GraphQL and Kubernetes by building Microservices in Go: Part 2 - GraphQL BFF'
date: '2024-04-20'
categories: ["Projects", "Go", "Microservice", "gRPC", "GraphQL", "Kubernetes"]
tags: ["Go", "English Article", "Kubernetes"]
---

# Intro

This is the second post in a series about learning gRPC, GraphQL and Kubernetes by building Microservices in Go. 
Here is a list of posts in the series:

- [Part 1 - gRPC Microservices](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part1)
- [Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)
- [Part 3 - Orchestrating by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

Full code is in [here](https://github.com/moonorange/go_programs/tree/main/microservices_tutorial)

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

# Summary

We have learned about BFF and GraphQL and implemented BFF.

In part 3 we will orchestrate services by Kubernetes.

# References

[REST Servers in Go: Part 7 - GraphQL](https://eli.thegreenplace.net/2021/rest-servers-in-go-part-7-graphql/)

[Building GraphQL servers in golang](https://gqlgen.com/getting-started/)
