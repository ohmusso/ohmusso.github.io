---
layout: post
title:  NUCLEO-F302R8 3相ブラシレスDCモータ
date:   2023-10-08 20:00:00 +0900
categories: iot
tags: stm32 arm bldc
---

以下の書籍の開発キットを使用して、3相ブラシレスDCモータを回転させるプログラムを作ります。

<https://shop.cqpub.co.jp/hanbai/books/48/48011.htm>

今回はとりあえずモータを回すプログラムを作成します。

## リファレンス

マイコンのRM(リファレンスマニュアル)、DS(データシート)とPM(プログラミングマニュアル)を参照します。

RM: <https://www.st.com/resource/en/reference_manual/rm0365-stm32f302xbcde-and-stm32f302x68-advanced-armbased-32bit-mcus-stmicroelectronics.pdf>

DS: <https://www.st.com/resource/en/datasheet/stm32f302r6.pdf>

PM: <https://www.st.com/resource/ja/programming_manual/pm0214-stm32-cortexm4-mcus-and-mpus-programming-manual-stmicroelectronics.pdf>

## HW構成

![HW構成](/assets/images/image-2023-10-29-motor-hw-structure.png)

* 書籍のキットに含まれているもの
  * マイコンボード
  * モータドライバ(モータドライバはマイコンボードに装着できるようになっています)
  * モータ(モータとモータドライバの結線も含む)
  * ホールセンサ(ホールとモータドライバの結線も含む)
* 自分で用意しないといけないもの
  * PC
  * USB miniB
  * DC 12V
  * はんだなどの電子工作ツール
    * ホールセンサにはんだ付けが必要
    * テスターやオシロスコープがあると、結線やマイコンの出力を確認可能

### マイコンボード X-NUCLEO-IHM07M1

* CPU: Arm Cortex-M4. Single core
* MCU: STM32F302R8
* 内蔵クロック: 8MHz
* <https://www.st.com/ja/evaluation-tools/nucleo-f302r8.html>

### モータードライバボード X-NUCLEO-IHM07M1

マイコンボードに取り付けることができるモータードライバを搭載した拡張ボードです。

<https://www.st.com/ja/ecosystems/x-nucleo-ihm07m1.html>

モータードライバは以下になります。

<https://www.st.com/en/motor-drivers/l6230.html?rt=um&id=UM1943>

* デッドタイム付き。つまり、マイコンで出力するPWMにデッドタイムが不要。
  * デッドタイムを簡単に解説されているページ <https://nitomath.hatenablog.jp/entry/2020/08/02/002053>

### ポート

* ホールセンサの入力
  * ホールセンサ信号: モータのN極に反応してHighを出力します。
  * 今回のプログラムでは割込み設定だけして、制御では使用していません。
    * H1: PA15: U相ホールセンサ
    * H2: PB3 : V相ホールセンサ
    * H3: PB10: W相ホールセンサ
* モータドライバボードの入力
  * イネーブル信号: ドライバの出力を有効/無効に設定します。無効時はドライバの出力がHigh-Zになります。
    * EN1: PC10: U相イネーブル信号
    * EN2: PC11: V相イネーブル信号
    * EN3: PC12: W相イネーブル信号
  * PWM入力: マイコンのPWM出力先です。
    * IN1: PA8 : U相PWM(TIM1_CH1)
    * IN2: PA9 : V相PWM(TIM1_CH2)
    * IN3: PA10: W相PWM(TIM1_CH3)

## SW構成

![SW構成](/assets/images/image-2023-10-29-motor-sw-structure.png)

FreeRTOSを組み込んでいます。タスクの機能しか使っていませんが。。。

[前回: FreeRTOS タスク]({% link _posts/2023-10-02-NUCLEO-F302R8-FreeRTOS-task.markdown %})

[前回: FreeRTOS タスクの優先度]({% link _posts/2023-10-03-NUCLEO-F302R8-FreeRTOS-task-priority.markdown %})

以下のタスクを作成します。

* Uart Task
  * PCから”+"、"-"の文字を受信して、Motor Taskに通知
* Motor Task
  * モータの回転速度を停止、低、中、最大の4レベルで制御
  * TimerのPWMをレベルに合わせて設定
    * Uart Taskから"+"を受けると、PWMの周波数を上げて、回転速度を上げる
    * Uart Taskから"-"を受けると、PWMの周波数を下げて、回転速度を下げる

## 回転の仕組み

IN1,IN2,IN3を制御することでモータに流れる電流の向きを制御できます。

モータに流れる電流によりモータ内のコイルに磁界が発生します。

