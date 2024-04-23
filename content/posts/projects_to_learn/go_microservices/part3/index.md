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

Important and basic components of Kubernetes is below:

Nodes:
- A node is a physical or virtual machine

Cluster:
- Set of worker machines(nodes)
- Every cluster has at least one worker node

Pods:
- Smallest unit of K8s
- Abstraction over container with shared storage and network resources
- The "one-container-per-Pod" model is the most common Kubernetes use case
- Pods are ephemeral resources. Each pod gets its own IP address(New IP address on re-creation)

Services:
- Provide endpoints(Permanent IP address) for pods
- Make the set of pods available on the network
- Load Balancer for pods

Ingress:
- Expose http and https routes from external components to services within the cluster

Volumes:
- Storage which can exists beyond the lifetime of a pod if it's persistent volumens

{{<figure src="./k8s_components.png" alt="k8s Basic Components" width="50%">}}

You can describe what will be deployed in Kubernetes cluster by Kubernetes manifest files, which are JSON or YAML files.

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

As I explained in the Understanding Kubernetes chapter, you can describe Kubernetes objects by manifest files, but it will be tedious and complicated to manage them if you have many files.

# Summary

In this series of articles, we’ve explored the Backend For Frontend (BFF) pattern, microservices, and gRPC. Additionally, we’ve touched upon Kubernetes and Minikube for orchestration and local testing.

While there are many other concepts related to microservices, such as Distributed Transactions, I hope this article has provided you with a foundational understanding of the microservices ecosystem.

# References

[Kubernetes Documentation](https://kubernetes.io/docs/home/)

[Kubernetes Tutorial for Beginners [FULL COURSE in 4 Hours]](https://www.youtube.com/watch?v=X48VuDVv0do)
