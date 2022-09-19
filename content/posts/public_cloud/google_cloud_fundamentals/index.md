---
title: 'Google Cloud Fundamentals'
date: '2022-09-17'
categories: ["Public Cloud"]
tags: ["GCP", "Japanese Article"]
---

## 概要

https://www.coursera.org/learn/gcp-fundamentals-jp/home/week/1

courseraのクラスのメモ

## レッドチーム演習

セキュリティを高めるため、サービスに攻撃を仕掛けセキュリティの有効性を確かめる演習

## U2F(Universal 2nd Factor)

OTPベースの二要素認証の代替として安全性を高めたもの
秘密鍵がユーザーのデバイスに保存されネットワークに晒されない。

## GCPの階層構造

全てのGCPリソースはプロジェクトにまとめられる。

オプションでフォルダでプロジェクトをまとめることができる。

フォルダの下にフォルダを作ることもできる。
フォルダを使う際には組織ノードを作る必要がある。

{{<figure src="./cloud-hierarchy.svg" alt="Cloud hierarchy" width="75%">}}

## IAM

誰が何をできるかを定義するサービス
Primitive, Predefined, Customの３種類のロールタイプがある。

Primitiveロールはプロジェクトに適用し、プロジェクト内のすべてのリソースに影響する。

Custom rolesはカスタマイズ性が高いロールを自分で作ることができる。

- Custom rolesをマネージしなければならない
- projectかorganizationレベルでしか適用できない。フォルダには適用できない


同じリソースに対して、階層の上位と階層の下位でIAMポリシーを適用した場合、より上位ポリシーの方が制限が緩い場合は、制限の緩い方が適用される。
上位ポリシーの方が制限が厳しい場合は、厳しい方が優先される。

> たとえば、プロジェクトに適用されているポリシーにより、ユーザー Jane に Cloud Storage バケットの変更権限が付与されているとします。一方、組織レベルのポリシーでは、彼女は Cloud Storage バケットを表示できるだけで、変更はできません。この場合、より制限の緩いポリシーが適用されます。したがって、Jane はバケットを変更できます。 


### サービスアカウント

サーバー間のインタラクションにルールを適用したい場合はservice accountを使う。

例えばVMからしか、Cloud Storageを操作できないようにVMにサービスアカウントを付与するなど。

サービスアカウントは他のリソースと同じように扱え、サービスアカウントにIAMロールを適用することができる。(e.g. Aさんはサービスアカウントを編集できるが、Bさんは見るだけなど)

## Google Cloud Platformの操作

主に4つの操作インターフェースがある。

- Cloud Platform Console(web user interface)
- Cloud Shell and Cloud SDK(Command-line Interface)
- Cloud Console Mobile App(For IOS and Android)
- REST-based API(For custom applications)


## VPC

VPCはグローバルスコープであり、世界中どのRegionにもサブネットを持てる。

サブネットはRegion内の複数ゾーンをまたぐこともできる。つまりリージョンスコープである。

サブネットのサイズは動的に割り当てるIP範囲を拡大するだけで、増やせ、構成済みのVMが影響を受けることはない。

単に2つのVPC間にピアリング関係を確立したい場合は"VPCピアリング"が使える。

一方IAM機能を活用して、別プロジェクトのVPCに対して誰が何をできるかを制御するには、"Shared VPC"を使う。

## Compute Engine

永続はストーレッジは２種類。標準とSSD。

バッチ処理などの動いたり、止まったりするようなワークロードには、プリエンプティブルVMを使うとコストを節約できる。

プリエンプティブルVMは使われていないときは、リソースを解放して止まる。

## Cloud Load Balancing

完全分散型のマネージドサービス。

管理対象のVM内で実行されないので、スケーリングも管理も不要。

HTTPS, HTTP, TCP, SSL, UDPなどの全てのトラフィックに対応している。

## Cloud Big Table

マネージドなNoSQLデータベースサービスである。

単一の参照キーを持つようなデータに適しており、ユースケースとしては購買履歴、トランザクション履歴などのタイムシリーズデータ、ユーザー間のコネクションなどのグラフデータの保存が考えられる。

IoTの使用履歴や財務分析データなど。

データは処理中、保存時にも暗号化される。

## Cloud Spanner

グローバルに分散された強整合性を備えたデータベース。

RDBの構造とNoSQLの水平スケーラビリティを兼ね備えている。

{{<figure src="./large_spanner.max-2200x2200.png" alt="Cloud hierarchy" width="75%">}}

## Kubernetes

オープンソースのコンテナオーケストレーター

{{<figure src="./kubernetes_engine.png" alt="Kubernetes Engine" width="75%">}}

"cluster"という一連のノードにコンテナをデプロイできる。システム全体をコントロールする"master"とコンテナを走らせる"node"のセットである。

"node"とはKubernetesではコンピューティングインスタンスであり、Google CloudではCompute Engine内のVMインスタンスである。

"pod"とはKubernetesの"最小デプロイ単位"であり、クラスタで実行中のプロセスのようなものである。
通常はpod一つにコンテナ一つだが、強い依存関係を持つコンテナが複数ある場合は単一のポッドにパッケージ化することがある。

"Service"とはロードバランシングを代表する際の基本的な手法である。つまりPublic IPを持つロードバランサをサービスに対して紐づけることができる。

"Service"は一連のpodをグループ化し、安定したエンドポイントを提供するものである。

### コマンドラインツール

- gcloud container clusters - GKEコンテナをデプロイしたり、削除したりする
- kubectl - Kubernetes clustersを操作する


## App Engine

フルマネージドなPaaSサービス。

インフラを気にせず、コードのみに集中できる。

NoSQL database, In-memory cache, Load Balancing, Health check, logging, user authenticationなどのwebサービスに必要なサービスが組み込まれている。

ワークロードが変動しやすく、予測できないwebアプリやモバイルアプリのバックエンドに適している。

アプリをローカルでテストし、App Engineサービスにデプロイできる。

### App Engine　二つの環境

App Engineはスタンダードとフレキシブルの二つの環境を提供する。

スタンダード環境はシンプルな環境でデプロイが簡単できめ細かいスケーリングが可能。

App Engineの2つの環境で1日のフリーquotaが存在するが、Standardだと使用量の少ないアプリは無料で実行できる場合がある。

スタンダード環境はランタイムの制限があり、コードはSandboxというもので実行される。

Sandboxの制限は下記
- Local filesへの書き込みはできない
- リクエストタイムアウトが60秒
- third-party　softwareは入れられない

上記のような制約を受けたくない場合はフレキシブル環境を採用する。

違いは下記の表にある。

{{<figure src="./app_engine_comparison.png" alt="App Engine" width="75%">}}

## Google Cloud EndpointsとApigee Edge

API管理ツール。

- Cloud EndpointsはGCPで実行されるアプリをサポート()
- ApigeeはGCP外部のバックエンドサービスも対応(レガシーシステムをマイクロサービスに徐々に移行していくときなどに使える)

## 参考

[Google Cloud Fundamentals: Core Infrastructure ](https://www.coursera.org/learn/gcp-fundamentals-jp/home/week/1)

[Cloud Spanner とは](https://cloud.google.com/blog/ja/topics/developers-practitioners/what-cloud-spanner)