発生した磁界によりモータ内の回転子の永久磁石が磁力を受けて、モータが回転します。

![モータの回転](/assets/images/image-2023-10-29-motor-rotation.png)

## 初期化

### 1. ホールセンサ

ホールセンサから出力されるLow、Highの信号をマイコンのGPIOに入力します。

GPIOに以下の設定をします。

* 入力ポートの設定
* ホールセンサ信号の立上り、立下りで割込みを発生

#### 1. Clock

GPIOの割り込みを有効にするためにSYSCFGを使用します。

まずはSYSCFGのクロックを有効にします。

* RCC_APB2ENR: SYSCFGEN 1b
  * SYSCFGのクロックを有効にします。

#### 1. SYSCFG

SYSCFGは文字通りシステムに関する設定を行うレジスタです。いろいろな設定項目があります。

ここでは、ホールセンサの入力で使用するGPIOの割り込みを有効にする設定を行います。

* SYSCFG_EXTICRx
  * EXTICR4_EXTI15: PA[15] 0000b
  * EXTICR1_EXTI3 : PB[3]  0001b
  * EXTICR3_EXTI10: PB[10] 0001b
    * 上からH1,H2,H3の割り込みを有効にします。

#### 1. GPIOx

GPIOの設定を行うレジスタです。

ホールセンサで使用するGPIOを入力モードに設定します。

* GPIOx_MODER
  * GPIOA_MODER15: Input mode 00b
  * GPIOB_MODER03: Input mode 00b
  * GPIOB_MODER10: Input mode 00b
    * 上からH1,H2,H3の入力に使用します。

#### 1. EXIT

外部割込みの設定を行うレジスタです。

ホールセンサ入力の割り込みの設定を行います。

* EXIT_IMR1
  * MR15: Interrupt request from Line x is not masked 1b
  * MR3 : Interrupt request from Line x is not masked 1b
  * MR10: Interrupt request from Line x is not masked 1b
    * 上からH1,H2,H3の割込み要求のマスクを無効にします。つまり、割込み要求がNVICに通知されます。
    * EMR(Event mask register)との違いについて調べたので記載します。
      * イベントで割込みハンドラは実行されません。
      * イベントは周辺機能間で使用されます。例えば、あるGPIOにHighの信号が入力されたときにADCのサンプリングタイミングとしてイベントを通知できます。
* EXTI_RTSR1
  * TR15: Rising trigger enabled  1b
  * TR3 : Rising trigger enabled  1b
  * TR10: Rising trigger enabled 1b
    * 上からH1,H2,H3のLow⇒Highのエッジによる割込みを有効にします。
* EXTI_FTSR1
  * TR15: Falling trigger enabled  1b
  * TR3 : Falling trigger enabled  1b
  * TR10: Falling trigger enabled 1b
    * 上からH1,H2,H3のHigh⇒Lowのエッジによる割込みを有効にします。

#### 1. NVIC

割込みを有効にするレジスタです。Arm CortexM4で定義されたレジスタです。

NVIC_ISER0～NVIC_ISER2まであり、各ビットがRM: Table 41. STM32F302x6/8 vector tableのPostiionに対応します。

* NVIC_ISERx
  * ISER1_SETENA[8]: EXTI15_10 interrupt enable 1b
    * H1(EXTI15),H3(EXTI10)の割り込みを有効にします。複数の割込み要因に対して1つのハンドラが実行されるので、EXTI_PR1を確認してどちらの割込みであるかを確認する必要があります。
  * ISER0_SETENA[9]: EXTI3 interrupt enable 1b
    * H2(EXTI3)の割り込みを有効にします。こちらは割込み要因とハンドラが1対1なので、EXTI_PR1の確認は必要ないです。

### 2. イネーブル信号

#### 2. Clock

イネーブル信号で使用するGPIOのクロックを有効にします。

* RCC_AHBENR: IOPCEN Enable 1b
  * イネーブルでPC10,11,12を使用するので、GPIOCのクロックを有効にします。

#### 2. GPIOC

出力ポートの設定を行います。今回は起動時にイネーブル信号をHighにしてPWM出力を有効にするだけなので、MODERの設定だけです。高速にイネーブル信号を制御する場合はOSPEEDRの設定も必要になると思います。

* GPIOx_MODER
  * MODER20: Output mode 01b: PC10
  * MODER22: Output mode 01b: PC11
  * MODER24: Output mode 01b: PC12

### 3. PWM

PWMはマイコンのタイマーモジュールを使用します。

