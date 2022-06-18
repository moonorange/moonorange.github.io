---
title: '人間の目と畳み込みニューラルネットワーク(CNN)'
date: '2020-04-24'
categories: ["ML"]
tags: ["CNN", "Japanese Article"]
---

# CNN とは

CNN（Convolutional Neural Networks: 畳み込みニューラルネットワーク）は､主に画像認識に利用されるネットワーク
構造的には､視神経系（視覚を司る神経系）を模した畳み込み層（Convolutional Layer）を、脳の神経系を模した全結合層（Fully-Connected Layer）に接続したもの

神経科学からインスパイアされており、生体の視神経をコンピューターで模したものである。

## 人間の目はどうやって物体を認識しているか

生物の視覚野には沢山のニューロン(神経細胞)がある。外界からの入力に対する反応によって２つに分けられる。１つは単純型細胞、もう一つは複雑型細胞である。
ニューロンは目の網膜の様々な部位と接続しており、ニューロンに影響を与える網膜上の範囲のことを受容野という。

### 網膜

![網膜](https://www.saiseikai.or.jp/medical/disease/retinal_detachment/zu1.jpg)

### 網膜上にある受容野のイメージ

![網膜上にある受容野のイメージ](https://cdn.clipkit.co/tenants/86/item_images/images/000/002/389/medium/5a4f39d6-7acc-42d7-8a43-3e8f30ba2ff9.png?1543999549)

### 単純型細胞の受容野のイメージ

特定の位置で特定の傾きを持った影や光に反応するような受容野を持っている

![単純型細胞の受容野](https://cdn.clipkit.co/tenants/86/item_images/images/000/002/391/medium/89fa3ef5-f724-42fe-8973-392065cbbeb1.png?1543999549)

### 複雑型細胞の受容野のイメージ

位置にかかわらず検出されるべき特徴的な構造に反応する受容野を持っている

![ 複雑型細胞の入力（線分）に対する反応](https://cdn.clipkit.co/tenants/86/item_images/images/000/002/393/medium/9c7b3dc2-5988-4aff-9f04-f50b47b0e10c.png?1543999549)

### 単純型細胞

![単純型細胞](https://cdn.clipkit.co/tenants/86/item_images/images/000/002/395/medium/aa86a0c4-1e5a-4b9d-8f2b-fdb1a476f376.png?1543999550)

※図左が受容野による入力、右の出力層が単純型細胞

入力層の限られた領域にのみ反応し(この場合は 4\*4)位置シフトによって反応するニューロンが変化する
単純型細胞は画像の濃淡パターンを検出する働き

### 複雑型細胞

![複雑型細胞](https://cdn.clipkit.co/tenants/86/item_images/images/000/002/397/medium/fe098863-dfba-41b9-90dd-8f7fff7f77a4.png?1543999550)

※１層が受容野の入力、中間層が単純型細胞、最終層が複雑型細胞

中間層の単純型細胞のどれか一つのニューロンでも反応していれば最終層のニューロンが反応する
位置シフトによらず特徴的な構造に反応することができる
入力パターンの変化に鈍感である
位置感度を低下させる働き

## 参考

[Deep learning で画像認識 ①〜Deep Learning とは？〜](https://lp-tech.net/articles/j4xoo)
