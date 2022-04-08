---
title: 'Redis'
date: '2021-09-21'
---

# Intro

Redis は Remote Dictionary Server の略で高速でオープンソースなインメモリ key-value store である。
"
データベース、キャッシュ、メッセージブローカー、キューとして利用される。

## Usage example

Docker を用い Redis Server を run する。

```console
docker run -dit --rm --name=my-redis -p 6379:6379 redis:6.0.8
docker exec -it my-redis redis-cli
```

Redis コンソール内

```console
SET name "keigo"
GET name
>> "keigo"
```

key を上書きするのを防ぐため Namespaces を作ることがある。

慣習的に Namespace を区切るために:や/を使うことが多い。

```console
SET user:keigo:address "Tokyo"
SET user:taro:address "Osaka"

GET user:keigo:address
>> "Tokyo"

GET user:taro:address
>> "Osaka"
```

### Mathematical commands

値を加算するなど算術処理をすることもできる。

```console
SET score:man_c 0
INCR score:man_c
>> (integer) 1
INCRBY score:man_c 3
>> (integer) 4
DECRBY score:man_c 5
>> (integer) -1

MSET score:man_c 5 score:man_u 3
MGET score:man_c score:man_u
>> 1) "5"
>> 2) "3"
```

### Data Types of Redis

Redis では様々なデータタイプを持つことができる。

#### HyperLogLog

Bloom Filters のようなデータ構造で false positive に対してある程度許容でき、データセットが大きいケースに有効。

> Bloom Filters とは m bit の領域を 0 で初期化し、追加する要素を k 個のハッシュ関数に通し得た 0~m-1 の値を bit 領域に対する index とみなし、その bit を立てることで実装される。

つまり値の存在確認、追加ともに O(k)で実行できるデータ構造になっている。

## 参考

[Redis official page](https://redis.io/)

[Redis とは?](https://aws.amazon.com/jp/redis/)

[Complete Intro to Databases](https://frontendmasters.com/courses/databases/key-value-store-databases/)

[ブルームフィルタ Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%96%E3%83%AB%E3%83%BC%E3%83%A0%E3%83%95%E3%82%A3%E3%83%AB%E3%82%BF)
