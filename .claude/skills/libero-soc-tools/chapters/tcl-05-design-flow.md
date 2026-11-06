# Libero Tcl: デザインフロー実行とツール設定（PDF p.43, 48-50, 87-95, 122-125, 146-149, 220-260）

## Core Idea
フローは **`configure_tool -name {TOOL} -params {KEY:VALUE}` で設定 → `run_tool -name {TOOL}` で実行**。上流が未実行なら自動で上流も走る（PLACEROUTE を叩けば SYNTHESIZE から実行）。状態は `get_tool_state`、設定値は `get_tool_options` で読める。戻り値は 0 = 成功、1 = 失敗なので `catch` で判定する。

## Commands & Syntax

### run_tool（p.122-125）
```
run_tool -name {tool_name} [-script {absolute or relative path to script file} ]
```
| `-name` | 内容 | `-script` |
|---|---|---|
| `SYNTHESIZE` | 合成（Synplify） | — |
| `COMPILE` | Compile | — |
| `SIM_PRESYNTH` / `SIM_POSTSYNTH` / `SIM_POSTLAYOUT` | 各段のシミュレーション（テストベンチ関連付けが前提） | — |
| `PLACEROUTE` | 配置配線 | — |
| `VERIFYTIMING` | SmartTime 解析 | SmartTime 用 Tcl |
| `VERIFYPOWER` | SmartPower 解析 | SmartPower 用 Tcl |
| `GENERATEPROGRAMMINGDATA` | Generate FPGA Array Data | — |
| `GENERATEPROGRAMMINGFILE` | Generate Bitstream | — |
| `PROGRAMDEVICE` | Run PROGRAM Action（デバイス書込み） | — |
| `CONFIGURE_CHAIN` | JTAG チェーン設定（FlashPro コマンドの Tcl を渡す、**必須**） | 必須 |
| `SMARTDEBUG` | SmartDebug コマンドを実行 | 必須 |
| `SSNANALYZER` | 同時スイッチングノイズ解析 | SSN 用 Tcl |
| `EXPORTNETLIST` | .v/.vhd ネットリスト出力 | — |
| `EXPORTSDF` | バックアノテート (.v + .sdf) 出力 | — |
| `GENERATEDEBUGDATA` | SmartDebug 用データ生成（PolarFire） | — |
| `GENERATE_SPI_FLASH_IMAGE` / `PROGRAM_SPI_FLASH_IMAGE` | SPI Flash イメージ生成 / 書込み（PolarFire） | — |
| `UPDATE_ENVM` | eNVM クライアント更新（SmartFusion 2/IGLOO 2） | .cfg |
| `UPDATE_UPROM` | uPROM 更新（RTG4） | .cfg |
| `GENERATE_MEMORY_MAP` | データシート XML（SmartFusion 2/IGLOO 2/RTG4） | — |
- `run_tool` を呼ぶ前に `set_root` が必要。
- `PROGRAMDEVICE` はプログラマ未接続で `PROGRAMDEVICE: No programmer is connected.`

### update_and_run_tool（p.146-149）
```
update_and_run_tool -name {<tool_name >} -script {<absolute or relative path to script file>}
```
- ソースに変更があるときだけ更新して実行。`-name` の候補は run_tool とほぼ同じ（SMARTDEBUG を除く）。

### configure_tool（p.48-50）
```
configure_tool -name {tool_name} -params {<parameter>:<value>}
```
- `-params` は繰り返し可。`-name` の候補: `COMPILE`, `CONFIGURE_ACTION_PROCEDURES`, `CONFIGURE_PROG_OPTIONS`, `SYNTHESIZE`, `PLACEROUTE`, `VERIFYTIMING`, `VERIFYPOWER`, `GENERATEPROGRAMMINGDATA`, `GENERATEPROGRAMMINGFILE`, `PROGRAMDEVICE`, `PROGRAM_OPTIONS`, `PROGRAMMER_INFO`, `SPM`, `FLASH_FREEZE`, `PROGRAM_RECOVERY`, `USER_PROG_DATA`, `INIT_LOCK` ほか（EXPORTSDF, IO_PROGRAMMING_STATE, PROGRAM_SPI_FLASH_IMAGE, SPM_OTP も Ch.6 で configure_tool 対象）。

