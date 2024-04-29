---
layout: post
title:  Flutterでバイナリベクトルタイルを描画
date:   2024-04-21 13:00:00 +0900
categories: webland
tags: flutter geomap
mermaid: false
---

FlutterのCustomPaintでバイナリベクトルタイルを描画します。

## バイナリベクトルタイルを使ってみたいと思った経緯

[不動産取引情報API](https://www.reinfolib.mlit.go.jp/)を見ているとバイナリベクトルタイルを返すAPIがありました。取引情報を地図上に表示することができます。GeoJSONという形式もあるようですが、バイナリベクトルタイルの方がサイズが小さくなりそうかなと思い、バイナリベクトルタイルにしました。ただし、日本地図のバイナリベクトルタイルの提供が実験中となっています。詳しくはリファレンスの国土地理院のサイトを確認してください。

次に、バイナリベクトルタイルを扱うパッケージがあるか探しました。地図を扱うパッケージで[flutter_map](https://pub.dev/packages/flutter_map)が有名ですが、バイナリベクトルタイルをサポートしていません。flutter_mapにバイナリベクトルタイルを扱うための[vector_map_tiles](https://pub.dev/packages/vector_map_tiles)というプラグインもありますが、あまり更新されてないように見えました。

ということで、CustomPaintを使って自分で描画してみることにしました。

## リファレンス

* バイナリベクトルタイル
  * [バイナリベクトルタイルの仕様](https://github.com/mapbox/vector-tile-spec/blob/master/2.1/README.md)
  * [国土地理院](https://github.com/GoogleCloudPlatform/functions-framework-nodejs?tab=readme-ov-file#functions-framework-for-nodejs)
* Webの参考にさせていただいた記事
  * [ベクトルタイルについて](https://qiita.com/aoinakanishi/items/35bef7de85fdee46bef9)
  * [GeoJSONをCustomPaintで描画](https://qiita.com/ciscorn/items/a9f0d420c5d0ccb728ba)

## protocol buffers

バイナリベクトルタイルはプロトコルバッファで定義されます。まずはDartでプロトコルバッファを扱えるようにします。

### Dartのチュートリアル

以下の公式チュートリアルに従って、チュートリアルのデータモデル addressbook.proto の書き込み/読み出しができることを確認しました。

<https://protobuf.dev/getting-started/darttutorial>

困ったところをメモしました。

* addressbook.protoをDart向けにコンパイルするときに"protoc_plugin"というDartのパッケージが必要。以下のコマンドでインストールできる。
  * dart pub global activate protoc_plugin
* addressbook.protoをコンパイルするとdartのファイルが出力されるが、コンパイルエラーが出力されている。以下の対応が必要。
  * [protobuf](https://pub.dev/packages/protobuf)のパッケージを追加する。プロトコルバッファを扱うためのパッケージ。*.protoをコンパイルするパッケージだけじゃない。
  * addressbook.protoをコンパイルするときにgoogle/protobuf/timestamp.protoもコンパイル時のコマンドに含める。[※参考記事](https://zenn.dev/konbu33/articles/fc38b26df150b1)

### ベクトルタイルのprotoをコンパイル

[ベクトルタイルのproto](https://github.com/mapbox/vector-tile-spec/blob/master/2.1/vector_tile.proto)も同じ要領で扱うことができます。

コンパイルしてテストしたコードが以下になります。

<https://github.com/ohmusso/map_app/tree/main/vecmap>

一つ注意点があります。以下のzigzagデコード関数で使用するxor演算がFlutter Webのjavascriptに変換されたときに正しく動作しませんでした※1。stackoverflowにあった対策コードで実装しています。Flutterでビット演算をするときは気をつける必要があります。

<https://github.com/ohmusso/map_app/blob/76a4cba92d4cc58fc3e18e9b5269fa6760b384d0/vecmap/lib/vecmap.dart#L46>

※1 戻り値をint型で定義したのに戻り値がUInt32で扱われてしまう。

## バイナリベクトルタイルの描画

まずは一つのタイルを描画してみます。

ソースコードは以下になります。

<https://github.com/ohmusso/map_app/tree/main/app_map>

assets/11_1796_811.pbfに描画するタイルを置いています。[京都市を表示するタイル](https://maps.gsi.go.jp/#11/34.956870/135.788784/&base=std&ls=std&disp=1&vs=c1g1j0h0k0l0u0t1z0r0s0m0f1)です。

### タイルの読み込み

以下のgetTileFromPbfでassetsフォルダに置いたタイルを読み込みます。将来的に<https://cyberjapandata.gsi.go.jp/xyz/experimental_bvmap/{z}/{x}/{y}.pbf>にアクセスして必要なタイルを読み込みます。

<https://github.com/ohmusso/map_app/blob/76a4cba92d4cc58fc3e18e9b5269fa6760b384d0/app_map/lib/vecmap_widget.dart#L7>

### CustomPaintでタイルを描画

以下のコードでタイルを描画しています。

<https://github.com/ohmusso/map_app/blob/76a4cba92d4cc58fc3e18e9b5269fa6760b384d0/app_map/lib/vecmap_widget.dart#L95>

いろいろと未対応です。

* 一部のレイヤでエラーが出力されたのでコメントアウトしています。
* 線の色は適当です。地図のスタイルを規定したjsonがあるので、これを読み込んで描画する必要があります。
  * <https://github.com/gsi-cyberjapan/gsimaps-vector-experiment?tab=readme-ov-file#stylejson>

## 動作確認結果

![確認結果](/assets/images/image-2024-04-29-binary-vector-tile-result.png)

## 以上
