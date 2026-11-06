# Timing Constraints Editor UG: SDC クロック系コマンド（PDF p.65-71, 110-118, 125-126, 137）

## Core Idea
クロック定義は `create_clock`（一次クロック・virtual clock）と `create_generated_clock`（PLL/CCC 出力・分周・転送クロック）の 2 本立て。クロック間の関係は `set_clock_groups`、ジッタやマージンは `set_input_jitter` / `set_clock_uncertainty`、外部遅延は `set_clock_latency -source` で補う。**PLL の `-phase` は波形シフトではなくクロック網の遅延として扱われる** 点に注意。

## Frameworks Introduced
- **クロック定義の判断**
  - 外部から入るクロック → `create_clock` を入力ポートに
  - FPGA 外の基準クロック（I/O の相手側クロック）→ ソース無しの `create_clock`（virtual clock）
  - PLL/CCC 出力・分周 FF 出力・出力ポートへ転送するクロック → `create_generated_clock`（`-source` は基準ピン）
  - 同一ポートに複数周波数の可能性 → 別名 + `-add`、その後 `set_clock_groups -physically_exclusive` など
- **ジッタ/マージンの入れ方（PolarFire）**: 入力クロックのジッタ = `set_input_jitter`（SDC ファイル内のみ）→ 追加マージン = `set_clock_uncertainty -setup` → 自動システムジッタの上書き = `set_system_jitter`（稀）

## Commands & Syntax
```tcl
create_clock [ -name clock_name ] [-add] -period period_value \
 [ -waveform edge_list ][ source_objects ]
```
- `-name` か source のどちらかは必須。`-add` には `-name` が必須で、名前が既存と同じなら `-add` でも上書き
- `-waveform {rise fall}`: 立上り→立下りの 2 値、fall > rise。例: 周期 19 で 3 ns 立上り / 8 ns 立下り → `[ 3 8 ]`
- source は ports / pins / nets。既にクロックがあるピンに定義すると置き換え

```tcl
create_clock -name {my_user_clock} -period 6 CK1
create_clock -name {my_other_user_clock} -period 6 -waveform {0 3} {CK2}
create_clock -name clk1 -period 10 -waveform {0 5} [get_ports clk1]
create_clock -name clk2 -add -period 20 -waveform {0 10} [get_ports clk1]
create_clock -name mainCLK -period 2.5 [get_nets {cknet}]
```

```tcl
create_generated_clock [-name clock_name ] [-add] \
 [-master_clock clock_name ] \
 -source reference_pin \
 [- divide_by divide_factor ] \
 [-multiply_by multiply_factor ] \
 [-invert] source [-edges values ] \
 [-edge_shift values ]
```
- `-source`（基準ピン）と source（定義点、1 ピンのみ）は必須
- `-divide_by` / `-multiply_by` は整数（1.2 や 4/2 は警告なしで切り捨て）。`-edges` とは併用不可
- `-edges {r f r}`: 基準クロックのエッジ番号 3 つ（昇順）。`-edge_shift` は ns で 3 値、`-edges` 必須
- `-add` には `-name` と `-master_clock` が必要（複数クロックがマスタピンに入る場合）
- 例で使われる追加オプション: `-phase`、`-pll_output`、`-pll_feedback`（外部フィードバック CCC、PolarFire/RTG4）
- **重要**: `-phase` は「クロック網内の遅延」として扱われ、setup/hold の期待エッジ決定には使われない（launch の最初のエッジ → capture の 2 番目のエッジのまま）

```tcl
create_generated_clock -name {my_user_clock} -divide_by 2 \
 -source [get_ports {CLK}] U1/reg1:Q
create_generated_clock -divide_by 3 -multiply_by 4 \
 -source clk [get_pins {myPLL:CLK1}]
create_generated_clock -name PULSE -source [get_ports clk] \
 -edges {1 3 3} -edge_shift {2 0 2} [get_pins NAND1/Y]
# 出力ポートへ転送するクロック（set_clock_groups の例より）
create_generated_clock -name clk_out -divide_by 1 \
 -source [ get_pins { ccc_100_0/ccc_100_0/pll_inst_0/OUT0 } ] \
 [ get_ports clk_out ]
```

