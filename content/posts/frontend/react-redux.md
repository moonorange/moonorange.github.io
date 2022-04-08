---
title: 'ReactとReduxについて(About React and Redux)'
date: '2021-03-26'
categories: ["Frontend"]
tags: ["React", "Redux", "JavaScript", "Japanese Article"]
---

# 概要

Summarizing what I learned about React and Redux

## React

React, Javascript 周辺の用語

- Babel 新しい仕様の JavaScript や JSX、TypeScript のコードを古いブラウザでも実行可能な JavaScript にコンパイルするコンパイラー
- webpack コンパイラと連携しつつ大量のソースコードファイルをひとつにまとめ、種々の最適化を施す bundler
- react-dom DOM を抽象化して React から操作できるようにするレンダラーのパッケージ
- jest オールインワンの javascript テスティングフレームワーク

## コンポーネント

https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

コンポーネントで大事な要素
• Props 　親コンポーネントから受け取る値
• Local State 　コンポーネント自身が内部に持つ状態
• ライフサイクル　初期化されてマウントされレンダリングされ、何らかの処理が行われて再レンダリングされたりして、最後にアンマウントされるまでの過程

## React Hooks

### useEffect

何かコンポネントのライフサイクル内でない処理をスケジュールするときに使う。
API 呼び出しやタイマーを設定するなど。

### useContext

"data tunneling" or "prop drilling"といった親コンポーネントから深い階層の子コンポーネントに props をバケツリレーのように運んでいかなければならないという問題があった。
この問題を解消するため、UserContext.Provider 内であればどこからでも使えるデータにしたのが useContext である。
コンテクストを使うとアプリケーションの複雑性が増すため真に application 全体で使う状態(user 情報や auth key など)にのみ使うようにした方が良い。
Redux のように機能するので一緒に使うのはあまりお勧めではない。

### useRef

レンダリングサイクルが終わった後にも保持したい state がある場合は ref を使う。
ref が保持している値が変わっても再描画はされない。

### useReducer

useState と成し遂げることは同じで違うやり方を取る hook、redux の reducer と同じ。
action と state を引数に取り action を state に適用する。
useState を比べて一見複雑に見えるが、state が変更するケースを全て一覧でみれるという利点がある。
単体テストも全てのケースの書きやすくなる。

### useMemo

計算量が多い関数などを特定の値が変わるまで再評価しないようにするために使う。
例えばフィボナッチ数列を再起的に計算する関数など。
アプリでパフォーマンスの問題が発生した場合に使うことを推奨し事前に使うことは複雑性を増すことになるのでお勧めしない。

```javascript
const fibonacci = (n) => {
  if (n <= 1) {
    return 1;
  }

  return fibonacci(n - 1) + fibonacci(n - 2);
};

useMemo(() => fibonacci(num), [num]);
```

### useCallback

関数を受け取り dependency に変更ない限りは同じ関数を呼び出すようにし re ー render を防ぐもの
useMemo と似ており内部的には useMemo が使われている。

### useLayoutEffect

useEffect とほぼ同じだが useEffect がいつスケジュールされるか分からないのに対し component が update されたすぐ後に実行されるという確約がある点で異なる。
animation などでイベントの後すぐに実行しなければならない関数などがある場合に使う。

### Code Splitting

コードの使う部分だけを load することを可能にする技術。
全てのコードが一度に bundle されるのを防げるためデータ量の削減、パフォーマンス向上に役に立つ。

## 参考

[Intermediate React, v3](https://frontendmasters.com/courses/intermediate-react-v3/)
