# Chapter 1: Introduction / Getting Started（UG p.1-7）

## Core Idea
MPFS-DISCO-KIT は MPFS095T-1FCSG325E（PolarFire SoC: RISC-V MSS + FPGA ファブリック）を評価する低コストキット。オンボード FlashPro5（eFP5）で USB-C 1 本から給電・書き込み・デバッグ・UART まで完結する。

## Frameworks Introduced
- **1 本ケーブル運用モデル**: USB-C（J4）= 5V/3A 給電 + JTAG（eFP5）+ 3ch UART（FT4232HL）
  - When to use: 通常の開発時（既定構成）
  - How: J47 を 1-2 クローズ（既定）のまま、付属 C-C ケーブルで PC の 5V/3A 対応 Type-C ポートへ
- **電源投入確認**: 5P0V / VDD / 1P8V の 3 LED 点灯 → 必要ならテストポイント（Table 1-3 のコンデンサ）で電圧実測

## Key Concepts
- **MPFS-DISCO-KIT**: 本キットの型番。RoHS 準拠、コスト最適化
- **MPFS095T-1FCSG325E (U1)**: 搭載デバイス。詳細は PolarFire SoC Datasheet
- **eFP5 (U16)**: Embedded FlashPro5。USB-to-JTAG。Identify / SmartDebug（ファブリック）と SoftConsole（組込み SW）のデバッグにも使う
- **FT4232HL (U16 表記)**: USB-quad UART ブリッジ。基板上では 3 UART を使用
- **VSC8221 (U4)**: 10/100/1000BASE-T PHY。SGMII モードで RJ45（J3、マグネティクス内蔵）へ
- **DDR4 (U2)**: MT40A512M16TB-062E:R、x16、1 GB
- **J10 / J11 / J5,J6 / J48**: RPi 40pin / RPi MIPI RX (CSI-2) / mikroBUS / 7 セグ(10pin)

## Reference Tables
**キット内容（Table 1-1）**: ボード ×1、USB 2.0 C-C ケーブル 1 m シールド ×1、Quickstart card ×1

**ボード部品（Table 1-2 抜粋）**
| 部品 | Ref | 要点 |
|---|---|---|
| USB-C 5V/3A | J4 | 既定の給電源 |
| 5V DC Jack | J7 | J47 開放で使用 |
| 50 MHz osc DSC1001DL5-050.0000 | X3 | シングルエンド、ファブリック用 |
| 125 MHz osc DSC1123BL5-125.0000 | X1 | 差動 LVDS、MSS リファレンスクロック |
| eFP5 | U16 | USB-JTAG |
| Gigabit Ethernet | J3 | VSC8221 / SGMII |
| microSD | J17 | |
| Push buttons | SW1-SW2 | デバッグ用 |
| DIP switch | SW8 | 8 bit |
| LEDs | LED1-LED8 | **アクティブ High** |

**電圧測定点（Table 1-3）**
| # | Rail | 測定点 | 期待値 | 許容 |
|---|---|---|---|---|
| 1 | 5P0V_IN | C175 | 5V | ±5% |
| 2 | 5P0V | C165 | 5V | ±5% |
| 3 | 3P3V | C123 | 3.3V | ±5% |
| 4 | VDD | C144 | 1.0V | ±3% |
| 5 | VDDA | C134 | 1.0V | ±3% |
| 6 | 2P5V | C153 | 2.5V | ±3% |
| 7 | 1P8V | C227 | 1.8V | ±5% |
| 8 | 1P2V_DDR4 | C284 | 1.2V | ±3% |
| 9 | 0P6V_VREF_DDR4 | C250 | 0.6V | ±3% |
| 10 | 3P3V_FT | C277 | 3.3V | ±5% |
| 11 | 1P8V_FT | C280 | 1.8V | ±5% |

## Anti-patterns
- **ドッキングステーションの USB-C から給電**: 未検証。ノート PC / PC 本体のポートを使う
- **J47 を確認せず DC ジャックを挿す**: J7 を使うなら J47 の 1-2 を開放する
- **ESD 対策なしで扱う**: ESD 対策の上で取り扱うこと
- **動作温度を前提にした設計**: UG では「将来改版で記載」とされ未定義

## Key Takeaways
1. 既定は USB-C 給電（J47 1-2 クローズ）。PC ポート 5V/3A を使う
2. 正常起動の目印は 5P0V・VDD・1P8V の LED
3. 電源異常の切り分けは Table 1-3 の 11 測定点
4. 底面に MIPI RX（J11）と microSD スロットがある
5. 書き込み・デバッグに外部プログラマは不要

## Connects To
- **Ch2**: ジャンパ設定（J45/J46/J47/J49）、電源ツリー
- **Ch5**: FlashPro Express での書き込み手順
- **Ch6**: 全コネクタのピン配置
