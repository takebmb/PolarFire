# 回路図: シート一覧 / 電源 / Bank 電圧 / 主要部品 / ジャンパ / クロック・リセット

> 出典: PolarFire SoC Discovery Kit Schematics Rev2（DVP-100-000578-002, 2023-09-04, 14 sheets, `C:/_Devs/PolarFire/doc/board/PolarFire_SoC_Discovery_Kit_Rev2_Schematics.pdf`）。ネット名・部品番号・ボール番号は回路図の表記のまま。読み取れなかった箇所は **要確認**。

## 0. 出典

| 項目 | 内容 |
|---|---|
| タイトル | PolarFire SoC Discovery Kit（Part Number: MPFS-DISCO-KIT） |
| 図面番号 | DVP-100-000578-002 |
| Rev | 2.0 |
| 日付 | Monday, September 04, 2023 |
| ページ数 | 14 |
| ローカルパス | `C:\_Devs\PolarFire\doc\board\PolarFire_SoC_Discovery_Kit_Rev2_Schematics.pdf` |
| 照合に使った資料 | `C:\_Devs\PolarFire\doc\board\PolarFire_SoC_FPGA_Discovery_Kit_User_Guide.pdf`（DS50003630E） |

PDF はパスワード保護されているため、Read ツールでは開けない。`pdftoppm -r 300` で画像にしてから目視で読んだ。

### シート一覧

| Sheet | タイトル（原文） | 内容 |
|---|---|---|
| 1 | TITLE PAGE | 目次 |
| 2 | Block Diagram | 全体ブロック図 |
| 3 | MSS_DDR4X16_Connection | U1-5 Bank 6 MSS DDR と U2 DDR4、終端、デカップリング |
| 4 | BANK4 & uSD Card Connection | U1-7 Bank 4 MSSIO、U85 レベル変換、J17 microSD |
| 5 | VSC8221 MAC Connection | U4 VSC8221 の MDI/SGMII/電源、J3 RJ45、U1-6 MSS SGMII、X1 125MHz、U1-11 SERDES-0（未使用） |
| 6 | VSC8221 G5 Interface Connection | VSC8221 の MDIO/リセット/CMODE/JTAG、Y1 25MHz、U5 EEPROM |
| 7 | Bank0,1,2 Connection | U1-3 Bank 0、U1-2 Bank 1、U1-8 Bank 2 のピン割当、X3 50MHz、7 セグ（U6/J48/J49）、DIP SW8、J12 |
| 8 | Peripheral Connection | mikroBUS J5/J6、プッシュボタン SW1/SW2、RPi 40pin J10、MIPI J11、LED1〜8、カメラ系レベル変換 |
| 9 | Programming Connection | U1-1 Bank 3（JTAG/SC-SPI）、DEVRST_N、U19 リセット IC |
| 10 | FT4232H Connection | U16 FT4232H、U84 3.3V LDO、U82 EEPROM、U3 レベル変換、X2 12MHz |
| 11 | Power & GND Connection | U1-9/U1-10 電源ピン、各 VDDIx、J45/J46、U1-4 GND |
| 12 | Decoupling Capacitor Connection | FPGA 電源ピンのデカップリング |
| 13 | 5V Supply | J4 USB-C、J7 DC ジャック、J47、U10 保護スイッチ、D3 ESD |
| 14 | Power Regulator Connection | U7/U72/U12/U9/U8/U11 レギュレータ、電源 LED |

FPGA（U1 = MPFS095T-1FCSG325E）は回路図上で U1-1〜U1-11 の 11 ブロックに分かれている。

| ブロック | 中身 | Sheet |
|---|---|---|
| U1-1 | Bank 3 JTAG/SC-SPI | 9 |
| U1-2 | Bank 1（GPIO） | 7 |
| U1-3 | Bank 0（HSIO） | 7 |
| U1-4 | GROUND | 11 |
| U1-5 | Bank 6 MSS DDR | 3 |
| U1-6 | MSS SGMII（Bank 5） | 5 |
| U1-7 | MSSIO Bank 4 | 4 |
| U1-8 | MSSIO Bank 2 | 7 |
| U1-9 | POWER-1（VDD18/VDD25/VDDAUX/VDDIx） | 11 |
| U1-10 | POWER-2（VDD/VDDA/XCVR_VREF） | 11 |
| U1-11 | SERDES-0（XCVR_0） | 5 |

---

## 1. 電源ツリー

### 1.1 5V 入力（Sheet 13）

