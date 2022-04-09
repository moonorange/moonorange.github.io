---
title: DB Random Notes(DB関連)
date: '2021-10-23'
categories: ["Middleware"]
tags: ["DB", "Japanese Article"]
---

## RAID

複数の Disk をまとめて一つの storage とする技術。

基本的にはデータを冗長化すること可用性を高めるためのものである。パフォーマンス向上のために用いられることも多い。

### RAID0(ストライピング)

データを複数の disk に分けて保持する方式。冗長性はなく、一つでも disk が壊れたらデータが失われる。

人によっては RAID と認めない人もおり実際に本番環境で用いるべき構成ではない。

### RAID1

別名ミラーリングと呼ばれる方式で複数の disk に全く同じデータを持つ方式。

複数の disk に同じデータを持つことで可用性はますが、性能向上には寄与せず disk 使用効率も悪い。

### RAID5

パリティ分散と呼ばれる方式をとっており最低 3 本の Disk を必要としデータ共にパリティと呼ばれるエラー訂正符号を格納する。

そのため、Disk の内一本までなら壊れてもデータを復旧でき、データも分散されているため I/O 効率もあげることができる。

デメリットとしてはパリティ計算の必要性があることで書き込み性能が落ちることであるがデータベースは読み込みが多いためそちらの効率の方が重要視される。

### RAID10(1+0)

名前が示す通り RAID1 と 0 を組み合わせた手法。始めに RAID1 のグループ(ミラーリング)を二つ作り、そのグループを使って RAID0 を作る。

RAID1 の高信頼性と RAID0 の高速性を両立させることができる。デメリットとしては最低でも Disk が 4 本いるためコストが高いことである。

## RAID まとめ

実際、運用ではお金に余裕がある場合は高信頼性と高速性を兼ね備えた RAID10 を使うのが好ましい。

そうではない場合は RAID5 を使うことである程度の信頼性と高速性を実現するのが望ましい。

## データベースに格納されるファイルについて

`データファイル`：user がデータベースに格納するデータを保持するファイル

`インデックスファイル`：テーブルに作成されたインデックスを保持するファイル

`システムファイル`：DBMS の内部管理用に使われるデータを保持するファイル

`一時ファイル`：DBMS 内部での一時的なデータを格納するファイル、SQL で使われたサブクエリを展開したデータや GROUP BY などの際のソートデータなどがある

ログファイル：DBMS がテーブルのデータに対する変更を受け付けたときに変更分を一旦溜め込むファイル、その後一括してデータファイルに変更を加える

{{<figure src="./db_files.png" alt="DB物理ファイル" width="75%">}}

## 正規化

データベースのデータの冗長性を排除し、一貫性と効率性を保持するためのデータ形式である正規系にすること。

通常の業務では`第３正規化まで`をするケースが多い。

### 第一正規化

スカラ値の原則。一つのセルに一つの値

スカラ値しか許されないのは仮に一つのセルに複数の値がある場合、主キーが決まれば値が一意に決まるといった\*関数従属性が満たされず定義に反するから。

\*関数従属性とは数学の関数と同様に X の値が決まれば Y の値が一つに定まる関係を言う。

### 第二正規化

主キーの一部の列に対して従属している列がある状態を`部分関数従属`と呼ぶ。

対して主キーを構成する全ての列に従属性があることを`完全関数従属`と呼ぶ。

部分関数従属を解消し、完全関数従属のみのテーブルを作ること。

### 第三正規化

テーブル内部に存在する段階的な従属関係のことを`推移的関数従属`と呼ぶ。

推移的関数従属の解消をするのが第三正規化。

第３正規化を行うことで全ての非キー列はキー列に対してのみ従属している状態にできる。

### 無損失分解

テーブルを分解する前の状態に戻せる、つまりテーブルの情報を完全に保存しつつ分解することを`無損失分解`と呼ぶ。

正規化したテーブルを非正規形の状態に戻すのは結合 SQL を用いて行うことができる。

第２、第３正規化はどちらも無損失分解である。

## インデックス

SQL チューニングの手段として非常にポピュラーなもので目的のデータを速く見つけるための索引である。

特徴として以下の 3 つを持つ。

- アプリケーション透過的(アプリケーションプログラムに影響を与えない)
- データ透過的(データに影響を与えない)
- かつ性能改善の効果が大きい

インデックスの種類には関してはビットマップインデックス、ハッシュインデックスなどいくつかあるあ B-tree が一番多い。

B-tree index は得意領域があると言うより平均して優秀なため、よく使われる。

{{<figure src="./b-tree_chart.png" alt="B-tree レーダーチャート" width="75%">}}

### 均一性

平衡木であるため、キーの値に関わらず探索を同じ計算量で扱える。

バランスを自動的に整える機能もあるが、テーブルへの更新、削除、挿入などでバランスが崩れることもある。

### 持続性

B-tree は木の高さが平均的に 3~4 程度であるため、データ量が膨大に増えたとしても性能劣化が緩やかである。

### 処理汎用性

挿入、更新、削除のコストも同じく O(log n)である。

ビットマップインデックスは検索に関しては B-tree を凌駕することもあるが、更新に多大な時間を要する。

### 非等値性

等号(=)による検索だけでなく不等号、BETWEEN などの検索でも高速化を実現できる。

### 親ソート性

B-Tree index は木構造のデータを生成するため、作成時にデータが sort されるという特徴がある。