### SYNTHESIZE パラメータ（p.251-254）
| KEY | 値・意味 |
|---|---|
| `CLOCK_ASYNC` | 非同期ピンのグローバル昇格しきい値（既定 12） |
| `CLOCK_GLOBAL` | クロックピン昇格しきい値（既定 2） |
| `CLOCK_DATA` | データピン昇格しきい値（既定 5000、1000〜200000） |
| `RETIMING` | リタイミング true/false |
| `RAM_OPTIMIZED_FOR_POWER` | true = 低消費電力 RAM 推論 |
| `SYNPLIFY_OPTIONS` | Synplify の `set_option ...;` を直接渡す（ユーザ Tcl より優先） |
| `SYNPLIFY_TCL_FILE` | Synplify 用 Tcl ファイル |
| `BLOCK_MODE` / `BLOCK_PLACEMENT_CONFLICTS` / `BLOCK_ROUTING_CONFLICTS` | ブロックフロー用（ERROR/KEEP/LOCK/DISCARD） |
| `LANGUAGE_SYSTEM_VLOG` / `LANGUAGE_VERILOG_2001` | 言語モード |
| `CDC_MIN_NUM_SYNC_REGS` / `CDC_REPORT` | CDC 同期段数（2〜9）とレポート |

### PLACEROUTE パラメータ（PolarFire 系, p.233-237）
| KEY | 既定 | 意味 |
|---|---|---|
| `TDPR` | true | タイミングドリブン |
| `PDPR` | false | パワードリブン |
| `IOREG_COMBINING` | false | I/O レジスタ結合 |
| `GB_DEMOTION` | true | グローバル降格 |
| `REPLICATION` | false | ドライバ複製 |
| `EFFORT_LEVEL` | false | 高エフォート |
| `INCRPLACEANDROUTE` | false | 前回配置を初期値に（毎回 false に戻る） |
| `REPAIR_MIN_DELAY` | false | ホールド違反修復 |
| `MULTI_PASS_LAYOUT` / `NUM_MULTI_PASSES`（5） / `START_SEED_INDEX` | false | マルチパス |
| `MULTI_PASS_CRITERIA` | VIOLATIONS | `SLOWEST_CLOCK` / `SPECIFIC_CLOCK` / `VIOLATIONS` / `TOTAL_POWER` |
| `SPECIFIC_CLOCK` / `DELAY_ANALYSIS`（MAX/MIN）/ `STOP_ON_FIRST_PASS` / `SLACK_CRITERIA`（WORST_SLACK/TOTAL_NEGATIVE_SLACK） | | マルチパス判定 |
| `RANDOM_SEED` / `RGB_COUNT`（1-18） | | シード / RGB 帯域上書き |

例:
```tcl
# 原文 PolarFire 例（抜粋）
configure_tool -name {PLACEROUTE} -params {TDPR:true} -params {EFFORT_LEVEL:true} \
  -params {REPAIR_MIN_DELAY:true} -params {IOREG_COMBINING:false} \
  -params {MULTI_PASS_LAYOUT:false}
if {[catch {run_tool -name {PLACEROUTE}}]} { puts "PLACEROUTE FAILED" }
```

