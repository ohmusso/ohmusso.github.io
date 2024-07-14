---
layout: post
title:  Flutter ベクトルタイルのSprite
date:   2024-07-15 01:00:00 +0900
categories: webland
tags: flutter
mermaid: false
---

ベクトルタイルで地図記号などを描画するために使用するSpriteについて記載します。

## Sprite

MapBox公式のSpriteについての仕様です。

<https://docs.mapbox.com/help/glossary/sprite/>

## 国土地理院地図のSprite

<https://github.com/gsi-cyberjapan/gsimaps-vector-experiment/tree/master/sprite>

"std"が標準の地図タイルで使用する地図記号です。これを使用します。

## 実際に使ってみた

### Spriteを地図記号ごとに切り取る

"std.png"はそれぞれの地図記号が"std.json"に定義された位置に描かれています。

![sprite](/assets/images/image-2024-07-14-map-sprite.png)

地図記号ごとに切り取り、pngファイルに保存します。切り取りには[imageライブラリ](https://pub.dev/packages/image)を使用します。

切り取ってpng形式のバイト列を出力するコードのサンプルです。

``` dart
  // import 'package:image/image.dart' as img;
  // string: 地図記号の名前, Uint8List: 切り取った地図記号(png形式)
  Map<String, Uint8List>? genMapPngIconImageBytes() {
    if (_image == null) {
      return null;
    }

    final Map<String, Uint8List> mapKeyImageBytes = Map();
    for (var key in _spriteJson.keys) {
      final jsonValue = _spriteJson[key];
      final x = jsonValue['x'];
      final y = jsonValue['y'];
      final width = jsonValue['width'];
      final height = jsonValue['height'];
      final cropedImaage =
          img.copyCrop(_image, x: x, y: y, width: width, height: height);
      mapKeyImageBytes[key] = img.encodePng(cropedImaage);
    }

    return mapKeyImageBytes;
  }
```

出力した地図記号のバイト列を*.pngで保存し、Flutterアプリのassetsに置いて読み込み、描画します。

以下は明石海峡大橋あたりを描画した結果です。国道を示すアイコンやインターチェンジを示すアイコンが表示できました。国道を示すアイコン(青色のハッチング)や高速道路の番号を示すアイコン(緑色のハッチング)は未完成で、アイコンの中央に国道や高速道路の番号を描画しないといけないです。

![描画結果](/assets/images/image-2024-07-14-map-draw-icon.png)

以上です。