STM32F302x6/8で使用できるタイマーモジュールはTIM1,TIM2,TIM15,TIM16,TIM17があります。TIM1が一番高機能です。それ以外は汎用タイマとなっています。※TIM2とTIM15,TIM16,TIM17でも機能に差がありそうですが、今回は使用していないので調べていません。

今回はTIM1を使用しますので、TIM1に関するレジスタを設定します。

### 3. 構成図

![PWMの構成](/assets/images/image-2023-10-29-motor-pwm-structure.png)

> RM: Figure 132. Advanced-control timer block diagram。

* CNT
  * カウンタです。0始まりです。
  * カウントする周期を設定できます。
  * カウント方法(Up/Down/ARRで折り返し)を設定できます。
* ARR(Auto-reload register)
  * カウンタの1周期の値です。
  * カウント方法により動作が変わります。
    * UpカウントのときはCNTが0からインクリメントされ、CNTがARRを超える場合はCNTが0に戻る。
    * DownカウントのときはCNTがARRからデクリメントされ、CNTが0未満になる場合はCNTがARRに戻る。
    * ARRで折り返すときはUpカウントでCNTが0からインクリメントされ、CNTがARRを超える場合はDownカウントとなりCNTがデクリメントされる。そして、CNTが0未満になる場合はUpカウントになりCNTがインクリメントされる。
* OC
  * High/Lowを出力するコントローラです。
  * 使っていませんが、相補出力もできます。
  * GPIOのポートから出力されます。GPIOの設定が必要です。
* CCR(Capture/Compare register)
  * 出力をHigh/Lowに切り替える閾値です。
  * カウント方法に応じたCCRとCNTが一致したときの動作パターン※何れもHigh/Lowの極性は変えられます。
    * Upカウントのとき、CCR未満はHigh、CCR以上はLowを出力する。
    * Downカウントのとき、CCR以上はHigh、CCR未満はLowを出力する。
    * ARRで折り返すとき、CCR未満はHigh、CCR以上はLowを出力する。Upカウントとの違いは出力が凹になる。
* GC5C1～3
  * OCXとOC5の論理積を出力することができます。
  * 今回はOC1～3に同じPWM設定を行い、OC5を常にLow出力として、GC5CXを切り替えてなんちゃって三相のパルスを出力します。次のタイミングチャートを見てください。

### 3. タイミングチャート

![PWMのタイミングチャート](/assets/images/image-2023-10-29-motor-pwm-timingchart.png)

#### 3. Clock

TIM1のクロックを有効にします。また、TIM1から出力するPWMはGPIOを使用しますので、PWMで使用するGPIOのクロックも有効にします。

* RCC_AHBENR: IOPAEN Enable 1b
  * TIM1でPA8,9,10を使用するので、GPIOAのクロックを有効にします。
* RCC_APB2ENR: TIM1EN Enable 1b
  * TIM1のクロックを有効にします。
* RCC_CFGR3: TIM1SW PCLK 00b
  * TIM1のクロックを選択できます。デフォルトのPCLK(Peripherals Clock)を使用します。
  * CK_INTになります。

#### 3. GPIOA

* GPIOx_MODER
  * MODER8: Alternate function mode 10b
  * MODER9: Alternate function mode 10b
  * MODER10: Alternate function mode 10b
    * AFで使用します。
* GPIOx_AFRH
  * AFR8: AF6 TIM1_CH1 0110b
  * AFR9: AF6 TIM1_CH2 0110b
  * AFR10: AF6 TIM1_CH3 0110b
    * PWMで使用します。

> AFは以下のリファレンスを参照してください。
> DS: Table 14. Alternate functions for Port A

#### 3. TIM1の基本設定

制御中は基本的に変えない設定を記載します。

* TIM1_CR1
  * CKD: CK_INT 00b
    * 分周の設定です。CK_INTの周波数をそのまま使用します。
  * ARPE: TIMx_ARR register is buffered 1b
    * カウンタの自動設定を有効/無効を設定します。
  * CMS: Center-aligned mode. Interrupt flags is set when the counter is counting down 01b
    * PWMのモードを設定します。
    * RM: Figure 167. Center-aligned PWM waveformsn に具体的な波形が描かれています。
  * DIR: Counter used as upcounter 0b
    * カウント方法(カウンタアップ/カウントダウン)を選びます。
  * OPM: Counter is not stopped at update event 0b
    * Oneパルスの有効/無効を設定します。有効(1b)の場合はカウンタ満了すると特定のレジスタ(CEN)をクリアするまでカウンタがストップします。
  * CEN: Counter enabled 1b
    * カウンタが有効/無効を設定します。
