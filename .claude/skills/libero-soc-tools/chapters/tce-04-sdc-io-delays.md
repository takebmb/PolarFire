# Timing Constraints Editor UG: SDC I/O 遅延系コマンド（PDF p.114-115, 120-125, 134-136）

## Core Idea
I/O タイミングは「FPGA 外の予算」で書くのが標準（`set_input_delay` / `set_output_delay`）。「FPGA 内の予算」を直接書く `set_external_check` / `set_clock_to_output` は Libero の STA だけが見る非標準コマンド。DDR やソース同期の制約は、**`-clock_fall` と `-add_delay` で立上り・立下りの 2 本を同じポートに積む** のが基本形。

## Frameworks Introduced
- **どれを使うか**
  - 標準 SDC（合成・P&R・STA すべてで有効）→ `set_input_delay` / `set_output_delay`
  - 内部予算を直接指定（STA のみ、P&R や Synplify は無視）→ `set_clock_to_output`、`set_external_check`
  - PLL 外部フィードバックの基板遅延 → `set_external_delay`
- **`-add_delay` の合成規則**（PrimeTime と同じ）
  - `-add_delay` 無しで同じポートに別クロックの制約を書くと、前の制約は上書き
  - `-add_delay` 付きなら両方有効（別クロック・別エッジのパスを同じポートに表現）
  - {-rise,-fall} × {-max,-min} の各組は独立の制約。同じ組が重なると -max は最大値、-min は最小値が残る

## Commands & Syntax
```tcl
set_input_delay delay_value -clock clock_ref [-max] [-min] [-clock_fall] [-rise] [-fall] [-
add_delay] \
input_list
```
- delay_value: クロックエッジ後、入力にデータが現れるまでの時間(ns)。通常は外部レジスタのクロックピンからの組合せ遅延を表す
- `-max` / `-min`: 最長 / 最短パス。どちらも無指定なら max = min
- `-clock_fall`: 基準クロックの立下りエッジ基準（既定は立上り）
- `-rise` / `-fall`: ポート側の立上り/立下り遷移に対する遅延（無指定なら共通）
- clock_ref: ポート名・ピン名（例 `reg1:CLK`）・`[get_clocks {clk}]` のいずれか（クロック定義済みであること）
- 双方向ポートは入力・出力の両モードに遅延を指定できる

```tcl
set_input_delay 1.2 -clock [get_clocks CLK1] [get_ports data1]
set_input_delay 1.0 -clock_fall -clock CLK2 -min {IN1}
set_input_delay 1.4 -clock_fall -clock CLK2 -max {IN1}
set_input_delay 1.0 -clock CLK1 -max {IN1}
set_input_delay 1.4 -add_delay -clock CLK2 -max {IN1}
```

```tcl
set_output_delay [-max] [-min] delay_value -clock clock_ref [-clock_fall] [-rise] [-fall] \
[-add_delay] output_list
```
- delay_value: クロックエッジの何 ns 前に信号が必要か。外部レジスタまでの組合せ遅延 + setup（max）/ + hold（min）
- 出力ポートは指定しない限り output delay を持たない（＝制約なし）
```tcl
set_output_delay 1.2 -clock [get_clocks CLK1] [get_ports OUT1]
set_output_delay -min {OUT1} 1.0 -clock_fall -clock CLK2
set_output_delay -max {OUT1} 1.4 -clock_fall -clock CLK2
set_output_delay 1.4 {OUT1} -add_delay -clock CLK2 -max
```

```tcl
set_clock_to_output delay_value -clock clock_ref [-max] [-min] output_list
set_external_check delay_value -clock clock_ref [-setup] [-hold] input_list
set_external_delay from value to value [min] [max] <delay_value>
```
- `set_clock_to_output`: launch エッジから出力ポートのデータ変化までの FPGA 内予算。`-clock` は出力 FF を駆動する実クロック。生成クロックなら初段クロック源から解析
- `set_external_check`: 入力の setup/hold 予算。`-setup` か `-hold` のどちらか必須
- `set_external_delay`: PLL 出力（-from 出力ポート）→ 基板 → PLL フィードバック（-to 入力ポート）の遅延。Synplify Pro 非対応
```tcl
set_clock_to_output -max 0.3 -clock { clk } [ get_ports { Q } ]
set_external_check -setup 0.2 -clock { clk } [ get_ports { A } ]
set_external_delay -from [ get_ports { GL0_CLK_OUT} ] -to [ get_ports { FB_CLK_IN } ] 3.0
```

