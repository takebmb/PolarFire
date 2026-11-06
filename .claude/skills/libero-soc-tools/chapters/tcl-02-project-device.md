# Libero Tcl: プロジェクトとデバイス設定（PDF p.45, 101-116, 125-134）

## Core Idea
`new_project` 1 発でデバイス・HDL・制約フロー・動作条件までが決まる。後から変えるのは `set_device`（デバイス）、`project_settings`（フロー設定）、`project_general_settings`（リンクファイルの相対/絶対）。デバイスを変えるとコンポーネントとフローが無効化され、再生成・再実行が必要になる。

## Commands & Syntax

### new_project（p.101-106）
```
new_project -name project_name \
  -location project_location \
  -family family_name \
  -project_description "brief text description of project" \
  -die device_die \
  -package package_name \
  -hdl HDL_type \
  -speed speed_grade \
  -die_voltage value \
  -part_range value \
  -block_mode {1 | 0} \
  -ondemand_build_dh {1 | 0} \
  -adv_options value \
  -use_relative_path {1 | 0} \
  -linked_files_root_dir_env root_dir_env \
  -standalone_peripheral_initialization {1 | 0} \
  -instantiate_in_smartdesign {1 | 0} \
  -use_enhanced_constraint_flow {1 | 0}
```
| 引数 | 値・意味 |
|---|---|
| `-location` | プロジェクトの場所。**既存ディレクトリ不可** |
| `-family` / `-die` / `-package` / `-speed` | デバイス。speed は `-1` か `STD` |
| `-die_voltage` | コア電圧（データシート依存） |
| `-part_range` | PolarFire 系: `EXT` / `IND` / `MIL` |
| `-hdl` | `VHDL` / `VERILOG` |
| `-block_mode` | 1 = デザインブロック作成モード（既定 0） |
| `-ondemand_build_dh` | 1 = On Demand Build Design Hierarchy |
| `-instantiate_in_smartdesign` | 1 = MSS/System Builder を作成時に SmartDesign へ自動配置（原文で既定 0/1 の記述が矛盾） |
| `-use_relative_path` / `-linked_files_root_dir_env` | リンクファイルを環境変数基準の相対パスで保持 |
| `-use_enhanced_constraint_flow` | 1 = Enhanced Constraint Flow（I/O PDC・FP PDC・SDC 分離。原文の全 PolarFire 例で 1） |
| `-adv_options` | `KEY:VALUE` を繰り返し指定（下表） |

**`-adv_options`（PolarFire 系で使うもの）**
| KEY | 値 |
|---|---|
| `IO_DEFT_STD` | 既定 I/O 規格: `LVTTL` / `LVCMOS 3.3V` / `LVCMOS 2.5V` / `LVCMOS 1.8V` / `LVCMOS 1.5V` / `LVCMOS 1.2V` |
| `RESTRICTPROBEPINS` | 1 = SmartDebug 用プローブピン予約 |
| `RESTRICTSPIPINS` | （RTG4 のみ）SPI ピン予約 |
| `SYSTEM_CONTROLLER_SUSPEND_MODE` | 1/0。**v2026.1 から adv_options で設定**（旧 CONFIGURE_PROG_OPTIONS の引数は廃止） |
| `TEMPR` | 解析用温度範囲。`TEMPR:0 25 70` のように min/typ/max も可 |
| `VCCI_1.2_VOLTR` 〜 `VCCI_3.3_VOLTR` | 各 VCCI の電圧範囲（COM/IND/MIL/Custom、PolarFire 例は EXT） |
| `VOLTR` | コア電圧範囲 |
| `RAD_EXPOSURE` | RT 品のみ |

例:
```tcl
# 原文 p.106 の PolarFire 例を MPFS095T-1FCSG325E 用に変更 (原文外: family/die/package/RESTRICTPROBEPINS 以外の値は AN5282 の GUI 既定から)
new_project -location {./MPFS_DISCO} -name {MPFS_DISCO} \
  -project_description {} -block_mode 0 -standalone_peripheral_initialization 0 \
  -instantiate_in_smartdesign 1 -ondemand_build_dh 1 \
  -use_enhanced_constraint_flow 1 -hdl {VERILOG} \
  -family {PolarFireSoC} -die {MPFS095T} -package {FCSG325} -speed {-1} \
  -die_voltage {1.0} -part_range {EXT} \
  -adv_options {IO_DEFT_STD:LVCMOS 1.8V} -adv_options {RESTRICTPROBEPINS:1} \
  -adv_options {SYSTEM_CONTROLLER_SUSPEND_MODE:0} -adv_options {TEMPR:EXT} \
  -adv_options {VCCI_1.2_VOLTR:EXT} -adv_options {VCCI_1.5_VOLTR:EXT} \
  -adv_options {VCCI_1.8_VOLTR:EXT} -adv_options {VCCI_2.5_VOLTR:EXT} \
  -adv_options {VCCI_3.3_VOLTR:EXT} -adv_options {VOLTR:EXT}
```
- family 文字列: 本書のエラーメッセージ・derive_constraints では `PolarFire SoC`（空白あり）表記。GUI/AN5282 は `PolarFireSoC`。**実際に通る表記は `export_script` の出力で確認する** (原文外)。

