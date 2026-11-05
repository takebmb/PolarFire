# Chapter 4-5: Board Components Placement / Demo Design（UG p.14-16）

## Core Idea
シルク図（Rev 2、SEP 2023、Pactron 製）で部品とコネクタの物理位置と信号名を確認する。出荷時のデバイスには FIR フィルタのデモが書き込まれている。

## Frameworks Introduced
- **シルクで信号名を確かめる**: ピン表（Ch6）は番号だけなので、mikroBUS・MIPI の信号名はシルクで照合する
  - mikroBUS J5 側: AN / RST / CS / SCK / MISO / MOSI / +3.3V / GND
  - mikroBUS J6 側: PWM / INT / RX / TX / SCL / SDA / +5V / GND
  - MIPI RX J11（底面）: GND, RX_N0/P0, RX_N1/P1, RX_CKN/CKP, CAM_GPIO, CAM_EN, I2C_SCL/SDA, 3P3V
- **出荷時デモの確認**: 届いた直後に電源を入れてデモが動けば、ボードは正常と判断できる。詳細は AN5165

## Key Concepts
- **Top view**: J10（RPi、pin 1/2 が右端・39/40 が左端）、J45/J46 は J10 の下、SW8、LED1-8、J3（RJ45）、J48（7 セグ）+ J49、SW1/SW2、J4（USB-C）、J7（DC ジャック）、J47、電源 LED（5P0V / 1P8V / VDD）、テストポイント GND1/GND4/GND5/TP_VDD/TP_1P8V、J12（Live probes）
- **Bottom view**: J11 MIPI RX、J17 microSD（push 式、LOCK/UNLOCK 表示）、U16（FT4232/eFP5 周辺）、X1（125 MHz）
- **基板表記**: `MPFS-DISCO-KIT REV 2`
- **FIR Filter demo**: 出荷時のデザイン。AN5165「PolarFire SoC DSP FIR Filter Demo」

## Anti-patterns
- **RPi 40pin のピン 1 を左端と思い込む**: シルクでは pin 1 側が基板の右寄り。HAT を挿す前に向きを確認する
- **デモを消す前に動作確認しない**: 自作の書き込みで消える前に、出荷時の状態で一度動作を見る

## Key Takeaways
1. mikroBUS・MIPI の信号名はシルクで確認する（ピン表には番号しかない）
2. MIPI と microSD は底面にある
3. 出荷時デモは FIR フィルタ（AN5165）

## Connects To
- **Ch6**: ピン番号と FPGA ピンの対応
- **Ch5**: デモの再書き込み・自作デザインの書き込み
