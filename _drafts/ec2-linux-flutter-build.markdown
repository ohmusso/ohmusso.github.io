---
layout: post
title:  AWS Linux Flutter Webをビルド
date:   2023-06-15 22:00:00 +0900
categories: aws
tags: aws linux flutter
---

### flutter install

以下の公式に従ってインストールします。
作業フォルダはEC2のデフォルトユーザのディレクトリです。

<https://docs.flutter.dev/get-started/install/linux>

#### SDKをダウンロード

snapコマンドは使えなかったので、wgetコマンドでSDKをダウンロードします。

``` bash
wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.10.2-stable.tar.xz
```

#### SDKを解凍

解凍するとflutterフォルダが作成されます。

``` bash
tar xf flutter_linux_3.10.2-stable.tar.xz
```

#### パスの設定

``` bash
export PATH="$PATH:`pwd`/flutter/bin"
```

##### gitのインストール

パスを設定してflutterを実行したのですが、"gitがない"とエラーが出たので、インストールしました。

``` bash
sudo yum install -y git
```

### flutter doctor

flutter doctorコマンドを実行して、開発環境をチェックします。
[✓]のところは今の環境でも実行可能です。
ビルドするだけなので、Flutterだけ[✓]になっていればOKです。

``` bash
[ec2-user@ip-xxx ~]$ flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.10.2, on Amazon Linux 2023 6.1.27-43.48.amzn2023.x86_64, locale C.UTF-8)
[✗] Android toolchain - develop for Android devices
    ✗ Unable to locate Android SDK.
      Install Android Studio from: https://developer.android.com/studio/index.html
      On first launch it will assist you in installing the Android SDK components.
      (or visit https://flutter.dev/docs/get-started/install/linux#android-setup for detailed instructions).
      If the Android SDK has been installed to a custom location, please use
      `flutter config --android-sdk` to update to that location.

[✗] Chrome - develop for the web (Cannot find Chrome executable at google-chrome)
    ! Cannot find Chrome. Try setting CHROME_EXECUTABLE to a Chrome executable.
[✗] Linux toolchain - develop for Linux desktop
    ✗ clang++ is required for Linux development.
      It is likely available from your distribution (e.g.: apt install clang), or can be downloaded from https://releases.llvm.org/
    ✗ CMake is required for Linux development.
      It is likely available from your distribution (e.g.: apt install cmake), or can be downloaded from https://cmake.org/download/
    ✗ ninja is required for Linux development.
      It is likely available from your distribution (e.g.: apt install ninja-build), or can be downloaded from
      https://github.com/ninja-build/ninja/releases
    ✗ GTK 3.0 development libraries are required for Linux development.
      They are likely available from your distribution (e.g.: apt install libgtk-3-dev)
[!] Android Studio (not installed)
[✓] Connected device (1 available)
[✓] Network resources

! Doctor found issues in 4 categories.
```

``` bash
wget https://github.com/ohmusso/flutter_aws_host/archive/refs/heads/main.zip
```
