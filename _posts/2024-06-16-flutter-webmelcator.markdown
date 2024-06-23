---
layout: post
title:  Flutter Webメルカトル
date:   2024-06-16 20:00:00 +0900
categories: webland
tags: flutter
mermaid: false
---

緯度、経度から対応するベクトルタイルの座標を取得するコードについて記載します。

## Webメルカトル

地球の表面(楕円の球面)を平面に投影する方法の一つです。

国土地理院から取得するベクトルタイルもこの図法として進めます。

> <https://maps.gsi.go.jp/development/siyou.html> 地図投影法を参照。

## 緯度、経度をWebメルカトル図上の座標に変換

緯度経度からWebメルカトル図上の座標に変換する数式です。詳細は以下のwikiを参照してください。

<https://en.wikipedia.org/wiki/Web_Mercator_projection#Formulas>

```math
x = 1/2π * 2^zoom_level * (π + λ)
y = 1/2π * 2^zoom_level * (π - ln(tan(π/4 + φ/2)))

※lnは自然対数eを底とする対数
```

今回使用するライブラリ内の計算ではyの式が以下の形で使われていました。変形されているだけで計算結果は同じです。

```math
y = 1/2π * 2^zoom_level * (π - ln((1 + sinφ) / (1 - sinφ)) * 1/2 )
```

変形については以下の資料を参照してください。tanの加法定理、2倍角の公式、累乗の対数あたりを使用して変形できます。

> <https://zenodo.org/records/35392>  
> P36 式(2.29)～(2.31)

## ライブラリを使って計算

ライブラリは以下を使用します。

<https://pub.dev/packages/latlng>

### スカイツリーのマップ座標を計算

スカイツリーの緯度経度を以下とします。

緯度: 35.709529
経度: 139.810713

マップのズームレベルが15のとき、緯度経度からマップの座標を取得するコードです。

<https://github.com/ohmusso/map_app/blob/5f3485ef6b9ee00e6520e387d240c024eb6bf5a7/vecmap/test/bl2xyf_test.dart>

### 実行結果

![実行結果](/assets/images/image-2024-06-23-webmelcator-calc-mapindex.png)

国土地理院の地図で計算した座標を確認すると、スカイツリーの位置になっていました。

<https://maps.gsi.go.jp/#15/35.709529/139.810713/&base=std&ls=std&disp=1&vs=c1g1j0h0k0l0u0t1z0r0s0m0f1>

![国土地理院地図で座標を確認した結果](/assets/images/image-2024-06-23-webmelcator-calc-mapresult.png)

以上です。
