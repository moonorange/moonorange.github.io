---
title: 'Essential Cloud Infrastructure: Foundation'
date: '2022-09-20'
categories: ["Public Cloud"]
tags: ["GCP", "Japanese Article"]
---

## 概要

https://www.coursera.org/programs/g-i-g-japan-program-5-nk1my?collectionId=vg5ue&currentTab=CATALOG&productId=uI7ZD6WGEeiuigqo2O4RrA&productType=course&showMiniModal=true

Courseraコースのメモ

## VPC

VPCネットワークはバーチャルバージョンの物理ネットワークであり、Andromedaを使ってGoogleの本番ネットワーク内に実装されている

VPCには３タイプある。

{{<figure src="./vpc_network_types.png" alt="VPC network types" width="75%">}}

- デフォルト: 全てのプロジェクト
- Auto Mode: 事前定義されたIPレンジが使用される、カスタムモードに変更できる
- Custom Mode: どのリージョンにどのIPレンジで作成するか決める

同じネットワークにあれば、リージョンが違ってもInternal IPを使い通信できる。

違うネットワークだと、同じリージョンでもexternal IPが必要。


