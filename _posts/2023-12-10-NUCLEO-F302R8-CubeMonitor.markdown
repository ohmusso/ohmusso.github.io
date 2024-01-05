---
layout: post
title:  NUCLEO-F302R8 STM32CubeMonitor
date:   2023-12-10 20:00:00 +0900
categories: NUCLEO-F302R8
tags: stm32 arm
---

前回、ソフトウェアのトリガでAD変換を行うプログラムを作成しました。

AD変換結果をUARTでPCに送信し、Tera Termで受信したデータをログに出力して、ログをエクセルでグラフにしていました。

今回はSWDを使用して変数をリアルタイムにモニタできるツール STM32CubeMonitor がST公式にあることを知りましたので、試してみた記事になります。

先駆者の方が書かれた記事がありますが、私のプロジェクトではSTM32CubeMXを使っていないことからか、以下のはまりポイントがあったので記事にしました。

* elfファイルを読み込んでも変数一覧が出てこない
* モニタを開始しても"not connected"となり変数がモニタできない

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

### CubeMonitor

以下の記事を参考にさせていただきました。この記事を読んでいただいたらこの章は飛ばしてもらってOKです。

<https://rt-net.jp/mobility/archives/22743>

#### ダウンロード

<https://www.st.com/ja/development-tools/stm32cubemonitor.html>

私はwindowsなので"STM32CubeMon-Win"をダウンロードしました。

#### インストール

ダウンロードしたzipを解凍してsetupSTM32CubeMonitor_*.*.*.exeを実行するとインストーラが起動します。

全部デフォルトで進め手インストールしました。

#### 使い方

公式のチュートリアル動画を見てください。

<https://www.youtube.com/watch?v=PhWAZiVwMkU>

### コンパイルオプション

elfを読み込んでも変数の一覧が出てこない原因はコンパイルオプションでデバッグ情報を出力していないことでした。当たり前だろと思われるかもしれませんが。。。

以下の部分を更新しました。

<https://github.com/ohmusso/NUCLEO-F302R8/commit/c84da0760953d2e55b32913c92035316cb600c63#diff-31b1447a4cd4f2cf2b927156b551b5f84cc82a702987deebe0bad1f52371804f>

オプションのレベルは4段階あるようです。

<https://github.com/ohmusso/NUCLEO-F302R8/commit/ced2cca2fb03ffa6fb71e5d55e8d1ee7a860fa96#diff-31b1447a4cd4f2cf2b927156b551b5f84cc82a702987deebe0bad1f52371804f>

### SWDの設定

モニタを開始しても"not connected"となり変数がモニタできない原因は以下のポート設定が必要でした。こちらもSWDを使用するので当たり前だろと思われるかもしれませんが。。。

* SWDIO
  * PA13: AF0: SWCLK- JTCK
* SWCLK
  * PA14: AF0: SWCLK- JTCK

コードは以下になります。

<https://github.com/ohmusso/NUCLEO-F302R8/commit/ced2cca2fb03ffa6fb71e5d55e8d1ee7a860fa96#diff-87d2ec847b5133c430b4210d9da8404641c0677b557afc9dc6a1ff53fa01f69f>

## モニタ結果

リファレンス電圧(ADC IN18)をモニタした結果です。波形をすぐに確認できるので良いですね！

![モニタ結果](/assets/images/image-2023-12-10-cubemonitor.png)
