---
layout: post
title:  WindowsからAmazon Linux2へのファイル転送
date:   2023-5-23 21:00:00 +0900
categories: webland
tags: aws scp
---

タイトルの通りです。

開発環境(Windows)でJekyllやFlutterを使って作成したサイトをAmazon Linux2でホストすることを目的に
ファイル転送方法をまとめました。

## PuTTYをダウンロード

以下からMSI (‘Windows Installer’)をダウンロード

[PuTTY ダウンロードサイト](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

### PuTTYとは

リモート接続のクライアントアプリ。

### PuTTYを使用する理由

AWS公式ドキュメントのおすすめです。この記事より公式ドキュメントを見たほうが早いかも。。。

[PuTTY を使用した Windows から Linux インスタンスへの接続](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/putty.html)

* scp
  * PuTTYに付属しており、sshを通してファイルコピーしてくれます。これを使用してEC2インスタンスにファイルを転送します。

* PuTTYgen
  * PuTTYに付属しており、Amazon EC2で自動生成される秘密鍵(.pem)はscpで使用できないため、scpで使用できる(.ppk)に変換します。

## PuTTYをセットアップ

ダウンロードしたインストーラを実行して、インストールする。
全部デフォルトのまま進めた。

## 秘密鍵を変換

### PuTTYgenを起動

インストールできていたら、powershellを開いて以下のコマンドを実行すると、PuTTYgenが起動する。

``` powershell
PuTTYgen
```

### EC2インスタンスに指定したキーペアの秘密鍵をロードする

キーペア作成時にpemファイルをダウンロードしているので、それを選択する。

![PuTTYgenでEC2インスタンスの秘密鍵をロード](/assets/images/image-2023-05-23-puttygen-import-pem.png)

選択すると、"PuTTYgen Notice"というタイトルのダイアログが出てくる。OKを選択。
鍵が読み込まれる。

### 変換された秘密鍵を保存する

秘密鍵保存を選択すると、PuTTYのscpで使用できるppkファイルで保存される。
ワーニングのダイアログが出てくるが、OKを選択。

⇒"出力される秘密鍵のファイルが暗号化されてないから危ない"というワーニング

![aaa](/assets/images/image-2023-05-23-puttygen-save-ppk.png)

## ファイル転送

### EC2インスタンスを実行する

AWS コンソール等から実行しておく。

### 転送するファイルを作成する

test.txtを作成し、"hellow ec2"と入力する。

### scpコマンドを実行

powershellで以下のコマンドを実行すると、ファイルが転送される。

``` powershell
pscp -i C:\work\aws_ssh\aws-test.ppk C:\work\aws_ssh\test.txt ec2-user@XXX.XXX.XXX.XXX:/home/ec2-user/
```

* -i C:\work\aws_ssh\aws-test.ppk
秘密鍵(*.ppk)へのパス。パスは自身の環境に合わせて変えてください。
* C:\work\aws_ssh\test.txt
転送するファイルへのパス
* ec2-user@XXX.XXX.XXX.XXX:/home/ec2-user/
転送先。
  * ec-user はデフォルトで作成されたAmazon Linux2のユーザ名。
  * @XXX.XXX.XXX.XXX はEC2インスタンスのパブリックipv4アドレス。パブリックDNSでもOK。
  * :/home/ec2-user/ は転送先のディレクトリ。

## 以上