```
J4 USB-C (USB4105-GF-A-120) VBUS ── USB_C_5V_CON ── F2 3A (1812L300/24SLER) ── USB_C_5V ── J47 pin1
                                                                                        J47 pin2 ── 5P0V_IN
J7 DC ジャック (PJK_5A_PJ-047AH, "5V POWER JACK @ 4A") pin1 ─────────────────────────────────── 5P0V_IN
5P0V_IN ── U10 TPD1S514-1YZR (VBUS_CON → VBUS_SYS) ── 5P0V
```

- J47（HDR_1X2_100MIL、HTSW-102-07-G-S、"6A Capable header"、Default=1&2）
  - 1&2 を閉じる: USB-C J4 から給電する
  - 開放: J7 ヘッダから給電する（回路図注記 "J47 - OPEN, When board need to be powered via header J7"）
  - J47 を閉じたまま J7 にも電源をつなぐと、USB_C_5V と 5P0V_IN が直結になる。回路図に逆流防止は見当たらない
- U10 の EN（Active low）は R33 10K で GND に落としてあるので常時有効。VBUS_POWER（B1）はテストポイント VBUS_PWR に出ている
- J4 の CC1/CC2 は R125/R126 5.1K/1% でプルダウンされている（シンク）。R123/R124 は NL
- J4 の D+/D- は D3 USBLC6-2SC6（ESD 保護）を通って DP_FT4232H / DM_FT4232H になり、U16 FT4232H につながる。USB-C 1 本で電源と FTDI（JTAG + UART 3 本）をまかなう
- 推奨アダプタ（回路図注記）: Power Adapter part#:SDI24-5-U-P5、Adapter Cable part#:AC-C13NA

### 1.2 レギュレータ（Sheet 14、Sheet 10）

| レール名 | 部品 | 型番 | 入力 | 回路図見出しの定格 | 分圧・設定 | 主な供給先 |
|---|---|---|---|---|---|---|
| VDD（1.0V） | U7 | MIC22705YML-TR | 5P0V | "VDD 1.0V@7A" | R88 1.1K / R92 2.55K、Vout=0.7(1+R1/R2) | FPGA VDD（コア）。R224 0R/0.5W を通して VDDA にも供給 |
| 3P3V | U72 | MIC22705YML-TR | 5P0V | "3P3V@7A" | R668 1.5K / R664 402R | VDDI2、VDDI3、VDDAUX2、RPi/mikroBUS/MIPI の 3.3V、VDD33A（PHY）、U8 入力、U11 BIAS |
| 1P2V_DDR4 | U12 | MIC22405YML-TR | 5P0V | "1.2V for DDR4, VDDI6 @4A" | R115 1.1K / R116 1.54K | VDDI6、U2 DDR4 の VDD/VDDQ、U11 VIN |
| 1P8V | U9 | MIC22405YML-TR | 5P0V | "1.8V@4A" | R120 1.1K / R101 698R | VDDI0、VDDI4、VDD18、U85/U6/U79/U29/U3 の低電圧側、SW1/SW2/SW8 のプルアップ |
| 2P5V | U8 | MIC69502WR（LDO） | 3P3V | "2.5V for PLL @5A" | R95 10K / R97 2.49K、Vout=0.5(1+R1/R2) | VDD25、VDDA25、VDD_XCVR_CLK、VDDAUX4、U2 VPP、J45/J46 の 2.5V 側 |
| 0P6V_VTT_DDR4 / 0P6V_VREF_DDR4 | U11 | MIC5166YML-TR | VIN=1P2V_DDR4、BIAS=3P3V | "DDR4 VTT & VREF" | — | DDR4 アドレス/コマンド終端（39R）、VREFCA、MSS_DDR4_VREF_IN |
| 3P3V_FT | U84 | MCP1726T-3302E/SN（LDO） | 5P0V | "3.3V LDO @ 1A" | 固定 3.3V | U16 FT4232H（VCCIO など）、U82 EEPROM、U3 の B 側 |
| 1P8V_FT | U16 内蔵 | FT4232H VREGOUT | 3P3V_FT | — | — | FT4232H の VCORE |
| 1P2V_PHY | U4 内蔵 | VSC8221 REGOUT + L1 4.7uH | VDD33A | — | — | VSC8221 VDD12 / VDD12A（FB3 経由で VDD12A_PHY） |
| VDDI1_5 | J45 で選択 | — | 3P3V または 2P5V | — | — | VDDI1（Bank 1）、VDDI5（Bank 5）、VSC8221 VDDIO 系、X1 125MHz の VDD |
| VDDAUX1 | J46 で選択 | — | 3P3V または 2P5V | — | — | FPGA VDDAUX1 |

