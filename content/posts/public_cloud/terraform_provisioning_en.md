---
title: 'Infrastructure Setup Guide on AWS with Terraform'
date: '2021-12-17'
categories: ["Public Cloud"]
tags: ["Terraform", "Terraform Cloud", "AWS", "English Article"]
---

## Infrastructure Setup Guide

This document outlines the steps for building a deployment environment for a web service on using AWS and Terraform. Infrastructure CI/CD is also achieved through Terraform Cloud and GitHub Actions.

Basic explanations about Terraform are provided.

Please note that the information in this document is current as of December 17, 2021.

The tools you are using, such as Terraform and Terraform Cloud, may have been updated, so be aware that things may not work exactly as described.

## Configuration of Terraform Cloud (TC) and Integration with GitHub

You can achieve CI/CD of infrastructure code using TC and GitHub Actions. 

The goal is to have linting and terraform plan run when a pull request is created, and apply executed when it is merged.

I followed the documentation provided in the following link: https://learn.hashicorp.com/tutorials/terraform/github-actions?in=terraform/automation

### TC Configuration

Create a workspace in TC.

1. Choose an API-driven workflow for integration with GitHub Actions. Additionally, name the workspace in such a way that it 
2. indicates the environment and application.

Once the workspace is created, you can configure environment variables, among other things. 

For constants you want to set across environments, you can use variable sets.

Example:

1. AWS_ACCESS_KEY_ID
2. AWS_SECRET_ACCESS_KEY

Here's how to set up variable sets: https://www.terraform.io/cloud-docs/workspaces/variables/managing-variables#variable-sets

