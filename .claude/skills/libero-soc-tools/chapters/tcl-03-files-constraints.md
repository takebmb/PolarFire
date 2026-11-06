# Libero Tcl: ファイル投入・階層・制約ファイルの関連付け（PDF p.37, 42-43, 50-51, 72-75, 98-100, 109-112, 130, 139, 584-593）

## Core Idea
ファイルは **import（プロジェクト内へコピー）** か **link（外部を参照）** で入れる。入れただけでは使われない。**`organize_tool_files` で「どのツールがどの制約を使うか」を明示**して初めて合成・配置配線・タイミング検証に効く。I/O 制約（`-io_pdc`）と フロアプラン制約（`-fp_pdc`）は別ファイル・別引数。

## Commands & Syntax

### import_files（p.98-100）
```
import_files \
-schematic {file} -symbol {file} -smartgen_core {file} -ccp {file} \
-stimulus {file} -hdl_source {file} -io_pdc {file} -fp_pdc {file} \
-edif {file} -sdc {file} -crt {file} -dcf {file} -vcd {file} -saif {file} \
-simulation {file} -profiles {file} -cxf {file} -templates {file} -ccz {file} \
-modelsim_ini {file} -library {file} -convert_EDN_to_HDL {true | false}
```
| 引数 | 値・意味 |
|---|---|
| `-hdl_source` | HDL ソース。**ファイルごとに引数を繰り返す** |
| `-hdl_source_folder` | HDL フォルダ |
| `-stimulus` | テストベンチ |
| `-io_pdc` | I/O 属性・ピン配置の PDC（`<prj>/constraint/io`） |
| `-fp_pdc` | フロアプラン（配置・領域）の PDC（`<prj>/constraint/fp`） |
| `-sdc` | タイミング制約 |
| `-ndc` / `-net_fdc` | Compile Netlist 制約 / Synplify Netlist 制約（構文欄にないがエラーメッセージの正式書式に記載） |
| `-library` | 取り込み先ライブラリ（無ければ作成） |
| `-edif` + `-convert_EDN_to_HDL` | EDIF→HDL 変換（PolarFire では `-edif` 非対応） |
- 旧 `-pdc` は PolarFire 系で廃止。`-io_pdc` / `-fp_pdc` に分ける。
- `-crt` / `-dcf` / `-pin` / `-gcf` は PolarFire 非対応。

例:
```tcl
# 原文 p.27 design.tcl を基に、ファイル名を LVDS 追加想定に変更 (原文外)
import_files -hdl_source {./src/lvds_rx.v} -library {work}
import_files -hdl_source {./src/top.v} -library {work}
import_files -sdc {./src/constraint/timing_user_constraints.sdc}
import_files -io_pdc {./src/constraint/io_lvds.pdc}
import_files -fp_pdc {./src/constraint/fp_constraints.pdc}
import_files -stimulus {./src/stimulus/tb_top.v} -library {stimulus}
```
- 原文 p.27 の例は `import_files -pdc {...}` を使っているが、p.98 の説明では `-pdc` は廃止。**`-io_pdc` / `-fp_pdc` を使う**。

### create_links（p.50-51）
```
create_links [-convert_EDN_to_HDL "TRUE | FALSE"] [-hdl_source_folder "Source folder"]* \
  [-library "library"] [-hdl_source "file"]* [-stimulus "file"]* [-edif "file"]* \
  [-sdc "file"]* [-ndc "file"]* [-fp_pdc "file"]* [-io_pdc "file"]* [-net_fdc "file"]* \
  [-verilog_netlist "file"]* [-vcd "file"]* [-dcf "file"]* [-pin "file"]* [-crt "file"]* [-gcf "file"]*
```
- 同じファイルを異なるライブラリへリンク不可。環境変数で根を指定可（`${MSCC_ROOT_1}/test.v`）。
- 関連: `change_link -file -path`（p.37）、`change_all_links`、`unlink_files`、`unlink_copy_locally`（リンク解除してローカルコピー）、`export_as_link`。

例:
```tcl
create_links -hdl_source {${MSCC_ROOT_1}/test.v} -library {work}
```

### build_design_hierarchy / set_root（p.37, 139）
```
build_design_hierarchy
set_root -module "module name"
```
- ソース変更で階層は無効化される → import 後に `build_design_hierarchy`。
- `set_root` は実装のトップを決める。`{top::work}` 形式も使われる（原文例 `set_root -module {prep1::work}`）。

### organize_tool_files（p.111-112）
```
organize_tool_files -tool {tool_name} [-file {absolute path to specific file} ] \
  [-module {<design_name>::work} ] -input_type {value}
```
| 引数 | 値・意味 |
|---|---|
| `-tool` | `SYNTHESIZE` / `COMPILE` / `PLACEROUTE` / `SIM_PRESYNTH` / `SIM_POSTSYNTH` / `SIM_POSTLAYOUT` / `VERIFYTIMING` |
| `-file` | 繰り返し可。**指定したリストで置き換え**。`-file {}` で何も渡さない |
| `-module` | 既定 `<root>::work` |
| `-input_type` | `constraint` / `source` / `simulation` / `stimulus` / `unknown`（必須） |