* TIM1_DIER
  * CC1IE: CC1(カウンタチャネル1) interrupt enable 1b
    * チャネル1の割り込みを有効にします。割込みでTIM1_CCR5のGC5C1,GC5C2,GC5C3の切り替えを行います。
    * 割込みが発生するタイミングは RM: Figure 167. を参照してください。CMS=01なので、ダウンカウントでカウンタがCCRxと一致したときに割込みが発生します。
    * 以下の設定も忘れないように。
      * Vector Tableにハンドラを登録。RM: Table 41. STM32F302x6/8 vector tableを参照。
      * NVICの割り込みを有効にする。
* TIM1_CCMR1
  * CC1S: CC1 channel is configured as output 00b
  * CC2S: CC2 channel is configured as output 00b
    * 入力/出力を選択します。
    * RMを見るとTIM1_CCMR1のセクションが二つに分かれています。一つ目のセクションがここで入力を設定した場合のレジスタの仕様で、2つ目のセクションがここで出力を設定した場合のレジスタの仕様です。
  * OC1M: CC1 PWM mode 1  0110b
  * OC2M: CC2 PWM mode 1  0110b
    * 出力モードを選択します。PWM mode1は設定したTIMx_CCR1(カウンタ比較値)未満の時にHigh、それ以外はLowとなるPWMです。
  * OC1PE: CC1 preload enable 1b
  * OC2PE: CC2 preload enable 1b
    * プリロードの有効/無効を設定します。プリロードが有効の場合、TIMx_CCR1に書き込んだ値はプリロードレジスタに書き込まれ、カウンタが1周したときにTIMx_CCR1へ書き込まれます。
* TIM1_CCMR2
  * CC3S: CC3 channel is configured as output 00b
  * OC3M: CC3 PWM mode 1  0110b
  * OC3PE: CC3 preload enable 1b
    * CC3の設定です。CC1-2と同じです。
* TIM1_CCMR3
  * OC5M: CC3 PWM mode 1  0110b
  * OC5PE: CC3 preload enable 1b
    * CC5の設定です。CC5は出力のみです。
    * CC5はCC5とCC1,2,3に対して論理積(AND)を行うことができます。詳細はRM: 20.3.14 Combined 3-phase PWM modeを参照してください。これを使用して3相のPWMを作ります。

##### 3. NVIC 割込みコントローラ

NVICはArm Cortex-M4の機能になります。レジスタの内容はリファレンスはPMになります。

ただ、ベクターテーブルはRMを参照します。

TIM1の割り込みを有効にします。

* NVIC_ISERx
  * 割込みを有効にするレジスタです。NVIC_ISER0～NVIC_ISER2まであり、各ビットがRM: Table 41. STM32F302x6/8 vector tableのPostiionに対応します。
  * ISER1_SETENA[27]: interrupt enable 1b
    * TIM1_CCを有効にします。
    * ベクターテーブルからTIM1_CCがPosition 27となっているので、それに対応するNVIC_ISER0のbit27を1bに設定します。

##### 3. TIM1 PWMのパルス幅とDuty

制御中に変更するレジスタです。

* TIM1_PSC
  * PSC: 8
    * CNTのプリスケーラを設定します。CK_INTの8MHzを1MHzにします。つまり、1カウント=1usです。
    * 今回は制御中に変更しないです。
* TIM1_ARR
  * ARR: 23750
    * CNTの最大値です。つまり、PWMの周期になります。
    * CMSをCenter-aligned modeに設定しているので、CNTは0, 1, ..., ARR - 1, ARR, ARR - 1, ..., 1, 0とカウントされます。
    * 初期値は低速時の値を設定しています。
* TIM1_CCR1~TIM1_CCR3
  * CCR: 0
    * カウンタの閾値を設定します。閾値と一致すると、パルスの出力が反転させます。
    * CMSをCenter-aligned modeに設定しているので、閾値は2回一致します。最初に一致でHigh⇒Lowになり、次の一致(折り返し中)でLow⇒Highになります。
    * Duty = ARR / 2 = 50% にしています。
    * 初期値は0にして、常にLow出力にします。つまり、モータは停止です。
* TIM1_CCR5
  * CCR: 0
    * CC1～CC3をマスクするPWMです。CCRを0にすると、常にLow出力になります。

##### 3. 出力の有効化

出力を有効にします。複数のレジスタの組み合わせで出力が有効になります。RD: Table 116. を参照してください。

* TIM1_CCER
  * CC1E: CC1 output enable 1b
  * CC2E: CC2 output enable 1b
  * CC3E: CC3 output enable 1b
