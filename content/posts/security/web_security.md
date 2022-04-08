---
title: 'Web脆弱性'
date: '2021-10-31'
categories: ["Security"]
tags: ["Web Vulnerability", "Japanese"]
menu: main
---

## CSRF(Cross Site Request Forgery)

外部サイトを経由したサーバーへの悪意のあるリクエストによって利用者の意図しない処理を実行する攻撃。

user がログインした状態で悪意あるリンクを踏みそれが DELETE リクエストを送り user の情報を消してしまうなどが起こりうる。

対策

- GET, HEAD, OPTIONS などの safe method を副作用を持たないようにする(サーバーサイドの状態を何も変えないものにする)
- POST, PUT, PATCH, DELETE などの unsafe のリクエスト時は csrf token を必要とする

## XSS

web アプリにスクリプトを埋め込み利用者のブラウザで不正なスクリプトが実行されてしまう脆弱性。
{{<figure src="./xss.png" alt="XSS" width="75%">}}

Cookie などに user の認証情報が入っていた場合盗まれるなどの問題が発生する。

対策

- web ページに出力するすべてのデータのエスケープ処理(web ページの出力に際して意味を持つ文字などを置換するなど)
- 入力値の内容チェック

## Dos attack(Denial of Service attack)

web サイトやサーバーに対して過剰なアクセスや不正データを送付しトラフィックを増やしたりすることで、サービス停止に陥らせようとする攻撃。

攻撃元となるサーバーを複数に分散して一斉に攻撃することを DDos attack(Distributed Denial of Service attack)と呼ぶ。

対策

- 攻撃元と思われる IP アドレスからのアクセスを制限
- 国レベルでアクセスを制限
- ネットワークトラフィックを監視するサービスを導入し攻撃元と思われる IP を制限

## 参考

[IPA 安全なウェブサイトの作り方](https://www.ipa.go.jp/security/vuln/websecurity.html)
