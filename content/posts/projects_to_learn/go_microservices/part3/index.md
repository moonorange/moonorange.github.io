---
title: 'Learn gRPC, GraphQL and Kubernetes by building Microservices: Part 3 - Kubernetes'
date: '2024-04-23'
categories: ["Projects", "Go", "Microservice", "gRPC", "Kubernetes"]
tags: ["Go", "English Article", "Kubernetes"]
---

# Intro

This is the third and last post in a series about learning gRPC, GraphQL and Kubernetes by building Microservices in Go.

Here is a list of posts in the series:

- [Part 1 - gRPC Microservices](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part1)
- [Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)
- [Part 3 - Orchestrating by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

Full code is in [here](https://github.com/moonorange/go_programs/tree/main/microservices_tutorial)

Directory structure is like below

```sh
tree .
.
├── Makefile
├── README.md
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
├── go.mod
├── go.sum
├── microservices // Microservices, query and command services
│   ├── command_service
│   │   ├── cmd
│   │   │   └── server
│   │   │       └── main.go
│   │   ├── go.mod
│   │   └── go.sum
│   └── query_service
│       ├── cmd
│       │   └── server
│       │       └── main.go
│       ├── go.mod
│       └── go.sum
└── proto_go // protobuf and auto-generated server and client stubs from the protobuf
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

# Orchestrate services by Kubernetes

## Understanding Kubernetes

Kubernetes is an open-source container orchestration platform. Its purpose is to automate deployment, scaling, and management of containerized applications.

You can define the desired state and 

## Understanding Minikube

Minikube is a powerful tool that allows us to run a Kubernetes cluster locally. 

It’s particularly useful for development and testing purposes.

Minikube creates a lightweight, single-node Kubernetes cluster on your local machine.

With Minikube, you can simulate a production-like environment without the need for a full-scale Kubernetes cluster.

We’ll leverage Minikube to run our project locally.

```sh
brew install minikube
```

## Understanding Helm

Helm is a package manager for Kubernetes.

It reduces

# Summary

In this series of articles, we’ve explored the Backend For Frontend (BFF) pattern, microservices, and gRPC. Additionally, we’ve touched upon Kubernetes and Minikube for orchestration and local testing.

While there are many other concepts related to microservices, such as Distributed Transactions, I hope this article has provided you with a foundational understanding of the microservices ecosystem.

# References

[Kubernetes Tutorial for Beginners [FULL COURSE in 4 Hours]
](https://www.youtube.com/watch?v=X48VuDVv0do)
