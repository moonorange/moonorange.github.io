---
title: 'awsでのwebサーバー構築について'
date: '2020-02-12'
categories: ["Public Cloud"]
tags: ["AWS", "Japanese Article"]
---

# chapter 1

サーバーというのは、「Linux」や「WindowsServer」など、サーバー用の OS をインストールしたコンピュータのこと

## VPC

AmazonVirtualPrivateCloud の略。ユーザー専用のプライベートなクラウド環境を提供するサービスである。

{{<figure src="./vpc_structure.png" alt="VPC structure" width="75%">}}

## NAT

NAT は NetworkAddressTranslation の略で、内部だけで通用するアドレスを外部とも通信できるアドレスに変換する技術

NAT を導入すると、「片方向だけの接続を許す」ことが実現できる。
たとえば、DB サーバーからはインターネットに接続できるが、逆に、インターネットから DB サーバーには接続できないという環境を実現できる

# chapter 2

## IP アドレス

TCP/IP で使われている通信先を特定するためのプロトコル。
IP アドレスは、32 ビットで構成されている。

{{<figure src="./ip_address.png" alt="IP Address" width="75%">}}

図のようにネットワーク部は同じネットワークに属する限りは同じ値
ホスト部が割り当てたいサーバーやクライアント、ネットワーク機器に対する値となる。

インターネットに接続する際に用いる IP アドレスのことを「パブリック IP アドレス」と言う。
インターネットで使われないものをプライベート IP アドレスという

{{<figure src="./private_ip.png" alt="private ip address" width="75%">}}

## IP アドレスの表記方法

### CIDR 表記(ClasslessInterDomainRouting)

IP アドレスを 2 進数で表記したとき、「ネットワーク部のビット長」を「/ビット長」で示す方法のこと
このビット長のことを「プレフィックス（prefix）」と言う。たとえば「192.168.1.0 ～ 192.168.1.255」の場合、プレフィックスは 24 ビット。
CIDR 表記だと 192.168.1.0/24」と記述される。

### サブネットマスク

「192.168.0.0 ～ 192.168.255.255」の場合、サブネットマスクは「255.255.0.0」で、「192.168.0.0/255.255.0.0」と表記する。

## パケット

{{<figure src="./packet.png" alt="パケット" width="75%">}}

### パケットフィルタリング

パケットフィルタリングは、流れるパケットを見て、通過の可否を決める仕組み。
パケットには、「IP アドレス」のほか「ポート番号」も含まれる
パケットフィルタリングは、「IP アドレス」と「ポート番号」など、パケットに付随する各種情報を見て、通過の可否を決める.

## DNS

ドメイン名を IP アドレスに変換するときに使われる仕組み
DNS を用いて、あるドメイン名から、それに対応する IP アドレスを引き出すことを「名前解決」と呼ぶ

## VPC

### VPC を作成する

「IPv4CIDR ブロック」は、使用する IP アドレス範囲。「10.0.0.0/16」などと設定する。
後ほどこちらのネットワーク内に db サーバーや api サーバー(ec2 インスタンス)を置く。

### subnet を作成する

上記で決めた範囲の CIDR ブロックをさらに細分化した CIDR ブロックのことを「サブネット（Subnet）」と言う。
サブネットを作るとその部分だけでネットワークを分けることができる。

物理的に分散させるため複数 Availability Zone(AZ)で subnet を作成する。地理的な要因で起こる障害に対応するため。既存のものでは３つ subnet を作っている。

パブリックサブネットとプライベートサブネットがあり、パブリックはインターネットにアクセスすることを目的としたサブネットである。

### internet gateway(ig)を作成する

VPC とインターネットとの接続ポイントを作る。

### Route Tables で internet gateway につなげる

routes は下記の通りに

Destination: 172.31.0.0/16
Target: local

Destination: 0.0.0.0/0
Target: 作成した ig

## RDS

postgreSQL エンジンで作る。

- Additional configuration で databasename をつけておく(後ほどデータベースサーバーにつなぐ際に使用)
- standby instance は小規模だと必要ないが、ある程度の規模のサービスだとアクセスを分散して I/O のパフォーマンスを安定させられるので必要

## EC2

Security Groups で Security Group を作る。
Inbound で HTTP(80)、HTTPS(443)、SSH(22)を開ける。Source は everywhere(0.0.0.0/0)。
Outbound は All traffic にする。

- EC2 インスタンスを Amazon ECS-Optimized Amazon Linux 2 AMI で作る.
  ecs に最適化された AMI である。ECS で AWS Marketplace で検索すれば出てくる。
