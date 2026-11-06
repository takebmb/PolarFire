# Timing Constraints Editor UG: シナリオ・SDC 入出力・解析/レポート Tcl（PDF p.64-110, 132-138）

## Core Idea
SmartTime の Tcl は「制約を入れる」以外に、**制約の検査（check_constraints）、SDC の読み書き（read_sdc / write_sdc）、パス列挙（list_paths / expand_path）、レポート生成（report）、解析オプション（set_options）** を持つ。スクリプトで Verify Timing を回し、結果をテキスト/CSV で取り出せる。

## Frameworks Introduced
- **制約ファイルの検査フロー**: `read_sdc` → `check_constraints`（構文 + ネットリストとのオブジェクト照合。不一致は `*_sdc.log` にエラー）→ `list_*` で中身確認 → `save`
- **シナリオ**: 制約のセット。解析用（`-analysis_scenario`）と TDPR 用（`-tdpr_scenario`）を別にできる。既定は Primary。`create_scenario` / `clone_scenario` / `set_current_scenario` より `organize_tool_files` の使用が推奨
- **パスの取り出し方**: set 指定は「`-set 名前`」か「`-clock` + `-type`」か「`-from_clock` + `-to_clock`（inter-clock）」か「`-in_to_out`」

## Commands & Syntax
```tcl
check_constraints
read_sdc \
 -add \
 -scenario scenario_name \
 -netlist ( user | optimized ) \
 -pin_separator ( : | / ) \
 -ignore_errors file_name
write_sdc \
 -scenario scenario_name \
 -pin_separator ( : | / ) \
 file name
save
```
- `read_sdc` は `-add` 無しだと既存制約を消してから読む。`-add` なら追加（衝突時は上書き）
- `-netlist user` = 合成後ネットリスト基準、`optimized` = 物理（解析用）ネットリスト基準
```tcl
read_sdc first.sdc
read_sdc -add second.sdc
write_sdc margin.sdc
```

```tcl
list_paths \
 -analysis <max | min> \
 -format <csv | text> \
 -set <name> \
 -clock <clock name> \
 -type <set_type> \
 -from_clock <clock name> \
 -to_clock <clock name> \
 -in_to_out \
 -from <port/pin pattern> \
 -to <port/pin pattern>
expand_path \
 -index value \
 -set name \
 -clock clock_name \
 -type set_type \
 -analysis {max| min} \
 -format {csv | text} \
 -from_clock clock_name \
 -to_clock clock_name
```
- `-analysis max` = setup、`min` = hold。`-clock` はワイルドカード不可
- 返す本数は `set_options -limit_max_paths`（既定 20）
```tcl
puts [ list_paths -clock clk1 -type reg_to_reg ]
puts [expand_path -clock { myclock } -type { reg_to_reg } -index { 1 2 3 } -format text]
```

```tcl
create_set -name <name> -parent_set <name> -type <set_type> -clock <clock name> \
 -source_clock <clock name> -sink_clock <clock name> -in_to_out \
 -source <port/pin pattern> -sink <port/pin pattern>
create_set -name { another_set } -source_clock { EXTERN_CLOCK } -sink_clock { MY_GEN_CLOCK }
```

```tcl
report -type (timing | violations | datasheet | bottleneck | constraints_coverage |
combinational_loops | cdc) \ -analysis <max|min> \ -format (csv|text) \ <filename> \
```
（続くオプション: timing 系 `-max_parallel_paths` `-max_paths` `-print_summary` `-use_slack_threshold` `-slack_threshold` `-print_paths` `-max_expanded_paths` `-include_user_sets` `-include_clock_domains` `-select_clock_domains` `-limit_max_paths` `-include_pin_to_pin`、bottleneck 系 `-cost_type (path_count|path_cost)` `-max_instances` `-from` `-to` `-set_type` `-set_name` `-clock` `-from_clock` `-to_clock` `-in_to_out`）
- `-type` と filename は必須。datasheet で `-analysis` を付けると失敗しうる。cdc は CSV のみ
```tcl
report -type violations -analysis max -use_slack_threshold no -limit_max_paths yes \
 -max_paths 2 -max_expanded_paths 1 timing_viol.txt
report -type timing -select_clock_domains { clk1 clk2 } report2.txt
```

