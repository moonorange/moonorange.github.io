---
title: 'GCP Decision Tree'
date: '2022-09-03'
categories: ["Public Cloud"]
tags: ["GCP", "English Article"]
---

## Decision Tree

Summarize decision trees to decide which gcp services you should use in a certain circumstance.

### Compute option

GCP offers a multiple compute services.

Here is a list of those services and how customizable/managed they are.

{{<figure src="./gcp_compute_services.png" alt="Compute Services List" width="75%">}}


Decision Tree

{{<figure src="./compute_options_dt.png" alt="Compute Options Decision Tree" width="75%">}}

Services

{{<figure src="./compute_services.jpeg" alt="Compute Services" width="75%">}}

#### Firebase

When you're a mobile or html developer and want the least of amount of server-side code possible, you should choose Firebase.

Firebase provides Database, Storage, Functions/Services, and Hosting with a little code.

#### Cloud Functions(Serverless)

If your application is an event-driven app and you want the least server-side code, you should use Cloud Functions.

Cloud Functions is like a AWS Lambda. Only pay for when your functions are running.

#### App Engine(Serverless)

If you don't want to separate codes into many independent tasks(functions) and still want serverless services, you can use App Engine.

Can Scale to high and low traffic.


#### Cloud Run

With Cloud Run, you can use any language, any library, any binary.

You can use Cloud Run if you use a container.

#### GKE

If you use Kubernetes for you container orchestration tool, you can use GKE.

## References

[Choosing the right compute option in GCP: a decision tree](https://cloud.google.com/blog/products/compute/choosing-the-right-compute-option-in-gcp-a-decision-tree)

[Where Should I Run My Stuff Choosing Google Cloud Compute Option](https://cloud.google.com/blog/ja/topics/developers-practitioners/where-should-i-run-my-stuff-choosing-google-cloud-compute-option)
