---
title: 'デザインパターン２'
date: '2020-08-05'
---

# 8 Abstract Factory

抽象的な部品を組み合わせて抽象的な製品を作る

# 実装

## 抽象的な部品

### Item

link と tray のスーパークラス
link と tray を同一視するためのクラス
caption field はこの項目の見出し
makehtml method は抽象メソッドで html の文字列が返り値となるようサブクラスで実装

### Link

html のハイパーリンクを抽象的に表現したクラス
url field はリンクの飛び先 url
makehtml を実装してないので抽象クラス

### Tray

複数の link や tray を集めてひとまとまりにしたもの
link や tray を add method で集める。それらの super class である item を引数にとる
makehtml を実装してないので抽象クラス

## 抽象的な製品

### Page

html ページ全体を抽象的に表現したクラス
link や tray が抽象的な部品であるなら page は抽象的な製品
title 　 field はページのタイトル
author はページの作者
add method で item を追加する
output method でタイトルを元にファイル名を決定し makeHTML method で内容をファイルに書き込む
makeHTMl を抽象メソッドとして定義

### 抽象的な工場

### Factory

get_factory method はクラス名を指定して具体的な工場のインスタンスを生成するもの。ただし戻り値は抽象的な工場
create_link, tray, page の各メソッドは抽象的な工場で部品や製品を作るときに用いるメソッド
具体的な作成はサブクラスだがシグニチャ(引数の型と数)とメソッドの名前は決められている

## 具体的な工場

### ListFactory

create_link, tray, page を実装

## 具体的な部品

### ListLink

link のサブクラス
make_html を実装

### ListTray

tray のサブクラス
make_html を実装。tray field の中の item を html のタグとして表現する

## 具体的な製品

### ListPage

Page のサブクラス
make_html method を実装

# 登場人物

## AbstractProduct

抽象的な部品や製品の interface

## AbstractFactory

AbstractProduct のインスタンスを作り出すためのインターフェース

## Client

AbstractFactory と AbstractProduct のインターフェースだけを使って仕事を行う役

## CreateProduct

AbstractProduct のインターフェースの実装をする

## ConcreteFactory

AbstractFactory のインターフェースを実装

# 知っておくべきこと

## 具体的な工場を新たに追加するのは簡単

いくら具体的な工場を追加しても抽象的な工場や main 関数に修正を加える必要はない

## 部品を新たに追加するのは困難

AbstarctFactory に新たな部品を追加しようとすると具体的な工場全てに修正加えなければならない

# 参考

[オブジェクト指向でなぜつくるのか 第 3 版 知っておきたい OOP、設計、アジャイル開発の基礎知識](https://www.amazon.co.jp/%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91%E3%81%A7%E3%81%AA%E3%81%9C%E3%81%A4%E3%81%8F%E3%82%8B%E3%81%AE%E3%81%8B-%E7%AC%AC3%E7%89%88-%E7%9F%A5%E3%81%A3%E3%81%A6%E3%81%8A%E3%81%8D%E3%81%9F%E3%81%84OOP%E3%80%81%E8%A8%AD%E8%A8%88%E3%80%81%E3%82%A2%E3%82%B8%E3%83%A3%E3%82%A4%E3%83%AB%E9%96%8B%E7%99%BA%E3%81%AE%E5%9F%BA%E7%A4%8E%E7%9F%A5%E8%AD%98-%E5%B9%B3%E6%BE%A4-%E7%AB%A0/dp/4296000187/ref=sr_1_3_sspa?adgrpid=97228450450&dchild=1&gclid=CjwKCAjwyvaJBhBpEiwA8d38vCSN9YLFZrO7BK1V4aME5GqEeH_TbfjAE2LQvkARmIrUhtbcuR1ITxoCEtYQAvD_BwE&hvadid=439342708792&hvdev=c&hvlocphy=1009301&hvnetw=g&hvqmt=e&hvrand=2989518601321199022&hvtargid=kwd-332957222879&hydadcr=21755_11003627&jp-ad-ap=0&keywords=%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E6%8C%87%E5%90%91+%E6%9C%AC&qid=1631459100&sr=8-3-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEzMEdWQUhTM1UxQjFUJmVuY3J5cHRlZElkPUEwNjUyMTA2MUlLUlk5WFIxREhESiZlbmNyeXB0ZWRBZElkPUEzSjA5MDNOTUpYTk9aJndpZGdldE5hbWU9c3BfYXRmJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==)
