---
layout: post
title:  NUCLEO-F302R8 LEDチカチカ コード説明
date:   2023-09-11 22:00:00 +0900
categories: iot
tags: arm stm32
---

以下の記事で書いたコードの説明を残します。

[環境構築]({% link _posts/2023-09-09-NUCLEO-F302R8-BlinkLED.markdown %})

## RM リファレンスマニュアル

CPUとマイコンのリファレンスマニュアルを参照します。

### Arm-RM

<https://developer.arm.com/documentation/ddi0403/latest/>

### F302R8-RM

<https://www.st.com/resource/en/reference_manual/rm0365-stm32f302xbcde-and-stm32f302x68-advanced-armbased-32bit-mcus-stmicroelectronics.pdf>

## stm32f3.ld

リンカスクリプトです。コンパイルしたデータのメモリ配置を決めます。

リンカで使用します。

![linker dataflow](/assets/images/image-2023-09-11-linker-dataflow.png)

### メモリ MEMORY

実メモリのアドレスを定義します。以下のようなイメージです。

メモリに配置するセクションはSECTIONSで記載します。

![linker memory](/assets/images/image-2023-09-11-linker-memory.png)

SRAMの点線はリプロ(プログラムの書き換え)で書き込まれないという表現です。SRAMはプログラムで書き込みます。

SRAMは起動時に何が書き込まれているか分からない状態なので、プログラムによる初期化が必要です。

#### 領域の分割

FLASH, SRAMはMain Flash Memory, SRAMのアドレス内で細かく分けることも可能です。その場合はセクションも分ける必要があります。

``` ld
/* 例 */
MEMORY {
    SRAM (rwx)    : ORIGIN = 0x20002000, LENGTH = 8K
    SRAM (rwx)    : ORIGIN = 0x20000000, LENGTH = 8K
    FLASH_OS (rx)    : ORIGIN = 0x08008000, LENGTH = 32K
    FLASH_USER (rx)  : ORIGIN = 0x08000000, LENGTH = 32K
}
```

### ENTRY

最初に実行される関数を定義します。

Reset_Handlerを設定しています。これは起動時にCPUから暗黙的に実行されるため、リンカがプログラム内で使っていないと判断し、リンカのオプションによってはReset_Handlerが消えてしまいます。ENTRYに定義することで、リンカに使用していることを伝えます。

Reset_Handlerは後述のベクターテーブルで使用します。

### _estack

スタックの先頭アドレスを定義します。ここではRAMの一番後ろのアドレス(0x2000 4000)としています。

後述のベクターテーブルで使用します。

### セクション SECTIONS

オブジェクトファイルのデータの配置先をセクションとして定義します。

リンカは定義したセクションのとおりデータを配置して、プログラムの関数や変数のアドレスを決めます。

以下のようなイメージです。

![linker section](/assets/images/image-2023-09-11-linker-section.png)

セクションの記述方法は以下のサイトが分かりやすいです。

<http://blueeyes.sakura.ne.jp/2018/10/31/1676/>

#### .isr_vector

ベクタ－テーブルを配置するセクションです。FLASHの先頭にベクターテーブルが配置されるように、最初に定義しています。

##### ALIGN

セクションの区切りを調整するために使われます。

以下のようなイメージです。

![linker align](/assets/images/image-2023-09-11-linker-align.png)

##### }> FLASH

セクションのデータをFLASHに配置します。

#### .text

主に命令を配置するセクションです。関数は丸々入るイメージです。関数内で使用される定数データも入ります。

##### \*(.text) \*(.text.*)

\*(.text)の"\*"はファイル名のワイルドカードです。つまり、コンパイルしたすべてのファイル(\*.obj)の.textを配置します。

\*(.text.*)はさらに末尾に識別子をつけたセクションを配置します。
後述のReset_Handlerはここに配置されます。

##### glue_7 glue_7t

このプログラムでは使用されていないセクションで、私も使ったことがないです。

以下は調べた内容です。

> Armのコンパイラ特有のセクションです。
> ArmにはTHUMB命令(16bit)とARM命令(32bit)があります。  
> THUMB命令はサイズが半分なので、ROMのサイズがARM命令に比べて小さくなります。
> ARM命令はサイズが増える代わりに処理が早くなります。
> 一つの命令で行える処理が増えるからです。  
> この二つの命令は実行中に切り替えることができます。基本はTHUMB命令を使用して、重たい処理をするところではArm命令に切り替えるということが可能です。
> この切り替えを行うために使われるセクションが、glue_7, glue_7tです。

---

##### KEEP(*(.init)) KEEP(*(.fini))

C++のコンストラクタとデストラクタが配置されます。

KEEPはセクション内のデータが使用されない場合でもデータを保持します。

##### .rodata

constで定義した変数が配置されます。

#### _sidata

初期値付き変数の初期値を配置したアドレス(FALSH)の先頭を定義します。後述のRAM初期化で使用します。

LOADADDRは引数で与えられたセクションを実アドレスに変換します。

#### .data

初期値付き変数を配置するセクションです。

##### SRAM AT> FLASH

セクションのデータをSRAMとFLASHに配置します。

