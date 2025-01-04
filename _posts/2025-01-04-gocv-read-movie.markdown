---
layout: post
title: gocv 動画の読み込み 書き込み
date:   2025-01-04 01:00:00 +0900
categories: gocv
tags: go
mermaid: false
---

## 前提

* goをインストール済み
* windows11
* visual studio code

## gocvのインストール

以下の公式のインストール手順の「Verifying the installation」まで行います。捕捉を記載します。

<https://gocv.io/getting-started/windows/>

### MinGW-W64

gocvはopencvをビルドして使用するようになってます。opencvをビルドするときにCMakeとMinGWを使用します。

CMakeは特にバージョンは気にする必要はなかったです。

MinGWはバージョンを気を付けないとopencvのビルドに失敗します。私は以下のMinGWをPCにインストールしました。

<https://github.com/niXman/mingw-builds-binaries/releases/download/14.2.0-rt_v12-rev0/x86_64-14.2.0-release-posix-seh-ucrt-rt_v12-rev0.7z>

### Verifying the installation

公式手順のここまで実行して、gocvとopencvのバージョンが表示されたらgocvのインストールは完了です。

## 動画からフレームを読み出す

### gocv.VideoCaptureを作成する

gocv.OpenVideoCaptureでgocv.VideoCaptureを作成します。VideoCaptureを通して動画のフレームを読み出すことができます。

今回作成したコード: <https://github.com/ohmusso/gocv_movie_read/blob/f8dece9b9682d142a22c4223ece8b3c6a87fc1c3/reader/reader.go#L23C2-L23C50>

### 1フレームを読み出す

gocv.VideoCaptureのReadメソッドでフレームを読み出します。戻り値がfalseならフレームが読み出せなかったことを示します。全てのフレームを読み出し、次に読み出すフレームがない場合もfalseになります。

今回作成したコード: <https://github.com/ohmusso/gocv_movie_read/blob/f8dece9b9682d142a22c4223ece8b3c6a87fc1c3/main.go#L108>

読み出したフレームはgocv.Matというクラスに格納されてます。

## gocv.Matについて

OpenCVのMatクラスが元です。Matは行列(Matrix)のことです。詳細は以下のリンクを参照してください。特に以下の用語は重要です。

* 次元: 行列の次元です。画像の場合は2次元になります。
* チャンネル: 行列の要素の次元です。RGB画像の場合は3チャンネルになります。

<http://opencv.jp/cookbook/opencv_mat.html>

動画のフレームは1枚の画像であるため、2次元、3チャンネルのMatとして読み出されます。OpenCVではMatのデータ構造を定数で定義されています。2次元、3チャンネルのMatはCV_8UC3という定数で定義されます。gocvではgocv.MatTypeCV8UC3が定義されています。

以下はMatのデータ構造の例です。注意としてRGB画像の場合はBGRの順番でデータが並びます。

![gocv.Mat](/assets\images\image-2025-01-04-gocv-mat.png)

Matでピクセルのデータにアクセスする方法がいくつかあります。以下のサイトの「画素へのアクセス」が参考になります。

<https://recruit.cct-inc.co.jp/tecblog/img-processor/opencv_cvmat/>

gocvでは以下の二つがありました。atメソッドは画像のようにチャネルが3つの場合にどのように使えばよいか分かり難く、結局ポインタと同じように行列のインデックスを計算しないといけないので、今回は使いませんでした。

* atメソッド GetUCharAt/SetUCharAt
* ポインタ DataPtrUint8
  先頭のピクセルのポインタから各ピクセルにアクセスします。

今回作成したコード: <https://github.com/ohmusso/gocv_movie_read/blob/f8dece9b9682d142a22c4223ece8b3c6a87fc1c3/reader/reader.go#L67>

## 動画にフレームを書き込む

### gocv.VideoWriterFileを作成する

gocv.VideoWriterFileでgocv.VideoWriterを作成します。gocv.VideoWriterを通して動画のフレームを書き込むことができます。

今回作成したコード: <https://github.com/ohmusso/gocv_movie_read/blob/f8dece9b9682d142a22c4223ece8b3c6a87fc1c3/main.go#L17>

### コーデック avc1

VideoWriterFileの引数でコーデックを指定します。今回はavc1を指定していますが、gocvの公式のインストール手順だけでは以下のエラーが発生してavc1で動画を作成できません。※avc1(H.264)はブラウザで一般的に使用されているエンコード方法です。win11の標準のメディアプレイヤーもこのコーデックで再生できました。

``` txt
Failed to load OpenH264 library: openh264-1.8.0-win64.dll
        Please check environment and/or download library: https://github.com/cisco/openh264/releases
```

エラーの通り、コーデックに必要なdllをダウンロードします。エラー出力の通りVer.1.8.0のdllが必要です。avc1のコーデックに関してはライセンスの問題でこのような対応が必要になっているようです。

openh264-1.8.0-win64.dll: <https://github.com/cisco/openh264/releases/download/v1.8.0/openh264-1.8.0-win64.dll.bz2>

ダウンロードした圧縮ファイルにopenh264-1.8.0-win64.dllが入っています。openh264-1.8.0-win64.dllをgo.exeと同じフォルダに置くとgocvがこのdllを使用してくれます。

### 1フレームを書き込む

gocv.VideoWriterのWriteメソッドでフレームを書き込みます。

今回作成したコード: <https://github.com/ohmusso/gocv_movie_read/blob/f8dece9b9682d142a22c4223ece8b3c6a87fc1c3/main.go#L39>

## デモ

VtuberさんのMVの一部をキャプチャさせていただき、R,G,Bに分解した動画ファイルを出力するデモを作成ました。

* MV: <https://www.youtube.com/watch?v=lUDPjyfmJrs&pp=ygUDaWlp>
  ※0:33～1:15あたりをキャプチャ
* キャプチャファイル: <https://github.com/ohmusso/gocv_movie_read/blob/f8dece9b9682d142a22c4223ece8b3c6a87fc1c3>
  test.mp4
* 出力ファイル: <https://github.com/ohmusso/gocv_movie_read/blob/f8dece9b9682d142a22c4223ece8b3c6a87fc1c3>
  * test_blue.mp4
  * test_green.mp4
  * test_red.mp4
