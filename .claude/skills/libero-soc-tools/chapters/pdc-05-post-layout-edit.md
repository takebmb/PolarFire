# PDC Commands Reference Guide: Post Layout Edit（edit_io / edit_instance_delay）と付録（PDF p.26-40）

## Core Idea
配置配線後（post-layout）の状態で I/O 属性や遅延だけを変えるためのコマンド。`edit_io` は `edit_post_layout_design` ツールで GPIO/HSIO/トランシーバ I/O の属性を変更する。**`-pin_name` `-fixed` `-io_std` は変更不可**（配置と規格は動かさない）。同一 GPIO バンクで VCM_RANGE を MID/LOW 混在させたいときの唯一の手段でもある。

## Frameworks Introduced
- **いつ edit_io を使うか**
  - ハードウェア上で受信マージンを調整したい（VCM_RANGE、ODT、IN_DELAY など）
  - 同一バンクの真差動入力で VCM_RANGE を混在させたい（set_io では不可）
  - XCVR の TX/RX アナログ設定を再配置なしで変えたい
- **いつ edit_instance_delay を使うか**: LANECTRL の DQS 遅延、ICB_CLKDIVDELAY・PLL_DELAY の遅延値を変える（フロアプラン PDC と edit_post_layout_design の両方で使用可）

## Key Concepts
- **edit_post_layout_design**: post-layout 編集フロー（Libero Design Flow UG の Post Layout I/O Editing）
- **GPIO/HSIO の引数**: set_io と同じ（pdc-03 参照）
- **RX_CALIBRATION**: データレートで決まり、PDC で変更不可
- **TX_EMPHASIS_AMPLITUDE**: 送信の振幅とディエンファシス
- **RX_CTLE / RX_DFE_COEFFICIENT_H1..H5**: 受信イコライザ設定（DFE 係数は 0–15）
- **RX_POLARITY**: P/N 入れ替え（Normal/Inverted）
- **LANECTRL**: IOD レーンコントローラ（RX/TX_DQS_DELAY を持つ）

## Commands & Syntax
```tcl
# GPIO / HSIO
edit_io -port_name <port_name>\
  [-OUT_LOAD <value>]\
  [-RES_PULL <value>]\
  [-LOCK_DOWN <value>]\
  [-CLAMP_DIODE <value>]\
  [-SCHMITT_TRIGGER <value>]\
  [-SLEW <value>]\
  [-VCIM_RANGE <value>]\
  [-ODT <value>]\
  [-ODT_VALUE <value>]\
  [-OUT_DRIVE <value>]\
  [-IMPEDANCE <value>]\
  [-SOURCE_TERM <value>]\
  [-IN_DELAY <value>]\
  [-OUT_DELAY <value>]

# Transceiver I/O
edit_io -port_name <port_name>\
  [-TX_EMPHASIS_AMPLITUDE <value>] [-TX_IMPEDANCE <value>]\
  [-TX_TRANSMIT_COMMON_MODE_ADJUSTMENT <value>] [-RX_INSERTION_LOSS <value>]\
  [-RX_CALIBRATION <value>] [-RX_CTLE <value>] [-RX_CDR_GAIN <value>]\
  [-RX_TERMINATION <value>] [-RX_PN_BOARD_CONNECTION]\
  [-RX_LOSS_OF_SIGNAL_DETECTOR_LOW <value>] [-RX_LOSS_OF_SIGNAL_DETECTOR_HIGH <value>]\
  [-RX_DFE_COEFFICIENT_H1 <value>] … [-RX_DFE_COEFFICIENT_H5 <value>] [-RX_POLARITY <value>]

edit_instance_delay \
  -inst_name <inst_name>\
  -properties {<property_name>:<property_value>[ <property_name>:<property_value>]*}
```
注: 構文中の `-VCIM_RANGE` は set_io の `-VCM_RANGE` に対応する（原文の綴り。実際に受理される綴りは要確認）。原文の例 `edit_io -port_name A -RES_PULL Down -CLAMP_DIODE LVCMOS15 -OUT_DRIVE 12` は CLAMP_DIODE に IOSTD 名を入れており誤り（値は ON/OFF）。