sdata,_edataはSRAM内の.dataの開始/終了アドレスとなります。

後述のRAM初期化処理でFLASHから初期値を読みだした、SRAMに書き込むことで初期値付き変数が実現されます。

#### .bss

初期値がなしまたは0の変数を配置するセクションです。

このセクションも後述のRAM初期化処理で0をSRAMに書き込みます。

##### \*(COMMON)

重複定義してよい変数を配置するセクションのようです。

配置する場合の具体的なコードの書き方が分かりません。。。

基本的にファイル間で変数を重複定義すると、リンカでエラーになります。

### Linker Options

ビルドでリンカを実行するときのオプションについてメモを残しておきます。

toolchain-STM32F302R8.cmake

``` cmake
# Linker
## linker script file
set(LINKER_FILE ${CMAKE_CURRENT_LIST_DIR}/stm32f3.ld)
### linker flags
#### -Wl,--gc-sections
set(CMAKE_EXE_LINKER_FLAGS "-mcpu=cortex-m4 -mthumb -mfloat-abi=hard -mfpu=fpv4-sp-d16 -nostdlib -T\"${LINKER_FILE}\" -Wl,-Map=out.map -Wl,--gc-sections")
```

#### -nostdlib

標準ライブラリをリンクしないようにするオプションです。これを外すと、標準ライブラリがリンクされます。

外しても次の記載する"--gc-sections"により、使用していなければバイナリから削除されます。

#### -Wl,XXX

一つのリンカオプションを渡すときはこの書き方をします。"XXX"がリンカオプションです。

このプロジェクトのcmakeは直接リンカ(arm-none-eabi-ld.exe)を実行しておらず、コンパイラ(arm-none-eabi-gcc.exe)から間接的にリンカを実行しています。そのため、このようにリンカオプションを書く必要があります。

##### -Wl,--gc-sections

未使用のセクションを削除するリンカオプションです。

## startup.s

起動時の処理を記載します。

起動シーケンスは以下のようになります。

![startup](/assets/images/image-2023-09-11-startup.png)

### BOOT

CPUが実行を開始するアドレスにプログラムをマッピングします。

CPUは0x0000 0000にあるベクターテーブルを起点に動作を開始します。

プログラムはMain Flash Memoryの0x0800 0000に書き込まれいるので、BOOTにより0x0000 0000 ～ 0x0000 FFFFへのアクセスが0x0800 0000 ～ 0x0800 FFFFに置き換えられます。

> F302R8-RM: 3.5 Boot configurationを参照

### ベクターテーブル vector table

ベクターテーブルは例外/割込みが発生したときに実行するアドレスを設定します。

ベクターテーブルの1行は4byte(.word)です。g_pfnVectorsと定義しています。

起動時はCPUにリセットの例外が発生して、ベクターテーブルのNo.0とNo.1にアクセスします。

リセットのときだけベクターテーブルの二つの行にアクセスします。

#### No.0 メインスタックポインタ初期値

CPUはここに格納された値をメインスタックポインタ(msp)に設定します。

_estackを設定して、RAMの一番後ろをスタックとして使用します。

#### No.1 リセット Reset

CPUはここに格納されたアドレスをリセットハンドラとして実行します。つまり、CPUに最初に実行される処理となります。

#### No.15 SysTick_Handler

CPUが一定周期で呼んでくれるハンドラです。

このプログラムでは時間を数えるために使用します。

後で解説します。

### リセットハンドラ Reset_Handler

RAMの初期化とmain()の呼び出しを行っています。

アセンブラを読むときのメモを残しておきます。(普段は使わないし、苦手です。。。)

* .section を記載したところからセクションが割り当てられる。次の.sectionまで同じセクションが割り当てられる。
* ループ処理はループ内の処理が先に書かれて、ループの判定が後に書かれる。CopyDataInit、LoopCopyDataInitのように。

## Cソース

ClockとGPIOの初期化を行い、一定周期でLEDをOFF/ONします。

### Driver/Clock.c

#### Clockの概要

今回設定するクロックの経路を黄色で塗っています。

![clock diagram](/assets/images/image-2023-09-11-clock-diagram.png)

> F302R8-RM: 9 Reset and clock control (RCC) Fig 14

コードの前に、図の説明をします。

##### HSI

High Speed Intanal Clockです。周波数は8MHzです。リセット時は
このクロックを元に動作します。

##### SYSCLK

システムクロックです。CPUもこのクロックで動きます。

SWを設定することで、SYSCLKをHSI,PLLCLK,HSEから選択できます。

リセット時はHSIになります。

以下のレジスタで設定できます。

> F302R8-RM: 9.4.2 Clock configuration register (RCC_CFGR) SW

###### HCLK

バスなどの周辺機能で使用されるクロックです。

* LEDを出力するGPIOが属するAHBバスのクロックになります。
* SysTickの外部クロックになります。ただし、図のように/8されているので、周波数は1MHzになります。

SYCLKをAHB prescalerで分周したクロックがHCLKとなります。

AHB prescalerはリセット時に1分周となっています。リセット時のまま使用しますので、周波数はSYCLKのままです。

以下のレジスタで設定できます。

