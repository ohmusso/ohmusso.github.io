---
layout: post
title:  NUCLEO-F302R8 FreeRTOS タスク 優先度
date:   2023-10-03 20:00:00 +0900
categories: NUCLEO-F302R8
tags: stm32 arm
---

前回はFreeRTOSでタスクを作ってLEDチカチカとUART送信を行いました。

[前回: FreeRTOS タスク]({% link _posts/2023-10-02-NUCLEO-F302R8-FreeRTOS-task.markdown %})

今回は異なるタスクに異なる優先度を与えて、優先度に応じたタスクの切り替えが行われることを試します。

## リファレンス

FreeRTOSのドキュメントを参照します。以下がトップページです。

<https://www.freertos.org/index.html>

## SW構成

![SW構成](/assets/images/image-2023-10-03-freertos-task-prio-structure.png)

以下のタスクが今回のポイントです。

* Low, Mid Highタスク
  * 優先度が異なるタスクを3つ作成します。Lowの優先度が一番小さいです。
  * 各タスクはUartタスクに送信したい文字を一定時間通知し続けます。タスクの処理を模擬しています。
  * vApplicationHookからSysTickカウントを受け取り、時間のカウントに使用します。
* Uartタスク
  * Low, Mid, Highタスクから文字を受け取り、Uartで送信します。
  * タスクの優先度はHighタスクよりも高くします。

## タスクの実行スケジュール

以下のイメージでタスクが実行されるように、タスクの優先度、タスクの処理時間、タスクのDelayを設定します。

![スケジュール](/assets/images/image-2023-10-03-freertos-task-schadule.png)

|タスク|優先度|処理時間[ms]|Delay[ms]|
|----|----|----|----|
|Uart|4|-|100|
|High|3|200|800|
|Mid|2|500|500|
|Low|1|-|-|

* High, Midタスクは処理時間だけループした後、Delay時間だけタスクをBlocked(待ち)にします。ループ中は優先度の低いタスクは実行されません。優先度の高いタスクがBlockedになると、優先度の低いタスクが実行されます。
* LowタスクはDelayを行わず、常にループさせます。他に実行しているタスクがいない場合に実行されます。
* Uartタスクの処理時間は実際にUartの送信レジスタへの書き込みにかかった時間になります。

Blockedはタスクの状態の一つです。以下のページを見てください。

<https://www.freertos.org/RTOS-task-states.html>

## コード

### タスク定義

<https://github.com/ohmusso/NUCLEO-F302R8/commit/ae6e9d8fdb82dbef23a464a77335ee9e85fa7797#diff-a372127da1d5f8d04f031c2e45adb12ed6c958c642aca859f8faffabe70d561a>

### タスク作成

<https://github.com/ohmusso/NUCLEO-F302R8/commit/ae6e9d8fdb82dbef23a464a77335ee9e85fa7797#diff-e0cf5b28d9b6b600f0af2bc78e8fd30ec675fd731a5da86f0c4283ffc0e40176>

## 動作確認

イメージしたスケジュール通りに文字が送信されていることが確認できます。

![uartでの通信](/assets/images/image-2023-10-03-freertos-task-test.gif)

## タスクの優先度を同じにすると

以下のコードのように、同じ優先度のタスクを作った場合の動きを確認します。

``` c
void taskAppLow1() {
    for (;;) {
        uartSendChar = '1';
    }
}

void taskAppLow2() {
    for (;;) {
        uartSendChar = '2';
    }
}

void taskAppLow3() {
    for (;;) {
        uartSendChar = '3';
    }
}
```

動作確認の結果です。ラウンドロビンで動いています。

![uartでの通信](/assets/images/image-2023-10-04-freertos-samepriotask-test.gif)

## 以上
