# 回路図: コネクタのピン表（RPi J10 / mikroBUS J5・J6 / MIPI J11 / microSD J17 / 7 セグ J48 / JTAG / UART / Ethernet）

> 出典: PolarFire SoC Discovery Kit Schematics Rev2（DVP-100-000578-002, 2023-09-04, 14 sheets, `C:/_Devs/PolarFire/doc/board/PolarFire_SoC_Discovery_Kit_Rev2_Schematics.pdf`）。ネット名・部品番号・ボール番号は回路図の表記のまま。読み取れなかった箇所は **要確認**。

## 4. コネクタのピン表

### 4.1 Raspberry Pi 40pin（J10、Sheet 8 / Sheet 7）

FPGA 側はレベル変換なしの直結。3/5 ピンは R169/R170 10K で 3P3V にプルアップ。24 ピンは R172 10K、26 ピンは R171 10K で 3P3V にプルアップ。28 ピンは R173（プルアップ）と R687（プルダウン）、27 ピンは R686（プルダウン）が付くが、いずれも 10K_NL（未実装）。

| Pin | ネット名 | FPGA | Bank | | Pin | ネット名 | FPGA | Bank |
|---|---|---|---|---|---|---|---|---|
| 1 | 3P3V | — | | | 2 | 5P0V | — | |
| 3 | RPi_GPIO2_SDA | E18 | 1 | | 4 | 5P0V | — | |
| 5 | RPi_GPIO3_SCL | F18 | 1 | | 6 | GND | — | |
| 7 | RPi_GPIO4_GCLK | E12 | 1 | | 8 | RPi_GPIO14_TXD0（RPI_TXD） | A20 | 1 |
| 9 | GND | — | | | 10 | RPi_GPIO15_RXD0（RPI_RXD） | B21 | 1 |
| 11 | RPi_GPIO17_GEN0 | G18 | 1 | | 12 | RPi_GPIO18_GEN1 | E10 | 1 |
| 13 | RPi_GPIO27_GEN2 | F21 | 1 | | 14 | GND | — | |
| 15 | RPi_GPIO22_GEN3 | F20 | 1 | | 16 | RPi_GPIO23_GEN4 | E21 | 1 |
| 17 | 3P3V | — | | | 18 | RPi_GPIO24_GEN5 | F19 | 1 |
| 19 | RPi_GPIO10_SPI_MOSI | A4 | 2（MSS） | | 20 | GND | — | |
| 21 | RPi_GPIO9_SPI_MISO | B4 | 2（MSS） | | 22 | RPi_GPIO25_GEN6 | A19 | 1 |
| 23 | RPi_GPIO11_SPI_SCLK | D6 | 2（MSS） | | 24 | RPi_GPIO8_SPI_CE0_N | D5 | 2（MSS） |
| 25 | GND | — | | | 26 | RPI_GPIO7 | B2 | 2（MSS） |
| 27 | RPi_ID_SD | D1 | 2（MSS） | | 28 | RPi_ID_SC | C2 | 2（MSS） |
| 29 | RPi_GPIO5 | D17 | 1 | | 30 | GND | — | |
| 31 | RPi_GPIO6 | D18 | 1 | | 32 | RPi_GPIO12 | B20 | 1 |
| 33 | RPi_GPIO13 | A15 | 1 | | 34 | GND | — | |
| 35 | RPi_GPIO19 | B19 | 1 | | 36 | RPi_GPIO16 | B15 | 1 |
| 37 | RPi_GPIO26 | B17 | 1 | | 38 | RPi_GPIO20 | B14 | 1 |
| 39 | GND | — | | | 40 | RPi_GPIO21 | E13 | 1 |

Bank 2 側の MSSIO 番号と、回路図に注記された MSS 機能:

