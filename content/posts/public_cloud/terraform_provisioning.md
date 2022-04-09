---
title: 'Terraformでのインフラ環境構築の手順書'
date: '2021-12-17'
categories: ["Public Cloud"]
tags: ["Terraform", "Terraform Cloud", "AWS", "Japanese Article"]
---

# インフラ構築の手順書

Terraformを使いWebサービスのデプロイ環境を構築していく際の手順を記述する。
Terraform CloudとGithub ActionsでインフラのCI/CDも実現している。

Terraformの基礎的な説明をしている。

情報は2021/12/17日現在のものであるため、使用しているTerraformやTerraform Cloudなどのツールがアップデートされ、資料通りではない可能性もあることに留意すること。

## Terraform Cloud（TC)の設定とGithubとの連携

TCとGithub Actionsを使うことでインフラコードのCI/CDを実現できる。
PRを作成した際にlint, terraform planが走り、mergeされたらapplyが実行されることを目指す。

下記のドキュメントを参考に進めた。
<https://learn.hashicorp.com/tutorials/terraform/github-actions?in=terraform/automation>

### TCの設定

TCでworkspaceを作る。

Github Actionsと連携するためAPI-driven workflowを選択する。またworkspace nameは環境名とアプリケーション名が分かる様につける。

workspaceが作成できたら環境変数などを設定できる。環境横断的に設定したい定数についてはvariable setsで設定できる。

例

1. AWS_ACCESS_KEY_ID
2. AWS_SECRET_ACCESS_KEY

variable sets設定方法。
<https://www.terraform.io/cloud-docs/workspaces/variables/managing-variables#variable-sets>

次に[Token Page](https://app.terraform.io/app/settings/tokens)にいき、”Create an API Token"でGit Actionsで使うためのAPI Tokenを作成する。
作成したTokenは後で使うのでコピーしておくこと。

最後にworkspaceのsettings/generalのTerraform Working Directoryを設定する。

### Githubの設定

Github ActionsでCI/CDを実現したいレポジトリの"Settings"から"Secrets"に飛ぶ。

TF_API_TOKENという名前で先程 TCで作ったAPI Tokenを定義する。

## Github Actions Workflowの作成

Github Actionsで実際にterraform commandsを実行するための設定ファイルを記述していく。

基本的には下記の記事を参考に進めていく。
<https://www.ai-shift.co.jp/techblog/1924>

プロジェクトルート配下に`.github/workflows/{任意の名前}.yml`を作成する。

中身は以下のようにしている。
developブランチでstaging環境を運用することを想定。

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
    # Checkout the repository to the GitHub Actions runner
    - name: Checkout
      uses: actions/checkout@v2

    # Install the latest version of Terraform CLI and configure the Terraform CLI configuration file with a Terraform Cloud user API token
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1
      with:
        cli_config_credentials_token: ${{ secrets.TF_API_TOKEN }}

    # Initialize a new or existing Terraform working directory by creating initial files, loading any remote state, downloading modules, etc.
    - name: Terraform Init
      id: init
      run: terraform init

    # Checks that all Terraform configuration files adhere to a canonical format
    - name: Terraform Format
      id: fmt
      working-directory: ./
      run: terraform fmt -check -recursive
      continue-on-error: true

    # Generates an execution plan for Terraform
    - name: Terraform Plan
      id: plan
      if: github.event_name == 'pull_request'
      run: terraform plan
      continue-on-error: true

    - name: Terraform Plan Status
      if: steps.plan.outcome == 'failure'
      run: exit 1
      # On push to main, build or change infrastructure according to Terraform configuration files
      # Note: It is recommended to set up a required "strict" status check in your repository for "Terraform Cloud". See the documentation on "strict" required status checks for more information: https://help.github.com/en/github/administering-a-repository/types-of-required-status-checks
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve
```

ここまで出来たらyamlで設定した通りの条件でpull requestなどを出したタイミングでworkflowが実行されるはずなので試してみる。

## Github actions workflowのテスト

環境ごとに作るフォルダ、ファイルの作成

```console
mkdir  staging
cd staging
touch {main,provider}.tf
```

mainはterraformコードのエントリーポイント、providerはawsなどクラウドプロバイダーを操作するための設定情報を書く。

provider.tf

```hcl
provider "aws" {
  region  = "ap-northeast-1"

  # These tags will be attached to every resource in this environment
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
    # TC organization name
    organization = "{Your organization name}"

    # TC workspace name
    workspaces {
      name = "{Your workspace name}"
    }
  }
}
```

先程、TCで定義した環境変数をこちらでも定義する。
またs3のmoduleだけ追加してみる。

modules/配下には環境を跨いで使うAWSのリソースが定義してある。

main.tf

```h
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

この状態でとりあえずterraform planを実行してみる。
planはどういう変更を加えようとするかを教えてくれるコマンドであり実際のリソースは作成されないので気軽に実行して良い。

```console
terraform plan
```

s3のmoduleだけを作ろうとするのでどういったバケットが作られるのか確認できるはず。

