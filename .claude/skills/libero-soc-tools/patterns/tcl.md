# パターン: Tcl スクリプト

(原文外) は資料外から補った部分。

## Tcl だけでプロジェクト作成から .job 出力まで
**When to use**: 再現性のあるビルドを CI やコマンドラインで回したいとき。リファレンスデザインの全自動版を自作するとき。
**How**: `libero script:build.tcl SCRIPT_ARGS:... logfile:build.log` で、`new_project`（-family PolarFireSoC -die MPFS095T -package FCSG325 -speed -1 -die_voltage 1.0 -part_range EXT）→ `import_files`（-hdl_source / -sdc / -io_pdc / -fp_pdc）→ `build_design_hierarchy` → `set_root` → `derive_constraints_sdc` → `organize_tool_files`（SYNTHESIZE / PLACEROUTE / VERIFYTIMING）→ `run_tool` を SYNTHESIZE, PLACEROUTE, VERIFYTIMING, GENERATEPROGRAMMINGDATA の順 → `generate_design_initialization_data` → `run_tool -name {GENERATEPROGRAMMINGFILE}` → `export_prog_job` → `close_project -save 1`。各 run_tool は `if {[catch {...}]}` で判定。詳細: tcl-01, tcl-02, tcl-03, tcl-05, tcl-06, tcl-07。
**Trade-offs**: GUI 不要で速く差分管理しやすい。一方 MSS 構成やコア VLNV の版はインストール環境に依存し、family 文字列等の細部は `export_script` で実機確認が要る。

## 既存プロジェクトに HDL と I/O PDC を追加して再ビルド
**When to use**: リファレンスデザインで生成したプロジェクトにファブリック回路（例: LVDS 受信）を足すとき。
**How**: `open_project -file {...prjx}` → `import_files -hdl_source {lvds_rx.v}` と `import_files -io_pdc {io_lvds.pdc}` → `build_design_hierarchy` → SmartDesign に `sd_instantiate_hdl_module`・`sd_create_scalar_port`・`sd_connect_pins` → `save_smartdesign` → `generate_component` → **`organize_tool_files -tool {PLACEROUTE}` に既存 PDC/SDC と新 PDC を全部並べ直す** → `run_tool -name {PLACEROUTE}` → `run_tool -name {VERIFYTIMING}` → `save_project`。詳細: tcl-03, tcl-04。
**Trade-offs**: プロジェクトを作り直さないので速い。ただし organize_tool_files はリストを置き換えるため既存ファイルの書き漏れで制約が外れる。再現性を重視するなら生成スクリプト側に同じ処理を追記する。

## GUI 操作を Tcl に落としてスクリプトへ移植
**When to use**: SmartDesign 接続やコア設定を GUI で試してから生成スクリプトに反映したいとき。
**How**: GUI で変更 → `export_script -file {./exported.tcl} -relative_path 1`（セッション全体）または `export_component_to_tcl`（コンポーネント単位）→ 必要部分を生成スクリプトへコピー。`sd_connect_pin_to_port` 等の「対話操作」はコンポーネント Tcl に出ないことがある。詳細: tcl-01, tcl-04。
**Trade-offs**: 引数を手で調べる必要がなく正確。出力には不要な操作も含まれるので取捨選択が要る。

## eNVM に HSS（ブートローダ）を入れて書き込む
**When to use**: リファレンスデザインの HSS_UPDATE 相当を自分のスクリプトで行うとき。
**How**: eNVM.cfg に `set_plain_text_client`（client_name / number_of_bytes / content_type MEMORY_FILE / content_file / start_page / reprogram 1）を書く → `configure_envm -cfg_file {ENVM.cfg}` → `generate_design_initialization_data` → `configure_tool -name {GENERATEPROGRAMMINGFILE} -params {program_envm:true} -params {program_fabric_snvm:true}` → `run_tool -name {GENERATEPROGRAMMINGFILE}` → `run_tool -name {PROGRAMDEVICE}` か `export_prog_job -bitstream_file_components {FABRIC SNVM ENVM}`。詳細: tcl-06, tcl-07。
**Trade-offs**: Libero 内で完結。HSS の hex 形式・ベースアドレス指定は原文の記述（Microchip-Binary のみ）と実スクリプトでずれがありうるので、リファレンスデザインの .cfg を手本にする（原文外）。

## タイミング未達で止まるフロー
**When to use**: 無人ビルドで違反のあるビットストリームを出したくないとき。
**How**: `configure_tool -name {VERIFYTIMING} -params {FORMAT:TEXT} ...` → `run_tool -name {VERIFYTIMING}` → `set st [get_tool_state -name {VERIFYTIMING}]` → 違反を示す文字列（Tool has timing violations / not been met）なら `error`。詳細: tcl-05。
**Trade-offs**: 安全。判定は状態文字列の照合なので、Libero の版で文言が変わると要修正（原文外）。
