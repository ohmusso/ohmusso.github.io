---
layout: post
title:  FullCAN BasicCAN
date:   2023-08-11 23:00:00 +0900
categories: Autosar
tags: R19-11 canif
---

FullCAN BasicCAN について調べたことをまとめた記事です。

## FullCAN BasicCAN とは

Autosar CanIfのHOH(Hardware Object Handle)の設定です。

### CanIfとは

Can Interfaceの略です。

上位のモジュールにCANの送信/受信,コントローラの設定/状態取得を行うインターフェースを提供します。

インターフェースは下位モジュール(CAN Driver)を使用して実装する構成になっています。

![構成図](/assets//images/image-2023-08-12-arc-canif.png)

> <https://www.autosar.org/fileadmin/standards/R22-11/CP/AUTOSAR_SWS_CANInterface.pdf#page=12>
> figure 1.1: AUTOSAR CAN Layer Model

### HOHとは

CAN mailboxのハンドル(扱い方)をHOHとしてCanIfで定義します。

一つのHOHに一つのHardware Object(CAN mailbox)が対応します。

ハンドルする項目を以下に記載します。ここでFullCAN,BasicCANが出てきます。

* 送信/受信の設定
  * 送信の場合はHTH(Hardware Transmit Handle)と定義します。
  * 受信の場合はHRH(Hardware Receive Handle)と定義します。
* FullCAN/BasicCANの設定
  * FullCAN
    * 一つのIDだけで使用します。
  * BasicCAN
    * 一つ以上のIDで共有して使用します。
      * IDXXX,IDYYY,...を格納
      * IDXXX～IDYYYの範囲を格納 ※全てのIDも可能
    * ソフトウェアによるIDのフィルタが必要です。上位レイヤで使用しないIDが格納される可能性があるためです。

図で表すと次のようになります。

![HOH to Mailbox](/assets//images/image-2023-08-12-hohToMailbox.png)

> <https://www.autosar.org/fileadmin/standards/R22-11/CP/AUTOSAR_SWS_CANInterface.pdf#page=33>
> Figure 7.1: Mapping between PDU Ids and HW object handles

FIFOのmailboxを使用する場合は次のようになります。

![HOH to FIFO Mailbox](/assets//images/image-2023-08-12-hohToFifoMailbox.png)

> FIFOのmailboxを使用するかを設定するのはCAN Driverになります。
> <https://www.autosar.org/fileadmin/standards/R22-11/CP/AUTOSAR_SWS_CANDriver.pdf#page=48>
> 7.6.1 Receive Data Consistency

### FullCAN

#### FullCAN 受信

* ソフトウェアによるフィルタが不要でCPUの負荷が小さいです。
* mailboxが他IDのメッセージで上書きされることがないです。
  * 必ず受信割り込みでデータを取り出す必要はないです。
  * 同じIDを連続して受信する場合はあります。しかし、基本的に受信アプリは最新のデータで動作すればよいです。システム設計で検討が必要です。

次の図はmailboxのデータを周期処理で取り出す場合を表しました。

![timme chart](/assets//images/image-2023-08-12-timechart-001.png)

#### FullCAN 送信

* 他IDと共有しないので、送信待ちがありません。

特に送信のFullCANで記載することはないです。

送信ではFullCAN/BasicCANによらず、送信完了を確認してから次に送信するメッセージをmailboxに格納する前提があり、基本的にBasicCANで問題ないと考えます。

### BasicCAN

#### BasicCAN 受信

* ソフトウェアによるフィルタが必要でCPUの負荷が大きいです。
* mailboxが他IDのメッセージで上書きされる(メッセージを取りこぼす)可能性があります。
  * mailboxの受信割り込みでデータを取り出す必要があります。
* mailboxを節約できます。

次の図はmailboxのデータを受信割り込みで取り出す場合を表しました。

![timme chart](/assets//images/image-2023-08-12-timechart-002.png)

#### BasicCAN 送信

* 他IDと共有するので、送信待ちがあります。
* mailboxを節約できます。

次の図は送信処理を表しました。

![timme chart](/assets//images/image-2023-08-12-timechart-003.png)

### 補足 CAN mailbox

※ FullCAN/BasicCANについては以上です。以降はmailboxについて調べたことをまとめました。

CANコントローラが持つ、CANの送信/受信データのバッファ(RAM)です。

mailboxの特徴を列挙します。

* mailboxにはCANメッセージのID, DLC, データが格納されます。
* FIFOのmailboxもあります。
* mailboxは送信/受信のどちらで使うかを設定する必要があります。

#### 受信のmailbox

送信のmailboxはデータをセットして送信のトリガをかけると送信されますが、受信のmailboxは受信ルールの設定が必要になります。

CANコントローラはメッセージを受信すると、各mailboxを参照し、受信ルールに適合した場合にメッセージを格納します。

受信ルールの項目を記載します。

* リモート/データフレーム
* DLC
* ID/IDE(拡張ID)

CANコントローラが受信ルールを参照するときに受信ルールをマスクする機能があります。

特に、IDに対するマスクによりmailboxが一つのIDだけ格納する(FullCAN)か複数のIDを格納する(BasicCAN)ことを設定できます。

IDをマスクする例を記載します。

* マスクのbit定義(マイコンによって異なるかもしれません)
  受信したメッセージのIDに対して
  * IDマスクのbitが0: そのbitは受信ルールのIDと一致したことにする(そのbitは気にしない)
  * IDマスクのbitが1: そのbitを受信ルールで設定したIDと比較する
* ID100だけ受信
  * 受信ルール ID: 0x100
  * IDマスク: 0xFFF
* ID100とID000だけ受信
  * 受信ルール ID: 0x100
  * IDマスク: 0x100
* ID100～ID1FFだけ受信
  * 受信ルール ID: 0x100
  * IDマスク: 0x1FF
* 全て受信
  * 受信ルール ID: 0xFFF(なんでもよい)
  * IDマスク: 0x000

#### 実際のマイコンのmailboxの記載

Renesasマイコンのmailboxを一例として載せておきます。

マイコンによってmailboxの実装が異なるため、CanIfでHOHとして抽象化されていることが分かります。

![mailbox](/assets//images/image-2023-08-12-mailbox.png)

> <https://www.renesas.com/jp/ja/document/mah/rh850f1km-rh850f1kh-users-manual-hardware-r01uh0684ej0130#page=2043>

## 以上
