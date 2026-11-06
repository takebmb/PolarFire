# Libero Tcl コマンド索引 (1/2): Project Manager / SmartDesign / HDL / Command Tools / MSS（PDF p.32-262）

> 出典: *Libero SoC v2026.1 Tcl Command Reference Guide*（DS50003101W）。頁は PDF 頁（= 本書の頁番号）。
> ★ = 詳細を章ファイルに記載（01: scripting-basics、02: project-device、03: files-constraints、04: smartdesign-components、05: design-flow、06: memory-init、07: program-export）。
> 説明は原文 Description の要約。

## Ch.3 Project Manager Tcl Commands（p.32-149）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| add_file_to_library | ファイルをライブラリへ追加 | 32 |
| add_library | VHDL ライブラリを追加 | 32 |
| add_modelsim_path | ModelSim シミュレーションライブラリを追加 | 33 |
| add_profile | ツールプロファイル（外部ツール）を追加 | 34 |
| associate_stimulus | テストベンチ（stimulus）を関連付け | 36 |
| build_design_hierarchy ★03 | Design/Stimulus 階層を再構築 | 37 |
| change_link | リンクファイルの参照先を変更 | 37 |
| change_all_links | 全リンクファイルの参照先を変更 | 38 |
| change_vault_location | IP Vault の場所を変更 | 39 |
| check_fdc_constraints | FDC（Synplify ネットリスト制約）を検査 | 40 |
| check_hdl | HDL ファイルの妥当性を検査 | 40 |
| check_ndc_constraints | NDC（Compile ネットリスト制約）を検査 | 41 |
| check_pdc_constraints ★03 | P&R 用 PDC（I/O・FP）を検査 | 42 |
| check_sdc_constraints ★03 | SDC を検査（synthesis/designer/timing） | 43 |
| cleanall_tool ★05 | 指定ツールまでクリーン、下流を Out of Date に | 43 |
| close_design | SmartDesign コンポーネントを閉じる | 44 |
| close_project ★02 | プロジェクトを閉じる（`-save`） | 45 |
| configure_core ★04 | 既存コアコンポーネントの構成変更 | 46 |
| create_and_configure_core ★04 | カタログ IP から構成済みコンポーネントを作成 | 47 |
| configure_tool ★05 | 任意ツールのパラメータ設定 | 48 |
| create_links ★03 | 外部ファイルへのリンクを作成 | 50 |
| create_set | タイミング解析用パスセットを作成 | 51 |
| create_smartdesign ★04 | SmartDesign を作成 | 53 |
| delete_component | コンポーネントを削除 | 53 |
| delete_files | プロジェクトからファイル削除（`-from_disk`） | 54 |
| download_core | コアを Vault へダウンロード | 55 |
| download_latest_cores | 最新コアを一括ダウンロード | 56 |
| edit_profile | ツールプロファイル編集 | 57 |
| edit_post_layout_design | P&R をやり直さず I/O SI パラメータ・外部タイミングを調整 | 58 |
| export_as_link | ファイルを別ディレクトリへ書き出しリンク化 | 59 |
| export_ba_files | バックアノテートファイル出力 | 60 |
| export_bitstream_file ★07 | ビットストリームを出力 | 61 |
| export_bsdl_file | BSDL（.bsd）を出力 | 68 |
| export_component_to_tcl ★04 | コンポーネントを再生成する Tcl を出力 | 69 |
| export_design_summary | ルート SmartDesign の HTML サマリ出力 | 70 |
| export_firmware | ファームウェア構成データ・SoftConsole プロジェクトを出力 | 71 |
| export_fp_pdc ★03 | フロアプラン PDC を出力（P&R 後） | 72 |
| export_ibis_file | IBIS モデル出力 | 73 |
| export_interrupt_map ★04 | 割込み接続マップを JSON 出力 | 73 |
| export_io_pdc ★03 | I/O PDC を出力（P&R 後） | 74 |
| export_job_data | Job Manager 用 JDC を出力 | 75 |
| export_netlist_file | Compile 後のネットリスト出力 | 77 |
| export_parameter_report | コンポーネントと Tcl パラメータの XML レポート | 77 |
| export_pin_reports | ピンレポート出力 | 78 |
| export_profiles | ツールプロファイルを .ini 出力 | 79 |
| export_prog_job ★07 | FlashPro Express ジョブ（.job）出力 | 80 |
| export_script ★01 | 現セッションの Tcl を出力 | 83 |
| export_sdc_file ★03 | SDC を出力 | 84 |
| export_smart_debug_data | SmartDebug 用 .ddc を出力 | 85 |
| force_update_design_flow ★05 | Synth/P&R の無効状態を強制 Pass に | 87 |
| generate_component ★04 | SmartDesign/コアの HDL を生成 | 88 |
| generate_sdc_constraint_coverage ★05 | SDC 制約カバレッジレポート | 89 |
| get_libero_release ★02 | リリース番号を返す | 90 |
| get_libero_version ★02 | バージョン番号を返す | 91 |
| get_tool_options ★05 | ツール設定値を返す | 92 |
| get_tool_state ★05 | ツール状態を返す | 94 |
| import_component | .cxf コンポーネントを取り込み | 96 |
| import_component_data | MDDR/FDDR/SerDes 初期化データ取り込み（SmartFusion 2 系） | 96 |
| import_files ★03 | ソース・制約を取り込み | 98 |
| import_mss_component ★04 | MSS（.cxz）を取り込み（PolarFire SoC） | 101 |
| is_synthesis_enabled | 合成可能か（1/0）を返す | 101 |
| new_project ★02 | 新規プロジェクト作成 | 101 |
| one_way_passcode | One Way Passcode（OWP）の有効/無効（PolarFire SoC） | 106 |
| open_project ★02 | プロジェクトを開く | 107 |
| open_smartdesign ★04 | SmartDesign を開く | 108 |
| organize_constraints ★03 | 制約ファイルの関連付け・順序 | 109 |
| organize_sources ★03 | ソースファイル順序の設定 | 110 |
| organize_tool_files ★03 | ツールごとに使うファイルを指定 | 111 |
| project_general_settings ★02 | リンクファイルの相対/絶対設定 | 112 |
| project_settings ★02 | プロジェクトのフロー設定変更 | 113 |
| publish_block | デザインブロックを公開 | 116 |
| refresh | プロジェクト更新・階層再構築・リンク確認 | 118 |
| remove_core | コアを削除 | 118 |
| remove_library | VHDL ライブラリを削除 | 119 |
| remove_profile | ツールプロファイル削除 | 120 |
| rename_file | ファイルをコピー/リネーム | 121 |
| rename_library | VHDL ライブラリ名変更 | 121 |
| run_tool ★05 | ツールを実行 | 122 |
| save_log | ログを保存 | 125 |
| save_project ★02 | プロジェクト保存 | 125 |
| save_project_as ★02 | 名前を付けて保存 | 126 |
| save_smartdesign ★04 | SmartDesign 保存 | 127 |
| select_profile | ツールプロファイル選択 | 128 |
| set_actel_lib_options | シミュレーション用デバイスライブラリ指定 | 128 |
| set_active_testbench | アクティブなテストベンチ指定 | 129 |
| set_as_target ★03 | 新規制約の書込み先 SDC/PDC を指定 | 130 |
| set_device ★02 | デバイス（die/package/speed 等）変更 | 131 |
| set_global_include_file | グローバル include ファイル指定 | 134 |
| set_global_include_path_order | グローバル include パス順序 | 135 |
| set_modelsim_options | ModelSim オプション設定 | 135 |
| set_option | モジュール単位の合成/FHB オプション | 138 |
| set_root ★03 | ルート（トップ）モジュール指定 | 139 |
| set_user_lib_options | ユーザライブラリのシミュレーション設定 | 140 |
| smartdesign | SmartDesign DRC（幅不一致等）の重大度切替 | 141 |
| unlink_copy_locally | リンク解除してローカルコピー | 141 |
| unlink_files | リンク解除 | 142 |
| unset_as_target | 制約書込み先の解除 | 143 |
| unset_global_include_file | グローバル include 解除 | 144 |
| update_component_version | インスタンスのコンポーネント版を更新（接続維持） | 144 |
| use_source_file | モジュールを定義するファイルを指定 | 145 |
| update_and_run_tool ★05 | 変更があれば更新して実行 | 146 |

