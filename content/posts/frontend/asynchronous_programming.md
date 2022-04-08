---
title: '非同期処理(Asynchronous Programming)'
date: '2021-04-05'
categories: ["Frontend"]
tags: ["Asynchronous Programming", "JavaScript", "Japanese"]
menu: main
---

# 概要

JavaScript を使う上で重要な概念である非同期処理について説明していく。
非同期処理とは

「ある関数が呼び出されたとき、戻り値として本来渡したい結果を返すのではなく、一度関数としては終了し（＝呼び出し元に戻る）、後で『本来渡したかった値』を返せる状態になったときに、呼び出し元にその値を通知する」という仕組み

JavaScript の非同期処理の重要概念は Callback、Promise、async/await の 3 つがあり、それらについて説明する。

## Callback 関数について

callback とは`ある関数を他の関数に渡すこと`または`引数として渡される関数`のことを言う。
他の関数は、何らかの条件が満たされたとき、または何らかの（非同期の）イベントが発生したときに、引数の関数を呼び出す。

```javascript
function foo() {
  fs.readFile('/etc/passwd', (err, data) => {
    if (err) throw err;
    console.log(data);
  });
  console.log('foo');
}
```

radFile 関数が終了したら 2 つ目の引数に渡した arrow 関数を実行する。
この場合、readFile の`終了を待たずにconsole.log('foo')が先に実行される。`

## Promise について

Promise とは Future パターン（Promise パターン）というデザインパターンの一種で、ECMAScript6（ES2015）で標準化された組み込みのクラスである。
Promise を使った非同期処理の関数は`Promise オブジェクト`(非同期処理の最終的な完了処理 (もしくは失敗) およびその結果の値を表現する)を返す。

Promise オブジェクトとは簡単に言うと、`今は値を返せないが、あとで返すことを約束する`オブジェクトである。

### Promise の状態について

Promise オブジェクトは 3 つの内部状態を持つ

- pending（保留）: まだ非同期処理は終わっていない（成功も失敗もしていない）
- fulfilled（成功）: 非同期処理が正常に終了した
- rejected（拒否）: 非同期処理が失敗した

初期状態は pending であり、`一度fulfilled or rejectedになるとそれ以降は状態が変わらず、終了時に返す値も変わらない`

### コンストラクター

Promise のコンストラクターは関数を引数に取る。そしてその関数は以下の特徴がある。

- 関数は 2 つの関数(resolve, reject)を引数に取る
  - resolve に引数を渡して実行すると状態が fulfilled になり、引数の値が Promise オブジェクトが保持する値になる
  - reject に引数を渡して実行すると状態が rejected になり、引数の値が Promise オブジェクトが保持する値になる
- 関数が例外を投げた場合も状態が rejected になり、投げた値があ Promise オブジェクトが保持する値になる
  - throw する値を rejected に渡して実行した時と同じ

fs.readFile()を Promise 化した例。

```javascript
// Promiseのコンストラクタにresolveとrejectを引数に取るarrow関数を渡す
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, (err, data) => {
      if (err) {
        reject(err); // 失敗: 内部状態をrejectedにする
      } else {
        resolve(data); // 成功: 内部状態をfulfilledにする
      }
    });
  });
}
```

### then(), catch()

非同期処理の結果を取り出す then()と catch()について説明する。

#### then

then()は`2つの関数`を引数に取る。
Promise の状態が fulfilled になったら 1 番目の関数が rejected になると二番目が実行される。

```javascript
readFilePromise('/etc/passwd').then(
  (data) => {
    // 読み出しに成功したらresolve()に渡した値が引数として渡される
    console.log('OK', data);
  },
  (err) => {
    // 読み出しに失敗するか fs.readFile() 自体が例外を投げたら
    // reject()に渡した値が引数として渡される
    console.log('error', err);
  }
);
```

## async 関数について

関数の前に async を宣言することにより、非同期関数（async function）を定義できる。
Promise オブジェクトを返す非同期処理をより簡単に書けるようにするものである。
async を関数の前につけると Promise オブジェクトを返すようになる。

```javascript
async function sample() {}
```

async function には以下の特徴がある。

- async function は呼び出されると Promise を返す
- async function が値を return した場合、Promise は戻り値を resolve する
- async function が例外や何らかの値を throw した場合はその値を reject する

### await

async function 内で Promise の結果（resolve、reject）が返されるまで待機する（処理を一時停止する）演算子のこと。
以下のように、関数の前に await を指定すると、その関数の Promise の結果が返されるまで待機する。

```javascript
function sampleResolve(value) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(value);
    }, 1000);
  });
}

async function sample() {
  const result = await sampleResolve();

  // sampleResolve()のPromiseの結果が返ってくるまで以下は実行されない
  console.log(result);
}
```

## まとめ

非同期処理とはあるタスクの完了を待つ間に別の処理をすることである。
JS の非同期処理の重要概念は call back 関数、Promise、async await の 3 つがある。

callback とはある関数に引数として与えられる関数のことを言い、何かしらの処理が完了された時に呼び戻す関数である。

Promise オブジェクトとはあとで値を返すことを約束するオブジェクトであり pending, fulfilled, rejected の 3 つの内部状態を持つ
Promise のコンストラクタに引数として与えられる関数の引数として resolve、reject をとる
resolve に引数を渡して実行すると状態が fulfilled になり、引数の値が Promise オブジェクトが保持する値になる
reject に引数を渡して実行すると状態が rejected になり、引数の値が Promise オブジェクトが保持する値になる
.then で値を取得することができる

async, await とは Promise より簡潔に非同期処理を記述するためのシンタックスシュガーである。
関数の前に async をつけるだけで Promise を返す非同期処理を書ける。
また async 関数の中で await を使うことで処理の完了を待つことができる。

## 参考

[async/await 入門（JavaScript）](https://qiita.com/soarflat/items/1a9613e023200bbebcb3)

[Asynchronous（非同期）](https://developer.mozilla.org/ja/docs/Glossary/Asynchronous)

[JavaScript の非同期処理を理解する その 2 〜Promise 編〜](https://knowledge.sakura.ad.jp/24890/)