### VERIFYTIMING パラメータ（p.257-260）
```
configure_tool -name {VERIFYTIMING} -params {parameter:value}
run_tool -name {VERIFYTIMING} -script {timing.tcl}
```
| KEY | 意味 |
|---|---|
| `MAX_TIMING_MULTI_CORNER` / `MIN_TIMING_MULTI_CORNER` 他 `*_VIOLATIONS_MULTI_CORNER` | マルチコーナーのレポート（既定 1） |
| `MAX_TIMING_SLOW_LV_HT` / `MIN_TIMING_FAST_HV_LT` / `*_SLOW_LV_LT`（PolarFire 系） | コーナー別レポート |
| `FORMAT` | `XML` / `TEXT` / `HTML` / `CSV` |
| `CONSTRAINTS_COVERAGE` | 制約カバレッジレポート |
| `MAX_PATHS_TIMING`（10）/ `MAX_PATHS_VIOLATION`（20）/ `SLACK_THRESHOLD_VIOLATION` | 出力パス数・しきい値 |
- `-script` の Tcl は SmartTime コマンド（`create_set`, `report`, `write_sdc`, `save`）。SDC 系コマンドの詳細は別担当。

### 状態と設定の問い合わせ（p.43, 87, 92-96）
```
get_tool_state -name {tool_name}
get_tool_options -name {tool_name} -params {parameter_names}
cleanall_tool -name {tool_name}
force_update_design_flow
generate_sdc_constraint_coverage -tool {PLACEROUTE | VERIFYTIMING}
```
| コマンド | 用途 |
|---|---|
| `get_tool_state` | `Tool successfully run` / `Tool failed` / `Tool inputs are out of date` / `Tool has timing violations` 等の文字列を返す |
| `get_tool_options` | `-params` 省略で全パラメータ。例 `puts [get_tool_options -name {SYNTHESIZE} -params {RETIMING}]` |
| `cleanall_tool` | 指定ツールまでクリーン。下流の Pass は Out of Date へ |
| `force_update_design_flow` | 無効化された Synthesize/P&R の状態を強制的に Pass へ |

例:
```tcl
# 原文例を基に、タイミング未達でスクリプトを止める (原文外: 判定文字列の部分一致は推測)
run_tool -name {VERIFYTIMING}
set st [get_tool_state -name {VERIFYTIMING}]
puts "VERIFYTIMING: $st"
if {[string match "*violation*" $st]} { error "timing not met" }
```

### レポート出力（p.70, 78, 85）
- `export_pin_reports`（ピンレポート）、`export_design_summary`（HTML サマリ）、`export_netlist_file`、`export_ba_files`、`export_smart_debug_data`（.ddc）。索引参照。

## Anti-patterns
- **configure_tool を run_tool の後に書く**: 設定は次回実行から。順序は configure → run
- **SYNPLIFY_OPTIONS と SYNPLIFY_TCL_FILE で同じオプションを別値に**: SYNPLIFY_OPTIONS が勝つ
- **INCRPLACEANDROUTE を常用前提にする**: 実行ごとに false に戻る
- **force_update_design_flow で変更を握りつぶす**: ソース変更後に使うと古い結果が Pass 扱いになる

## Key Takeaways
1. 書込みまでの名前: `SYNTHESIZE` → `PLACEROUTE` → `VERIFYTIMING` → `GENERATEPROGRAMMINGDATA` → `GENERATEPROGRAMMINGFILE` → `PROGRAMDEVICE`
2. 上流は自動実行されるので、リファレンスデザインの `PROGRAM` 引数も実質 `run_tool -name {PROGRAMDEVICE}` 1 回で全フローが走る (原文外: 挙動からの推測)
3. タイミング確認は `get_tool_state -name {VERIFYTIMING}` の文字列で行う
4. 設定の現在値は `get_tool_options` で読み、スクリプトに固定化する

## Connects To
- [tcl-03-files-constraints.md](tcl-03-files-constraints.md): 各ツールに渡す制約
- [tcl-06-memory-init.md](tcl-06-memory-init.md) / [tcl-07-program-export.md](tcl-07-program-export.md): メモリ初期化と GENERATEPROGRAMMINGFILE / PROGRAMDEVICE の設定
- SmartTime/SDC コマンドは SDC 担当の章