| FPGA | MSSIO | 回路図の機能注記 | ネット |
|---|---|---|---|
| D6 | MSSIO30B2 | SPI_1_CLK (B) | RPI_GPIO11_SPI_SCLK |
| D5 | MSSIO31B2 | SPI_1_SS0 (B) | RPi_GPIO8_SPI_CE0_N |
| A4 | MSSIO32B2 | SPI_1_DO (B) | RPI_GPIO10_SPI_MOSI |
| B4 | MSSIO33B2 | SPI_1_DI (B) | RPI_GPIO9_SPI_MISO |
| B2 | MSSIO34B2 | （注記なし） | RPI_GPIO7 |
| C2 | MSSIO35B2 | I2C_0_SCL (B) | RPi_ID_SC |
| D1 | MSSIO36B2 | I2C_0_SDA (B) | RPi_ID_SD |

### 4.2 mikroBUS（J5 / J6、Sheet 8）

すべて Bank 1（VDDI1_5）の直結。ソケット側のピン名は回路図シンボルの表記（PIC のピン名）。

| J5 Pin | シンボル名 | ネット名 | FPGA | 備考 |
|---|---|---|---|---|
| 1 | AN/RA2 | MBUS_AN | G12 | |
| 2 | RST/RE1 | MBUS_RST | B16 | R139 10K で 3P3V にプルアップ |
| 3 | CS/RE0 | MBUS_SPI_CS# | C16 | R145 10K で 3P3V にプルアップ |
| 4 | SCK/RC3 | MBUS_SPI_CLK | E17 | |
| 5 | MISO/RC4 | MBUS_SPI_MISO | A18 | |
| 6 | MOSI/RC5 | MBUS_SPI_MOSI | E11 | |
| 7 | +3.3V | 3P3V | — | |
| 8 | GND | GND | — | |

| J6 Pin | シンボル名 | ネット名 | FPGA | 備考 |
|---|---|---|---|---|
| 1 | RC0/PWM | MBUS_PWM | D13 | |
| 2 | RB0/INT | MBUS_INT | G17 | R146 10K_NL（未実装） |
| 3 | RC7/RX | MBUS_UART_RXD | E14 | |
| 4 | RC6/TX | MBUS_UART_TXD | E15 | |
| 5 | RC3/SCL | MBUS_I2CSCL | D11 | R147 10K で 3P3V にプルアップ |
| 6 | RC4/SDA | MBUS_I2CSDA | D12 | R148 10K で 3P3V にプルアップ |
| 7 | +5V | 5P0V | — | |
| 8 | GND | GND | — | |

MBUS_UART_RXD / MBUS_UART_TXD の信号の向き（FPGA から見て入力か出力か）は、回路図のオフページ記号からは確定できない（要確認）。ネット名とシンボル名はホスト側 RX/TX の表記になっている。

### 4.3 MIPI カメラ（J11、Sheet 8）

15pin FPC。差動ペアは Bank 1 の GPIO に直結。制御線は Bank 0（1.8V）からレベル変換して 3.3V で出している。

| J11 Pin | ネット名（コネクタ側） | 経由 | FPGA 側ネット | FPGA |
|---|---|---|---|---|
| 1 | GND | | | |
| 2 | GPIO_MIPI_RX_N0 | 直結 | GPIO_MIPI_RX_N0 | B11 |
| 3 | GPIO_MIPI_RX_P0 | 直結 | GPIO_MIPI_RX_P0 | C11 |
| 4 | GND | | | |
| 5 | GPIO_MIPI_RX_N1 | 直結 | GPIO_MIPI_RX_N1 | A10 |
| 6 | GPIO_MIPI_RX_P1 | 直結 | GPIO_MIPI_RX_P1 | B10 |
| 7 | GND | | | |
| 8 | GPIO_MIPI_RX_CKN | 直結 | GPIO_MIPI_RX_CKN | B12 |
| 9 | GPIO_MIPI_RX_CKP | 直結 | GPIO_MIPI_RX_CKP | A12 |
| 10 | GND | | | |
| 11 | CAM_EN | U29 TXB0102 | CAM_EN_B0 | Y14（Bank 0） |
| 12 | CAM_GPIO | U29 TXB0102 | CAM_GPIO_B0 | Y17（Bank 0） |
| 13 | CAM_I2C_SCL | U79 TXS0102 | CAM_I2C_SCL_B0 | T21（Bank 0） |
| 14 | CAM_I2C_SDA | U79 TXS0102 | CAM_I2C_SDA_B0 | T20（Bank 0） |
| 15 | 3P3V | | | |