そのため DBMS においては専用のメモリ領域を一旦割り当てられるようなコストの高い sort 処理を skip することができる。

### インデックスをどのように作るか

インデックスを作る際にいくつかの指針がある。

#### 大規模なテーブルに対して作成する

フルスキャンでも十分に速く探索できるようなデータ数が少ないテーブルに対してではなく、データが多いテーブルに作る。

目安としてはレコード数が１万件以下の場合はほぼ効果がないと考えて良い。

ハードウェアのスペックによって変わるので実行計画などを見て確認すると良いかも。

#### カーディナリティの高い列に作成する

データがあまり分散していない列に作成しても効果が薄いためカーディナリティ(データの種類)が多く、分散している列に作るべき。

目安としては`全体のレコード数の 5%程度`に絞り込めるカーディナリティがある列が望ましい。

注意点としてはカーディナリティは複合 index を作る場合は複合列の組み合わせで考えること。

#### B-tree index が効かないケース

`インデックス列に演算を行なっている`

```sql
WHERE index_col * 1.1 > 100
```

インデックスは裸で用いらなければならない。

インデックスはあくまで `index_col` に設定されており `index_col \* 1.1` に設定されているわけではないため。

`IS NULL 述語を使っている`

```sql
WHERE index_col IS NULL
```

一般的に B-tree は null をデータとみなさず保持していないため、null をキーとすることは有効でない。

DBMS の仕様によってはできることもあるが、汎用的ではない。

`否定形を用いている`

```sql
WHERE index_col <> 100
```

否定形を用いた場合は探索範囲がキー以外となり狭まらいため、利用できない。

`ORを用いている`

```sql
WHERE index_col = 99 OR index_col = 100
```

OR を用いた場合は index が効かないため IN で書き換える。

```sql
WHERE index_col IN(99,100)
```

`後方一致、中間一致のLIKE述語を用いている`

```sql
WHERE index_col LIKE '%a';

WHERE index_col LIKE '%a%';
```

LIKE 述語を使う際は前方一致(`a%`など)の場合のみ index が効く。

`暗黙の型変換を行なっている`

\*index_col は CHAR 型で定義されているとする

```sql
x SELECT * FROM SomeTable WHERE index_col = 100; 
o SELECT * FROM SomeTable WHERE index_col = '100';
o SELECT * FROM SomeTable WHERE index_col = CAST(100, AS CHAR(2));
```

明示的に型変換をせず WHERE や結合条件に使ってしまうと index が効かなくなる。

### B-tree index 作成によるデメリット

更新性能を低下させる。

インデックスの対象の列が更新される際、インデックスの値も更新する必要がある。

### B-tree index 以外の index について

B-tree 以外にも index は存在する。

ビットマップインデックスとハッシュインデックスについて説明する。

#### ビットマップインデックス

ビットマップインデックスはデータの値からビットデータを作成しそれを index とするものである。

{{<figure src="./bitmap_index.png" alt="ビットマップインデックス" width="75%">}}

ビットデータの演算で検索を行うことができるため、B-tree が苦手だったカーディナリティが低い列に対しても検索性能が良い。

また OR 条件に対しても利用できる点やビットデータはサイズが小さく index のサイズが小さくなるといった利点もある。

デメリットとしてはデータ更新のたびにビットデータを更新しなければならず、更新性能が悪いといった欠点がある。

#### ハッシュインデックス

ハッシュインデックスとはその名の通り、ハッシュ関数を使いデータの値をハッシュ値に変換してインデックスを保存するものである。

このため等値検索においては O(1)と非常に高速な検索が行えるという利点があるがそれ以外の検索では使えないというデメリットがある。

## 統計情報

統計情報とはテーブルのデータや index についてのデータ、メタデータである。

SQL はこの統計情報をもとにアクセスパスを決定する。

### オプティマイザと実行計画

{{<figure src="./table_access_flow.png" alt="テーブルアクセスフロー" width="75%">}}

DBMS が SQL 文を受け取る際のフローは上図にのようになっている。

まず`パーサー`と呼ばれる module が `SQL 文が正常な構文であるかどうかチェック`する。

もし文法的に間違っているような場合はエラーを返す。

パーサーによるチェックの後は`オプティマイザ`と呼ばれる実行計画を決める module に送られる。

オプティマイザは`カタログマネージャー`という module に統計情報の照会をかけ、それを元に最短と思われるパスを選択し SQL を手続きに変換する。

その時得られた手続き(実行計画)に従ってテーブルデータにアクセスする。

### 統計情報の収集タイミングについて

データが大きく更新された際はデータの値の分布などが変わっている可能性があるので出来るだけ早く更新したい。

しかし現実的にはサービスがよく使われているような時間に統計情報を更新するといった重い処理をするのはパフォーマンスの低下を招く可能性があるので夜間に実施されることが多い。

DBMS によってはデフォルトの設定で定期的に実行してくれることもある。

統計情報のロックを行い更新されないように凍結することもできる。これは現時点でのアクセスパスが最適だと分かっているケースなどに利用される。

# 参考

達人に学ぶ DB 設計徹底指南書～初級者で終わりたくないあなたへ
ミック. 達人に学ぶ DB 設計 徹底指南書 (Japanese Edition). Kindle 版.

[B-tree インデックス入門](https://qiita.com/kiyodori/items/f66a545a47dc59dd8839)

[ビットマップインデックスの仕組み](https://qiita.com/gohandesuyo/items/b3a684157b2eefc69a79)