例:
```tcl
# 原文 p.27 を基に PDC 名を変更 (原文外)
organize_tool_files -tool {SYNTHESIZE} -file {./src/constraint/timing_user_constraints.sdc} \
  -module {top::work} -input_type {constraint}
organize_tool_files -tool {PLACEROUTE} \
  -file {./src/constraint/timing_user_constraints.sdc} \
  -file {./src/constraint/io_lvds.pdc} \
  -file {./src/constraint/fp_constraints.pdc} \
  -module {top::work} -input_type {constraint}
organize_tool_files -tool {VERIFYTIMING} -file {./src/constraint/timing_user_constraints.sdc} \
  -module {top::work} -input_type {constraint}
```
- 原文 p.27 は `-module {$prj_root}` と波括弧内に変数を書いており展開されない。変数を使うなら `-module "$prj_root::work"` 等にする (原文外)。

### organize_sources / organize_constraints（p.109-111）
```
organize_sources [-file filename ] [-mode "new | add | remove" ] -module value -tool "synthesis | simulation" [-use_default value ]
organize_constraints [-file "filename" ] [-mode "new | add | remove" ] [-designer_view "designer view" ] [-module "module" ] -tool "value"
```
- ソースの順序を手動管理するとき `-use_default FALSE`。

### set_as_target / unset_as_target（p.130, 143）
```
set_as_target [-type { constraint_file_type to associate } ] -file { file to set as target }
```
- `-type`: `sdc` / `io_pdc` / `fp_pdc`。GUI（I/O Editor 等）で追加した制約の書き込み先ファイルを指定。

### 制約チェック（p.42-43）
```
check_pdc_constraints -tool {designer}
check_sdc_constraints -tool {synthesis | designer | timing}
```
- PDC チェックは Compile Netlist 後に実行可。SDC チェックも Compile 後。

### 制約のエクスポート（p.72-75, 84）
```
export_io_pdc -file { absolute or relative path and name of *.pdc file }
export_fp_pdc -file { ... } [-mode { PDC_PLACE | PDC_FULL_PLACEMENT }]
export_sdc_file -file { absolute path and name of *.sdc file }
```
- いずれも **Place and Route 後**に実行。出力は参照用で、反映させるには既存 PDC へ手で足すか Constraints Editor で import。
- `export_fp_pdc -mode PDC_FULL_PLACEMENT` は I/O・バンク・領域・グローバルまで全物理制約。

例:
```tcl
export_io_pdc -file {./export/io_after_pnr.pdc}   ;# (原文外: パス変更)
```

### derive_constraints_sdc（p.28）と derive_constraints ユーティリティ（p.584-593）
```
derive_constraints_sdc
```
- Libero 内で、コンポーネント（PF_CCC、MSS 等）の SDC から派生タイミング制約を生成する（原文は Ch.2 のフロー例でのみ登場し、独立した節はない）。
- Libero 外の単体ツール `<libero>/bin{64}/derive_constraints derive.tcl` は Ch.17 の専用コマンド（`set_device -family -die -speed`、`read_verilog`、`read_vhdl`、`set_top_level`、`read_sdc -component`、`derive_constraints`、`write_sdc`、`write_pdc`）を使う。

## Anti-patterns
- **import しただけで organize しない**: Enhanced Constraint Flow ではツールに渡らない
- **`-pdc` を使う**: 廃止済み。I/O は `-io_pdc`、配置は `-fp_pdc`
- **organize_tool_files で 1 ファイルだけ渡す**: 既存の関連付けが置き換わる。必要なファイルを全部 `-file` で並べる
- **export_io_pdc の出力を元ファイルと思い込む**: 参照用。ソースの PDC を直す

## Key Takeaways
1. 追加の順番: `import_files` → `build_design_hierarchy` → `set_root` → `organize_tool_files`
2. I/O PDC は PLACEROUTE にだけ渡す。SDC は SYNTHESIZE / PLACEROUTE / VERIFYTIMING の 3 つ
3. コンポーネント由来のクロック制約は `derive_constraints_sdc` で作る
4. 制約の正しさは `check_pdc_constraints` / `check_sdc_constraints` で事前確認

## Connects To
- [tcl-02-project-device.md](tcl-02-project-device.md): `-use_enhanced_constraint_flow`
- [tcl-05-design-flow.md](tcl-05-design-flow.md): 制約を使うツールの実行
- PDC の中身（`set_io` 等）は PDC/I/O 制約資料の章（別担当）