- 2P5V は 3P3V から作るので、3P3V が立ち上がった後に立ち上がる
- VDDA は VDD と同じ 1.0V（R224 0R）。R690 10K_NL は未実装

### 1.3 シーケンスと PG（Sheet 14）

- MIC22705/MIC22405 の 4 個（U7/U72/U12/U9）はすべて同じ構成:
  - EN/DLY には 5P0V へのプルアップ（R137/R673/R90/R104、いずれも 10K_NL＝未実装）と GND への 1000pF（C178/C1107/C248/C189）がある
  - PG は 47.5K（R94/R674/R117/R105）で 5P0V にプルアップされているだけで、ほかのレギュレータの EN にはつながっていない
  - つまり PG による連鎖シーケンスは組まれていない。立ち上がり順は各 EN/DLY の容量で決まる（4 個とも同じ値）。EN/DLY 内部の挙動はデータシートで要確認
- U8（2P5V）の EN: 3P3V を R91 20K / R93 20K で分圧し、C208 0.1uF を付けて入れている
- U11（VTT）の EN_VTT は R100 10K で 3P3V にプルアップ、PG_VTT は R113 1K で 3P3V にプルアップ。PG_VTT の行き先は回路図上に見当たらない
- 電源 LED（Sheet 14、すべて APHHS1005CGCK 緑）:
  - 5P0V: 直結、R685 698R
  - VDD: U33（MBT2222ADW1T1G）を VDD で R286 2K 経由でドライブ。LED は 3P3V から R684 432R
  - 1P8V: 同じく U33 を R288 2K 経由でドライブ。R683 432R
  - LED の部品番号（参照記号）は図中に "5P0V" / "VDD" / "1P8V" としか書かれていない（要確認）
- テストポイント: TP_VDD、TP_1P8V（HDR1X1_NL）、GND1/GND4/GND5（HDR1X1、61300111121）

---

## 2. FPGA I/O バンクと電圧（Sheet 11）

| Bank | 回路図ブロック | 電源ピン名 | 電源ネット | 電圧 | 電源ボール |
|---|---|---|---|---|---|
| Bank 0（HSIO） | U1-3 | VDDI0 | 1P8V | 1.8V 固定 | U16, V20, Y13, Y18 |
| Bank 1（GPIO） | U1-2 | VDDI1 | VDDI1_5 | J45 で 3.3V/2.5V | A11, B13, B18, B8, D10, D20, E16, G15 |
| Bank 2（MSSIO） | U1-8 | VDDI2 | 3P3V | 3.3V 固定 | B3, D2, H5 |
| Bank 3（JTAG/SC-SPI） | U1-1 | VDDI3 | 3P3V | 3.3V 固定 | E7, G8 |
| Bank 4（MSSIO, SD） | U1-7 | VDDI4 | 1P8V | 1.8V 固定 | J2, L4 |
| Bank 5（MSS SGMII） | U1-6 | VDDI5 | VDDI1_5 | J45 で 3.3V/2.5V（Bank 1 と共通） | M7, N2 |
| Bank 6（MSS DDR） | U1-5 | VDDI6 | 1P2V_DDR4 | 1.2V 固定 | R5, U11, V2, V6, Y1, Y4, Y8 |

その他の FPGA 電源ピン:

| ピン名 | ネット | ボール |
|---|---|---|
| VDD（1.0V） | VDD | J10, J12, K11, K13, K15, K9, L10, L12, M11, M13, M15, M9, N10, N12 |
| VDDA | VDDA（= VDD、R224 0R） | J20, L18, L20, N20 |
| VDD18 | 1P8V | P15, P7, R10, R12 |
| VDD25 | 2P5V | G7, H15, R15, R7 |
| VDDA25 | 2P5V | L17, L19 |
| VDD_XCVR_CLK | 2P5V | K17, M17 |
| VDDAUX1 | VDDAUX1（J46） | F17, G11, G13 |
| VDDAUX2 | 3P3V | F5, G4 |
| VDDAUX4 | 2P5V | J5, J7 |
| XCVR_VREF | XCVR_VREF（R81 10K で GND） | G20, H18 |

- 回路図注記（Sheet 5/7/11）: "VDDI1_5 and VDDAUX1 / 2.5V for MIPI and Ethernet PHY operation / 3.3V for RPI and MikroBus operation"
- PDC の I/O 規格の目安: Bank 0 は LVCMOS18、Bank 1 は J45 の設定に合わせて LVCMOS33 または LVCMOS25

---

## 3. 主要部品

