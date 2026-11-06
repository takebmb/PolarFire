# I/O Editor User Guide: Floorplanner View・各ウィンドウ・PDC 書き出し・PolarFire SoC MSS I/O（PDF p.37-53）

## Core Idea
Floorplanner View はセルレベルでチップ全体を表示し、マクロ配置・リージョン作成・I/O バンク設定を GUI で行う。結果は I/O 制約 PDC と `fp.pdc`（フロアプラン）として書き出せる。PolarFire SoC の MSS I/O（Bank 2/4/5/6）は MSS Configurator の設定がそのまま使われ、I/O Editor では**読み取り専用**。

## Frameworks Introduced
- **Floorplanner の 2 モード**: Macro Manipulation（マクロの配置/解除、プロパティ参照、Ctrl で複数選択）/ Region Manipulation（リージョンのサイズ変更・改名・削除、マクロやネットの割り当て）。ボタンで切替
- **バンク電圧の GUI 設定**: アイコン「I/O Bank Settings」で個別設定、「Auto Assign I/O Bank」で未設定バンクに電圧（と必要なら VREF ピン）を自動付与
- **PDC 書き出し**: File > Export Physical Constraint (PDC) > I/O Constraint（User または Full 制約）/ > Floorplan Constraint（fp.pdc）。任意の場所に保存可
- **MSS I/O の扱い**: MSS ポートは固定位置なので自動配置。PDC 不要。正しい配置と IOSTD を書いても受理されるが、MSS 固有属性を設定しようとすると失敗。Chip Planner が生成する PDC には書かれない

## Key Concepts
- **Display Options**: レイヤ/色の設定グループ System / Pin_Planner / Grey_Scale。Fill Device Cells, Use Cluster Mode, Consolidate Globals
- **Highlighting**: Logical/Net Object Browser でマクロ・ネットに永続色を付ける（鉛筆アイコン、消しゴムで解除）
- **Cross Probing**: 外部アプリからのクロスプローブ表示（無効化不可、透明色にはできる）
- **Netlist Viewer – Hier / Flat**: post-synthesis 階層ビュー / post-compile 平坦化ビュー
- **World View**: Floorplanner の表示範囲を赤枠で示す
- **Log Window**: Error/Warning/Info でフィルタ。保存時に更新された PDC のパスを表示
- **Object Window タブ**: Port, Logical, Net, Region（Ctrl+F で浮動ウィンドウ）
- **Properties Window**: マクロ名、セル種別、配置座標、リソース使用、リージョン、ネット表、ロック状態、I/O ならポート/IOSTD/バンク/ピン

## Reference Tables
**Floorplanner View アイコン（Table 9-1、主要なもの）**
| アイコン | 機能 |
|---|---|
| Rubber Band Zoom / Select | 範囲ズーム / 範囲内マクロ選択 |
| Zoom to Location | X-Y 座標へ移動 |
| Check Design Rules | Prelayout Checker（P&R 前のネットリスト検査） |
| Check DRC Rules for Selected Interfaces | 選択インターフェースの DRC |
| I/O Bank Settings | バンクの I/O 技術を設定 |
| Auto Assign I/O Bank | Auto I/O Bank and Globals Assigner を実行 |
| Create Empty / Inclusive / Exclusive | ユーザーリージョン作成 |
| Show Nets For Macros | マクロに繋がる全ネット表示（既定 off） |

**MSS バンク設定（Table 12-1、PDF 画像で確認）**
| 種別 | Bank | 対応 VDDI |
|---|---|---|
| MSS_IO（Peripheral IOs） | Bank2 | 1.2 / 1.5 / 1.8 / 2.5 / 3.3 |
| MSS_IO（Peripheral IOs） | Bank4 | 1.2 / 1.5 / 1.8 / 2.5 / 3.3 |
| MSS_SGMII_IO（RefClk, SGMII） | Bank5 | 2.5 / 3.3 |
| MSS_DDR_IO | Bank6 | DDR3 1.5, DDR3L 1.35, DDR4 1.2, LPDDR3 1.2, LPDDR4 1.1 |

