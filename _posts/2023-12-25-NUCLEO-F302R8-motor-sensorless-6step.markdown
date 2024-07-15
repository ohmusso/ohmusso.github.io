---
layout: post
title:  NUCLEO-F302R8 モータ センサーレス6step
date:   2023-12-25 20:00:00 +0900
categories: iot
tags: stm32 arm bldc
---

以下の書籍の開発キットを使用して、3相BLDCモータを回転させるプログラムを作ります。

<https://shop.cqpub.co.jp/hanbai/books/48/48011.htm>

前回は6step制御を行いました。

[前回: 6step制御]({% link _posts/2023-11-25-NUCLEO-F302R8-mortor_02.markdown %})

今回はBEMFを検出して6step制御を行い、モータを回します。

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

## SW構成

![SW構成](/assets/images/image-2023-12-25-motor-sensorless-6step-swarch.png)

## 6step制御

1. PWM出力設定を行います。INxxとENxxを設定します。
  ![各stepのPWM出力設定](/assets/images/image-2023-11-25-motor-6step.png)
1. High-Zにした相でBEMFのAD変換を開始します。AD変換の割込みでAD値が閾値を超えたときにモータタスクに通知します。
1. AD変換の割込みから通知を受けるまで待機します。
1. 次のstepに移り、1.～3.を行います。

## コード

<https://github.com/ohmusso/NUCLEO-F302R8/blob/mortor/src/app/app.c#L56C40-L56C40>

## 動作確認

* stepTimeが1stepの時間です。1tick=1μsです。1stepをrpmに変換する式は以下になります。
  * 1相の電気角 = 360° / 極数 / 3 = 360° / 7 / 3 = 17.1°
  * 1stepの電気角 = 1相の電気角 / 2 = 17.1° / 2 = 8.6°
  * 角速度 = 1stepの電気角 / 1stepの時間 = 8.6° / 1stepの時間
  * rpm = 角速度 / 1stepの時間 \* 360° \* 60s

### 回転速度

rpm = 8.6° / 3664us \* 360° \* 60s = 389[rpm]
![389rpm](/assets/images/image-2023-12-25-motor-sensorless-6step-bemf01.png)

rpm = 8.6° / 3027us \* 360° \* 60s = 471[rpm]
![471rpm](/assets/images/image-2023-12-25-motor-sensorless-6step-bemf02.png)

rpm = 8.6° / 864us \* 360° \* 60s = 1653[rpm]
![1653rpm](/assets/images/image-2023-12-25-motor-sensorless-6step-bemf03.png)

rpm = 8.6° / 431us \* 360° \* 60s = 3314[rpm]
![3314rpm](/assets/images/image-2023-12-25-motor-sensorless-6step-bemf04.png)

<video controls width="100%" preload loop muted="true" src="/assets/movies/movie-2023-12-25-motor.mp4" type="video/mp4" >
 Sorry, your browser doesn't support embedded videos.
</video>

## 次回

次は回転速度の指令値を与えて、指令値の回線速度になるように制御します。

## 以上