| 参照記号 | 型番 | 機能 | Sheet |
|---|---|---|---|
| U1 | MPFS095T-1FCSG325E | PolarFire SoC FPGA | 3〜11 |
| U2 | MT40A512M16TB-062E:R（Micron） | DDR4 512M x16（8Gb = 1GB） | 3 |
| U4 | VSC8221XHH | 10/100/1000BASE-T Ethernet PHY（SGMII） | 5, 6 |
| J3 | SI-61001-F | RJ45（マグネティクス内蔵） | 5 |
| U5 | AT24CM01-SSHD-B | VSC8221 用 I2C EEPROM（アドレス 1010000） | 6 |
| U16 | FT4232HL-REEL（FT4232H） | USB → JTAG + UART x3 | 10 |
| U82 | 93LC56BT-I/OT | FT4232H 設定 EEPROM | 10 |
| U85 | NVT4857UKAZ | microSD 用レベル変換（1.8V ↔ カード側） | 4 |
| J17 | MEM2075-00-140-01-A（GCT） | microSD ソケット | 4 |
| U6 | TXB0108PWR（代替 NXB0108PW-Q100J） | 7 セグ信号のレベル変換（1.8V ↔ VCCB） | 7 |
| U3 | TXB0102DCUR（代替 TXB0102DCUT / NXB0102DCH） | FTDI UART_D のレベル変換（3.3V ↔ 1.8V） | 10 |
| U79 | TXS0102DCTR | カメラ I2C のレベル変換（1.8V ↔ 3.3V） | 8 |
| U29 | TXB0102DCUR（代替 TXB0102DCUT / NXB0102DCH） | CAM_EN / CAM_GPIO のレベル変換（1.8V ↔ 3.3V） | 8 |
| U14, U83, U86, U87 | MBT2222ADW1T1G | LED1〜8 ドライブ用デュアル NPN | 8 |
| U33 | MBT2222ADW1T1G | 電源 LED（VDD/1P8V）ドライブ | 14 |
| U19 | MCP121T-315E/TT | DEVRST_N 用電圧監視リセット IC | 9 |
| U10 | TPD1S514-1YZR | 5V 入力保護ロードスイッチ | 13 |
| D3 | USBLC6-2SC6（代替 SRL05-TP） | USB D+/D- ESD 保護 | 13 |
| SW8 | 1-1825058-9（DIP_SW_8） | 8 ビット DIP スイッチ | 7 |
| SW1, SW2 | PTS 647 SM38 SMTR2 LFS（SW_SPST_TS02） | プッシュボタン | 8 |
| LED1, 3, 5, 7 | LTST-C191KSKT（黄 YEL/2.1V/20mA） | ユーザー LED | 8 |
| LED2, 4, 6, 8 | LTST-C191KRKT（赤 RED/2V/20mA） | ユーザー LED | 8 |
| LD1〜LD3 | APHHS1005CGCK（緑） | VSC8221 LED0〜LED2 | 5 |
| J10 | 61304021121（Wurth、HDR/20X2/100MIL） | Raspberry Pi 40pin | 8 |
| J11 | SFW15R-1STE1LF（FPC_15x1_1P0MM_RA） | Raspberry Pi MIPI カメラ（15pin） | 8 |
| J5, J6 | PPTC081LFBN-RC | mikroBUS（8pin x2） | 8 |
| J48 | PH2RA-10-UA（代替 61301021021、HDR_5X2_RA_100MIL） | 7 セグ表示ボード接続 | 7 |
| J12 | 61300421121（HDR/2X2/100MIL_NL） | ライブプローブ（未実装） | 7 |

UG の RPi コネクタ型番は 61204021621 だが、回路図は 61304021121。どちらが実装品かは要確認。

### クロック源

| 参照記号 | 型番 | 周波数・仕様 | 電源 | 行き先 |
|---|---|---|---|---|
| X1 | DSC1123BL5-125.0000（代替 SIT9120AC-2C3-XXE125.000000） | 125MHz 差動、2.25V-3.63V | VDDI1_5 | MSS_REFCLK_IN_P（P5）/ MSS_REFCLK_IN_N（P4） |
| X3 | DSC1001DL5-050.0000（代替 ECS-2520S18-500-FN-TR） | 50MHz、1.8V-3.3V、10ppm | 1P8V | R62 0R → 50Mhz_HSIO → R18（Bank 0） |
| Y1 | FL2500213Z | 25MHz 水晶 / 20pF（C65/C66 33pF） | — | VSC8221 XTAL1/XTAL2 |
| X2 | ECS-120-12-30-JGN-TR | 12MHz 水晶 / 12pF（C285/C296 27pF） | — | FT4232H OSCI/OSCO |
| —（U4 出力） | VSC8221 CLKOUT | 125MHz | — | VSC_CLKOUT_125Mhz → D21（Bank 1） |

