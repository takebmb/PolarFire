# PDC Commands Reference Guide: set_location・NDC（set_ioff / set_preserve）・フロアプラン FDC（PDF p.16-25）

## Core Idea
I/O 以外の配置制約は 3 系統。`set_location`（マクロ・DDR・PLL/DLL・TxPLL の位置固定）、NDC（Compile 前に読む *.ndc。**`set_ioff` で I/O レジスタ取り込み**、`set_preserve`）、FDC（リージョン定義と割り当て）。IOD/CDR/RGMII/SGMII/XCVR には配置ルールがあり、違反は PRPF_xxx エラーになる。

## Frameworks Introduced
- **I/O FF 取り込み（set_ioff）の条件**: *.ndc に書いて Compile/Synthesis 前に読む / IN_REG・OUT_REG・EN_REG のうち 1 つだけ有効 / 取り込める FF は 1 個で fanout 1 / スカラーポートのみ（バスは for ループかワイルドカード）
- **リージョンの種類の選び方**: ロジックを寄せたい → Inclusive（他も入れる）/ Exclusive（割り当てたものだけ）。何も置かせない → Empty。`-route true` で内部ネットの配線も閉じ込める
- **リージョンの順序**: `define_region` → `assign_region`。別 PDC ファイルならファイル順が重要

## Key Concepts
- **set_location -x -y**: Chip Planner で座標を調べて指定。グローバルバッファの位置指定は推奨しない（Libero に任せる）
- **Edge_Anchor**: DDR3 の配置位置（NORTH_NE/NW, SOUTH_SE/SW, WEST_NW/SW）
- **PLL/DLL 位置**: PLL0_NW, PLL1_NW, PLL0_NE, PLL1_NE, PLL0_SW, PLL1_SW, PLL0_SE, PLL1_SE（DLL も同形式）
- **TxPLL 位置**: Q0–Q3 の TXPLL0 / TXPLL1 / TXPLL_SSC
- **NDC**: Netlist Design Constraint。Compile エンジンが post-synthesis ネットリスト最適化に使う
- **Hard macro**: 相対配置が固定されたロジックセル。リージョンに割り当てられるのはこれのみ
- **AFL 名**: assign_net_macros のネット名は平坦化ネットリスト名（大小区別なし）

## Commands & Syntax
```tcl
set_location -inst_name <macro_inst_name> -fixed <true|false> -x <integer> -y <integer>
set_location -inst_name <hierarchical path to DDR instance> -location <edge>_<anchor>
set_location -inst_name <hierarchical inst name> -location <PLL location>   ;# DLL/TxPLL も同形
# 例
set_location -inst_name mem_data_in\[57\] -fixed true -x 7 -y 2
set_location -inst_name {DDR3_TOP/DDR3_0}\ -location {NORTH_NE}

# NDC
set_ioff -port_name <portname> \ [-IN_REG true/1|false/0] \
  [-OUT_REG true/1|false/0] \ [-EN_REG true/1|false/0]
set_ioff -port_name {DataA[*]} -IN_REG 1
for { set i 0 } { i &lt; 32 } { incr i } { set_ioff -port_name "DataA\[$i\]" -IN_REG 1 }
set_preserve -inst_name <hier_inst_name>        ;# 例: set_preserve -inst_name "test1/AND2_0"

# FDC
define_region -region_name <region_name> -type <inclusive|exclusive|empty> -x1 <integer> -y1 \
  <integer> -x2 <integer> -y2 <integer> [-route <true|false>]
assign_region -region_name <region_name> -inst_name <macro_name>+
assign_net_macros -region_name <region_name> -net_name <net_name> -include_driver <true|false>
move_region -region_name <region_name> -x1 <integer> -y1 <integer> -x2 <integer> -y2 <integer>
# 例
define_region -region_name UserRegion1 -type empty -x1 100 -y1 46 -x2 102 -y2 50
assign_region -region_name User_region2 -inst_name des01/Counter_1/*
```
注: 原文 for ループ例の `&lt;` は HTML エスケープの残骸。実際は `<`（Tcl として `$i < 32` が正しい形。原文外）。set_ioff の値は PolarFire 系では true/false か 1/0（`yes` はエラー PDCPF-426）。戻り値 0=成功 / 1=失敗。

## Reference Tables
**配置ルール違反エラー（Table 3-22〜3-24）**
| コード | 内容 |
|---|---|
| PRPF_001 | インターフェースの全 PAD を set_io で配置してから P&R |
| PRPF_002 | SGMII・IOG CDR は RX_/TX_ PAD を同一レーンに。RGMII は RX[ ] と RXCLK を同一レーン |
| PRPF_003 | SGMII・IOG CDR ではレーンの DQS_N ピンは内部用に予約。ポートを割り当てない |
| PRPF_004 | TRAINING/OVERLAY 付き IOD インスタンスは配置しない（ツールが自動配置） |
| PRPF_005 | RGMII の RX_CLK はレーンの DQS（P パッド）へ |
| PRPF_007 | Tx PLL は set_location で P&R 前に配置 |
| PRPF_008/009 | XCVR 専用ポート・REFCLK は set_io で P&R 前に配置 |
| PRPF_010 | 1 コーナーあたり PLL/DLL 参照クロック＋ファブリッククロックは計 6 以下 |
| PRPF_011 | 1 コーナーあたりファブリック駆動の PLL/DLL 参照クロックは 2 以下 |

**リージョン型（Table 5-4）**: Empty=マクロ不可 / Exclusive=割り当てマクロのみ / Inclusive=割り当て・非割り当て両方可。`-route` 既定 false。ローカル/グローバルクロックとリージョン境界をまたぐインターフェースネットは -route の対象外。

## Anti-patterns
- **set_ioff を PDC に書く**: NDC ファイル専用
- **set_ioff で 2 オプション同時有効**: 1 つだけ
- **IOG CDR のレーンで DQS_N にポートを置く**: PRPF_003
- **グローバルバッファを set_location で固定**: 原文は Libero に任せるよう推奨
- **リージョン定義より前の PDC で assign_region**: 順序エラー

## Key Takeaways
1. I/O 直結レジスタは NDC の `set_ioff`（IN_REG/OUT_REG/EN_REG のいずれか 1 つ）
2. IOD 系インターフェース（CDR, RGMII, SGMII）はレーン単位の配置ルールがある
3. PLL/DLL の位置は 4 コーナー × 2 個、コーナーごとのクロック数制限あり
4. リージョンは Inclusive/Exclusive/Empty、`-route` で配線も閉じ込め
5. DDR3 の Edge_Anchor ごとに最大幅が違う（Table 3-21 は MPF200/MPF300 のみ）

## Connects To
- **pdc-02-set-io-iostd**: PAD は set_io で配置
- **ioed-02-interface-views**: IOD View の DRC（同じレーン制約を GUI で）
- **ioed-03-floorplanner-export-mss**: Floorplanner でリージョン作成
- **mlg-05-post-layout-macros**: IOREG（IOINFF/IOUTFF/IOENFF）＝ set_ioff で取り込まれる先