- 3.3V 側 I2C のプルアップ R107/R109、1.8V 側の R676/R677 はすべて 10K_NL（未実装）
- U79/U29 の OE は R678/R679 4.7K で 1P8V にプルアップ（常時有効）
- 回路図注記: "Cable to cable mating, Raspberry Camera module V2 board/Camera HQ / Cable Part Supplied with Camera Module - Opposite Sided Contacts"

### 4.4 microSD（J17、Sheet 4）

FPGA（Bank 4、1.8V）と J17 の間に U85 NVT4857UKAZ が入る。U85 の VCCA=1P8V、VSD=3P3V、VCCB=SD_VCCB（C1126 2.2uF のみ。U85 内部で生成すると思われるが要確認）。

| J17 Pin | J17 名 | カード側ネット | U85（B 側 / A 側） | FPGA 側ネット | FPGA | MSSIO |
|---|---|---|---|---|---|---|
| 1 | DAT2 | SD_DAT2 | A4 / A1 | SD_LT_DATA2 | M5 | MSSIO4B4 |
| 2 | CD/DAT3 | SD_DAT3 | B4 / B1 | SD_LT_DATA3 | L5 | MSSIO5B4 |
| 3 | CMD | SD_CMD | C4 / C1 | SD_LT_CMD | M4 | MSSIO1B4 |
| 4 | VDD | 3P3V | — | — | — | — |
| 5 | CLK | SD_CLK | D3 / D2 | SD_LT_CLK | L2 | MSSIO0B4 |
| 6 | VSS1 | GND | — | — | — | — |
| 7 | DAT0 | SD_DAT0 | D4 / D1 | SD_LT_DATA0 | L3 | MSSIO2B4 |
| 8 | DAT1 | SD_DAT1 | E4 / E1 | SD_LT_DATA1 | K1 | MSSIO3B4 |
| 9 | Card Det | SD_CD# | U85 CD（B2）にも接続 | SD_CD# | K2 | MSSIO6B4 |

U85 の制御ピン:

| U85 ピン | ネット | FPGA | MSSIO |
|---|---|---|---|
| SEL（E3） | SD_LT_SEL | K7 | MSSIO9B4 |
| CLK_FB（E2） | SD_CLK_FB ← R695 0R ← SD_LT_POW | L7 | MSSIO8B4 |

- SD_LT_POW は名前に反して、R695 0R を介して U85 の CLK_FB につながっている。回路図の描画どおりに記載した
- H2（MSSIO7B4）、H1、K4、K5、J1（MSSIO10〜13B4）は未接続
- SEL の論理（どちらでカード側が 1.8V になるか）は NVT4857 のデータシートで要確認

### 4.5 7 セグメント表示ボード（J48 / J49、Sheet 7）

Bank 0（1.8V）の B0_SEG_DSP_IOx を U6 TXB0108PWR（VCCA=1P8V、VCCB=VCCB、OE は R354 4.7K で 1P8V にプルアップ）で VCCB レベルに変換し、SEG_DSP_IOx として J48 に出している。

