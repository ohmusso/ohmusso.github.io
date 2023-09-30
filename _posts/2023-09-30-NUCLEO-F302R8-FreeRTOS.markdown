---
layout: post
title:  NUCLEO-F302R8 FreeRTOS 導入
date:   2023-09-30 11:00:00 +0900
categories: NUCLEO-F302R8
tags: stm32 arm
---

前回はUARTを実装しました。今回はFreeRTOSを組み込みます。

[前回: UART]({% link _posts/2023-09-26-NUCLEO-F302R8-uart.markdown %})

## ソースコード

<https://github.com/ohmusso/NUCLEO-F302R8/tree/freertos>

## リファレンス

FreeRTOSのドキュメントを参照します。以下がトップページです。

<https://www.freertos.org/index.html>

## SW構成

![SW構成](/assets/images/image-2023-09-30-freertos-structure.png)

## FreeRTOS

### ディレクトリ

以下にプロジェクトのディレクトリを図示します。

赤字はFreeRTOSのgitから取ってきます。

青字はFreeRTOSの設定ファイルです。似ている開発ボードのdemoプロジェクトから取ってきました。公式のページは<https://www.freertos.org/a00110.html>です。

緑字はCMakeのルートファイルで、FreeRTOSのビルド設定を追加します。

![ディレクトリ](/assets/images/image-2023-09-30-freertos-directory.png)

[FreeRTOS git](https://github.com/FreeRTOS/FreeRTOS-Kernel)

[FreeRTOSConfig.h](https://github.com/FreeRTOS/FreeRTOS/blob/main/FreeRTOS/Demo/CORTEX_M4F_STM32F407ZG-SK/FreeRTOSConfig.h)

#### FreeRTOSConfig.h

このプロジェクトで変更が必要になった設定を説明します。

* #define configUSE_IDLE_HOOK
  * 実行するタスクがないときに呼ばれる関数(vApplicationIdleHook)を使用する場合は"1"を設定します。使用する場合は関数を定義しないとビルドエラーになります。使用しないので"0"を設定します。
* #define configUSE_TICK_HOOK
  * 一定周期で呼ばれる関数(vApplicationTickHook)を使用する場合は"1"を設定します。今までmain.cの無限ループで書いていた処理をこちらに移して、同じように動くことを確認します。使用するので"1"を設定します。
* #define configCPU_CLOCK_HZ
  * CPUのクロック周波数を設定します。driver/clockに初期値付き変数で定義しています。開発ボードに標準で付いている内部クロックの周波数8MHzを設定しています。
* #define configTICK_RATE_HZ
  * SysTickの周波数です。1000[Hz]を設定しています。つまり、1tickは1/1000 = 1msです。vApplicationTickHookが1ms周期で実行されます。
  * SysTickはユーザコードのdrive/clockで設定しますが、FreeRTOS(xPortStartScheduler->vPortSetupTimerInterrupt)がSysTickのレジスタを上書きします。
* #define configTOTAL_HEAP_SIZE
  * HEAPのサイズです。大きいとRAMに入らずリンクエラーとなります。1KByteにしています。
* #define configCHECK_FOR_STACK_OVERFLOW
  * スタックオーバーフローが発生したときに呼ばれる関数(vApplicationStackOverflowHook)を使用する場合は"1"を設定します。使用する場合は関数を定義しないとビルドエラーになります。使用しないので"0"を設定します。
* #define configUSE_MALLOC_FAILED_HOOK
  * MALLOC(動的メモリ確保)に失敗した時に呼ばれる関数(vApplicationMallocFailedHook)を使用する場合は"1"を設定します。使用する場合は関数を定義しないとビルドエラーになります。使用しないので"0"を設定します。

#### ルートCMakeLists.txtのFreeRTOSビルド設定

下記がFreeRTOSのビルド設定を記述した箇所です。

``` CMake
# FreeRTOS
set(FREERTOS_CONFIG_FILE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" CACHE STRING "Absolute path to the directory with FreeRTOSConfig.h")
set(FREERTOS_PORT "GCC_ARM_CM4F" CACHE STRING "FreeRTOS port name")
set(FREERTOS_HEAP "1" CACHE STRING "FreeRTOS heap model number. 1 .. 5. Or absolute path to custom heap source file")
include_directories("${FREERTOS_CONFIG_FILE_DIRECTORY}")
add_subdirectory(lib/freertos)
・
・
・
target_link_libraries(sample freertos_kernel driver app)
```

* FREERTOS_CONFIG_FILE_DIRECTORY
  * FreeRTOSConfig.hを置いたディレクトリを設定します。
* FREERTOS_PORT
  * コンパイラ、CPU毎に実装が異なる部分を補うコードです。選択できる値は<https://github.com/ohmusso/NUCLEO-F302R8/blob/6b5e2d6c1c805ce6f74c22892f76115351d17169/lib/freertos/CMakeLists.txt#L34>を参照してください。
* FREERTOS_HEAP
  * 5種類のHEAPが選べます。一番単純というHEAP_1を設定しています。<https://www.freertos.org/a00111.html>
* include_directories
  * FreeRTOSConfig.hへのインクルードパスを追加します。
* add_subdirectory(lib/freertos)
  * FreeRTOSのCMakeLists.txtを実行します。freertos_kernelという名前のライブラリが作成されます。
* target_link_libraries(sample freertos_kernel driver app)
  * ビルドの一番最後で、各ライブラリをリンクしています。ここにfreertos_kernelを追加します。

## アプリ

LEDチカチカとUartの送信を行うアプリです。これまでの記事と同じです。実行している関数がvApplicationTickHookになっただけです。

<https://github.com/ohmusso/NUCLEO-F302R8/blob/freertos/src/app/app.c>

## 動作確認

前回と同じように、PCで送信されたデータを受信できました。

![uartでの通信](/assets/images/image-2023-09-26-uart-test.gif)

## 以上

次はvApplicationTickHookではなく、タスクを作ってアプリを動かします。