## Code Examples: LVDS ソース同期 DDR への当てはめ（原文外）
原文にソース同期・DDR・LVDS 専用の例は無い。以下は上記の文書化済みオプションを組み合わせた一般形。
```tcl
# 受信: 送信側から転送クロック rx_clk が来る DDR 入力
create_clock -name rx_clk -period 4.0 [get_ports rx_clk]
set_input_delay -max 0.6 -clock rx_clk [get_ports {rx_data*}]
set_input_delay -min -0.6 -clock rx_clk [get_ports {rx_data*}]
set_input_delay -max 0.6 -clock rx_clk -clock_fall -add_delay [get_ports {rx_data*}]
set_input_delay -min -0.6 -clock rx_clk -clock_fall -add_delay [get_ports {rx_data*}]

# 送信: クロックを出力ポートへ転送する DDR 出力
create_generated_clock -name tx_clk_fwd -divide_by 1 \
 -source [get_pins {<PLL出力ピン>}] [get_ports tx_clk]
set_output_delay -max 0.5 -clock tx_clk_fwd [get_ports {tx_data*}]
set_output_delay -min -0.5 -clock tx_clk_fwd [get_ports {tx_data*}]
set_output_delay -max 0.5 -clock tx_clk_fwd -clock_fall -add_delay [get_ports {tx_data*}]
set_output_delay -min -0.5 -clock tx_clk_fwd -clock_fall -add_delay [get_ports {tx_data*}]
```
- 値の目安（原文外）: 入力 max = 送信側データ有効窓の終端側スキュー、min = 開始側スキュー（edge-aligned なら負になることが多い）。出力 max = 受信側 tsu + 基板スキュー、min = −(受信側 th) + 基板スキュー
- 差動ペアは P/N どちらのポート名で制約するかをネットリスト（`get_ports` の結果）で確認する（原文外）
- CCC/PLL や IOD 系 IP が生成する SDC は Constraints Manager の **Derive Constraints** で取り込むのが推奨（p.115 の記述）

## Reference Tables
| エラー | 意味 |
|---|---|
| SDC0004 | clk が既存のクロック名/ソースに一致しない |
| SDC0015 | ポートリスト不正 |
| SDC0054 | min delay > max delay |
| SDC0061 | パラメータ不正 |
| SDC0078 | set_external_delay の -from/-to ポート共有 |

## Anti-patterns
- **DDR の立下り側を `-add_delay` 無しで書く**: 立上り側の制約が上書きされて消える
- **`set_clock_to_output` で出力を締めて P&R も追従すると思う**: P&R と Synplify はこのコマンドを無視する。P&R に効かせるなら `set_output_delay` / `set_max_delay`
- **出力ポートに output delay を入れない**: 制約なしとして扱われ、Constraints Coverage で Untested になる
- **-min を省略**: max と同値とみなされ、hold 検査が非現実的になる

## Key Takeaways
1. 標準は `set_input_delay` / `set_output_delay`（外部予算）。内部予算コマンドは STA 専用
2. DDR は rise 用と `-clock_fall -add_delay` の fall 用の 2 組を書く
3. `-add_delay` 無しの別クロック指定は上書き。重複時は max は最大、min は最小が残る
4. 出力側の転送クロックは `create_generated_clock` を出力ポートに定義して基準にする（原文の clk_out 例と同形）
5. CCC/IP 由来のクロック制約は Derive Constraints で取り込む

## Connects To
- **tce-03**: 基準にするクロック（virtual / generated）の定義
- **tce-05**: I/O パスに max/min delay・multicycle を掛ける場合
- **sta-01 / sta-03**: External Setup / External Hold / Clock to Out セットでの確認
