# Chapter 7 (Appendix B): User Hardware Pinout（UG p.21-23）

## Core Idea
コネクタのピンと FPGA パッケージピンの対応表。PDC 制約や MSS の IO 割当はここから作る。信号名（GPIO 番号・SD の DAT など）は UG に無いので、**[推定]** の列は各コネクタの標準規格とシルクから補った（回路図で要確認）。

## Reference Tables

### Raspberry Pi 40pin（J10、Table 7-1）
| Pin | FPGA | [推定] RPi 名 | | Pin | FPGA | [推定] RPi 名 |
|---|---|---|---|---|---|---|
| 1 | 3.3V | | | 2 | 5V | |
| 3 | E18 | GPIO2 SDA1 | | 4 | 5V | |
| 5 | F18 | GPIO3 SCL1 | | 6 | GND | |
| 7 | E12 | GPIO4 | | 8 | A20 | GPIO14 TXD |
| 9 | GND | | | 10 | B21 | GPIO15 RXD |
| 11 | G18 | GPIO17 | | 12 | E10 | GPIO18 |
| 13 | F21 | GPIO27 | | 14 | GND | |
| 15 | F20 | GPIO22 | | 16 | E21 | GPIO23 |
| 17 | 3.3V | | | 18 | F19 | GPIO24 |
| 19 | **A4** | GPIO10 MOSI | | 20 | GND | |
| 21 | **B4** | GPIO9 MISO | | 22 | A19 | GPIO25 |
| 23 | **D6** | GPIO11 SCLK | | 24 | **D5** | GPIO8 CE0 |
| 25 | GND | | | 26 | **B2** | GPIO7 CE1 |
| 27 | **D1** | GPIO0 ID_SD | | 28 | **C2** | GPIO1 ID_SC |
| 29 | D17 | GPIO5 | | 30 | GND | |
| 31 | D18 | GPIO6 | | 32 | B20 | GPIO12 |
| 33 | A15 | GPIO13 | | 34 | GND | |
| 35 | B19 | GPIO19 | | 36 | B15 | GPIO16 |
| 37 | B17 | GPIO26 | | 38 | B14 | GPIO20 |
| 39 | GND | | | 40 | E13 | GPIO21 |

**太字の 7 本**（SPI + ID I2C）= MSS I/O Bank 2。残り 21 本 = GPIO Bank 1（VDDI1_5）。Ch3 の「SPI, I2C (7) → MSS、UART・I2C 1 組・その他 (21) → Bank 1」と本数が一致する。

### DIP SW8（Table 7-2）/ Push buttons（Table 7-8）/ LED（Table 7-3）
| SW8 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| FPGA | U17 | Y16 | R17 | AA15 | AA20 | Y20 | V21 | AA19 |

| LED | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| FPGA | T18 | V17 | U20 | U21 | AA18 | V16 | U15 | E1 (MSS) |

Push button: **Switch 1 = T19**, **Switch 2 = U18**（押下時の論理レベルは UG に記載なし）

### mikroBUS（J5 / J6、Table 7-4）
| J5 Pin | FPGA | [シルク] | | J6 Pin | FPGA | [シルク] |
|---|---|---|---|---|---|---|
| 1 | G12 | AN | | 1 | D13 | PWM |
| 2 | B16 | RST | | 2 | G17 | INT |
| 3 | C16 | CS | | 3 | E14 | RX |
| 4 | E17 | SCK | | 4 | E15 | TX |
| 5 | A18 | MISO | | 5 | D11 | SCL |
| 6 | E11 | MOSI | | 6 | D12 | SDA |
| 7 | 3.3V | +3.3V | | 7 | 5V | +5V |
| 8 | GND | GND | | 8 | GND | GND |

### MIPI RX（J11、Table 7-5）
| Pin | FPGA | [推定 RPi CSI-2 15pin] |
|---|---|---|
| 1 | GND | GND |
| 2 / 3 | B11 / C11 | D0_N / D0_P |
| 4 | GND | |
| 5 / 6 | A10 / B10 | D1_N / D1_P |
| 7 | GND | |
| 8 / 9 | B12 / A12 | CLK_N / CLK_P |
| 10 | GND | |
| 11 | Y14 | CAM_GPIO / CAM_EN |
| 12 | Y17 | CAM_GPIO / CAM_EN |
| 13 | T21 | I2C_SCL |
| 14 | T20 | I2C_SDA |
| 15 | 3.3V | 3.3V |

### microSD（J17、Table 7-6）。FPGA 側は電圧変換 IC の先
| Pin | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|
| FPGA | M5 | L5 | M4 | 3.3V | L2 | GND | L3 | K1 | K2 |
| [推定] | DAT2 | DAT3/CD | CMD | VDD | CLK | VSS | DAT0 | DAT1 | 検出 |

### 7 セグメント（J48、Table 7-7）
| Pin | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| FPGA | Y15 | T17 | V15 | Y19 | AA17 | W20 | W16 | V18 | VCCB | GND |
VCCB は J49 で 3.3V / 5V（既定 5V）。FPGA 側は Bank 0（1.8V）。

## Code Examples
<!-- UG 外。ピン表から作った PDC の例 -->
```tcl
# Push buttons / DIP (Bank 0, 1.8V)
set_io -port_name {PB[0]}  -pin_name T19 -fixed true -io_std LVCMOS18 -DIRECTION INPUT
set_io -port_name {PB[1]}  -pin_name U18 -fixed true -io_std LVCMOS18 -DIRECTION INPUT
set_io -port_name {DIP[0]} -pin_name U17 -fixed true -io_std LVCMOS18 -DIRECTION INPUT
# mikroBUS UART (Bank 1, J45/J46 = 3.3V profile)
set_io -port_name {MIKRO_TX} -pin_name E15 -fixed true -io_std LVCMOS33
set_io -port_name {MIKRO_RX} -pin_name E14 -fixed true -io_std LVCMOS33
```
- **What it demonstrates**: Bank 0 は LVCMOS18、Bank 1 は J45/J46 に合わせて LVCMOS33 または 2.5V 系。mikroBUS の RX/TX の向きは Click 側の表記なので、回路図で確認する

## Anti-patterns
- **RPi の SPI（A4/B4/D6/D5/B2）をファブリックの PDC に書く**: これらは MSS Bank 2。MSS Configurator で割り当てる
- **[推定] 列をそのまま確定情報として使う**: 必ず回路図（MPFS-DISCO-KIT Documentation）で照合する

## Key Takeaways
1. RPi 40pin は 7 本が MSS（SPI・ID I2C）、21 本が Bank 1
2. LED・SW・7 セグは Bank 0、1.8V
3. mikroBUS は J5（アナログ/SPI 側）と J6（UART/I2C/PWM 側）の 2 列
4. MIPI のレーンは 2 本（D0, D1）+ CLK、I2C は T21/T20

## Connects To
- **Ch2**: Bank 電圧・J45/J46/J49
- **Ch3**: MSS / ファブリックの振り分け
- **Ch4**: シルクの信号名