| J48 Pin | ネット | FPGA 側ネット | FPGA |
|---|---|---|---|
| 1 | SEG_DSP_IO8 | B0_SEG_DSP_IO8 | Y15 |
| 2 | SEG_DSP_IO4 | B0_SEG_DSP_IO4 | T17 |
| 3 | SEG_DSP_IO7 | B0_SEG_DSP_IO7 | V15 |
| 4 | SEG_DSP_IO3 | B0_SEG_DSP_IO3 | Y19 |
| 5 | SEG_DSP_IO6 | B0_SEG_DSP_IO6 | AA17 |
| 6 | SEG_DSP_IO2 | B0_SEG_DSP_IO2 | W20 |
| 7 | SEG_DSP_IO5 | B0_SEG_DSP_IO5 | W16 |
| 8 | SEG_DSP_IO1 | B0_SEG_DSP_IO1 | V18 |
| 9 | VCCB | — | — |
| 10 | GND | — | — |

VCCB の選択は J49（7 章）で行う。

### 4.6 JTAG（Sheet 9 / Sheet 10）

外付け JTAG ヘッダは回路図に無い。FPGA の JTAG は FT4232H のチャネル A（MPSSE）に直結され、これがオンボード FlashPro5（eFP5）になる。JT_TCK / JT_TDI / JT_TDO / JT_TMS / JT_TRST# はテストポイント（TP_SMD_C_30MIL）として出ている。

| 信号 | FPGA | Bank 3 ピン名 | FT4232H | 付加部品 |
|---|---|---|---|---|
| PF_JTAG_TCK | A8 | TCK | AD0（16） | R134 1K で GND |
| PF_JTAG_TDI | A9 | TDI | AD1（17） | |
| PF_JTAG_TDO | A7 | TDO | AD2（18） | R130 22R/1% 直列 |
| PF_JTAG_TMS | B7 | TMS | AD3（19） | |
| PF_JTAG_nTRST | B9 | TRSTB | AD4（21） | R129 1K で GND |

Bank 3 のその他のピン:

| FPGA | ピン名 | 接続 |
|---|---|---|
| E8 | DEVRST_N | DEVRST_N（8 章） |
| G9 | FF_EXIT_N | R128 4.7K で 3P3V にプルアップ |
| E9 | IO_CFG_INTF | R131 10K で GND |
| G10 | SPI_EN | R133 10K で GND |
| C6 | SCK | R127 10K で GND |
| D8 | SDI | R112 10K で 3P3V にプルアップ |
| A5 | SDO | 未接続 |
| B6 | SS | R694 10K で GND |

### 4.7 UART（Sheet 10 / Sheet 7）

UART 用の専用ヘッダは無い。FT4232H（U16、VCCIO=3P3V_FT）の 4 チャネルを次のように使っている。

| FT4232H ch | FT4232H ピン | ネット | 経由 | FPGA | Bank | 回路図の MSS 機能注記 |
|---|---|---|---|---|---|---|
| A | AD0〜AD4 | PF_JTAG_* | 直結 | A8/A9/A7/B7/B9 | 3 | JTAG（eFP5） |
| B | BD0 TXD（26） | FTDI_UART_B_TXD | 直結 | F2（MSSIO19B2） | 2 | MMUART_1_RXD |
| B | BD1 RXD（27） | FTDI_UART_B_RXD | 直結 | F4（MSSIO20B2） | 2 | MMUART_1_TXD |
| C | CD0 TXD（38） | FTDI_UART_C_TXD | 直結 | G2（MSSIO15B2） | 2 | MMUART_4_RXD |
| C | CD1 RXD（39） | FTDI_UART_C_RXD | 直結 | H4（MSSIO16B2） | 2 | MMUART_4_TXD |
| D | DD0 TXD（48） | FTDI_UARTD_TXD → FTDI_UART_D_TXD_B0 | U3 TXB0102（3.3V ↔ 1.8V） | W21 | 0 | ファブリック |
| D | DD1 RXD（52） | FTDI_UARTD_RXD → FTDI_UART_D_RXD_B0 | U3 TXB0102 | Y21 | 0 | ファブリック |