## Ch.4 SmartDesign Tcl Commands（p.150-210）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| create_smartdesign_testbench | 空の SmartDesign テストベンチ作成 | 150 |
| sd_add_pins_to_group | ピングループへピン追加 | 150 |
| sd_apply_presentation | 表示レイアウトをファイルから適用 | 151 |
| sd_clear_pin_attributes | ピン属性（反転・未使用等）をクリア | 152 |
| sd_configure_core_instance ★04 | 直接インスタンス化したコアの構成 | 153 |
| sd_connect_instance_pins_to_ports ★04 | インスタンス全ピンをトップポート化 | 154 |
| sd_connect_net_to_pins ★04 | 名前付きネットにピン/ポートを接続 | 155 |
| sd_connect_pins_to_constant ★04 | ピンを GND/VCC/定数へ接続 | 156 |
| sd_connect_pins ★04 | ピン/ポート同士を接続 | 157 |
| sd_connect_pin_to_port ★04 | ピンをトップポートへ昇格（Promote to Top） | 158 |
| sd_copy_paste | ポート・ネット・インスタンスのコピー＆ペースト | 159 |
| sd_create_bif_net | バスインターフェース（BIF）ネット作成 | 161 |
| sd_create_bif_port | BIF ポート作成 | 162 |
| sd_create_bus_net ★04 | バスネット作成 | 163 |
| sd_create_bus_port ★04 | バスポート作成 | 165 |
| sd_create_pin_group | ピングループ作成（表示整理用） | 166 |
| sd_create_pin_slices ★04 | バスをスライス分割 | 167 |
| sd_create_scalar_net ★04 | スカラネット作成 | 168 |
| sd_create_scalar_port ★04 | スカラポート作成 | 169 |
| sd_cut_paste | 別 SmartDesign へカット＆ペースト | 170 |
| sd_delete_bif_pin | BIF ピン削除 | 171 |
| sd_delete_instances | インスタンス削除 | 173 |
| sd_delete_nets | ネット削除 | 173 |
| sd_delete_pin_group | ピングループ削除 | 175 |
| sd_delete_pin_slices | スライス削除 | 176 |
| sd_delete_ports | ポート削除 | 177 |
| sd_disconnect_instance | インスタンスの全接続を解除 | 177 |
| sd_disconnect_pins | ピン/ポートをネットから外す | 178 |
| sd_duplicate_instance | インスタンス複製 | 180 |
| sd_hide_bif_pins | BIF 内部ピンを隠す | 181 |
| sd_instantiate_component ★04 | コンポーネントをインスタンス化 | 182 |
| sd_instantiate_core ★04 | カタログ IP を直接インスタンス化 | 183 |
| sd_instantiate_hdl_core ★04 | HDL+ コアをインスタンス化 | 184 |
| sd_instantiate_hdl_module ★04 | HDL モジュールをインスタンス化 | 185 |
| sd_instantiate_macro ★04 | プリミティブマクロをインスタンス化 | 186 |
| sd_invert_pins ★04 | ピン/ポートを反転 | 187 |
| sd_mark_pins_unused ★04 | 出力ピンを未使用にマーク | 188 |
| sd_remove_pins_from_group | ピングループからピン除去 | 189 |
| sd_rename_instance | インスタンス名変更 | 190 |
| sd_rename_net | ネット名変更 | 191 |
| sd_rename_pin_group | ピングループ名変更 | 192 |
| sd_rename_port | ポート名変更 | 193 |
| sd_replace_component | インスタンスの定義を別コンポーネントへ差替え | 194 |
| sd_reset_layout | キャンバスのレイアウトをリセット | 196 |
| sd_set_port_synth_attr | ポートに合成属性を付与 | 196 |
| sd_set_net_synth_attr | ネットに合成属性を付与 | 198 |
| sd_set_inst_synth_attr | インスタンスに合成属性を付与 | 199 |
| sd_set_comp_synth_attr | SmartDesign モジュールに合成属性を付与 | 201 |
| sd_remove_port_synth_attr | ポートの合成属性を削除 | 202 |
| sd_remove_net_synth_attr | ネットの合成属性を削除 | 203 |
| sd_remove_inst_synth_attr | インスタンスの合成属性を削除 | 205 |
| sd_remove_comp_synth_attr | モジュールの合成属性を削除 | 206 |
| sd_save_core_instance_config ★04 | 直接インスタンスの構成を保存 | 207 |
| sd_show_bif_pins | BIF 内部ピンを露出 | 208 |
| sd_update_instance ★04 | インスタンスを最新定義に更新 | 209 |

