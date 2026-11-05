# Chapter 2: Installation and Settings（UG p.8-10）

## Core Idea
ソフトは Libero SoC（無償 Silver ライセンス）+ SoftConsole を入れる。ハードは 4 つのジャンパで I/O 電圧と電源元を決める。特に **J45（VDDI1_5）と J46（VDDAUX1）はセットで切り替える**。

## Frameworks Introduced
- **I/O 電圧プロファイル（J45 + J46）**
  - **3.3V プロファイル（既定: 両方 1-2）**: Raspberry Pi GPIO、mikroBUS 用
  - **2.5V プロファイル（両方 2-3）**: MIPI RX、Ethernet 用
  - ルール: J45 で VDDI1_5 を 2.5V にしたら、J46 で VDDAUX1 も必ず 2.5V
  - VDDI1_5 は Bank 1（GPIO）と Bank 5（MSS SGMII）の両方に供給されるため、両プロファイルの機能は同時に最適化できない
- **ソフトウェア準備 4 点**: Libero SoC（プログラマドライバ同梱）/ SoftConsole / DirectCores（IP Core Tools）/ Firmware Catalog のドライバ

## Key Concepts
- **VDDI1_5**: Bank 1 と Bank 5 の I/O 電源（2.5V / 3.3V）
- **VDDAUX1**: Bank 1 の補助電源（2.5V / 3.3V）。VDDI1_5 と揃える
- **Silver license**: Microchip Portal で生成する無償 Libero ライセンス
- **Power LEDs**: VDD（1V コア）、1P8V、5P0V
- **Test points**: GND1 / GND4 / GND5（GND）、TP_VDD（1V コア）、TP_1P8V（1.8V）
- **Power Supply Topology**: 5V（J4 または J7、J47 で選択）→ 各レギュレータ → VDD / 3P3V / 1P2V_DDR4 / 1P8V / 2P5V / 0P6V_VTT/VREF

## Reference Tables
**ジャンパ（Table 2-1）**
| Jumper | 役割 | 1-2 | 2-3 / 開放 | 既定 |
|---|---|---|---|---|
| J45 | VDDI1_5（Bank1+5） | 3.3V（RPi GPIO, mikroBUS） | 2-3: 2.5V（MIPI RX, Ethernet） | 1-2 |
| J46 | VDDAUX1 | 3.3V | 2-3: 2.5V | 1-2 |
| J47 | 電源元 | クローズ: USB-C J4 | 開放: DC Jack J7 | 1-2 クローズ |
| J49 | 7 セグ電圧 | 3.3V | 2-3: 5V | **2-3（5V）** |

**Bank と I/O 電圧（Table 2-4）**
| Bank | 種別 | Rail | 電圧 |
|---|---|---|---|
| 0 | HSIO | 1P8V | 1.8V |
| 1 | GPIO | VDDI1_5 | 2.5V / 3.3V |
| 2 | MSS I/O | 3P3V | 3.3V |
| 3 | JTAG | 3P3V | 3.3V |
| 4 | MSS SD | 1P8V | 1.8V |
| 5 | MSS SGMII | VDDI1_5 | 2.5V / 3.3V |
| 6 | MSS DDR | 1P2V_DDR4 | 1.2V |

**レギュレータ（Table 2-5）**
| Rail | Part | 種別（原文） | 電流 |
|---|---|---|---|
| VDD 1.0V | MIC22705YML-TR | Buck adj 7A | 7A |
| 1P2V_DDR4 | MIC22405YML-TR | Buck adj 3A | 4A |
| 1P8V | MIC22405YML-TR | Buck adj 3A | 4A |
| 2P5V | MIC69502WR | Linear pos adj 5A | 5A |
| 3P3V | MIC22705YML-TR | Buck adj 7A | 7A |
| VTT/VREF | MIC5166YML-TR | 3A HS DDR term | 3A |

## Anti-patterns
- **J45 だけ 2.5V にする**: VDDAUX1 が 3.3V のまま残る。J46 も 2-3 へ
- **3.3V プロファイルのまま MIPI カメラを使う**: UG は MIPI RX に 2.5V を指定
- **2.5V プロファイルで 3.3V の Click / RPi HAT を繋ぐ**: Bank 1 の電圧が合わない
- **J49 既定（5V）のまま 3.3V 専用の 7 セグ基板を挿す**: 先に J49 を 1-2 にする

## Key Takeaways
1. J45/J46 は常にペアで切り替える
2. 3.3V = RPi/mikroBUS、2.5V = MIPI/Ethernet（UG の記述どおり）
3. Bank 0（LED・SW・7 セグ・50 MHz）は 1.8V 固定 → ファブリック制約は LVCMOS18
4. 電源元の切替は J47。DC ジャック使用時は開放
5. 原文の不整合: Table 2-5 で MIC22405 の説明が「3A」なのに Current 列は 4A

## Connects To
- **Ch1**: 電圧測定点（Table 1-3）
- **Ch3**: 各インターフェースがどの Bank にあるか
- **Ch6**: ピン配置（Bank 1 = RPi/mikroBUS/MIPI、Bank 0 = LED/SW/7 セグ）