**MSS ポートの IOSTD（Table 12-2/12-3）**
| ポート | IOSTD |
|---|---|
| Bank2/Bank4 ペリフェラル | 電圧に応じ LVCMOS12（原文は "LVCMOS22" と誤記）/15/18/25/33 |
| Bank5 REFCLK | LVTTL, PCI, LVCMOS33/25/18, SSTL25I, SSTL18I, LVDS25, LVPECL33 |
| Bank5 SGMII | LVDS25, LVDS33, RSDS25/33, MINILVDS25/33, SUBLVDS25/33, PPDS25/33, LCMDS25/33 |
| Bank6 DDR | DDR3: SSTL15I, DDR3L: SSTL135I, LPDDR3: HSUL12I, LPDDR4: LVSTL11I, DDR4 DQ/DQS/DM: POD12I |

**MSS 未使用インターフェースの既定バンク（Table 12-4）**: Bank2=3.3, Bank4=3.3, Bank5=3.3, Bank6=1.5（VDDI）。MSS マクロが無いデザインではバンク未設定・非ロック（変更しても影響なし）。

**MSS I/O 属性（read-only、Table 12-5/12-6/12-9）**
| 対象 | 属性: 値 |
|---|---|
| Reference CLK | Resistor Pull: None, Up / User I/O Lock Down: OFF / Odt: OFF, 100 / Thevenin: OFF, 150, 75, 50 / Schmitt: OFF, ON |
| Peripheral・GPIO | Resistor Pull: None, Up, Down, Hold / Clamp Diode, Persist, Lock Down: OFF, ON / Schmitt: 1.2/1.5/1.8V バンクでは常に ON / Low Power Mode In/Out: OFF, ON / Output Drive: 2,4,6,8,10,12,16,20 mA |
| SGMII | Resistor Pull: None, Up, Down / Odt: OFF, 100 / Vcm Input Range: MID, LOW / Source Termination: OFF, 100 / Output Drive: 1.5, 2, 3, 3.5, 4, 6 mA |
| DDR ODT（DQ/DQS） | DDR3/3L: 120,60,40,30 / DDR4: 120,80,60,40,30 / LPDDR3: 240,120,80,60,40,30 / LPDDR4: 120,80,60,48,40 |
| DDR Impedance | DDR3/3L: 48,34,30,24 / DDR4: 60,48,34,27 / LPDDR3・LPDDR4: 60,48,40,34 |

## Anti-patterns
- **MSS I/O の属性を PDC で上書きしようとする**: 失敗する。MSS Configurator で設定
- **ファブリックの LVDS を Bank 5（MSS SGMII）に置こうとする**: Bank 5 は MSS_SGMII_IO。ファブリック GPIO の LVDS は Bank 1 などファブリックバンクで（Discovery Kit のバンク割当は姉妹 skill 参照）
- **進捗ダイアログを閉じてリージョン割り当てを止めようとする**: 閉じても処理は止まらない

## Key Takeaways
1. Floorplanner の I/O Bank Settings / Auto Assign I/O Bank は set_iobank の GUI 版
2. 書き出しは I/O 制約（User/Full）と fp.pdc の 2 種
3. MSS I/O は自動配置・read-only、PDC 不要
4. Bank 5 は SGMII 用に LVDS25/33 等を持つが MSS 専用
5. 原文表に誤記（LVCMOS22）あり

## Connects To
- **pdc-01-syntax-iobank**: set_iobank
- **pdc-04-location-ndc-floorplan**: define_region / assign_region（Floorplanner の操作結果）
- 姉妹 skill `polarfire-soc-mss`: MSSIO・SGMII の詳細
- 姉妹 skill `polarfire-soc-discovery-kit`: 実ボードのバンク電圧
