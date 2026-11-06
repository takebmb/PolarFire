# Libero Tcl: スクリプト実行と Tcl 基礎（PDF p.13-31）

> 出典: *Libero SoC v2026.1 Tcl Command Reference Guide*（DS50003101W, 06/2026）Ch.1「Tcl Scripting Overview」、Ch.2「Building a Libero Design Using Tcl」

## Core Idea
Libero の Tcl は「GUI 操作の 1 対 1 写像」。`libero script:<file>.tcl` で GUI を出さずにバッチ実行でき、引数は `SCRIPT_ARGS:` で渡す。推奨構成は **parameters.tcl（デバイス変数）+ design.tcl（import/organize/configure）+ libero.tcl（new_project → フロー実行）** の 3 分割。各ステップは `catch` で包んで成否を `puts` する。

## Commands & Syntax

### コマンドラインからの実行（p.21-22）
```
<Libero インストール先>\Designer\bin\libero.exe script:<filename>.tcl
<Libero インストール先> SCRIPT:<filename>.tcl "SCRIPT_ARGS:a b c" LOGFILE:<output.log>
```
| 引数 | 値・意味 |
|---|---|
| `script:` / `SCRIPT:` | 実行する Tcl（相対/絶対パス）。この節のキーワードは大文字小文字を区別しない |
| `"SCRIPT_ARGS:..."` | スクリプトへの引数。原文例はスペース区切りをダブルクォートで囲む |
| `LOGFILE:` | ログ出力先 |

例:
```tcl
# 原文例
C:/Microchip/Libero/Designer/bin/libero.exe SCRIPT:myscript.tcl "SCRIPT_ARGS:one two three"
C:\libero\designer\bin\libero SCRIPT:testTclparam.tcl "SCRIPT_ARGS:a b c" LOGFILE:testTclparam.log
# Discovery Kit リファレンスデザイン（原文外: polarfire-soc-discovery-kit skill の README より。区切りは +）
libero script:MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.tcl SCRIPT_ARGS:I2C_LOOPBACK+HSS_UPDATE+PROGRAM logfile:MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.log
```

### スクリプト内での引数取得（p.19）
```
$argv0            ;# スクリプト名
lindex $argv N    ;# N 番目の引数
llength $argv     ;# 引数の個数
```
- 原文は「`lindex $argv 0` = Tcl ファイル名、`lindex $argv 1` = 第 1 引数」と書く一方、例では `set des_name [lindex $argv 0]` を設計名に使っており記述が揺れている。実挙動はスクリプト冒頭で `puts $argv` して確認する (原文外)。
- `+` 区切りの引数（リファレンスデザイン方式）は `split [lindex $argv 0] "+"` のようにスクリプト側で分解している可能性が高い (原文外。実スクリプトで確認)。

### GUI からの実行 (原文外)
- Project > Execute Script（CTRL+U）でファイルと引数を指定。リファレンスデザイン README では GUI の引数は**スペース区切り・クォートなし**。

### export_script（p.83）— 現セッションの操作を Tcl に書き出す
```
export_script -file { absolute or relative path to constraint file } -relative_path <value>
```
| 引数 | 値・意味 |
|---|---|
| `-file` | 出力 Tcl のパス |
| `-relative_path` | `1` = 相対パス、`0` = 絶対パス（必須） |

例:
```tcl
export_script -file {./exported.tcl} -relative_path 1
```
- GUI 版は File > Export Script File（p.22）。「Include Commands from Current Design [Project] Only」で他プロジェクトの操作を除外。スクリプトを別マシンへ持ち出すなら Qualified（フルパス）、移動しないなら Relative。
- 出力されるのはツールコマンドと引数のみ。

### extended_run_lib（p.22-24）— マルチパス配置配線のバッチ実行
```
$ACTEL_SW_DIR/bin/libero script:$ACTEL_SW_DIR/scripts/extended_run_lib.tcl logfile:extended_run.log "script_args:-root path/designer/module_name [-n numPasses] [-starting_seed_index numIndex] [-compare_criteria value] [-c clockName] [-analysis value] [-slack_criteria value] [-stop_on_success] [-timing_driven|-standard] [-power_driven value] [-placer_high_effort value]"
```
| 引数 | 値・意味 |
|---|---|
| `-n` | パス数（既定 5） |
| `-starting_seed_index` | 1〜100 |
| `-compare_criteria` | `frequency` / `violations` / `power` |
| `-analysis` | `max`（既定）/ `min` |
| `-slack_criteria` | `worst`（既定）/ `tns` |
- Tcl コマンドではなくシェルから叩くスクリプト。全パスの設計ファイルは保存されず、レポートだけ残る。