* TIM1_BDTR
  * モーターに安全機能を付けたいときに重要になるレジスタです。
  * MOE: : OC and OCN outputs are enabled 1b
    * TIM1の全体の出力を有効/無効化する設定です。
    * break信号が入力されると自動で0bとなり、出力が無効になります。
    * 出力が有効の場合はPWMが出力されます。
    * 出力が無効の場合はHi-Zとなります。無効時の出力をLow/Highで固定したい場合はOSSIを設定します。

## 割込みハンドラ

### 関数の宣言

割込みハンドラは関数の宣言で以下のように__attribute__((interrupt("IRQ")))を付けて宣言します。

``` c
/* TIM1の割込み */
void IRQ_TIM1_CC_Handler() __attribute__((interrupt("IRQ")));
void IRQ_TIM1_CC_Handler() { /* 処理 */ }
```

これにより、コンパイラが割込みハンドラ用の開始/終了処理を追加します。

__attribute__((interrupt("IRQ")))を付けない場合と比べて、スタックに退避するレジスタが多くなります。

``` asm
00000090 <IRQ_TIM1_CC_Handler>:
  /* レジスタをスタックにpush */
  90:   4668            mov     r0, sp
  92:   f020 0107       bic.w   r1, r0, #7
  96:   468d            mov     sp, r1
  98:   b481            push    {r0, r7}
  9a:   af00            add     r7, sp, #0
  /* 処理 */
  9c:   bf00            nop
  /* スタックからレジスタをpopしてリターン */
  9e:   46bd            mov     sp, r7
  a0:   bc81            pop     {r0, r7}
  a4:   4770            bx      lr
```

### Vector Tableへ登録

割込みハンドラをVector Tableに登録します。割込みハンドラを登録しないと割込みが発生したときにリセットがかかります。

コードでVector Tableをg_pfnVectorsと定義しています。コードを見てください。

### 割込み要求フラグのクリア

割込みハンドラで割込み要求フラグをクリアします。クリアしないと割込みが発生し続けます。

### ホールセンサー

ホールセンサーで使用する割込みハンドラは以下になります。ホールセンサー信号の立上り、立下りで割り込みが発生して、割込みハンドラが実行されます。

* IRQ_EXTI3_Handler
  * EXTI3専用。
* IRQ_EXTI15_10_Handler
  * EXTI10~EXTI15で共用。EXTI11などの割込みも使用する場合は次の割込み要求フラグを確認して処理を分岐させる必要がある。

割込みハンドラでは以下の割り込み要求フラグをクリアします。クリアしないと割込みハンドラが実行され続けます。他の割込みハンドラも同じです。

割込み要求フラグのレジスタは以下になります。

* EXTI_PR1
  * PR15: Triger request occured 1b
  * PR3 : Triger request occured 1b
  * PR10: Triger request occured 1b
    * 割込み要求があれば1bがセットされます。
    * 割込みハンドラが共用の場合はフラグを見て割込みを識別します。
      * 同時に複数の割込み要求がセットされる場合もあります。その場合は1回の割込みハンドラの実行で全ての割込み要求を処理するか、1回の割込みハンドラの実行で1つの割込み要求を処理するかを検討する必要があります。

今回は割込みハンドラが実行されることだけ確認し、処理は行っていません。今後、ホールセンサーの割込みでモータの回転速度の測定、回転位置の検出で使用します。

### PWM制御

PWMで使用する割込みハンドラは以下になります。

* IRQ_TIM1_CC_Handler
  * CNTがDownカウント時にCCRと一致したときに割込みが発生します。
  * このハンドラもIRQ_EXTI15_10_Handlerと同じようにCCRXで共用なので、次の割込み要求フラグを確認する必要があります。ただ、今回はCCR1の割込みだけ有効にしているので、CCR1専用として処理します。

割り込み要求フラグのレジスタは以下になります。

* TIM1_SR
  * CC1IF: Triger request occured 1b
    * カウンタのイベント発生時に割り込みが発生し、UIFがセットされます。

割込みハンドラで3. タイミングチャートのようにGC5Cを設定します。

* TIM1_CCR5
  * GC5C1
  * GC5C2
  * GC5C3

## コード

<https://github.com/ohmusso/NUCLEO-F302R8/tree/mortor>

## 動作確認

* 低速時に回転しなかったです。
* 高速時は回転しました。

<video controls width="100%" preload loop muted="true" src="/assets/movies/movie-2023-10-29-motor.mp4" type="video/mp4" >
 Sorry, your browser doesn't support embedded videos.
</video>

## 今後

* センサを使って回転数を検出できるようにします。
* PWMの制御を改善していきます。

## 以上
