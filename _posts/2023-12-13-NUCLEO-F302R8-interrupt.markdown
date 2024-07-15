---
layout: post
title:  NUCLEO-F302R8 FreeRTOS xTaskNotifyFromISR 
date:   2023-12-17 20:00:00 +0900
categories: iot
tags: stm32 arm bldc
---

xTaskNotifyFromISR の使用についてまとめた記事です。

## 経緯

AD変換の割込みでAD変換結果が閾値以上であればxTaskNotifyFromISRでタスクに通知するというプログラムを作っていました。

しかし、Arm Cortex-Mの割込み優先度設定が不適切でFreeRTOSのASSERTでプログラムが止まってしまったので、その対処で勉強したことをまとめました。

FreeRTOSの公式ドキュメントにも割込みについてのCoretx-Mの割込み優先度についての記載がありますので、こちらも参考にしてください。

<https://www.freertos.org/RTOS-Cortex-M3-M4.html>

## リファレンス

* マイコンのRM(リファレンスマニュアル)
  * RM: <https://www.st.com/resource/en/reference_manual/rm0365-stm32f302xbcde-and-stm32f302x68-advanced-armbased-32bit-mcus-stmicroelectronics.pdf>
  * マイコンの基本的な資料
* マイコンのDS(データシート)
  * DS: <https://www.st.com/resource/en/datasheet/stm32f302r6.pdf>
  * マイコンのPin配置やポートの機能一覧、HW特性
* マイコンのPM(プログラミングマニュアル)
  * PM: <https://www.st.com/resource/ja/programming_manual/pm0214-stm32-cortexm4-mcus-and-mpus-programming-manual-stmicroelectronics.pdf>
  * CPU、アセンブラ
* モータドライバーボードの製品シート
  * IHM: <https://www.st.com/resource/en/data_brief/x-nucleo-ihm07m1.pdf>
  * マイコンとの接続が分かる回路図
* センサーレス制御のアプリケーションノート
  * AP: <https://www.st.com/resource/en/application_note/an1946-sensorless-bldc-motor-control-and-bemf-sampling-methods-with--st7mc-stmicroelectronics.pdf>
  * 6step制御、BEMFの検出方法について記載されています。

## Arm Cortex 割込み優先度

Arm Cortexは各割り込みに対して8bit値の優先度を設定できるしようになっています。

ただし、実際の優先度のビット幅はマイコンの実装依存になります。

F302R8は8bitの内、上位4bitを使用できます。

![割込み優先度](/assets/images/image-2023-12-17-interrupt-priority.png)

> PM: 4.3.7 割込み優先度レジスタ（NVIC_IPRx）

## Arm Cortex 割込み優先度のグループ化

優先度値を上位、下位のビットに分けて、優先度をグループ化することができます。

* グループ優先度
  * 実行中の割込みハンドラよりグループ優先度が高い割込みが発生した場合、実行中の割込みを中断してグループ優先度が高い割込みハンドラを実行します。
  * 優先度が同じ場合は割込み番号が低い割込みが優先されます。
* サブ優先度
  * グループ優先度が同じ割り込みが複数発生した場合、サブ優先度が高い割込みから実行します。
  * 優先度が同じ場合は割込み番号が低い割込みが優先されます。

> PM: 4.4.5 アプリケーション割込みおよびリセット制御レジスタ（AIRCR）  
> PRIGROUP

## xTaskNotifyFromISR

公式のドキュメントを参照してください。

<https://www.freertos.org/xTaskNotifyFromISR.html>

## コード

今回の対応をコミットしたリビジョンです。

* グループ優先度
  * なしに設定しました。つまり、全ての優先度ビットをサブ優先度として使用します。
* サブ優先度
  * 全て5に設定しました。FreeRTOSのconfigLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITYで設定した優先度値以下でないと、vPortValidateInterruptPriority()のASSERTで止まります。

<https://github.com/ohmusso/NUCLEO-F302R8/commit/596b6163ceaf88d01b0c4a1e372e3d26862b7bec>
