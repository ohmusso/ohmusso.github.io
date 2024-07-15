---
layout: post
title:  WindowsでAWS CodeBuild Local
date:   2023-06-09 23:00:00 +0900
categories: webland
tags: aws codebuild
---

## 目的

AWSのCodeBuildをLocalで実行します。

以下の公式ドキュメントに沿って行います。

<https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/use-codebuild-agent.html>

## 前提条件

* Windows向け
* WSLのUbuntuでDockerが実行できること
  * 以下の記事に環境構築でWSLにUbuntuとインストールしてDokcerを実行する手順をまとめてます。
  * [WSLのUbuntuでDockerを実行]({% link _posts/2023-06-09-wsl-ubuntu.markdown %})

## Dockerイメージのダウンロード

### 注意

LinuxのDockerイメージをダウンロードしますが、サイズが13GBありました。

私のPCはイメージがあるだけでWSL(Vmmem)がメモリ－をたくさん使用して動作が重くなりました。

以下の手順でWSLのメモリ使用量を制限することができます。

#### .wslconfigを作成

ログインしているUserフォルダに.wslconfigを作成します。

テキストで以下のように記述することで、メモリの使用量を制限できます。

``` text
[wsl2]
memory=4GB
```

#### WSLを再起動

再起動することで、設定が反映されます。

まずは以下のコマンドでシャットダウンします。

``` powershell
wsl --shutdown
```

待っていると以下の通知が出てきますので、リスタートするとWSLも起動します。

![リスタート](/assets/images/image-2023-06-10-docker-restart.png)

### ダウンロードするイメージ

以下の二つのDockerイメージをダウンロードします。

公式ドキュメントのとおりにdocker pullコマンドでダウンロードしました。

* Amazon Linux2
  * CodeBuildで実際にビルドを行うコンテナのイメージです。
  * Amazon Linux以外のLinuxイメージも使用できます。
* CodeBuildエージェント
  * ユーザからCodeBuildの指示を受けてAmazon Linuxにビルドの命令を行うコンテナです。

## CodeBuildの構成

以下の構成でCodeBuildを行います。

XXXはログインしているユーザ名です。

``` text
C:\Users\XXX\aws\codebuild\test
\test
| codebuild_build.sh
| buildspec.yml
| output
```

* codebuild_build.sh: CodeBuildエージェントを実行するスクリプトです。後に説明します。
* buildspec.yml: ビルド手順などを記述するファイルです。今回はechoだけ出力してCodeBuildがどのように実行されるかを確認します。後に説明します。
* output: CodeBuildの成果物を出力するフォルダです。このbuildspecで作成されるものはないですが、必ず必要です。

### codebuild_build.shをダウンロード

以下からダウンロードします。

[Github](https://github.com/aws/aws-codebuild-docker-images/blob/master/local_builds/codebuild_build.sh)

### buildspec.yml

以下をファイル内に記述します。

後でいろいろ試してみてください。

``` yml
version: 0.2

phases:
  install:
    commands:
      - echo install
  pre_build:
    commands:
      - echo pre_build
  build:
    commands:
      - echo build
  post_build:
    commands:
      - echo post_build
```

## CodeBuildを実行

### WSLのUbuntuにログイン

以下のコマンドでログインします。

XXXはUbuntuをインストールしたときに入力したユーザ名です。

``` powershell
wsl -d Ubuntu -u XXX
```

### テスト用のフォルダに移動

cdコマンドで先ほど作成したテスト用のフォルダに移動します。

### 実行

以下のコマンドでCodeBuildエージェントを実行します。

"amazonlinux2"のところは用意したLinuxのイメージ名を入力してください。イメージIDじゃないです。

``` bash
./codebuild_build.sh -i public.ecr.aws/codebuild/amazonlinux2-x86_64-standard:3.0 -a ./output
```

buildspecに記述したechoのメッセージが出力されていれば完了です。

実行するたびにCodeBuildエージェントのコンテナが作成されます。

## 以上
