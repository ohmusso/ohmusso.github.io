---
layout: post
title:  NUCLEO-F302R8 FreeRTOS タスク
date:   2023-10-02 21:00:00 +0900
categories: iot
tags: stm32 arm
---

前回はFreeRTOSの導入を行いました。今回はFreeRTOSでタスクを作ってLEDチカチカとUART送信を行います。

[前回: FreeRTOS導入]({% link _posts/2023-09-30-NUCLEO-F302R8-FreeRTOS.markdown %})

## リファレンス

FreeRTOSのドキュメントを参照します。以下がトップページです。

<https://www.freertos.org/index.html>

## SW構成

LEDタスクとUartタスクを作成します。

vApplicationTickHookとIDLEタスクはFteeRTOSが自動で作成します。※設定で変更可能

![SW構成](/assets/images/image-2023-10-02-freertos-task-create.png)

## xTaskCreate タスク作成

タスクを作成できます。

[APIリファレンス](https://www.freertos.org/a00125.html)

## スタック

スタックの設定によりタスクが動かせなかったので、はまったポイントを記載します。

そもそもタスクのスタックサイズが何かわからない場合は以下の記事が分かりやすいです。

<https://uquest.tktk.co.jp/embedded/learning/lecture07-2.html>

* usStackDepth
  * xTaskCreateの引数です。タスクのスタックサイズを設定します。設定したサイズがそのままスタックサイズにならず、設定したサイズ × 4byteがタスクサイズになります。
    使用しているマイコンが32ビットのメモリなので4byteのアライメントでサイズが確保されます。
* ヒープ
  * スタックはヒープのメモリを使用します。ヒープのメモリが足りない場合はタスクが作成できず、xTaskCreateはerrCOULD_NOT_ALLOCATE_REQUIRED_MEMORYを返します。
* IDLEタスク
  * vTaskStartSchedulerを実行すると、ユーザが作成したタスク以外にIDLEタスクが作成されます。IDLEタスクのスタックサイズはconfigMINIMAL_STACK_SIZE × 4byteになります。
  * IDLEタスクの作成に失敗すると、他のタスクも開始されません。

## コード

### アプリ タスク定義

LEDチカチカとUart送信を行うタスクを定義している箇所です。これまでの記事と同じです。実行している関数がタスクになっただけです。

* taskAppLedBlink: LEDチカチカ
* taskAppUartTx: Uart送信

<https://github.com/ohmusso/NUCLEO-F302R8/commit/ca7a36cd0c1596cd436e84ebdcf169f59a5cb8cc#diff-a372127da1d5f8d04f031c2e45adb12ed6c958c642aca859f8faffabe70d561a>

### タスク作成

タスク作成を行う箇所です。

<https://github.com/ohmusso/NUCLEO-F302R8/commit/ca7a36cd0c1596cd436e84ebdcf169f59a5cb8cc#diff-e0cf5b28d9b6b600f0af2bc78e8fd30ec675fd731a5da86f0c4283ffc0e40176>

## 動作確認

前回と同じように、PCで送信されたデータを受信できました。

![uartでの通信](/assets/images/image-2023-09-26-uart-test.gif)

## 以上

次は異なるタスクに異なる優先度を与えて、タスクの切り替えができるかを試してみようと思います。