---

## 7. ジャンパ

| ジャンパ | 部品 | 回路上の接続 | 設定と意味 | 回路図の既定値 |
|---|---|---|---|---|
| J45 | HDR/1X3/100MIL（61300311121） | pin1 = 3P3V、pin2 = VDDI1_5、pin3 = 2P5V | 1-2: VDDI1_5 = 3.3V（RPi / mikroBUS）。2-3: 2.5V（MIPI / Ethernet） | Default=1&2 |
| J46 | HDR/1X3/100MIL（61300311121） | pin1 = 3P3V、pin2 = VDDAUX1、pin3 = 2P5V | 1-2: VDDAUX1 = 3.3V。2-3: 2.5V | Default=1&2 |
| J47 | HDR_1X2_100MIL（HTSW-102-07-G-S）、シャント SNT1（SNT-100-BK-T） | pin1 = USB_C_5V、pin2 = 5P0V_IN | 閉: USB-C J4 から給電。開: J7 ジャックから給電 | Default=1&2 |
| J49 | HDR/1X3/100MIL（61300311121） | pin1 = 3P3V、pin2 = VCCB、pin3 = 5P0V | 1-2: 7 セグ側 3.3V。2-3: 5V | Default=2&3（5V） |
| J12 | HDR/2X2/100MIL_NL（61300421121、未実装） | pin1 = C20（LPRB_B）、pin2 = C21（LPRB_A）、pin3/4 = GND | ライブプローブ出力。回路図注記 "Nets from pins C20 & C21 should be routed differentially." | — |

- J45 を 2-3（2.5V）にしたときは J46 も 2-3 にする（UG Table 2-1 の注意書き。回路図では J45 と J46 は独立）
- J45 は Bank 1 と Bank 5 の両方の VDDI を切り替える。Bank 1 の RPi / mikroBUS と Bank 5 の Ethernet は同時に推奨電圧にできない
- J49 の既定値: 回路図と UG Table 2-1 は 2&3（5V）。`mpfs-discovery-kit-embedded-software-user-guide.md` の表だけ "1 & 2" になっている（要確認。実機で確認すること）

---

## 8. クロックとリセット

### 8.1 クロック

| 用途 | 発振源 | ネット | FPGA ピン |
|---|---|---|---|
| ファブリック用システムクロック 50MHz | X3 DSC1001DL5-050.0000（1P8V 給電） | 50Mhz_HSIO | R18（HSIO56PB0/CLKIN_N_8/CCC_NE_CLKIN_N_8） |
| MSS 基準クロック 125MHz（差動） | X1 DSC1123BL5-125.0000（VDDI1_5 給電、OE は R84 1K で VDDI1_5） | MSS_REFCLK_IN_P / _N | P5 / P4 |
| PHY からの 125MHz | U4 VSC8221 CLKOUT | VSC_CLKOUT_125Mhz | D21（GPIO11PB1/CLKIN_S_9） |
| PHY 用 25MHz | Y1 FL2500213Z | — | （PHY XTAL1/XTAL2） |
| FT4232H 用 12MHz | X2 ECS-120-12-30-JGN-TR | — | （FT4232H OSCI/OSCO） |

- X3 の STDBY（pin1）は未接続
- XCVR_0A_REFCLK（J18/J17）と XCVR_0B_REFCLK（N17/N18）は未接続。ファブリック SERDES 用の基準クロックは載っていない
- X1 は VDDI1_5 から給電されるので、J45 の設定によって 3.3V または 2.5V で動く（X1 の仕様は 2.25V-3.63V）

### 8.2 リセット

| リセット | ネット | 回路 | 行き先 |
|---|---|---|---|
| FPGA デバイスリセット | DEVRST_N | U19 MCP121T-315E/TT の RST → R132 0R。R691 1K で 3P3V にプルアップ、C300 1uF | E8（Bank 3 DEVRST_N） |
| DDR4 リセット | MSS_DDR4_RAM_RST_N | R28 10K で GND | FPGA T3 → U2 P1 |
| PHY リセット | VSC_RESET# | R50 10K で VDDI1_5 にプルアップ | FPGA A14 → U4 A2 |
| PHY ソフトリセット | VSC_TXDIS_SRESET# | R45 10K でプルアップ | FPGA E20 → U4 G8 |
| FT4232H リセット | FT4232H_RST | R162 10K で 3P3V_FT にプルアップ、C279 1uF | U16 RESET# |

- DEVRST_N を手で押すリセットボタンは回路図に無い。UG の "debug Reset" が何を指すかは要確認