## Ch.5 HDL Tcl Commands（p.211-219）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| create_hdl_core | HDL モジュールから HDL+ コアを作成 | 211 |
| remove_hdl_core | HDL+ コアを削除 | 212 |
| hdl_core_add_bif | HDL+ コアにバスインターフェース追加 | 212 |
| hdl_core_assign_bif_signal | BIF 信号をポートに割当て | 213 |
| hdl_core_delete_parameters | HDL+ コアのパラメータ削除 | 214 |
| hdl_core_extract_ports_and_parameters | ポート・generic を自動抽出 | 215 |
| hdl_core_remove_bif | BIF 削除 | 216 |
| hdl_core_rename_bif | BIF 名変更 | 217 |
| hdl_core_unassign_bif_signal | BIF 信号割当て解除 | 218 |

## Ch.6 Command Tools（configure_tool / run_tool の -name, p.220-260）
| ツール名 | 1 行説明 | PDF 頁 |
|---|---|---|
| CONFIGURE_ACTIONS_PROCEDURES ★07 | アクションの任意/推奨プロシージャ設定 | 220 |
| CONFIGURE_CHAIN ★05 | FlashPro コマンドで JTAG チェーン設定（run_tool） | 221 |
| CONFIGURE_PROG_OPTIONS ★07 | design_version / back_level / silicon_signature | 222 |
| EXPORTNETLIST ★05 | ネットリスト出力 | 224 |
| EXPORTSDF ★05 | バックアノテート出力（DELAY_TYPE） | 225 |
| FLASH_FREEZE | Flash*Freeze 時の RAM 状態・MSS クロック | 226 |
| GENERATEDEBUGDATA ★05 | SmartDebug 用データ生成 | 227 |
| GENERATEPROGRAMMINGDATA ★06 | FPGA Array Data 生成 | 227 |
| GENERATEPROGRAMMINGFILE ★07 | ビットストリーム生成（コンポーネント選択） | 228 |
| INIT_LOCK | レジスタロックビット設定（SmartFusion 2/IGLOO 2） | 230 |
| IO_PROGRAMMING_STATE | 書込み中の I/O 状態（.ios）を読込み | 232 |
| PLACEROUTE ★05 | 配置配線の設定と実行 | 233 |
| PROGRAMDEVICE ★07 | デバイス書込みアクション設定と実行 | 237 |
| PROGRAMMER_INFO ★07 | FlashPro3〜6 の TCK/SCK・Vpump 設定 | 239 |
| PROGRAM_SPI_FLASH_IMAGE ★06 | SPI Flash イメージの書込み/消去/検証 | 242 |
| SIM_PRESYNTH ★05 | 合成前シミュレーション | 243 |
| SIM_POSTSYNTH ★05 | 合成後シミュレーション | 244 |
| SIM_POSTLAYOUT ★05 | レイアウト後シミュレーション | 244 |
| SPM | セキュリティ（鍵・ロック）設定 | 245 |
| SPM_OTP | 恒久ロック設定（PolarFire SoC では利用不可と記載） | 250 |
| SYNTHESIZE ★05 | 合成の設定と実行 | 251 |
| UPDATE_ENVM | eNVM クライアント更新（SmartFusion 2/IGLOO 2） | 254 |
| USER_PROG_DATA | design_version/silicon_signature（旧ツール、廃止扱い） | 255 |
| VERIFYPOWER ★05 | 電力解析（SmartPower スクリプト） | 256 |
| VERIFYTIMING ★05 | タイミング解析の設定と実行 | 257 |

## Ch.7 MSS Tcl Commands（p.261-262）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| mss_configure_envm ★04 | MSS の ENVM コアへ .cfg でクライアント指定 | 261 |
| mss_configure_instance ★04 | MSS 内コアインスタンスのパラメータ設定 | 261 |
| mss_disable_instance ★04 | MSS 内インスタンス無効化 | 261 |
| mss_enable_instance ★04 | MSS 内インスタンス有効化 | 262 |
| mss_save_instance_config ★04 | MSS 内インスタンス構成を保存 | 262 |
