---
layout: post
title:  Flutter WebをAmazon Linux2でホスト
date:   2023-5-27 14:00:00 +0900
categories: aws flutter 
---

いろいろなやり方があると思います。

ここではローカルで作成したFlutter WebアプリをApacheを起動したEC2インスタンスにファイル転送します。

## EC2

以前作成したような構成となるCloudFormationのテンプレートを用意しています。※ RDSなし

自身で作成していただいてもOKです。

[EC2の構成]({% link _posts/2023-05-21-aws-wordpress-infrastructure.markdown %})

### テンプレート

[EC2テンプレート](https://github.com/ohmusso/ohmusso.github.io/tree/main/assets/src/aws/cloud_formation/flutterweb)

## Flutter

FlutterのWebアプリは適当に用意してください。

ここではデフォルトで作成されるアプリを用意しました。

``` powershell
# デフォルトで作成されるアプリをカレントフォルダに作成
flutter create .
```

### ビルド

``` powershell
flutter build web -o build/html
```

build/webに出力される。

### 圧縮

7zipなどでzip形式で圧縮

## EC2に圧縮したファイルを転送

開発PC(Windows)からEC2(Linux)へのファイル転送方法の詳細は以下の記事に記載しています。

[ファイル転送手順]({% link _posts/2023-05-23-aws-transport-file-to-ec2-from-windows.markdown %})

``` powershell
$key = "C:\*\XXX.ppk" # EC2インスタンスの秘密鍵をpscpで使用できるように変換したファイルのパス
$webapp = "C:\*\html.zip" # ビルドして圧縮したファイルのパス
$ec2IpAddress = "XXX.XXX.XXX.XXX" # EC2インスタンスのipv4アドレス
pscp -i $key $webapp "ec2-user@${ec2IpAddress}:/home/ec2-user/"
```

## EC2にて

EC2インスタンスにログインしてください。

### 解凍

``` bash
unzip html.zip
```

### /var/www/htmlフォルダを上書き

Apacheのデフォルトのドキュメントにコピー

``` bash
cp -rf html /var/www
```

継続的に運用する場合は以下の操作も実施するほうが良いと思います。

#### 解凍前に前回回答したフォルダが残っていたら削除

``` bash
rm -rf html
```

#### htmlフォルダを上書きする前にhtmlフォルダを削除

``` bash
rm -rf /var/www/html
```

## アクセス

ELBのDNSにアクセスしてFlutterのカウンタアプリが表示されると完了です。

![カウンタアプリ](/assets/images/image-2023-05-27-flutter-couter-app.png)

## 以上

手動でデプロイするのはやっぱりめんどくさいです。

JenkinsやAWSのコードパイプラインでの自動デプロイにもトライしてみます。