### パート番号の分解（p.25）
| 項目 | 規則 | MPFS095T-1FCSG325E の場合 (原文外: 当てはめ) |
|---|---|---|
| `-die` | `-` の前 | `MPFS095T` |
| `-speed` | `-` 直後の数字（無ければ `STD`） | `-1` |
| `-package` | 続く英数字（末尾の 1 文字は含めない） | `FCSG325` |
| `-part_range` | 末尾の文字 | `E` → `EXT` |

## Tcl の書き方の要点（p.13-20）
- コマンド名は大文字小文字を区別、引数は区別しない。
- 行継続は行末 `\`。継続行の直後にコメント行を置くとコメントが無視される。
- `[ ]` は入れ子コマンド。ポート名などに `[ ]` を含めるときは `{}` で囲むか `\[` でエスケープ。
- パスは `{C:\dir\file}` か `C:/dir/file`。変数を含むパスは `{}` だと展開されないので `"$dir/file"` を使う。
- `catch { cmd } result` で失敗を拾う。`run_tool` 等は失敗時に非 0 を返す。

## Ch.2 推奨フロー（p.26-31）
```tcl
# libero.tcl（原文例。デバイス変数は Discovery Kit 用に変更 (原文外)）
set prj_family  "PolarFireSoC"   ;# 原文は "PolarFire"
set prj_die     "MPFS095T"       ;# 原文は "MPF100T"
set prj_package "FCSG325"        ;# 原文は "FCG484"
set prj_speed   "-1"
set prj_root    "top"
set libero_cmd "new_project -location {./exprj} -name {exprj} \
  -family {$prj_family} -die {$prj_die} -package {$prj_package} \
  -speed {$prj_speed} -hdl {VERILOG}"
eval file delete -force ./exprj
eval $libero_cmd
source ./design.tcl          ;# import_files / organize_tool_files / configure_tool
build_design_hierarchy
set_root $prj_root
derive_constraints_sdc
if {[catch {run_tool -name {SYNTHESIZE}}]} { puts "SYNTHESIZE FAILED" } else { puts "SYNTHESIZE PASSED" }
# ... PLACEROUTE / VERIFYTIMING / GENERATEPROGRAMMINGDATA / generate_design_initialization_data / GENERATEPROGRAMMINGFILE
close_project -save 1
```
- 更新時（p.31）: HDL・制約を差し替えて libero.tcl を再実行するのが原文の推奨。`file delete -force` でプロジェクトを作り直す前提。

## Anti-patterns
- **引数区切りを混同**: 原文は `"SCRIPT_ARGS:a b c"`（クォート + スペース）、リファレンスデザインは `SCRIPT_ARGS:A+B+C`。スクリプトが期待する形式に合わせる
- **継続行 `\` の後ろにコメント**: 次行のコメントが無視され、意図しない引数が混入する
- **`{}` 内に `$var`**: 展開されない。パスに変数を使うなら `"..."`
- **`catch` なしでフローを連結**: 途中失敗に気付かず後段が古いデータで走る

## Key Takeaways
1. バッチ実行は `libero script:x.tcl SCRIPT_ARGS:... logfile:...`。GUI 不要で高速
2. GUI 操作の Tcl は `export_script`（File > Export Script File）で得られる。手書きより確実
3. パート番号 MPFS095T-1FCSG325E → die `MPFS095T` / speed `-1` / package `FCSG325` / range `EXT`
4. 各フローステップは `if {[catch {...}]}` で成否判定する

## Connects To
- [tcl-02-project-device.md](tcl-02-project-device.md): new_project / set_device の全引数
- [tcl-05-design-flow.md](tcl-05-design-flow.md): run_tool の名前一覧と configure_tool
- polarfire-soc-discovery-kit skill の `references/reference-design.md`: 引数一覧（HSS_UPDATE, PROGRAM, EXPORT_FPE 等）
