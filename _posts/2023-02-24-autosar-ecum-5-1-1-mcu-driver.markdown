---
layout: post
title:  Autosar EcuM 5.1. SPAL Modules
date:   2023-02-24 23:00:00 +0900
categories: Autosar
tags: R19-11 ecum driver
---

## 5.1.1 MCU Driver

1. BCWの中でMCUドライバを最初に初期化すること。
マイコンの設定を行う。主には各ドライバへのクロック設定を行う。
各ドライバとはポート入出力(GPIO)、ADコンバータ、通信モジュール(CAN、LIN、EtherNet)、不揮発メモリ、HSMなど。

1. MCU_Initに加えて、必要なMCUドライバの初期化を行うこと。
上記でドライバにクロックが供給されて、ドライバが動作できる状態となっている。
ここで、各ドライバの詳細な設定を行う。
また、BSWモジュールのメモリを初期しておく。

1. EcuMは二つのコールアウトを提供すること。
コールアウトで追加のMCUドライバの初期化を実行する。

* EcuM_AL_DriverInitZero
* EcuM_AL_DriverInitOne

2回に分かれているのは、ドライバに依存関係があるため。
1回目にドライバ単体での初期化を行った後、2回目の初期化で他ドライバのデータを参照して残りの初期化を行う。

## 5.1.2 Driver-Dependencies-and-Initialization-Order

1. 設計時に以下の初期化関数の実行順序を決定すること。

* EcuMDriverInitListZero
* EcuMDriverInitListOne
* EcuMDriverRestartList
* EcuMDriverInitListBswM

本章でも、ドライバ間の依存関係を考慮すること。