Next, go to the [Token Page](https://app.terraform.io/app/settings/tokens) and create an API Token for use with GitHub Actions. Make sure to copy the created token as you'll need it later.

Finally, set the Terraform Working Directory in the workspace's settings/general.

### GitHub Configuration

In GitHub Actions, navigate to the `Settings` of the repository where you want to implement CI/CD and go to `Secrets`.

Define the API Token created in TC as `TF_API_TOKEN`

## Creating a GitHub Actions Workflow

To actually execute Terraform commands in GitHub Actions, you need to create a configuration file. 

You can refer to the following article for the basics: https://www.ai-shift.co.jp/techblog/1924

Create a `.github/workflows/{any_name}.yml` file under the project's root.

Here's an example of the content, assuming you want to operate the staging environment on the develop branch:

```yaml
name: 'Terraform Plan'

on:
  push:
    paths:
    - envs/staging/**
    - modules/**
    branches:
      - develop
  pull_request:
    paths:
    - envs/staging/**
    - modules/**
    branches:
      - develop

jobs:
  terraform:
    name: 'Terraform staging'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./envs/staging
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      with:
        cli_config_credentials_token: ${{ secrets.TF_API_TOKEN }}

    - name: Terraform Init
      id: init
      run: terraform init

    - name: Terraform Format
      id: fmt
      working-directory: ./
      run: terraform fmt -check -recursive
      continue-on-error: true

    - name: Terraform Plan
      id: plan
      if: github.event_name == 'pull_request'
      run: terraform plan
      continue-on-error: true

    - name: Terraform Plan Status
      if: steps.plan.outcome == 'failure'
      run: exit 1

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve
```

Once you have this setup, the workflow should run when you create pull requests or push commits according to the conditions you've specified.

## Testing the GitHub Actions Workflow

Create folders and files for each environment:

```bash
mkdir staging
cd staging
touch {main,provider}.tf
```

- main is the entry point for Terraform code.
- provider contains configuration information for cloud providers like AWS.

`provider.tf:`

```yaml
provider "aws" {
  region  = "ap-northeast-1"

  default_tags {
    tags = {
      Service = "{your service name}"
      Env     = "production"
    }
  }
}

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.65.0"
    }
  }

  required_version = ">= 1.0.0"

  backend "remote" {
    organization = "{Your organization name}"

    workspaces {
      name = "{Your workspace name}"
    }
  }
}
```

Define the environment variables that you previously set in TC here as well. Additionally, let's add an S3 module.

Under the modules/ directory, you can define AWS resources that you want to use across environments.

`main.tf:`

```yaml
locals {
  env     = "prod"
  service = "{your service name}"
}

variable "AWS_ACCESS_KEY_ID" {}
variable "AWS_SECRET_ACCESS_KEY" {}

module "s3_bucket" {
  source = "../../modules/s3"

  bucket = "{your bucket name}"
  acl    = "private"
}
```

`provider.tf:`

```yaml
provider "aws" {
  region  = "ap-northeast-1"

  default_tags {
    tags = {
      Service = "{your service name}"
      Env     = "production"
    }
  }
}

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.65.0"
    }
  }

  required_version = ">= 1.0.0"

  backend "remote" {
    organization = "{Your organization name}"

    workspaces {
      name = "{Your workspace name}"
    }
  }
}
```

Define the environment variables that you previously set in TC here as well. Additionally, let's add an S3 module.

Under the modules/ directory, you can define AWS resources that you want to use across environments.

`main.tf:`


```yaml
locals {
  env     = "prod"
  service = "{your service name}"
}

variable "AWS_ACCESS_KEY_ID" {}
variable "AWS_SECRET_ACCESS_KEY" {}

module "s3_bucket" {
  source = "../../modules/s3"

  bucket = "{your bucket name}"
  acl    = "private"
}
```

With this setup, you can run `terraform plan` to see what kind of bucket will be created when you try to create only the S3 module. 

`terraform plan` doesn't actually create resources, so you can safely run it.

```bash
terraform plan
```

If the S3 module is created successfully, proceed to create a pull request and confirm that the GitHub Actions workflow runs successfully.

In the repository settings, under `Branches`, add branch protection rules. Set `Require status checks to pass before merging` to true for the main branch. This ensures that the Terraform production workflow must pass for PRs to be merged.

## Explanation of Each .tf File

Here's an explanation of the three main types of files: `var.tf`, `output.tf`, and `main.tf`.

### var.tf

This file is used to define variables. Usage is straightforward: values that you want to change dynamically in `modules*/main.tf` are defined in `var.tf` and passed from `envs/*/main.tf`.

`Example: modules/sg/var.tf`

```yaml
variable "env" {
  type        = string
  description = "Environment. e.g. staging, prod"
}

variable "service" {
  type        = string
  description = "Service name"
}

variable "vpc_id" {
  type        = string
  description = "VPC ID"
}

variable "ec2_sg_name" {
  type        = string
  description = "Security group name for ec2 instance"
}
```

### output.tf

This file is used to describe values that you want to use outside the module.

For example, if you want to use the security group's ID inside the RDS module, you would use this file.

`Example: modules/sg/output.tf`

```yaml
output "alb_sg_id" {
  description = "Security group id for alb"
  value       = aws_security_group.alb.id
}

output "fargate_sg_id" {
  description = "Security group id for fargate tasks"
  value       = aws_security_group.fargate_sg.id
}

output "db_sg_id" {
  description = "Security group id for db"
  value       = aws_security_group.db.id
}

output "ec2_sg_id" {
  description = "Security group id for ec2 for script"
  value       = aws_security_group.ec2_sg.id
}
```

### main.tf

This is the entry point for Terraform. The resources defined in this file will be created.

`envs/*/main.tf` serves as the entry point for all resources, and `modules/*/main.tf` is the entry point for each module.

`Example: Part of modules/sg/main.tf`

```yaml
resource "aws_security_group" "fargate_sg" {
  name        = "${var.service}-${var.env}-fargate-sg"
  vpc_id      = var.vpc_id
  description = "For fargate tasks"

  egress {
  }

  ingress {
  }

  ingress {
  }

  tags = {
    Name = "${var.service}-sg-fargate-${var.env}"
  }
}

resource "aws_security_group" "db" {
  vpc_id = var.vpc_id
  name   = "${var.service}-sg-db-${var.env}"

  ingress {
    from_port       =
    to_port         =
    protocol        = "tcp"
    security_groups = [aws_security_group.fargate_sg.id, aws_security_group.ec2_sg.id]
  }

  egress {
  }

  tags = {
    Name = "${var.service}-sg-db-${var.env}"
  }
}
```

`Example: Part of envs/staging/main.tf`

In this file, you pass variables and other configurations to the resources defined in each module to create all modules.


```yaml
# VPC and subnets settings
module "vpc" {
  source  = "../../modules/vpc"
  env     = local.env
  service = local.service

  cidr_block = "172.31.0.0/16"
}

# Security group
module "secutiry_group" {
  # moduleまでの相対pathを指定する
  source      = "../../modules/sg"
  # moduleで使う変数を定義する。
  env         = local.env
  service     = local.service
  # modules/vpc/output.tfで定義した値を使用
  vpc_id      = module.vpc.vpc_id
  ec2_sg_name = "{sg name}"
}
```

## Creating Resources Required for Web Service Deployment

With the setup of GitHub Actions and TC for CI/CD and an explanation of Terraform file structure, you can proceed to create the necessary resources for deploying a web service.

### Modules to Create

Listed below are the modules that should be created. 

When all of these modules are created, you will have everything needed for a release:

- iam_role (To create a role that allows ECS tasks to run)
- s3_bucket (For storing static files and more)
- vpc (Virtual network environment, subnets, Internet Gateway, route table)
- domain (Domain, SSL/TLS certificate, and API host records)
- security_group (To manage outbound from instances and inbound to instances)
- alb (Application Load Balancer for load balancing and HTTPS)
- ecr (To store Docker container images)
- rds (A service for creating databases)
- ecs (To manage Docker containers and run services)

You can refer to Terraform's official documentation and other resources to create each module.

#### rds

- [instance_class](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/Concepts.DBInstanceClass.html) Determine based on data volume and other factors in production

#### ecs

- image_uri: Copy the image URI from ECR.
- cpu: Choose an appropriate value (e.g., 1024, 2048). You can decide based on CPU usage observed in the ECS console after - deployment.
- memory: Choose an appropriate value (e.g., 1024, 2048). You can decide based on memory usage observed in the ECS console after - deployment.
- desired_count: Determine based on how many tasks you want. Generally, the more you have, the more load it can handle.

### Running terraform apply

Once you have created the resources, execute `terraform apply` to actually provision them.

You can choose to create modules one by one and run terraform plan to confirm there are no issues, which can be easier and involve fewer changes.

Please note that you should not create the `ECS module` at this stage.

Once the modules other than ECS are successfully created, push your Docker image to the ECR you created.

Go to the ECR repository page and select the repository you created. Then, choose View push commands to see the push command. You can generally use the push command shown there as a reference.

Obtain an authentication token and authenticate the Docker client with the ECR registry.

Build the Docker image from your Dockerfile.

The --platform part is explicitly specifying amd64 because the author is using an ARM Mac.

```console
docker build -t {tag_name} -f docker/app/Dockerfile --platform amd64 .
```

Tag the built image.

Push the image to the ECR repository.

Once the image is pushed, copy the image URI. Go to the repository you created and click `Copy URI`.

After completing these steps, you can proceed to write the ECS module.

Paste the previously copied image_uri and make any necessary changes to other variables. Then, run terraform plan and terraform apply.

With this, your infrastructure setup is complete. If you encounter any issues, be sure to check `CloudWatch logs` or other relevant resources for troubleshooting.
