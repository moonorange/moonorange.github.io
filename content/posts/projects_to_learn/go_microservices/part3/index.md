---
title: 'Learn gRPC, GraphQL and Kubernetes by building Microservices: Part 3 - Kubernetes'
date: '2024-04-23'
categories: ["Projects", "Go", "Microservice", "gRPC", "Kubernetes"]
tags: ["Go", "English Article", "Kubernetes", "Helm", "GraphQL"]
---

# Intro

This is the third and last post in a series about learning gRPC, GraphQL and Kubernetes by building Microservices in Go.

Here is a list of posts in the series:

- [Part 1 - gRPC Microservices](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part1)
- [Part 2 - GraphQL BFF](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part2)
- [Part 3 - Deploy services by Kubernetes](https://moonorange.github.io/posts/projects_to_learn/go_microservices/part3)

Full code is in [here](https://github.com/moonorange/go_programs/tree/main/microservices_tutorial)

We have implemented gRPC servers and BFF in parts 1 and 2.

In part 3, we will deploy those services using Kubernetes and Minikube.

k8s directory structure:

```sh
tree .
.
├── k8s // k8s has Kubernetes resources
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
```

## Understanding Kubernetes

Kubernetes is an open-source container orchestration platform. Its purpose is to automate deployment, scaling, and management of containerized applications.

Key components of Kubernetes include:

**Nodes:**
- Represent either physical or virtual machines in a Kubernetes cluster.

**Cluster:**
- A collection of worker nodes that work together.
- Every Kubernetes cluster must have at least one worker node.

**Pods:**
- The smallest deployable units in Kubernetes.
- Represent one or more containers that share resources such as storage and network.
- The most common use case is the "one-container-per-Pod" model.
- Pods are ephemeral, with each pod getting its own IP address, which changes upon recreation.

**Services:**
- Provide stable endpoints (with permanent IP addresses) for pods.
- Make a set of pods available on the network and can act as load balancers for those pods.

**Ingress:**
- Exposes HTTP and HTTPS routes from external components to services within the cluster.

**Volumes:**
- Volumes provide storage that can exist beyond the lifetime of a pod, especially with persistent volumes.

{{<figure src="./k8s_components.png" alt="k8s Basic Components" width="50%">}}

You can describe the desired state of your Kubernetes cluster using Kubernetes manifest files, typically written in JSON or YAML format.

Kubernetes pods can be configured imperatively or declaratively, with manifest files commonly used for declarative configuration.

## Understanding Minikube

Minikube is a powerful tool that allows us to run a Kubernetes cluster locally.

It’s particularly useful for development and testing purposes.

Minikube creates a lightweight, single-node Kubernetes cluster on your local machine.

With Minikube, you can simulate a production-like environment without the need for a full-scale Kubernetes cluster.

We’ll leverage Minikube to run our project locally.

```sh
brew install minikube
```

Once installed, you can start Minikube using:

```sh
minikube start
```

## Understanding Helm

Helm is a package manager for Kubernetes that simplifies the management and deployment of applications.

It introduces the concept of charts, which are packages of pre-configured Kubernetes resources and configurations.

Helm charts can be customized using templates and values, making them reusable and adaptable to different environments.

Helm charts consist of several files, including:

- Chart.yaml: Contains metadata about the chart, such as its name, version, and description.
- values.yaml: Defines default values used to parameterize the chart's templates.
- templates: Contains the actual Kubernetes resource definitions, written using Go template syntax.

By leveraging Helm, you can streamline the deployment process and ensure consistency across different environments.

## Containerization

To begin, let's containerize our microservices.

The following code snippet is for the bff service, although the Dockerfile is nearly identical for the query and command services as well.

```Dockerfile
# Use the official Golang image as base
FROM golang:1.22.2-alpine AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy the Go module files first to help Docker utilize the Docker layer caching mechanism more efficiently
COPY go.mod go.sum ./

# Download and install dependencies
RUN go mod download

# Copy the rest of the application source code
COPY . .

# Build the application's binary
# Compiles the code into a static binary meaning it includes all necessary dependencies within the binary itself with CGO Disabled.
# -a: tells the Go toolchain to rebuild all packages, even if they are up to date.
# -installsuffix: Used with CGO to distinguish between CGO-enabled and CGO-disabled builds
# -ldflags '-extldflags "-static"': Sets the external linker flags to include -static, which instructs the linker to statically link all libraries, including C libraries, into the binary.
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-extldflags "-static"' -o bff ./cmd/server

# Start a new stage from scratch for multi-stage builds.
# This is to reduce the finale docker image size and to isolate build dependencies
FROM alpine:latest

# Set the current working directory inside the container
WORKDIR /app

# Copy the compiled binary from the previous stage
COPY --from=builder /app/bff .

# Expose the port on which the server will run
EXPOSE 8080

# Command to run the application when starting the container
CMD ["./bff"] 
```

Build and Push the images to DockerHub.

Follow these steps to build and push the images to DockerHub:

**Login to DockerHub**: Execute the following command to login to DockerHub:

```sh
docker login
```

**Place the Makefile**: Ensure the Makefile provided below is placed in the project root directory (gomicroservice):

```Makefile
# Define variables for image names and paths
BFF_IMAGE := bff:latest
QUERY_IMAGE := query_service:latest
COMMAND_IMAGE := command_service:latest
DOCKERHUB_REPO := keigokida/gomicroservices

build:
	docker build -t $(BFF_IMAGE) ./bff
	docker build -t $(QUERY_IMAGE) ./microservices/query_service
	docker build -t $(COMMAND_IMAGE) ./microservices/command_service
	docker tag $(BFF_IMAGE) $(DOCKERHUB_REPO):bff
	docker tag $(QUERY_IMAGE) $(DOCKERHUB_REPO):query_service
	docker tag $(COMMAND_IMAGE) $(DOCKERHUB_REPO):command_service

push_images:
	docker push $(DOCKERHUB_REPO):bff
	docker push $(DOCKERHUB_REPO):query_service
	docker push $(DOCKERHUB_REPO):command_service

build_push: build push_images

.PHONY: build push_images build_push
```

**Build and Push Images**: Execute the following command to build and push the images to DockerHub:

```sh
make build_push
```

**Verify on DockerHub**: After the process completes, verify that all three images were successfully pushed to DockerHub.

{{<figure src="./dockerhub.png" alt="DockerHub" width="100%">}}

## Deploying on Kubernetes

Start Minikube:

```sh
minikube start
```

Utilize Helm to deploy services on Minikube:

```sh
helm create microservice
```

Modify the deployment.yaml file as follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}
spec:
  {% comment %} This line specifies the desired number of replicas (pods) for the Deployment {% endcomment %}
  replicas: {{ .Values.replicaCount }}
  {% comment %} The selector defines how the Deployment finds the pods it manages. {% endcomment %}
  selector:
    matchLabels:
      app: {{ .Values.name }}
  {% comment %} This section defines the template for the pods that the Deployment will create and manage.
  The labels defined here will be applied to the pods. {% endcomment %}
  template:
    metadata:
      labels:
        app: {{ .Values.name }}
    {% comment %} This section defines the container(s) that will run in the pods managed by the Deployment. {% endcomment %}
    spec:
      containers:
        - name: bff
          image: "{{ .Values.container.image.repository }}:{{ .Values.container.image.tag }}"
          imagePullPolicy: {{ .Values.container.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.port }}
              protocol: TCP
          {% comment %} env variables {% endcomment %}
          env:
            - name: PORT
              value: "{{ .Values.service.port }}"
            - name: QUERY_SERVICE_HOST
              value: "{{ .Values.container.dns.query }}.default.svc.cluster.local:{{ .Values.service.queryPort }}"
            - name: COMMAND_SERVICE_HOST
              value: "{{ .Values.container.dns.command }}.default.svc.cluster.local:{{ .Values.service.commandPort }}"
```

The following selector part in the deployment.yaml determines which pods the deployment should manage:

```yaml
 selector:
    matchLabels:
      app: {{ .Values.name }}
```

Modify the service.yaml file as follows:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.name }}
  labels:
    app: {{ .Values.name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: {{ .Values.name }}
```

The selector in the service determines which pods should receive traffic from the service

The `selector` should match with `matchLabels` in deployment.yaml.

By ensuring that the labels and selectors match, the service can route traffic to the correct set of pods managed by the deployment.

Modify the values.yaml file as follows:

```yaml
# Default values for microservice.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

name: microservice
replicaCount: 1
container:
  image:
    repository: keigokida/gomicroservices
    tag: microservice
    pullPolicy: IfNotPresent
  dns:
    query: query-service
    command: command-service

service:
  type: ClusterIP
  port: 8080
  queryPort: 8081
  commandPort: 8082

ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: microservice.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
```

This file is going to be default injected values.

Modify the Chart.yaml file as follows:

```yaml
apiVersion: v2
name: microservice
description: A Helm chart for my microservice

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"
```

The provided Charts.yaml, deployment.yaml and service.yaml files demonstrate how to define Kubernetes resources for deploying microservices.

These files use Helm templates to inject values from the values.yaml file, allowing for customization and reusability.

Next, create separate YAML files (e.g., bff.yaml, query-service.yaml, command-service.yaml) to inject values specific to each service.

Ensure that NodePort is used for the bff service to allow access from external services:

```yaml
name: bff
replicaCount: 1
container:
  image:
    repository: keigokida/gomicroservices
    tag: bff
    pullPolicy: IfNotPresent
  dns:
    query: query-service
    command: command-service

service:
  type: NodePort
  port: 8080
  queryPort: 8081
  commandPort: 8082
```

Once the Kubernetes resources are defined, we can use Helm to install the charts and deploy the microservices onto the Kubernetes cluster.

By running helm install, we can deploy the microservices with a single command:

```sh
helm install -f k8s/bff.yaml bff ./k8s/microservice
helm install -f k8s/query-service.yaml query-service ./k8s/microservice
helm install -f k8s/command-service.yaml command-service ./k8s/microservice
```

Verify that all services were deployed successfully:

```sh
kubectl get deployments
```

Pod status should resemble the following:

```sh
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
bff               1/1     1            1           10m
command-service   1/1     1            1           10m
query-service     1/1     1            1           10m
```

To troubleshoot any issues, check the detailed status of each deployment:

```sh
kubectl describe deployment -n default bff
```

If all pods are running successfully, proceed to check the services from a browser.

Retrieve the service name for the bff service:

```sh
kubectl get service
```

Obtain a URL to connect to the service:

```sh
minikube minikube service bff
```

Finally, verify that all servers are running and returning expected values.

{{<figure src="./graphql_playground.png" alt="GraphQL Playground" width="100%">}}

# Summary

In this series of articles, we’ve explored the Backend For Frontend (BFF) pattern, microservices, and gRPC. Additionally, we’ve touched upon Kubernetes and Minikube for deploying services locally.

Although there are numerous other concepts related to microservices, including Service Mesh, Distributed Transactions, Distributed Logs, and Fault Tolerance, I hope that this article has equipped you with a foundational understanding of the microservices ecosystem.

# References

[Kubernetes Documentation](https://kubernetes.io/docs/home/)

[Kubernetes Tutorial for Beginners [FULL COURSE in 4 Hours]](https://www.youtube.com/watch?v=X48VuDVv0do)

[How to deploy application on Kubernetes with Helm](https://wkrzywiec.medium.com/how-to-deploy-application-on-kubernetes-with-helm-39f545ad33b8)