```tcl
set_clock_groups [-name name ] \
 [-physically exclusive | -logically exclusive | -asynchronous] \
 [-comment comment_string ] \
 -group clock_list
```
- 引数表上の名前は `-physically_exclusive` / `-logically_exclusive`。3 つのフラグは排他で 1 つだけ
- 指定グループ間は両方向とも解析しない。同一グループ内は同期として解析
- 同じ名前・同じフラグで再定義すると前の定義を置き換え
```tcl
set_clock_groups -logically_exclusive -group clk_1 -group clk_2
set_clock_groups -asynchronous -group { clk_in ccc_100 clk_out } -group clk_50
```

```tcl
set_clock_latency -source [-rise] [-fall] [-early] [-late] delay clock
```
- FPGA では内部（network）latency は使わない（クロックツリーは既知）。外部（source）latency 専用
- `-source` だけ、または early/late の片方だけでは setup/hold に影響しない。**`-early` と `-late` の両方を与えたときのみ setup に影響**（hold は同一エッジ検査なので常に影響なし）。`-source` 無しは FPGA では非推奨
```tcl
set_clock_latency -source -rise -early 0.4 { main_clock }
set_clock_latency -source -fall 1.2 { main_clock }
```

```tcl
set_clock_uncertainty [-setup] [-hold] uncertainty [object_list -from from_clock |
 -rise_from rise_from_clock | -fall_from fall_from_clock -to to_clock | -rise_to rise_to_clock
 |
 -fall_to fall_to_clock ]
```
- 不確かさは required time 側にだけ適用（latency は arrival と required の両方）
- **ジッタのモデル化には `-setup` を明示**。無指定だと setup と hold の両方に掛かり、ジッタとしては誤り
- Simple はクロック/クロック網上のピンに設定し、その先のキャプチャレジスタに効く（複数掛かれば伝搬順で最後のもの）。Clock-to-clock は from/to 両方必須で Simple より優先
```tcl
set_clock_uncertainty 2 -setup [get_clocks clk]
set_clock_uncertainty 10 -from [get_clocks { Clk1 }] -to [get_clocks { Clk2 }]
```

```tcl
set_input_jitter <input_jitter> <clock>
set_system_jitter <system_jitter>
```
- どちらも Tcl コマンドとしては使えず **.sdc ファイル内でのみ有効**。Synplify Pro は `set_input_jitter` を無視
- ジッタは ns、正の値。clock は create_clock で付けた名前
```tcl
create_clock -name { clockName } -period 20 -waveform {0 10 } [ get_ports { CLK } ]
set_input_jitter 2.1 [ get_clocks { clockName } ]
```

## Reference Tables
| `-setup` | `-hold` | uncertainty の適用 |
|---|---|---|
| No | No | setup と hold |
| No | Yes | hold のみ |
| Yes | No | setup のみ（ジッタ用） |
| Yes | Yes | setup と hold |

主なエラー: SDC0001（clock source 不正）、SDC0006（period 不正）、SDC0007（waveform 不正）、SDC0062（`-edges` は 3 値）、SDC0063（`-edges` 非昇順）、SDC0065（`-edges` と乗除算の併用）、SDC0069（`-add` に名前なし）。

## Anti-patterns
- **PDF から例をコピペする**: 原文の例に en dash（`–period`）が混じっている箇所がある。ASCII の `-` に直す
- **90° 位相の PLL 出力で center-aligned 取り込みを「波形がずれる」前提で読む**: `-phase` は遅延扱いで、エッジ対応は変わらない
- **`set_input_jitter` を Tcl コンソールで打つ**: 効かない。SDC に書く
- **`set_clock_uncertainty` を `-setup` なしでジッタに使う**: hold にも掛かり過剰に悲観的
- **同名 + `-add`**: 追加ではなく上書きになる

## Key Takeaways
1. 一次クロックは `create_clock`、PLL/分周/転送クロックは `create_generated_clock`（`-source` は基準ピン）
2. Virtual clock = ソース無し `create_clock`。外部デバイス側クロックの表現に使う
3. `-phase` は波形シフトではなくクロック網遅延として扱われる
4. 非同期・排他クロックは `set_clock_groups`（3 フラグは排他）
5. `set_input_jitter` / `set_system_jitter` は SDC ファイル専用
6. ジッタを uncertainty で表すなら `-setup` を付ける

## Connects To
- **tce-04**: これらのクロックを基準にした I/O 遅延
- **tce-02**: GUI 側の Clock Groups / Uncertainty / Latency
- **sta-02**: inter-clock domain 解析（共通周期 = 周期の最小公倍数）
- **sta-04**: CDC レポートの Clock Group Cstr 列
