---
layout: post
title:  NUCLEO-F302R8 LEDチカチカ コード 
date:   2023-09-15 22:00:00 +0900
categories: NUCLEO-F302R8
tags: aws linux flutter
---

コードの説明を残します。

### flutter install


 GPIOB_MODER13
  01b: general output

/*  LSI : Low Speed Internal    */
/*  LSE : Low Speed External    */
/*  HSI : High Speed Internal   */
/*  HSE : High Speed External   */

## SysTick

<https://developer.arm.com/documentation/ddi0403/ee/?lang=en>

## toolchain

<https://github.com/vpetrigo/arm-cmake-toolchains>

## 開発環境

windowsです。

ツール類もwindows用を使います。

## ツール

### cmakeのダウンロード

<https://cmake.org/download/>

* cmake-x.xx.x-windows-x86_64.zip

上記を適当なフォルダに展開します。

bin/cmake.exeを環境変数のパスに追加します。

### ninjaのダウンロード

<https://github.com/ninja-build/ninja/releases>

* ninja-win.zip

展開すると、ninja.exeがあります。

これを適当なフォルダにおいて環境変数のパスに追加します。

### arm gccのダウンロード

<https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads>

* arm-gnu-toolchain-xx.x.rel1-mingw-w64-i686-arm-none-eabi.zip

zipを適当な場所に展開します。

binフォルダにコンパイラなどが入ってます。

* コンパイラ: arm-none-eabi-gcc.exe
* リンカ: arm-none-eabi-ld.exe

パスは後述のbuild用のスクリプトで一時設定して使用します。

### STM32CubeProgrammer

https://www.st.com/ja/development-tools/stm32cubeprog.html

STLink ファームのアップデートも必要でした。

<https://www.st.com/en/development-tools/stsw-link007.html>

解凍したフォルダのWindowsにあるexeを実行するだけです。

## テストプログラム

### 構成

``` none
test
 - src
  - main.c: テスト用のコード
 - build.ps1: ビルド用のスクリプト
 - CMakeLists.txt
 - toolchain.cmake
```

### ビルドスクリプトの作成

* build.ps1: ビルド用のバッチファイル
* toolchain-STM32F302R8.cmake: STM32F302R8向けCMakeの設定
  * 参考: <https://github.com/vpetrigo/arm-cmake-toolchains/tree/master>
* CMakeLists.txt: CMake本体

### リンカスクリプトの作成

#### マイコンメモリマップ

公式のリファレンスマニュアル

<https://www.st.com/resource/en/reference_manual/rm0365-stm32f302xbcde-and-stm32f302x68-advanced-armbased-32bit-mcus-stmicroelectronics.pdf>

<https://sourceware.org/binutils/docs/ld/index.html#SEC_Contents>

#### Linker Options

--gc-sections: 未使用のセクションを削除

--specs=nosys.specs: ホストPC用のライブラリをリンクしないようにする。
<https://stackoverflow.com/questions/19419782/exit-c-text0x18-undefined-reference-to-exit-when-using-arm-none-eabi-gcc>

#### ENTRY

リセットして最初に実行する関数を設定します。

Vectorテーブルにも設定するReset_Handlerを設定します。

ただし、Startupで参考にした資料を見ると、Cortex Mシリーズは自動でVectorテーブルのリセットハンドラを最初に実行します。

ENTRYが必要なのは、Reset_Handlerがコンパイラの最適化で削除されないように、ENTRYに設定して使用していることを明示するためです。

#### MEMORY

a

#### startup

以下のサイトを参考に作成しました。
<https://medium.com/@csrohit/stm32-startup-script-in-c-b01e47c55179>

<https://sourceware.org/binutils/docs/as/index.html#SEC_Contents>
