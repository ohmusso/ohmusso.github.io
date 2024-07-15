---
layout: post
title:  NUCLEO-F302R8 LEDチカチカ
date:   2023-09-9 20:00:00 +0900
categories: iot
tags: stm32 arm
---

## 評価ボード NUCLEO-F302R8

私はモータ制御の勉強で買ったものを使用しました。

<https://shop.cqpub.co.jp/hanbai/books/48/48011.htm>

* CPU: Arm Cortex-M4. Single core
* MCU: STM32F302R8
* 内蔵クロック: 8MHz

### UM(ユーザマニュアル)

<https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf>

以降の説明でUMを引用します。

### USBケーブル mini B

ボードと開発用のPCをUSBケーブルで接続します。

USBで電力も供給されます。

ボードのUSBタイプがminiBです。

私はelecomのU2C-MF10BKを使用しました。

### ユーザLED LD2

ユーザが制御できるLEDがLD2になります。

ボードでの位置はUMを確認してください。

> UM: Figure 3. Top layout

MCUでのポートは以下になります。

pin34 PB13

> UM: 6.4 LEDs  
> User LD2: the green LED is a user LED connected to ARDUINO® signal D13 corresponding  
to STM32 I/O PA5 (pin 21) or PB13 (pin 34) depending on the STM32 target. Refer to  
Table 11 to Table 23 when

## 開発環境

### OS

Windows 10

### Arm GNU Toolchain

Arm向けのコンパイラやリンカなどが入ってます。

<https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads>

zipをダウンロードして、適当なフォルダに解凍しました。

ex) C:\arm-none-eabi

### Buildツール

CMakeとNinjaというツールを使ってビルドを便利で簡単に行います。

#### CMake

* version: 3.24.3
* Download: <https://cmake.org/download/>

zipをダウンロードして、適当なフォルダに解凍しました。

環境変数のPATHにCmakeのbinフォルダのパスを追加してください。

ex) Path: C:\Program Files\CMake\bin

### Ninja

* version: 1.11.1
* Download: <https://github.com/ninja-build/ninja/releases>

zipをダウンロードして、適当なフォルダに解凍しました。

環境変数のPATHにNinjaのexeを置いたパスを追加してください。

ex) Path: C:\Program Files\ninja

## ソースコード

<https://github.com/ohmusso/NUCLEO-F302R8>

ここではフォルダ構成のみ説明します。

コードの説明は以下の記事に記載します。

[コードの説明]({% link _posts/2023-09-12-NUCLEO-F302R8-BlinkLED.markdown %})

```txt
|--CMakeList.txt: CMakeのスクリプト。
|--toolchain-STM32F302R8.cmake: ビルド対象固有の設定を記載したCMakeスクリプト。
|                               CMakeList.txtから読み込む。
|--stm32f3.ld: リンカスクリプト。メモリマップを定義。
|--/src
|   |--startup.s: 起動処理を行う。
|   |--main.c: 必要なドライバの初期化とLEDチカチカの処理を行う。
|   |--/Driver
|      |--/Clock: クロックの設定を行う。
|      |--/Port: LEDが接続されているポートの設定を行う。
|--build.ps1: ビルドを実行するスクリプト。
```

## Build

上記のソースコードをダウンロードして、powershellのbuild.ps1を実行します。

```powershell
# ターミナル上
./build.ps1
```

実行すると、buildフォルダが作成されます。

buildフォルダのsample.binがLEDチカチカプログラムで、これをボードに書き込みます。

## 書き込み

### ボードとPCをUSBケーブルで接続

接続すると

* デバイスドライバがインストールされます。場合によってはボードのファームウェアを更新する必要があります。
  * <https://www.st.com/en/development-tools/stsw-link007.html>
* USBドライブを認識したらOKです。

![USB Drive](/assets/images/image-2023-09-09-UsbDrive.png)

### バイナリをドラッグアンドドロップ

![Drag and Drop](/assets/images/image-2023-09-09-DraAndDrop.png)

書き込み中はLD1が点滅します。

書き込みが完了して、LD2がチカチカすればOKです。

## 以上

次はUARTを実装しました。

[UART]({% link _posts/2023-09-26-NUCLEO-F302R8-uart.markdown %})