- Auto-assign Public IP は disable にする。Public IP は固定にしたいので Elastic IP address (EIP) を使うため
- IAM は ecsInstanceRole
- 先述の Security Group をアタッチして port を開ける。
- Elastic IPs で Allocate new address で EIP をつくった ec2 インスタンスに割り当てる。

## Route53 　ドメイン取得

任意のドメインを取得する。
サブドメインが必要な場合はこちらで登録する

{{<figure src="./original_domain.png" alt="独自ドメイン" width="75%">}}

## ACM(Certificate Manager) SSL 証明書の発行

取得したドメインの証明書をリクエストする

- DNS validation
- SSL の証明書の検証をするためのドメインを自動で作ってくれる

## EC2

### Target Groups

target group を作成する

- Protocol は HTTP
- Health checks は DRF の場合、path が"/"で Success codes が 400

### Load Balancers

- internet facing
- Listeners は HTTPS
- Availability Zones は作っておいた subnet ３つ
- Certificate は acm から、作っておいた証明書を使う

### Route53 ドメインを ALB に割り当て

作った ALB(Application Load Balancer)にドメインを割り当てる。

## ECR 　コンテナを push

Amazon Elastic Container Registry (ECR) は、完全マネージド型の Docker コンテナレジストリです。このレジストリを使うと、開発者は Docker コンテナイメージを簡単に保存、管理、デプロイできます。

ECR にイメージを push して登録

## ECS

フルマネージド型のコンテナオーケストレーションサービス
docker をローカルで使うのは簡単だが、複数のクラスタ(ec2 インスタンス群) 上で管理するのは大変
それをマネージしてくれるのが ecs、コンテナの管理、配置などをよしなにしてくれる

create cluster

- EC2 Linux + Networking template を使う
- Provisioning Model は On-Demand Instance
- Subnets は３つ共 attach する

create new task definition

- Task Role は ecsTaskExecutionRole
- Task memory(MiB)と Task CPU(unit) はタスクが２つ走っても大丈夫なよう確保しておく
  (Add Container)
- add continer で ecr に push したコンテナを add する
- コンテナの hard limits(これを越えると OOMKiller が走り、コンテナが停止される)soft limits(ホストのメモリに余裕がある時は制限以上のメモリを使用する場合もあり)がある
- Port mappings は host は 80 で container port は DRF(Django Rest framework)だと 8000
- Environment variables で必要な環境変数を定義する
- Log configuration で log を出力するようにして置くとエラー対応がしやすい

create service

- launch type は ec2
- Load balancer type は Application Load Balancer で作っておいた lg を指定する
- Add to Load Balancer で作っておいた tg を指定する

Run new Task

- task definition を使って task を走らせる

## テスト

ここまで出来たらドメインにアクセスして正常に表示されるかチェックする。
出来ていない場合は Cloud Watch の Log gropus などで log を確認してデバッグする。

## Circle CI で自動デプロイ

kts_api や actaba_api の.circleci/config.yml を参考にして書く
circle_sha1 でイメージ version をユニークに
master ブランチのみデプロイ
circle ci で aws のアクセス token などを登録、aws 系は全部必要になる

## 参考

[Amazon Web Services 基礎からのネットワーク＆サーバー構築　改訂 3 版 Kindle 版](https://www.amazon.co.jp/Amazon-Web-Services-%E5%9F%BA%E7%A4%8E%E3%81%8B%E3%82%89%E3%81%AE%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%EF%BC%86%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E6%A7%8B%E7%AF%89-%E6%94%B9%E8%A8%823%E7%89%88-%E5%A4%A7%E6%BE%A4-ebook/dp/B084QQ7TCF/ref=sr_1_5?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&dchild=1&keywords=aws&qid=1602238206&sr=8-5)

[AWS：無料で SSL 証明書を取得する方法](https://qiita.com/iwaseasahi/items/1687426add4124899fe3)

[AWS のサービスでドメインを取得し ALB で SSL で接続出来るようにする](https://qiita.com/keitakn/items/4b2db95eae81044a779c)

[AWS ECS で Docker コンテナ管理入門（基本的な使い方、Blue/Green Deployment、AutoScaling などいろいろ試してみた）](https://qiita.com/uzresk/items/6acc90e80b0a79b961ce)

[Amazon EC2 Container Service(ECS)の概念整理](https://qiita.com/NewGyu/items/9597ed2eda763bd504d7#:~:text=%25E3%2582%25AF%25E3%2583%25A9%25E3%2582%25B9%25E3%2582%25BF%25E3%2583%25BC%25E3%2581%25AFContainer%2520Instance%25E3%25)

[ECS の CPU・メモリ設定と Docker コマンドの対応](https://qiita.com/omukaik/items/66ad4181eb13531f9c4a)