- U3 の OE は R32 4.7K で 1P8V にプルアップ
- FT4232H の RESET# は FT4232H_RST（R162 10K で 3P3V_FT にプルアップ、C279 1uF）
- 埋め込みソフト UG（`mpfs-discovery-kit-embedded-software-user-guide.md`）では「1 番目の COM ポート = MMUART1（HSS）、2 番目 = MMUART4（U-Boot/Linux）」としており、回路図の注記と一致する
- ほかの UART: RPi J10 の 8/10 ピン（A20/B21、Bank 1）、mikroBUS J6 の 3/4 ピン（E14/E15、Bank 1）

### 4.8 Ethernet（Sheet 5 / Sheet 6）

RJ45 J3 → U4 VSC8221 → **MSS SGMII（U1-6、Bank 5）**。ファブリックのトランシーバ SERDES-0（U1-11）はすべて未接続。

| 区間 | ネット | FPGA / PHY ピン | 備考 |
|---|---|---|---|
| MAC TX+ | VSC_TDP | FPGA N7 MSS_SGMII_TXP0 → PHY K9 TDP | C67 0.01uF で AC 結合 |
| MAC TX- | VSC_TDN | FPGA N5 MSS_SGMII_TXN0 → PHY K10 TDN | C68 0.01uF |
| MAC RX+ | VSC_RDP | PHY K8 RDP → FPGA M2 MSS_SGMII_RXP0 | C69 0.01uF。R82 100R_NL（未実装） |
| MAC RX- | VSC_RDN | PHY K7 RDN → FPGA M1 MSS_SGMII_RXN0 | C70 0.01uF |
| REFCLK | MSS_REFCLK_IN_P / _N | FPGA P5 / P4 ← X1 125MHz | R85/R86 0R。R83 100R_NL |
| 未使用 | MSS_SGMII_RXN1/RXP1/TXN1/TXP1 | R2 / P2 / P1 / N1 | 未接続 |

PHY 制御線（Bank 1）:

| ネット | FPGA | PHY ピン | 備考 |
|---|---|---|---|
| VSC_MDC | D15 | J10 MODDEF1/MDC | R42 4.7K で VDDI1_5 にプルアップ |
| VSC_MDIO | D16 | J9 MODDEF2/MDIO | R43 4.7K |
| VSC_MDINTB | A13 | H9 MDINTB | R44 4.7K |
| VSC_RESET# | A14 | A2 RESETB | R50 10K で VDDI1_5 にプルアップ。C61 NL |
| VSC_TXDIS_SRESET# | E20 | G8 TXDIS/SRESETB | R45 10K でプルアップ |
| VSC_CLKOUT_125Mhz | D21 | A1 MODDEF0_CLKOUT | GPIO11PB1/CLKIN_S_9 |
| VSC_RXLOS_SIGDET0 | A17 | B1 RXLOS/SIGDET0 | |

- PHY アドレス: 01011（回路図見出し "PHY address[4:0]: 01011"）。CMODE0〜3 は R54 5.9K / R55 4.02K / R56 22.6K / R689 0R で VDD33A に接続
- LED: LED0（B3）→ LD1、LED1（A3）→ LD2、LED2（A4）→ LD3。いずれも R192/R73/R78 1K で VDD33A へ。注記は "LED[2:0] = {Link/Activity, Link/Activity, Fault}"
- J3 の内蔵 LED 端子（LEDG_C/LEDG_A/LEDY_C/LEDY_A）は未接続
- 電源: VDDIOC_TRL / VDDIO / VDDIOMICRO = VDDI1_5、VDD33A_1/_2 = 3P3V（FB2 経由で VDD33A）、VDD12 = 1P2V_PHY（内蔵レギュレータ）
- J3 の MDI: TRD1+/- = TXVPA/TXVNA … TRD4+/- = TXVPD/TXVND。COM_CT は C74 0.1uF/50V で GND、シールドは C76 1000pF/3KV