```tcl
set_options \
 [-max_opcond value ] [-min_opcond value ] \
 [-interclockdomain_analysis value ] [-use_bibuf_loopbacks value ] \
 [-enable_recovery_removal_checks value ] [-break_at_async value ] \
 [-filter_when_slack_below value ] [-filter_when_slack_above value ] \
 [-remove_slack_filters] [-limit_max_paths value ] \
 [-expand_clock_network value ] [-expand_parallel_paths value ] \
 [-analysis_scenario value ] [-tdpr_scenario value ] [-reset]
set_options -max_opcond {fast_hv_lt} -min_opcond {fast_hv_lt}
set_options -tdpr_scenario {My_TDPR_Scenario}
```

## Reference Tables
**set_options の主な既定値（PolarFire）**
| オプション | 値 | 既定 | TDPR に影響 |
|---|---|---|---|
| `-max_opcond` | slow_lv_ht / slow_lv_lt / fast_hv_lt | slow_lv_lt | - |
| `-min_opcond` | 同上 | fast_hv_lt | - |
| `-interclockdomain_analysis` | yes / no | yes | あり |
| `-use_bibuf_loopbacks` | yes / no | no（loopback は false path 扱い） | - |
| `-enable_recovery_removal_checks` | yes / no | no | - |
| `-break_at_async` | yes / no | yes | あり |
| `-limit_max_paths` | 整数 | 20 | - |
| `-expand_clock_network` | yes / no | yes | - |
| `-expand_parallel_paths` | 整数 | 1 | - |
| `-analysis_scenario` / `-tdpr_scenario` | 名前 | Primary | tdpr はあり |

（SmartFusion2/IGLOO2/RTG4 は worst / typical / best。既定 max = worst、min = best）

**set_type（-type 値）**: `reg_to_reg` / `async_to_reg` / `reg_to_async` / `external_recovery` / `external_removal` / `external_setup` / `external_hold` / `clock_to_out`

**確認・削除系（引数なしの list と、-id または条件指定の remove）**
| list | remove |
|---|---|
| `list_clocks` / `list_generated_clocks` | `remove_clock -name clock_name \| -id constraint_ID` / `remove_generated_clock -name clock_name \| -id constraint_ID` |
| `list_input_delays` / `list_output_delays` | `remove_input_delay -clock clock_name port_pin_list \| -id constraint_ID` / `remove_output_delay -clock clock_name port_pin_list` |
| `list_false_paths` / `list_max_delays` / `list_min_delays` / `list_multicycle_paths` | `remove_false_path [-from from_list] [-to to_list] [-through through_list] \| -id constraint_ID`（max/min/multicycle も同形） |
| `list_clock_groups` / `list_clock_latencies` / `list_clock_uncertainties` / `list_disable_timings` | `remove_clock_groups` / `remove_clock_latency -source clock_name \| -id constraint_ID` / `remove_clock_uncertainty` / `remove_disable_timing` |
| `list_objects <object>` / `list_scenario` | `remove_all_constraints` / `remove_scenario name` / `remove_set -name name` |

## Anti-patterns
- **`read_sdc` で 2 本目を読むとき `-add` を忘れる**: 1 本目の制約が消える
- **`-ignore_errors` を常用**: IP 由来の derived 制約専用
- **operating condition を 1 つだけ見て安心する**: Timing Report Explorer は全コーナーの最悪を見るが、SmartTime ビューと CDC レポートは選択中の 1 コーナー（sta-04）
- **TDPR と解析で別シナリオなのに気付かない**: `-tdpr_scenario` と `-analysis_scenario` を確認

## Key Takeaways
1. 制約を入れたら `check_constraints` で構文とオブジェクト一致を確認（結果は `*_sdc.log`）
2. `read_sdc` は既定で置換、`-add` で追加
3. PolarFire の既定コーナー: setup = slow_lv_lt、hold = fast_hv_lt。slow_lv_ht も選べる
4. `report -type cdc` は CSV のみ。violations/timing はテキストか CSV
5. `list_paths` の本数は `set_options -limit_max_paths`（既定 20）

## Connects To
- **sta-03**: report の各種類の読み方
- **sta-04**: CDC レポート
- **sta-01**: SmartTime Options ダイアログ（set_options の GUI）
