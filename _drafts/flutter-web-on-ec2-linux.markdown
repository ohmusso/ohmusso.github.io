---
layout: post
title:  Flutter WebをAmazon Linux2でホスト
date:   2022-10-31 22:34:22 +0900
categories: AWS
---

## ビルド

``` powershell
flutter build web
```

build/webに出力される。

## 圧縮

7zipなどでzip形式で圧縮

## EC2に圧縮したファイルを転送

``` powershell
pscp -i C:\work\aws_ssh\aws-test.ppk C:\src\flutter_project\web\build\web.zip ec2-user@XXX.XXX.XXX.XXX:/home/ec2-user/
```

## EC2にて

### 解凍

``` bash
unzip web.zip -d html
```

### htmlフォルダに移動

``` bash
cp -rf html/web var/www/html
```
