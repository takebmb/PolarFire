# Timing Constraints Editor UG: SDC 例外コマンドとオブジェクト取得（PDF p.62-64, 75-79, 119, 121-131）

## Core Idea
例外コマンド（false path / max・min delay / multicycle / disable timing）は `-from` / `-through` / `-to` でパスを絞り込む。**少なくとも 1 つの -from/-to/-through が必須**。対象指定には `get_ports` / `get_pins` / `get_nets` / `get_cells` / `get_clocks` / `all_inputs` / `all_outputs` / `all_registers` を使うが、それぞれ使える場所に制限がある。

## Frameworks Introduced
- **起点/終点/経由点の有効オブジェクト**
  - `-from`: クロック、一次入力、inout ポート、順序セルのクロックピン
  - `-to`: クロック、一次出力、inout ポート、順序セルのデータピン
  - `-through`: ピン、ポート、ネット、インスタンス（ただし警告上は net か pin が正しい型）。1 つの `-through` に複数を書くと「いずれかを通る」
- **multicycle の setup/hold の考え方**
  - `-setup`/`-hold` 無し: ncycles は setup に適用、hold は既定の 0（ncycles−1 ではない）
  - `-setup N` のあと hold を戻したいときは `-hold` で別途指定
  - `-end`（setup の既定）: capture エッジを後ろへ。`-start`（hold の既定）: launch エッジを動かす
  - `-setup_only`: setup パスだけに乗数を適用
  - Microchip では multicycle 同士の優先度管理は無く、全て同じ優先度

## Commands & Syntax
```tcl
set_false_path [-ignore_errors] [-from from_list ] [-through through_list ] [-to to_list ]
```
- setup と hold の両方を外す。multicycle と max delay より常に優先
- `-ignore_errors`: IP 由来の derived 制約が、論理最適化で消えた対象を指して止まるのを防ぐ。用途外では使わない
```tcl
set_false_path -from [get_clocks {clk1}] -to reg_2:D
set_false_path -through U0/U1:Y
set_false_path -through [get_nets {Tblk/rm/n*}]
set_false_path -ignore_errors -through [ get_pins {PCIe_Demo_0/SYSRESET_POR/POWER_ON_RESET_N } ]
```

```tcl
set_max_delay delay_value [-ignore_clock_latency] [-from from_list ] [-to to_list ] [-through
through_list ]
set_min_delay delay_value [-ignore_clock_latency][-from from_list ] [-to to_list ] [-through
through_list ]
```
- 起点/終点のスキュー、input/output delay、setup を含めて評価
- `-ignore_clock_latency`: スキューとジッタを除外しデータパスのみ。非同期クロック間向け。Synplify Pro 非対応
```tcl
set_max_delay 5 -from {ff1a:CLK ff1b:CLK} -to {ff2e:D}
set_max_delay 3.8 -to [get_ports out*]
set_max_delay 2 -from [all_inputs] -to [all_outputs]
set_min_delay 5 -from {ff1a:CLK ff1b:CLK} -to {ff2e:D}
```

```tcl
set_multicycle_path ncycles [-setup] [-hold] [-setup_only] [-from from_list] \
[-through through_list ] [-to to_list] [ -start ] [ -end ]
```
- ncycles は 2 以上の整数（2.2 や 4/2 は警告なしで切り捨て）
```tcl
set_multicycle_path 3 -from [get_pins {reg1}] -to [get_pins {reg2}]
set_multicycle_path 4 -setup -from [get_clocks {ck1}]
set_multicycle_path 2 -hold -from [get_clocks {ck1}]
set_multicycle_path -setup_only 4 -from [ get_clocks { REF_CLK_0 } ]
set_multicycle_path -setup -start 2 -from [ get_clocks clk1 ] -to [ get_clocks clk2 ]
set_multicycle_path -setup -end 2 -from [ get_clocks clk1 ] -to [ get_clocks clk2 ]
```

```tcl
set_disable_timing -from value -to value name
```
- インスタンス内のアーク（例 A→Y）を無効化。`-from` と `-to` は両方あるか両方無いか。P&R と Verify Timing 用（合成は無視）
```tcl
set_disable_timing -from A -to Y
```

```tcl
all_inputs
all_outputs
all_registers [-clock clock_name ] [-async_pins] \
 [-output_pins] [-data_pins] [-clock_pins]
get_cells pattern
get_clocks pattern
get_nets pattern
get_pins pattern
get_ports pattern
```
- ワイルドカード `*` 可。一致なしは空文字列を返す
- ピン名は `inst:PIN`（例 `reg1:CLK`）、階層は `/`（例 `toplevel/block1/reg2:CLK`）

## Reference Tables
**オブジェクト取得コマンドの使用制限**
| コマンド | 返すもの | 使える場所 |
|---|---|---|
| `all_inputs` / `all_outputs` | 全入力（+inout）/ 全出力（+inout） | set_min/max_delay, set_multicycle_path, set_false_path の `-from`/`-to` のみ（`-through` 不可） |
| `all_registers` | レジスタセル（既定）またはピン | 同上の `-from`/`-to` |
| `get_cells` | インスタンス | set_max_delay, set_multicycle_path, set_false_path の `-from`/`-to` |
| `get_nets` | ネット | create_clock / create_generated_clock の source、例外の `-through` |
| `get_pins` / `get_ports` / `get_clocks` | ピン / ポート / クロック | 制限の記述なし |

**主なエラー**
| コード | 内容 |
|---|---|
| SDC0021 / SDC0023 | -from / -to の値が不正 |
| SDC0022 / SDC0024 / SDC0026 | -from / -to / -through が空 |
| 警告 | `-through` に cell や port を渡すと「net か pin であるべき」として **制約が無効化** される |

## Anti-patterns
- **`-through [get_ports ...]` や `-through [get_cells ...]`**: 警告が出て制約自体が無効になる。`get_pins` / `get_nets` を使う
- **`all_inputs` を `-through` に使う**: 不可
- **`set_multicycle_path 2` だけ書いて hold も 1 サイクルずれると思う**: `-setup`/`-hold` 無指定では hold は既定 0。意図を明示して `-hold` を書く
- **ワイルドカードが空振りしたのに気付かない**: get_* は一致なしで空文字列を返す。`check_constraints` や `list_objects` で確認（tce-06）
- **非同期クロック間に max delay だけ掛ける**: スキューを含めて評価される。データパスだけで縛るなら `-ignore_clock_latency`

## Key Takeaways
1. 例外は -from/-to/-through のいずれか必須。起点はクロックピン側、終点はデータピン側
2. `-through` には pin/net を渡す。cell/port は制約無効化
3. multicycle は setup と hold を分けて考える。`-setup_only`、`-start`/`-end` で launch/capture エッジを制御
4. `all_*` / `get_cells` は使える引数位置が限られる
5. `-ignore_errors` は IP の derived 制約用。乱用しない

## Connects To
- **tce-02**: 例外の優先順位と GUI
- **tce-06**: check_constraints / list_* / remove_* で制約を確認・削除
- **sta-02**: SmartTime の Path List から右クリックで例外を追加（cross-probe）