> F302R8-RM: 9.4.2 Clock configuration register (RCC_CFGR) HPRE

#### GPIOBのクロックを有効化

点灯させるLED2と繋がっているPB13はGPIOBに属しています。

リセット時はGPIOBのクロックが無効になっているので、クロックを有効にします。

コードは以下になります。

``` c
void Clock_Init()
{
  stpRCC->AHBENR = Init_RCC_AHBENR;
  // 省略
}
```

レジスタの詳細は以下を参照してください。

> F302R8-RM: 9.4.6 AHB peripheral clock enable register (RCC_AHBENR)

#### SysTickの有効化

時間をカウントするためにArm CortexのSysTick例外を使用します。

SysTickにカウンタを設定できます。SysTickはカウンタをデクリメントし、カウンタが0になると例外を発生させます。今回は1ms毎に例外が発生するようにSysTickのクロックとカウンタを設定します。

リセット時はSysTickが有効になっておらず、SysTick例外は発生しません。

コードは以下になります。

``` c
void Clock_Init()
{
  stpSYST->CSR = Init_SYST_CSR;
  stpSYST->RVR = Init_SYST_RVR;
  stpSYST->CVR = Init_SYST_CVR;
  // 省略
}
```

レジスタの詳細は以下を参照してください。Armの仕様です。

> Arm-RM: B3.3 The system timer, SysTick

##### CSR

SysTickを設定するレジスタで難しい感じたCSRを補足します。

SycTickの有効化と、SysTickで使用するクロックを選択できます。

* CLKSOURCE: 内部か外部のクロックをSysTickのクロックとして選択できます。内部はHCLKと同じです。外部はHCLKを/8した周波数と同じです。どちらでも問題ないですが、ここでは外部(HCLK/8=1MHz)を使用しています。つまり、SysTickの1カウントは1μsです。
* TICKINT: カウンタが0になった時の例外を発生させるかを設定できます。例外を発生させる設定にしています。
* ENABLE: SysTickのカウントを有効/無効に設定できます。使用するので有効にしています。

### Driver/Port.c

#### Portの概要

ポートの構成は以下のようになります。

![port diagram](/assets/images/image-2023-09-11-port-diagram.png)

##### GPIOB

このマイコンはGPIOがA～Fまであります。

ボードのLED2が繋がっているのがpin32です。

PB13(GPIOBの13)がpin32を制御していますので、PB13をプログラムで制御します。

GPIOBのレジスタは以下になります。

![port register](/assets/images/image-2023-09-11-port-register.png)

> F302R8-RM: Table 4. STM32F302x6/x8 peripheral register boundary addresses  

#### PB13を出力ポートに設定

LEDを点灯するのでPB13を出力ポートとして使用します。

コードは以下になります。

``` c
void Port_Init()
{
  stpGPIOB->MODER = Init_GPIOB_MODER;
 // 省略
}
```

レジスタの詳細は以下を参照してください。

> F302R8-RM: 10.4.1 GPIO port mode register (GPIOx_MODER)

#### PB13の出力速度の設定

ポートの出力速度を設定します。1秒間隔でOn/Offするだけなので、リセット時の値であるLowでも問題ないです。

今回はMidに設定しています。

コードは以下になります。

``` c
void Port_Init()
{
  stpGPIOB->OSPEEDR = Init_GPIOB_OSPEEDR;
 // 省略
}
```

レジスタの詳細は以下を参照してください。

> 10.4.3 GPIO port output speed register (GPIOx_OSPEEDR)

#### PB13への書き込み

On/Offを書き込みます。

Onだとpinの電圧がHighになり、LEDが点灯します。

Offだとpinの電圧がLowになり、LEDが消灯します。

コードは以下になります。

``` c
void Port_Write(
    PortOnOff value
)
{
  //省略
  stpGPIOB->ODR = (uint32)((value & 0x01) << GPIOB_ODR_SHIFT_13);
}
```

レジスタの詳細は以下を参照してください。

> 10.4.6 GPIO port output data register (GPIOx_ODR)

### main.c

#### ドライバの初期化

クロックとポートの初期化を行います。

コードは以下になります。

``` c
int main()
{
  Clock_Init();
  Port_Init();
 //省略
}
```

#### LEDチカチカ

1秒周期でPB13にOn/Offを書き込むことで、LEDをチカチカさせます。

コードは以下になります。

``` c
int main()
{
 //省略
  const uint32 blinkTime = 1000;
  while(1){
    if( systickCount <= blinkTime ){
      Port_Write(Port_Off);
    } 
    else if( systickCount <= (blinkTime * 2) ){
      Port_Write(Port_On);
    }
    else{
      Port_Write(Port_Off);
      systickCount = 0;
    }
  }
}
```

##### 1ms周期でコールされる例外ハンドラ

1msに設定したArm CortexのSysTick例外を使用して、1秒をカウントします。

コードは以下になります。

``` c
void SysTick_Handler()
{
  systickCount++; // 1カウントが1ms。1000カウントで1秒。
}
```

## 以上

次は以下にチャレンジして、勉強したことをまとめたいです。

* デバッグの仕方
* wifi
* モータ制御
