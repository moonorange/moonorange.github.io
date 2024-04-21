---
title: 'Learn gRPC, Go and Kubernetes by building Microservices: Part 3 - Kubernetes'
date: '2024-04-21'
categories: ["Projects", "Go", "Microservice", "gRPC", "Kubernetes"]
tags: ["Go", "English Article", "Kubernetes"]
---

# Intro

This is the third and last post in a series about learning gRPC, Go and Kubernetes by building Microservices in Go.

Here is a list of posts in the series:

[Part 1 - gRPC Microservices](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part1)
[Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)
[Part 3 - Orchestrating by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

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

In this series of articles, we’ve explored the Backend For Frontend (BFF) pattern, microservices, and gRPC. Additionally, we’ve touched upon Kubernetes and Minikube for orchestration and local testing.

While there are many other concepts related to microservices, such as Distributed Transactions, I hope this article has provided you with a foundational understanding of the microservices ecosystem.
