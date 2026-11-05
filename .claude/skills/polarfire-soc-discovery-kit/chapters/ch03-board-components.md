# Chapter 3: Board Component and Operations（UG p.11-13）

## Core Idea
各周辺が **MSS 側の Bank（2/4/5/6）か、ファブリック側の Bank（0/1）か** で、設定する場所が変わる。MSS 側は MSS Configurator、ファブリック側は RTL と PDC 制約で扱う。

## Frameworks Introduced
- **MSS / ファブリック振り分けマップ**
  | 周辺 | Bank | 側 |
  |---|---|---|
  | DDR4 | 6 (MSS DDR) | MSS |
  | microSD | 4 (MSS SD)、電圧変換 IC 経由 | MSS |
  | Ethernet VSC8221 SGMII | 5 (MSS SGMII) | MSS |
  | 125 MHz MSS_REFCLK | 5（P5/P4） | MSS |
  | UART_B, UART_C | 2 (MSS I/O) | MSS |
  | UART_D | 0 (HSIO) | ファブリック |
  | RPi SPI + ID I2C（7 本） | 2 (MSS I/O) | MSS |
  | RPi UART・I2C 1 組・その他 GPIO（21 本） | 1 (GPIO) | ファブリック |
  | mikroBUS（UART/SPI/I2C） | 1 | ファブリック |
  | 7 セグ（J48） | 0 | ファブリック |
  | SW1-2, SW8, LED1-7 | 0 | ファブリック |
  | LED8 | 2 (MSS I/O, E1) | MSS |
  | 50 MHz（R18） | 0 | ファブリック |
  | Live probes J12 | GPIO Bank | ファブリック/デバッグ |
  | Debug Reset | 3 | — |
  - When to use: 新しい周辺を使うとき、まずどちら側で設定するかを決める

## Key Concepts
- **DDR4**: Micron MT40A512M16TB-062E:R、1 GB、800 MHz（UG 表記の「Frequency range」）、MSS Bank 6
- **microSD**: GCT MEM2075-00-140-01-A、MSS Bank 4、カードと FPGA の間に電圧変換 IC
- **FT4232HL**: FTDI 製。UART_B/C → MSS Bank 2、UART_D → HSIO Bank 0。COM ポートの認識には FT4232H ドライバが必要
- **RPi 40pin**: Würth 61204021621
- **mikroBUS**: 16pin（J5+J6 の 8pin×2）。UART/SPI/I2C。Click ボード（mikroe.com）
- **7-Segment**: 10pin、Serial 7-Seg 8-Digit 基板用、HSIO Bank 0
- **Debug**: Push button 2、DIP 8、Debug LED 8（+ 電源 LED 3）
- **DEVRST_n (E8)**: 入力専用リセットパッド。チップ全体をリセット。MCP121T-315E/TT（3P3V 監視）で駆動
- **50 MHz osc**: ±10 ppm、1.8V、ファブリック R18（Bank 0）= システム基準クロック
- **125 MHz osc**: ±10 ppm、LVDS、VDDI1_5 給電、Bank 5 P5(P)/P4(N) = MSS 基準クロック
- **Form factor**: 約 4.15 × 3.3 inch
- **Pin List**: パッケージ全ピンは PPAT（Package Pin Assignment Tables）参照

## Reference Tables
**Debug LED（Table 3-1）**
| LED | Bank | Pin |
|---|---|---|
| LED1 | 0 | T18 |
| LED2 | 0 | V17 |
| LED3 | 0 | U20 |
| LED4 | 0 | U21 |
| LED5 | 0 | AA18 |
| LED6 | 0 | V16 |
| LED7 | 0 | U15 |
| LED8 | 2 | E1 |

## Code Examples
<!-- UG にコードは無い。以下はピン表から作った Libero PDC の例（UG 外） -->
```tcl
# 50 MHz fabric clock and LED1-7 on Bank 0 (1.8V)
set_io -port_name {CLK_50MHZ} -pin_name R18 -fixed true -io_std LVCMOS18 -DIRECTION INPUT
set_io -port_name {LED[0]}    -pin_name T18 -fixed true -io_std LVCMOS18 -DIRECTION OUTPUT
set_io -port_name {LED[1]}    -pin_name V17 -fixed true -io_std LVCMOS18 -DIRECTION OUTPUT
set_io -port_name {LED[6]}    -pin_name U15 -fixed true -io_std LVCMOS18 -DIRECTION OUTPUT
```
- **What it demonstrates**: Bank 0 は 1.8V 固定なので LVCMOS18。LED は High で点灯

## Anti-patterns
- **LED8 を他の LED と同じく PDC で割り当てる**: LED8（E1）は MSS I/O Bank 2。MSS Configurator 側で扱う
- **UART が見えないのをハード不良と判断する**: 先に FT4232H ドライバを入れる
- **MSS クロックを 50 MHz と考える**: MSS の基準は 125 MHz（X1、Bank 5）。50 MHz はファブリック用

## Key Takeaways
1. DDR4・SD・Ethernet・UART_B/C・RPi SPI は MSS 側
2. LED1-7・SW・7 セグ・UART_D・50 MHz はファブリック Bank 0（1.8V）
3. RPi 40pin は MSS（7 本）とファブリック Bank 1（21 本）に分かれている
4. 書き込みは eFP5 経由の JTAG
5. リセットは DEVRST_n（E8）、3.3V 監視 IC 付き

## Connects To
- **Ch2**: Bank 電圧と J45/J46
- **Ch5**: JTAG 書き込み
- **Ch6**: 全ピン表
