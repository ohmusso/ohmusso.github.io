---
layout: post
title:  NUCLEO-F302R8 BLDC センサーレス6step
date:   2024-01-05 16:00:00 +0900
categories: NUCLEO-F302R8
tags: stm32 arm bldc
---

以下の書籍の開発キットを使用して、3相BLDCモータを回転させるプログラムを作ります。

<https://shop.cqpub.co.jp/hanbai/books/48/48011.htm>

前回は6step制御を行いました。

[前回: 6step制御]({% link _posts/2023-12-25-NUCLEO-F302R8-motor-sensorless-6step.markdown %})

今回はBEMFから回転速度を算出して、指令値の回転数に近づくように速度制御を行うプログラムを追加してモータを回します。

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
  * 回路図
* モータドライバーのデータシート
  * l6230: <https://www.st.com/resource/en/datasheet/l6230.pdf>
  * モータドライバー入出力のブロック図
* センサーレス制御のアプリケーションノート
  * AP: <https://www.st.com/resource/en/application_note/an1946-sensorless-bldc-motor-control-and-bemf-sampling-methods-with--st7mc-stmicroelectronics.pdf>
  * 6step制御、BEMFの検出方法

## SW構成

![SW構成](/assets/images/image-2023-12-25-motor-sensorless-6step-swarch.png)

## 速度制御

モータ制御の概要を表したフローチャート、速度制御について記載した状態遷移図を以下に記載します。

![速度制御](/assets/images/image-2024-01-05-bldc-sensorless-6step-speed-controll.png)

* 6step制御は前回の記事を見てください。
* 速度制御はBEMFの閾値とPWMのDutyを調整します。
* BEMFの閾値
  * BEMF閾値はBEMFのAD変換割込みからモータタスクへ通知するかを判断するための閾値として使用します。モータタスクは通知を受けてステップを切り替えます。
  * 回転数が早くなるとBEMFが大きくなります。回転数に合わせてBEMFの閾値を調整します。
  * BEMFの閾値が適切な場合、固定子磁界が回転子を回転方向に引き付けます。その結果、モータの回転が速くなります。
  * BEMFの閾値が不適切な場合、固定子磁界が回転子を回転方向とは逆向きに引き付けます。その結果、モータの回転が遅くなります。
  * [BEMFの検出]({% link _posts/2023-12-13-NUCLEO-F302R8-adc2.markdown %})の動作確認が参考になると思います。
  * BEMFの閾値を大きくしすぎると通知が発生しなくなり、最悪の場合は回転が止まります。
* PWMのDuty
  * PWMのDutyを大きくすると固定子磁界が大きくなります。固定子の磁界が大きくなると回転子を引き付ける力も大きくなります。その結果、モータの回転が速くなります。
  * PWMのDutyを小さくすると固定子磁界が小さくなります。固定子の磁界が小さくなると回転子を引き付ける力も小さくなります。その結果、モータの回転が遅くなります。

## コード

<https://github.com/ohmusso/NUCLEO-F302R8/blob/a5f40ccf42c928380b98bf6383ab22cd16eeac8f/src/app/app.c#L91>

## 動作確認結果

* stepTimeが1stepの時間です。1tick=1μsです。1stepをrpmに変換する式は以下になります。
  * 1相の電気角 = 360° / 極数 / 3 = 360° / 7 / 3 = 17.1°
  * 1stepの電気角 = 1相の電気角 / 2 = 17.1° / 2 = 8.6°
  * 角速度 = 1stepの電気角 / 1stepの時間 = 8.6° / 1stepの時間
  * rpm = 角速度 / 1stepの時間 \* 360° \* 60s
* 注意: stepTimeはプログラムの制御時間であり、stepTimeから算出した回転数が実際の回転数と一致しているかは検証していません。

指令値: 1step = 1000[us], 1428[rpm]
![SW構成](/assets/images/image-2024-01-05-bldc-sensorless-6step-speed01.png)

指令値: 1step = 800[us], 1785[rpm]
![SW構成](/assets/images/image-2024-01-05-bldc-sensorless-6step-speed02.png)

指令値: 1step = 500[us], 2857[rpm]
![SW構成](/assets/images/image-2024-01-05-bldc-sensorless-6step-speed03.png)

指令値: 1step = 250[us], 5714[rpm]
![SW構成](/assets/images/image-2024-01-05-bldc-sensorless-6step-speed04.png)

<video controls width="100%" preload loop muted="true" src="/assets/movies/movie-2024-01-05-motor.mp4" type="video/mp4" >
 Sorry, your browser doesn't support embedded videos.
</video>

## 次回

モータ制御はいったんここまでにします。書籍は正弦波駆動や空間ベクトル制御の記載もあり、いつか試してみようと思います。

## 以上
