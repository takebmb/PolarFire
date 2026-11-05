# 回路図: UG 未記載・推定項目の照合結果 / 要確認項目

> 出典: PolarFire SoC Discovery Kit Schematics Rev2（DVP-100-000578-002, 2023-09-04, 14 sheets, `C:/_Devs/PolarFire/doc/PolarFire_SoC_Discovery_Kit_Rev2_Schematics.pdf`）。ネット名・部品番号・ボール番号は回路図の表記のまま。読み取れなかった箇所は **要確認**。

## 9. UG 未記載・推定項目の回路図での確認結果

skill 本体（SKILL.md、`chapters/ch06-hardware-pinout.md`）で **[推定]** または「UG に記載なし」とされている項目を回路図と照合した結果。

### 9.1 [推定] の信号名

**RPi 40pin（ch06 Table 7-1 の [推定] 列）: 確認済み（根拠: Sheet 8 J10、Sheet 7 Bank 1 / Bank 2）。**
- FPGA ボール番号は 28 本すべて UG と一致した
- [推定] の GPIO 番号もすべて回路図のネット名と一致した。正確なネット名は 4.1 節の表を参照（例: 11 ピンは `RPi_GPIO17_GEN0`、8 ピンは `RPi_GPIO14_TXD0`）
- 差分は 3 点:
  - 26 ピン（B2）: [推定] は "GPIO7 CE1"。回路図のネット名は `RPI_GPIO7` で、MSSIO34B2 に SPI の機能注記は付いていない。SPI_1 の SS1 として使えるかは MSS の IO 割当で要確認
  - 27/28 ピン: [推定] は "GPIO0 ID_SD" / "GPIO1 ID_SC"。回路図のネット名は `RPi_ID_SD` / `RPi_ID_SC` で、それぞれ MSS I2C_0_SDA（D1）/ I2C_0_SCL（C2）の注記が付いている
  - 3/5 ピン: 回路図のネット名は `RPi_GPIO2_SDA` / `RPi_GPIO3_SCL`。MSS ではなく Bank 1 のファブリック（E18/F18）につながっている。10K プルアップ付き

**MIPI J11（ch06 Table 7-5 の [推定 RPi CSI-2 15pin] 列）: 確認済み（根拠: Sheet 8 J11）。**
- 2/3 = `GPIO_MIPI_RX_N0`/`_P0`、5/6 = `_N1`/`_P1`、8/9 = `_CKN`/`_CKP`、1/4/7/10 = GND、15 = 3P3V。[推定] と一致する
- ch06 で "CAM_GPIO / CAM_EN" と未確定だった 11/12 ピンは確定した: **11 = CAM_EN（Y14 の `CAM_EN_B0`）、12 = CAM_GPIO（Y17 の `CAM_GPIO_B0`）**
- 11〜14 ピンは U29 / U79 で 1.8V ↔ 3.3V に変換されている（FPGA 側は Bank 0）

**microSD J17（ch06 Table 7-6 の [推定] 行）: 確認済み（根拠: Sheet 4 J17 / U85）。**
- 1 = DAT2、2 = CD/DAT3、3 = CMD、4 = VDD（3P3V）、5 = CLK、6 = VSS、7 = DAT0、8 = DAT1、9 = Card Det（`SD_CD#`）。[推定] と一致する
- FPGA 側のネット名は `SD_LT_*`（M5 = SD_LT_DATA2 など）。U85 を経由するので、FPGA ピンからカードへの直結ではない
- UG の表に無い制御線が 2 本ある: `SD_LT_SEL`（K7）、`SD_LT_POW`（L7。U85 の CLK_FB につながる）

### 9.2 プッシュボタン SW1/SW2 の押下時論理

**確認済み（根拠: Sheet 8 "Push Button" の R150/R149、SW1/SW2）。** 10K で 1P8V にプルアップされており、押すと GND に落ちる。**Active Low**（押下で '0'）。499R + 0.1uF の RC フィルタ付き。SWITCH1 は T19、SWITCH2 は U18。

参考: DIP SW8 も ON = '0'（Sheet 7 の表）。LED は Active High（Sheet 8 の NPN ドライブ）。

### 9.3 FT4232H の各チャネルの接続先