ここまで出来てplanも正常に実行されていたら実際にPRを作り、Github actionsのwork-flowが正常に実行される確認すると良さそう。

settingsのbranchesからbranch protection rulesでルールを追加する。
mainブランチでRequire status checks to pass before mergingをTrueにする。

これによってTerraform prodのワークフローが通らないとPRがマージできなくなる。

## 各tfファイルの説明

主に使う３種類のファイルであるvar.tf, output.tf, main.tfについてそれぞれのファイルが何をしているかを説明する。

### var.tfについて

変数を定義するファイル。
<https://www.terraform.io/language/values/variables>
使い方は単純で`modules*/main.tf`で動的に変更したい値はvar.tf内に書いて`envs/*/main.tf`から渡すようにする。

例: `modules/sg/var.tf`

```h
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

### output.tfについて

module外で使いたい値について記述するファイル。
<https://www.terraform.io/language/values/outputs>
例えばsecurity groupのidをrds module内で使いたい際などに使う。

例: `modules/sg/output.tf`

```h
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

### main.tfについて

Terraformのエントリーポイント。このファイル内に記述されているリソースが作成される。`envs/*/main.tf`は全てのリソースのエントリーポイントであり、`modules/*/main.tf`はそのmoduleのエントリーポイントである。

例：`modules/sg/main.tf`の一部

```h
resource "aws_security_group" "fargate_sg" {
  # 文字列に変数を埋め込みたい場合は${変数名}のように書く
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
    # ファイル内の値についてはリソースタイプ名.ローカル名.欲しい値などで使うことができる
    security_groups = [aws_security_group.fargate_sg.id, aws_security_group.ec2_sg.id]
  }

  egress {
  }

  tags = {
    Name = "${var.service}-sg-db-${var.env}"
  }
}
```

例：`envs/staging/main.tf`の一部

このファイルで各moduleで定義したリソースに変数などを渡し、全てのmoduleを作成する。

```h
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

## Webサービスのデプロイに必要なリソースの作成

Github Actions, TCでのCI/CDの構築が終了しTerraformファイルの書き方を説明したので、次にWebサービスのデプロイに必要なリソースを作成する。

### 作るべきmodulesについて

作るべきmodulesについて羅列する。下記のmoduleが全て作成されればリリースに必要なものが揃っていることになる。

- iam_role(ecsのタスクを実行できるroleを作成する)
- s3_bucket(静的ファイルなどの保存先)
- vpc(仮想ネットワーク環境, subnets, InternetGateway, route table)
- domain(ドメイン、SSL/TLS証明書、apiホストのレコードなど)
- security_group(インスタンスからの発信、インスタンスへの受信を管理する)　<https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/VPC_SecurityGroups.html>
- alb(Application Load Balancer、負荷分散、HTTPS化のために使用)
- ecr(Dockerコンテナイメージを登録するところ)
- rds(データベースを作成できるサービス)
- ecs(Dockerコンテナをマネージしサービスを動かすもの. ECRが作成されイメージをpushした後に作る)

基本的にはterraformの公式ページなどを参考に各moduleを作成していけば良い。

#### rds

- [instance_class](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/Concepts.DBInstanceClass.html) 本番でのデータ量などに応じて決める

#### ecs

- image_uri: ECR上のイメージuriをコピペする
- cpu 適当な値(1024, 2048など)。デプロイ後、ECSのコンソールでCPU使用率などを見てから決めても良い
- memory 適当な値(1024, 2048など)。デプロイ後、ECSのコンソールでメモリ使用率などを見てから決めても良い
- desired_count 何個のタスクが望ましいかに応じて決める。基本的に多いほど重い処理に耐えられる。

### terraform apply実行

リソースが作成できたらterraform applyを実行しリソースを実際にprovisioningしていく。
一気にmoduleを書いていってもいいが1moduleずつ書いて`terraform plan`を実行し挙動に問題がないかを確認していくと変更が少なく見やすい。
この時、ecs moduleはまだ書かないことに注意する。

ecs以外のmoduleが正常に出来たら、作成したECRにDocker Imageをpushする。

ECRのリポジトリページに行き、作成したレポジトリを選択する。
その後、`プッシュコマンドの表示`を選択する。
<https://ap-northeast-1.console.aws.amazon.com/ecr/repositories?region=ap-northeast-1>

基本的には上記で表示されたpushコマンドを参考にコマンドを記述する。

認証トークンを取得し、ECRレジストリに対して Docker クライアントを認証

DockerfileからDocker imageをbuildする。
--platform部分は筆者がarm Macなので明示的にamd64を指定している。

```console
docker build -t {tag_name} -f docker/app/Dockerfile --platform amd64 .
```

buildしたイメージにtagをつける。

作成したECRにイメージをpushする。

イメージがpush出来たらイメージのuriをコピーする。
作成したリポジトリ画面に行きURIのコピーを押す。

ここまでした後にecsのmoduleを記述する。
先程、コピーしたimage_uriをコピペし、他の変数も必要に応じて変更しplan, applyを実行する。

以上で環境構築が完了する。
何か問題がある場合はCloud watchのログなどを確認すること。