LVDS 受信の VCM を post-layout で LOW に変える例（原文外の組み立て）:
```tcl
edit_io -port_name LVDS_RX_P -VCIM_RANGE LOW
```

## Reference Tables
**XCVR 用 edit_io 引数（Table 6-1〜6-11）**
| 引数 | 値 | 既定 |
|---|---|---|
| -TX_EMPHASIS_AMPLITUDE | 100mV_with_0dB … 1000mV_with_-6.0dB（100/200/300/400/500/600/800/1000mV × 0〜-6.0dB の組） | 400mV_with_-1.0dB |
| -TX_IMPEDANCE | 150, 100, 85, 180 | 100 |
| -TX_TRANSMIT_COMMON_MODE_ADJUSTMENT | 50, 60, 70, 80（%。DC 結合時のみ調整） | 50 |
| -RX_INSERTION_LOSS | 6.5 dB, 17.0 dB, 25.0 dB | 6.5 dB |
| -RX_CALIBRATION | None_CDR, On Demand, On Demand and First Lock, None_DFE | （変更不可） |
| -RX_CDR_GAIN | Low, High | — |
| -RX_TERMINATION | 150, 100, 85 | 100 |
| -RX_PN_BOARD_CONNECTION | AC_COUPLED_WITH_EXT_CAP, DC_COUPLED | AC_COUPLED_WITH_EXT_CAP |
| -RX_LOSS_OF_SIGNAL_DETECTOR_LOW/HIGH | OFF, PCIE, SATA, BMR, 1–7 | OFF |
| -RX_DFE_COEFFICIENT_H1..H5 | 0–15（5bit、MSB は符号） | — |
| -RX_POLARITY | Normal, Inverted | Normal |

**edit_instance_delay のプロパティ（Table 6-12）**
| マクロ | プロパティ | 値 |
|---|---|---|
| LANECTRL | RX_DQS_DELAY | 0–255 |
| LANECTRL | TX_DQS_DELAY | 0–255 |
| ICB_CLKDIVDELAY | DELAY | 0–127, 128, 130, 132, …, 254 |
| PLL_DELAY | DELAY | 0–127 |

**付録 A（p.32-39）**: PolarFire（MPF050/100/200/300/500）のデバイス×パッケージ×スロット（Edge_Anchor）×メモリ種（DDR2/DDR3/DDR4/QDR II+ x8/x9/x18/x36）の対応一覧。PolarFire SoC（MPFS）の行は無い。

## Anti-patterns
- **edit_io でピンや IOSTD を変えようとする**: 非対応。set_io を直して再配置
- **AC 結合系で TX_TRANSMIT_COMMON_MODE_ADJUSTMENT を変更**: 既定のまま使う
- **RX_CALIBRATION を PDC で指定**: データレートで決まる

## Key Takeaways
1. edit_io は属性だけを post-layout で変更（ピン・規格・ロックは不可）
2. GPIO バンク内の VCM_RANGE 混在は edit_io で実現
3. XCVR の振幅/エンファシス/CTLE/DFE/終端も edit_io で調整
4. edit_instance_delay で LANECTRL・ICB_CLKDIVDELAY・PLL_DELAY の遅延を変更
5. 原文の構文・例には誤記（VCIM_RANGE、CLAMP_DIODE LVCMOS15）がある

## Connects To
- **pdc-03-set-io-attributes**: GPIO/HSIO 引数の値表
- **ioed-02-interface-views**: XCVR View の Signal Integrity View（同じ項目を GUI で表示）
- **mlg-05-post-layout-macros**: PLL_DELAY, ICB_CLKINT