**確認済み（根拠: Sheet 10 U16、Sheet 7 MSSIO-B2 の機能注記）。**

| ch | 接続先 |
|---|---|
| A | FPGA JTAG（Bank 3: TCK A8 / TDI A9 / TDO A7 / TMS B7 / TRSTB B9）。eFP5 |
| B | MSS Bank 2: F2 = MMUART_1_RXD、F4 = MMUART_1_TXD（**MMUART1**） |
| C | MSS Bank 2: G2 = MMUART_4_RXD、H4 = MMUART_4_TXD（**MMUART4**） |
| D | U3 TXB0102 でレベル変換して、ファブリック Bank 0: W21 = FTDI_UART_D_TXD_B0（FPGA 入力）、Y21 = FTDI_UART_D_RXD_B0（FPGA 出力） |

MMUART 番号は回路図に小さく注記された MSS 機能名による。実際の割当は MSS Configurator の IO 設定に従う。

### 9.4 Ethernet は VDDI1_5 = 2.5V が必要か

**部分的に確認済み（根拠: Sheet 11 VDDI5 = VDDI1_5、Sheet 5 の VSC8221 VDDIO 系 = VDDI1_5、Sheet 5/7/11 の注記）。**
- Ethernet は Bank 5 MSS SGMII（U1-6: TX N7/N5、RX M2/M1、REFCLK P5/P4）につながる。ファブリックの SERDES-0 は未使用
- 次の 3 つがすべて VDDI1_5（J45 で選択）から給電されている:
  - Bank 5（VDDI5、M7/N2）
  - VSC8221 の VDDIO / VDDIOC_TRL / VDDIOMICRO
  - 125MHz 発振器 X1
- PHY の管理線（MDC/MDIO/RESET など）は Bank 1 にあり、これも VDDI1_5
- 回路図の注記は "2.5V for MIPI and Ethernet PHY operation / 3.3V for RPI and MikroBus operation" と明記している。**設計意図としては、Ethernet を使うときは J45 と J46 を 2-3（2.5V）にする**
- 既定値の 3.3V のままで SGMII リンクが動くかどうかは、**回路図では判断できない**（MSS SGMII の I/O 規格と VSC8221 VDDIO の許容範囲はデータシートで要確認）

### 9.5 ライブプローブ J12 の接続先 Bank

**確認済み（根拠: Sheet 7 Bank 1 の C20 / C21、J12）。** **Bank 1**（GPIO4NB1/LPRB_B = C20 → J12 pin1、GPIO4PB1/LPRB_A = C21 → J12 pin2、pin3/4 = GND）。
- ネット名は `GPIO52NB1/LPRB_B` / `GPIO52PB1/LPRB_A` と書かれているが、パッド名は GPIO4。ネット名の表記揺れとみられる
- J12 は `HDR/2X2/100MIL_NL`（未実装）
- Bank 1 なので電圧は VDDI1_5（J45 で選択）

### 9.6 Table 2-5 の MIC22405 の電流

**確認済み（根拠: Sheet 14 の見出し "POWER:1.2V for DDR4, VDDI6 @4A"（U12）、"POWER:1.8V@4A"（U9））。** どちらの MIC22405YML-TR も **4A**。
- 参考: MIC22705（U7 VDD、U72 3P3V）は "@7A"、MIC69502WR（U8 2P5V）は "@5A"
- UG Table 2-5 の Description / Current 列は行がずれているように見える。表の値より回路図の見出しを優先すること

---

## 10. 読み取れなかった・不確かな項目（要確認）

- 電源 LED（5P0V / VDD / 1P8V）の参照記号（図中にレール名しか書かれていない）
- MIC22705 / MIC22405 の EN/DLY の立ち上がり遅延の値と、レギュレータ間の起動順（データシートの範囲）
- U85 の SD_VCCB の生成元と SEL の論理
- mikroBUS UART（MBUS_UART_RXD/TXD）の向き
- RPi 26 ピン（RPI_GPIO7、B2）を SPI_1 の SS1 として使えるか
- J49 の既定値（ドキュメント間で食い違いがある）
- RPi コネクタの型番（UG 61204021621 と回路図 61304021121）
- Ethernet を VDDI1_5 = 3.3V で使えるか
- DEVRST_N 用の手動リセットスイッチの有無（回路図上には無い）