### set_device（p.131-134）
```
set_device [-family "family" ] [-die "die" ] [-die_voltage "value" ] \
  [-part_range "part_range" ] [-package "package" ] \
  [-speed "speed_grade" ] [-adv_options "value" ]
```
- 一度決めた family は変更不可。それ以外は変更可。`<project>/tooldata/set_device.log` を生成。
- 温度範囲の既定（min/typ/max）: EXT 0/25/100、IND -40/25/100、MIL -55/25/125。

例:
```tcl
# 原文 PolarFire 例を基に die/package を変更 (原文外)
set_device -family {PolarFireSoC} -die {MPFS095T} -package {FCSG325} -speed {-1} -part_range {EXT} \
  -adv_options {TEMPR:0 25 70} -adv_options {SYSTEM_CONTROLLER_SUSPEND_MODE:0}
```

### project_settings（p.113-116）
```
project_settings [-hdl "VHDL | VERILOG"] [-verilog_mode "SYSTEM_VERILOG | VERILOG_2K"] \
  [-vhdl_mode "VHDL_2008 | VHDL_93"] [-auto_update_modelsim_ini "TRUE | FALSE"] \
  [-standalone_peripheral_initialization "TRUE | FALSE"] [-ondemand_build_dh "TRUE | FALSE"] \
  [-auto_generate_synth_hdl "TRUE | FALSE"] [-auto_run_drc "TRUE | FALSE"] \
  [-auto_file_detection "TRUE | FALSE"] [-enable_set_mitigation "TRUE | FALSE"] \
  [-display_fanout_limit "display_fanout_limit"] [-block_mode "TRUE | FALSE"] \
  [-abort_flow_on_sdc_errors "TRUE | FALSE"] [-abort_flow_on_pdc_errors "TRUE | FALSE"] \
  [-instantiate_in_smartdesign "TRUE | FALSE"] [-vm_netlist_flow "TRUE | FALSE"] ...
```
| 引数 | 値・意味 |
|---|---|
| `-verilog_mode` | 既定 `SYSTEM_VERILOG` |
| `-vhdl_mode` | 既定 VHDL-2008 |
| `-abort_flow_on_pdc_errors` / `-abort_flow_on_sdc_errors` | 制約エラーでフロー中断（既定 ON） |
| `-display_fanout_limit` | ログに出す高ファンアウトネット数（既定 10） |
| `-system_verilog_mfcu` | SV の Multi-File Compilation Unit |

### project_general_settings（p.112-113）
```
project_general_settings [-use_relative_path { 1 | 0 }] [-linked_files_root_dir_env {linked files root directory}]
```
- 絶対 → 相対へ一度変えると戻せない。

### open_project / save_project / save_project_as / close_project（p.45, 107, 125-127）
```
open_project -file "project file" [-do_backup_on_convert "TRUE | FALSE"] [-backup_file "backup archive file name"]
save_project
save_project_as -name "project name" -location "project location" [-files "all | project | sources" ] [-replace_links "TRUE | FALSE" ]
close_project [-save value]
```
| 引数 | 値・意味 |
|---|---|
| `open_project -file` | `.prjx` のフルパス |
| `-do_backup_on_convert` / `-backup_file` | 旧版プロジェクトを開くとき `.zip` にバックアップ |
| `close_project -save` | 既定 true（保存して閉じる） |

例:
```tcl
open_project -file {./MPFS_DISCO/MPFS_DISCO.prjx}   ;# (原文外: パス変更)
save_project
close_project -save 1
```

### その他の問い合わせ（p.90-91）
- `get_libero_version` / `get_libero_release`: バージョン文字列を返す。スクリプト冒頭のバージョンチェックに使える。

## Anti-patterns
- **既存ディレクトリに new_project**: 失敗する。原文フローは `file delete -force` してから作る
- **SYSTEM_CONTROLLER_SUSPEND_MODE を CONFIGURE_PROG_OPTIONS で設定**: v2026.1 で廃止。`-adv_options` へ
- **-part_range を COM**: COM は SmartFusion 2/IGLOO 2 専用。MPFS095T-1FCSG325E は末尾 E → `EXT`
- **set_device で family を変えようとする**: 不可。プロジェクト作り直し

## Key Takeaways
1. Discovery Kit は `-die {MPFS095T} -package {FCSG325} -speed {-1} -die_voltage {1.0} -part_range {EXT}`
2. `-use_enhanced_constraint_flow 1` が前提（I/O PDC と FP PDC が別ファイル）
3. 動作条件（TEMPR, VCCI_*_VOLTR, VOLTR）は Verify Timing/Power の解析条件に伝わる
4. 既存プロジェクトの改造は `open_project` → 変更 → `save_project`

## Connects To
- [tcl-01-scripting-basics.md](tcl-01-scripting-basics.md): パート番号の分解
- [tcl-03-files-constraints.md](tcl-03-files-constraints.md): プロジェクト作成後のファイル投入
- [tcl-08-index-project-sd.md](tcl-08-index-project-sd.md): その他の Project Manager コマンド
