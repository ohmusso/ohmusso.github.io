---
layout: post
title:  WSL2でUbuntu
date:   2023-06-10 12:00:00 +0900
categories: webland
tags: docker wsl
---

## 目的

AWSのCodeBuildをLocalで実行したいです。

以下の公式ドキュメントがありますが、Linux向けなのでWindowsだと動かせなかったです。。。

<https://docs.aws.amazon.com/ja_jp/codebuild/latest/userguide/use-codebuild-agent.html>

なので、WindowsでLinuxを動かす方法としてWSLを使うことにしました。

このドキュメントではWSLでUbuntuを動か方法をまとめました。

## 参考

<https://docs.docker.com/desktop/windows/wsl/>

## 前提条件

* Windows向け
* WindowsにDocker Desktopをインストールされていること。
* wsl2がインストールされていること。

## WSL2にUbuntuをインストール

WSLにUbuntuがインストール済みであればスキップしてください。

### Ubuntuのインストール確認

以下のコマンドを実行してUbuntuのバージョン2が表示されていればOKです。

``` powershell
wsl -l -v
```

![Ubuntuのインストール確認](/assets/images/image-2023-06-09-wsl-check-install.png)

### Ubuntuのインストール

Ubuntuが表示されなかったら、以下のコマンドでインストールしてください。

``` powershell
wsl --install -d Ubuntu
```

インストールが完了すると、Ubuntuが別ウインドで起動します。

ユーザ名とパスワードの入力を求められるので、入力してください。

## Ubuntuのバージョンアップ

インストールが完了した後にインストールの確認コマンドを実行すると、Ubuntuは表示されますが、バージョンが1になっていると思います。

バージョン2にアップデートする必要があります。

以下のコマンドを実行してください。

``` powershell
wsl --set-version Ubuntu 2
```

私は30分くらいかかりました。

完了後にもう一度インストールの確認コマンドを実行して、バージョン2になっていればOKです。

## WSL Integration

Docker Desktopの作業になります。

### Docker Desktopを起動

### WSL Integrationを有効化

UbuntuのWSLバージョン2がインストールされていると、以下の画面のようになります。

スライドボタンでUbuntuを有効にして、Apply & restartをクリックすると、WSLのUbuntuでDockerが使えます。

![WSL Integrationを有効化](/assets/images/image-2023-06-09-wsl-integ.png)

## UbuntuでDockerコマンドを確認

powershell(WSLのUbuntu)の操作になります。

### Ubuntuにログイン

以下のコマンドでログインします。

UserNameにはUbuntuをインストールしたときに入力したユーザ名を入力してください。

``` powershell
wsl -d Ubuntu -u UserName
```

### Dockerが使えることを確認

ログイン後に以下のコマンドでDockerが使えることを確認します。

Dockerのヘルプが表示されたらOKです。

WSL2～と表示された場合はUbuntuのバージョンが1になっていると思います。

``` bash
docker
```

## 補足

### ホームディレクトリの確認

以下のコマンドでホームディレクトリ(ログイン直後のディレクトリ)内を確認します。

``` bash
ls
```

いろいろ表示されると思いますが、表示される内容はWindowsの"C:\Users\<ログインユーザ名>"ディレクトリです。ユーザフォルダ以下が共有されます。

シェル上にも"/mnt/c/Users/UserName/"と表示されていると思います。

## 以上

Ubuntuからは以下のコマンドでログアウトできます。

``` bash
exit
```

次はこのUbuntu上でAWS CodeBuildを行います。
