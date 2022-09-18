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

サブネットはRegion内の複数ゾーンをまたぐこともできる。

サブネットのサイズは動的に割り当てるIP範囲を拡大するだけで、増やせ、構成済みのVMが影響を受けることはない。

## 参考

[Google Cloud Fundamentals: Core Infrastructure ](https://www.coursera.org/learn/gcp-fundamentals-jp/home/week/1)