# Cheatsheet — MPFS-DISCO-KIT（DS50003630E Rev E）

## ジャンパ
| J | 1-2 | 2-3 / 開放 | 既定 |
|---|---|---|---|
| J45 VDDI1_5 (Bank1+5) | 3.3V: RPi, mikroBUS | 2.5V: MIPI, Ethernet | 1-2 |
| J46 VDDAUX1 | 3.3V | 2.5V（J45 と揃える） | 1-2 |
| J47 電源元 | クローズ = USB-C J4 | 開放 = DC J7 | クローズ |
| J49 7 セグ | 3.3V | 5V | 2-3 |

## Bank 電圧
| Bank | 0 HSIO | 1 GPIO | 2 MSS IO | 3 JTAG | 4 MSS SD | 5 MSS SGMII | 6 MSS DDR |
|---|---|---|---|---|---|---|---|
| V | 1.8 | 2.5/3.3 | 3.3 | 3.3 | 1.8 | 2.5/3.3 | 1.2 |

## 主要ピン
| 信号 | Pin | Bank |
|---|---|---|
| CLK 50 MHz | R18 | 0 |
| MSS REFCLK 125 MHz P/N | P5 / P4 | 5 |
| DEVRST_n | E8 | — |
| LED1-7 | T18 V17 U20 U21 AA18 V16 U15 | 0 |
| LED8 | E1 | 2 (MSS) |
| SW1 / SW2 | T19 / U18 | 0 |
| SW8[1:8] | U17 Y16 R17 AA15 AA20 Y20 V21 AA19 | 0 |
| J48 pin1-8 | Y15 T17 V15 Y19 AA17 W20 W16 V18 | 0 |
| mikroBUS J5 AN RST CS SCK MISO MOSI | G12 B16 C16 E17 A18 E11 | 1 |
| mikroBUS J6 PWM INT RX TX SCL SDA | D13 G17 E14 E15 D11 D12 | 1 |
| MIPI D0 N/P, D1 N/P, CLK N/P | B11/C11, A10/B10, B12/A12 | 1 |
| MIPI I2C SCL/SDA, GPIO | T21/T20, Y14 Y17 | 1 |
| RPi SPI MOSI MISO SCLK CE0 CE1 | A4 B4 D6 D5 B2 | 2 (MSS) |
| RPi ID_SD / ID_SC | D1 / C2 | 2 (MSS) |
| RPi TXD/RXD (pin 8/10) | A20 / B21 | 1 |
| RPi SDA1/SCL1 (pin 3/5) | E18 / F18 | 1 |

## 電源
- LED: 5P0V / VDD（1V コア）/ 1P8V
- テストポイント: TP_VDD、TP_1P8V、GND1/4/5
- 測定: 5P0V_IN C175 5V、5P0V C165 5V、3P3V C123、VDD C144 1.0V±3%、VDDA C134 1.0V、2P5V C153、1P8V C227、1P2V_DDR4 C284、VREF C250 0.6V、3P3V_FT C277、1P8V_FT C280
- レギュレータ: VDD・3P3V = MIC22705（7A）、1P2V_DDR4・1P8V = MIC22405（4A）、2P5V = MIC69502WR（5A）、VTT = MIC5166（3A）

## どちら側で設定するか
- **MSS Configurator**: DDR4、microSD、Ethernet（SGMII）、UART_B/C、RPi SPI・ID I2C、LED8
- **RTL + PDC**: LED1-7、SW、DIP、7 セグ、UART_D、mikroBUS、RPi の 21 本、MIPI、50 MHz

## 書き込み
FlashPro Express → New Job Project → `.job` → OK → Programmer 確認（Refresh/Rescan）→ RUN → RUN PASSED

## 部品
DDR4 MT40A512M16TB-062E:R 1 GB / PHY VSC8221 / USB-UART FT4232HL / microSD GCT MEM2075 / RPi Würth 61204021621 / Reset MCP121T-315E/TT / 基板 4.15×3.3 in
