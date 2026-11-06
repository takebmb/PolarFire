# Libero Tcl コマンド索引 (2/2): SmartTime / SmartPower / Programming / FlashPro Express / JTAG / SmartDebug / その他（PDF p.263-593）

> 出典: *Libero SoC v2026.1 Tcl Command Reference Guide*（DS50003101W）。頁は PDF 頁。★ = 章ファイルに詳細（06: programming、03: files-constraints、05: design-flow）。
> Ch.8 の SDC 系（create_clock 等）の詳細は SDC 担当の章を参照。

## Ch.8 SmartTime Tcl Commands（p.263-339）— `run_tool -name {VERIFYTIMING} -script` 内や SmartTime で使う
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| all_inputs | 全入力/双方向ピンのオブジェクト | 263 |
| all_outputs | 全出力/双方向ピンのオブジェクト | 263 |
| all_registers | レジスタのピン/セル | 264 |
| check_constraints | 現シナリオのタイミング制約を検査 | 265 |
| clone_scenario | シナリオを複製 | 266 |
| create_clock | クロック制約（仮想クロック可） | 266 |
| create_generated_clock | 生成クロック制約 | 268 |
| create_scenario | タイミングシナリオ作成 | 272 |
| create_set | 解析用パスセット作成 | 273 |
| expand_path | パス詳細を表示 | 274 |
| get_cells | インスタンス集合を返す | 276 |
| get_clocks | クロック集合を返す | 277 |
| get_current_scenario | 現シナリオ名 | 277 |
| get_nets | ネット集合を返す | 278 |
| get_pins | ピン集合を返す | 279 |
| get_ports | ポート集合を返す | 280 |
| list_clock_groups | クロックグループ一覧 | 281 |
| list_clock_latencies | クロックレイテンシ一覧 | 282 |
| list_clock_uncertainties | クロック不確かさ一覧 | 282 |
| list_clocks | クロック制約一覧 | 283 |
| list_disable_timings | disable timing 一覧 | 284 |
| list_false_paths | フォールスパス一覧 | 284 |
| list_generated_clocks | 生成クロック一覧 | 285 |
| list_input_delays | 入力遅延一覧 | 285 |
| list_max_delays | 最大遅延制約一覧 | 286 |
| list_min_delays | 最小遅延制約一覧 | 287 |
| list_multicycle_paths | マルチサイクル一覧 | 287 |
| list_objects | オブジェクト一覧 | 288 |
| list_output_delays | 出力遅延一覧 | 289 |
| list_paths | ワースト n パスを返す | 289 |
| list_scenario | シナリオ一覧 | 291 |
| read_sdc | SDC を読込みシナリオへ追加 | 291 |
| remove_all_constraints | 全タイミング制約を除去 | 292 |
| remove_clock | クロック制約除去 | 293 |
| remove_clock_groups | クロックグループ除去 | 294 |
| remove_clock_latency | レイテンシ除去 | 295 |
| remove_clock_uncertainty | 不確かさ除去 | 295 |
| remove_disable_timing | disable timing 除去 | 297 |
| remove_false_path | フォールスパス除去 | 298 |
| remove_generated_clock | 生成クロック除去 | 299 |
| remove_input_delay | 入力遅延除去 | 300 |
| remove_max_delay | 最大遅延除去 | 301 |
| remove_min_delay | 最小遅延除去 | 302 |
| remove_multicycle_path | マルチサイクル除去 | 303 |
| remove_output_delay | 出力遅延除去 | 304 |
| remove_scenario | シナリオ削除 | 305 |
| remove_set | ユーザパスセット削除 | 306 |
| rename_scenario | シナリオ名変更 | 307 |
| report | タイミング/違反等のレポート出力 | 308 |
| save | SmartTime の変更を保存 | 311 |
| set_clock_groups | クロックグループ間の解析を無効化 | 311 |
| set_clock_latency | 外部クロックソース遅延 | 313 |
| set_clock_to_output | 出力の FPGA 内タイミング予算 | 315 |
| set_clock_uncertainty | クロック不確かさ | 317 |
| set_current_scenario | 使用シナリオを指定 | 319 |
| set_disable_timing | セル内タイミングアークを無効化 | 320 |
| set_external_check | 入力の外部セットアップ/ホールド | 321 |
| set_external_delay | チップ外のポート間遅延 | 321 |
| set_false_path | フォールスパス | 322 |
| set_input_delay | 入力遅延 | 324 |
| set_input_jitter | 入力ジッタ | 326 |
| set_max_delay | 最大遅延 | 327 |
| set_min_delay | 最小遅延 | 329 |
| set_multicycle_path | マルチサイクルパス | 331 |
| set_options | SmartTime 解析オプション | 333 |
| set_output_delay | 出力遅延 | 335 |
| set_system_jitter | システムジッタ上書き | 338 |
| write_sdc | SDC へ書き出し（`-scenario`） | 338 |

## Ch.9 SmartPower Tcl Commands（p.340-412）— `run_tool -name {VERIFYPOWER} -script` 内で使う
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| smartpower_add_new_custom_mode | カスタム動作モード作成 | 340 |
| smartpower_add_new_scenario | カスタムシナリオ作成 | 341 |
| smartpower_add_pin_in_domain | クロック/セットドメインにピン追加 | 342 |
| smartpower_battery_settings | バッテリ容量設定 | 343 |
| smartpower_change_clock_statistics | ドメインの既定周波数・確率変更 | 344 |
| smartpower_change_setofpin_statistics | ピンセットの既定周波数・確率変更 | 346 |
| smartpower_commit | 変更を設計ファイルへ保存 | 347 |
| smartpower_compute_vectorless | ベクタレス解析 | 348 |
| smartpower_create_domain | クロック/セットドメイン作成 | 348 |
| smartpower_edit_custom_mode | カスタムモード編集 | 349 |
| smartpower_edit_scenario | シナリオ編集 | 350 |
| smartpower_export_mpe_report | MPE レポート（XML）出力 | 351 |
| smartpower_get_temperature | 温度取得（原文説明未記入） | 352 |
| smartpower_get_tetaja | θJA 取得（原文説明未記入） | 353 |
| smartpower_get_thermalmode | 熱モード取得（原文説明未記入） | 354 |
| smartpower_import_vcd | VCD 取り込み | 354 |
| smartpower_init_do | 周波数・確率の一括初期化 | 356 |
| smartpower_initialize_clock_with_constraints | 制約からクロック/データ周波数初期化 | 358 |
| smartpower_init_set_clocks_options | 全クロックドメインの周波数初期化 | 359 |
| smartpower_init_set_combinational_options | 組合せ出力の初期化 | 361 |
| smartpower_init_setofpins_values | ピンセットの初期化 | 362 |
| smartpower_init_set_othersets_options | その他セットの初期化 | 363 |
| smartpower_init_set_primaryinputs_options | 一次入力の初期化 | 364 |
| smartpower_init_set_registers_options | レジスタ出力の初期化 | 365 |
| smartpower_init_set_set_reset_options | セット/リセットネットの初期化 | 366 |
| smartpower_remove_all_annotations | モードの全注釈除去 | 367 |
| smartpower_remove_custom_mode | カスタムモード削除 | 368 |
| smartpower_remove_domain | ドメイン削除 | 369 |
| smartpower_remove_file | VCD をモードから除去 | 370 |
| smartpower_remove_pin_frequency | ピン周波数指定を除去 | 371 |
| smartpower_remove_pin_of_domain | ドメインからピン除去 | 371 |
| smartpower_remove_pin_probability | ピン確率指定を除去 | 373 |
| smartpower_remove_scenario | シナリオ削除 | 373 |
| smartpower_report_power | 電力レポート | 374 |
| smartpower_report_power_activity_map | アクティビティ・ハザードレポート | 380 |
| smartpower_report_power_history | 電力履歴レポート（原文説明未記入） | 383 |
| smartpower_report_power_peak_analyzer | サイクル精度のピーク電力レポート | 384 |
| smartpower_report_power_sequencer | シナリオ電力レポート | 387 |
| smartpower_restore | コミット済み情報の復元 | 390 |
| smartpower_set_cooling | 冷却方式設定 | 390 |
| smartpower_set_default_io_enable_rate | I/O イネーブル率既定値（原文説明未記入） | 391 |
| smartpower_set_mode_for_analysis | サイクル精度解析のモード | 392 |
| smartpower_set_mode_for_pdpr | PDPR 用動作モード | 393 |
| smartpower_set_operating_conditions | 動作条件設定（2 節あり） | 394, 395 |
| smartpower_set_pin_frequency | ピン周波数（MHz） | 397 |
| smartpower_set_pin_probability | ピン確率 | 398 |
| smartpower_set_preference | 単位・モード等の設定 | 399 |
| smartpower_set_process | プロセス条件 | 400 |
| smartpower_set_scenario_for_analysis | 解析シナリオ指定 | 401 |
| smartpower_set_tambient | 周囲温度（原文説明未記入） | 401 |
| smartpower_set_temperature_opcond | 温度条件の選択 | 402 |
| smartpower_set_thermalmode | 接合温度の計算方法 | 403 |
| smartpower_set_voltage_opcond | 電圧条件の選択 | 404 |
| smartpower_temperature_opcond_set_design_wide | 設計全体の温度条件 | 405 |
| smartpower_temperature_opcond_set_mode_specific | モード別温度条件 | 407 |
| smartpower_voltage_opcond_set_design_wide | 設計全体の電圧条件 | 408 |
| smartpower_voltage_opcond_set_mode_specific | モード別電圧条件 | 410 |

## Ch.10 Programming and Configuration Tcl Commands（p.413-437）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| configure_design_initialization_data ★06 | 初期化データの段構成・アドレス・タイムアウト | 413 |
| configure_ram ★06 | Fabric RAM 初期化クライアント（.cfg） | 416 |
| configure_envm ★06 | eNVM クライアント（.cfg） | 416 |
| configure_snvm ★06 | sNVM クライアント（.cfg） | 417 |
| configure_spiflash ★06 | SPI Flash クライアント（.cfg） | 418 |
| configure_uprom | uPROM クライアント（.cfg） | 419 |
| export_spiflash_image ★06 | SPI Flash イメージ出力（BIN/STP） | 419 |
| generate_design_initialization_data ★06 | メモリファイル生成・クライアント追加・検証 | 420 |
| generate_initialization_mem_files | 初期化用メモリファイル生成パラメータ | 421 |
| modified_client | Fabric RAM .cfg 内の変更クライアント記述 | 422 |
| nvm_update_serialization_client | eNVM シリアライズクライアント更新（SmartFusion 2 系） | 423 |
| nvm_update_storage_client | eNVM ストレージクライアント更新（SmartFusion 2 系） | 424 |
| remove_permanent_locks | SPM_OTP のロックを全解除 | 426 |
| select_programmer ★07 | 使用プログラマを 1 台に限定 | 426 |
| set_auto_update_mode ★06 | SPI Flash 自動更新（spiflash.cfg 内） | 427 |
| set_manufacturer ★06 | SPI Flash メーカー指定 | 428 |
| set_programming_interface ★07 | JTAG / SPI_SLAVE 選択 | 428 |
| set_cipher_text_auth_client ★06 | sNVM 暗号文認証クライアント | 429 |
| set_plain_text_auth_client ★06 | sNVM 平文認証クライアント | 430 |
| set_plain_text_client ★06 | sNVM/eNVM 平文クライアント | 432 |
| set_usk_client ★06 | sNVM USK クライアント | 433 |
| set_client ★06 | SPI Flash / Fabric RAM クライアント | 434 |
| set_data_storage_client | uPROM データストレージクライアント | 435 |
| update_storage_client | RTG4 uPROM ストレージクライアント更新 | 436 |

## Ch.11 FlashPro Express Tcl Commands（p.438-462）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| close_project | FPExpress プロジェクトを閉じる | 438 |
| configure_flashpro3_prg | FlashPro3 設定 | 438 |
| configure_flashpro4_prg | FlashPro4 設定 | 439 |
| configure_flashpro5_prg | FlashPro5 設定 | 441 |
| configure_flashpro6_prg | FlashPro6 設定 | 441 |
| create_job_project ★07 | .job から FPExpress プロジェクト作成 | 443 |
| auto_construct_job_project | 自動チェーン構築でジョブプロジェクト作成（開発者モード） | 443 |
| get_connected_programmers | 接続中プログラマ一覧 | 444 |
| enable_prg | プログラマ有効/無効 | 446 |
| enable_prg_type | 種別単位でプログラマ有効/無効 | 446 |
| export_script | FPExpress セッションの Tcl 出力 | 447 |
| open_project | FPExpress プロジェクト（.pro）を開く | 448 |
| ping_prg | プログラマへ ping | 449 |
| refresh_prg_list | プログラマ一覧を更新 | 449 |
| remove_prg | プログラマを一覧から削除 | 450 |
| run_selected_actions ★07 | 選択アクションを実行 | 451 |
| save_log | ログ保存 | 452 |
| save_project | プロジェクト保存 | 452 |
| scan_chain_prg | チェーンをスキャン | 453 |
| self_test_prg | プログラマ自己診断 | 454 |
| set_fpexpress_mode | Operator / Developer モード切替 | 454 |
| set_hsm_params | HSM パラメータ設定 | 455 |
| set_prg_name | プログラマ名変更 | 456 |
| set_programming_action ★07 | デバイスのアクション選択（PROGRAM 等） | 457 |
| set_programming_file | デバイスの書込みファイル（stp/ppd）指定 | 458 |
| set_programming_interface | JTAG / SPI_SLAVE 選択 | 459 |
| set_spi_flash_action | SPI Flash の書込み/検証/消去指定 | 460 |
| set_spi_flash_file | SPI Flash 書込みファイル（.bin）指定 | 460 |
| update_fp6_programmers | FlashPro6 ファーム更新 | 461 |

## Ch.12 Configure JTAG Chain Tcl Commands（p.463-477）— `run_tool -name {CONFIGURE_CHAIN} -script` 内で使う
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| add_actel_device | Microchip デバイスをチェーンへ追加 | 463 |
| select_libero_design_device | Libero 設計対象デバイスを選択 | 464 |
| copy_device | デバイスをクリップボードへコピー | 464 |
| cut_device | デバイスを切り取り | 465 |
| enable_device | デバイス有効/無効（無効はバイパス） | 466 |
| remove_device | デバイス削除 | 467 |
| set_device_name | デバイスのユーザ名変更 | 468 |
| add_non_actel_device | 他社デバイス追加（BSDL か IR/TCK） | 468 |
| add_non_actel_device_to_database | BSDL で他社デバイスを DB 登録 | 469 |
| construct_chain_automatically | チェーン自動構築 | 470 |
| paste_device | クリップボードから貼付け | 471 |
| remove_non_actel_device_from_database | 他社デバイスを DB から削除 | 472 |
| set_bsdl_file | 他社デバイスに BSDL 指定 | 473 |
| set_device_ir | 他社デバイスの IR 長 | 474 |
| set_device_order | チェーン順序設定 | 474 |
| set_device_tck | 他社デバイスの最大 TCK | 475 |
| set_device_type | Microchip デバイスのファミリ変更 | 476 |

## Ch.13 SmartDebug Tcl Commands（p.478-561）— `run_tool -name {SMARTDEBUG} -script` 内で使う
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| add_probe_insertion_point | プローブ挿入点追加 | 478 |
| add_to_probe_group | プローブグループへ追加 | 479 |
| check_flash_memory | フラッシュメモリ検査 | 480 |
| close_project | SmartDebug プロジェクトを閉じる | 481 |
| complete_debug_job | デバッグジョブ完了 | 482 |
| create_probe_group | プローブグループ作成 | 483 |
| debug_ddr | ファブリック DDR デバッグ | 483 |
| debug_mss_ddr | MSS DDR デバッグ | 484 |
| debug_iod | IOD デバッグ | 485 |
| ddr_read / ddr_write | DDR 読み/書き | 486, 487 |
| delete_active_probe | アクティブプローブ削除 | 488 |
| load_live_probe_list | ライブプローブ一覧読込み | 489 |
| eye_monitor_power | アイモニタ電源 | 490 |
| event_counter | イベントカウンタ | 491 |
| execute_dfe_calibration | DFE キャリブレーション | 492 |
| export_ddr_training_data / export_mss_ddr_training_data | DDR トレーニングデータ出力 | 493 |
| fhb_control | FPGA Hardware Breakpoint 制御 | 493 |
| get_programmer_info | プログラマ情報 | 495 |
| get_user_clock_frequencies | ユーザクロック周波数 | 496 |
| import_ddc_file | .ddc 取り込み | 497 |
| load_active_probe_list | アクティブプローブ一覧読込み | 498 |
| load_SI_design_defaults | SI 設計既定値読込み | 499 |
| loopback_mode / loopback_test | トランシーバループバック | 499, 500 |
| move_to_probe_group | プローブグループ移動 | 502 |
| mss_add_register / mss_read_register / mss_write_register | MSS レジスタ追加・読み・書き | 502, 503, 505 |
| mss_import_register / mss_export_register | MSS レジスタ一覧入出力 | 506, 507 |
| new_project / open_project | SmartDebug プロジェクト作成/オープン | 508, 509 |
| optimize_dfe / optimize_receiver | DFE / 受信最適化 | 509, 510 |
| pcie_config_space / pcie_ltssm_status | PCIe コンフィグ空間 / LTSSM 状態 | 511, 512 |
| plot_eye | アイ図 | 513 |
| prbs_test | PRBS テスト | 514 |
| process_job_request | ジョブ要求処理 | 515 |
| program_probe_insertion | プローブ挿入を書込み | 516 |
| read_active_probe | アクティブプローブ読出し | 517 |
| read_envm_memory | eNVM 読出し | 518 |
| read_id_code | IDCODE 読出し | 519 |
| read_device_status | デバイス状態読出し | 520 |
| read_lsram / read_usram | LSRAM / uSRAM 読出し | 521, 522 |
| read_snvm_memory | sNVM 読出し | 523 |
| read_uprom_memory / read_flash_memory | uPROM / フラッシュ読出し | 525, 526 |
| record_actions | 操作を記録 | 527 |
| remove_from_probe_group / remove_probe_insertion_point | プローブ削除系 | 528, 529 |
| rescan_programmer | プログラマ再スキャン | 530 |
| run_frequency_monitor | 周波数モニタ | 531 |
| save_active_probe_list / save_live_probe_list | プローブ一覧保存 | 532 |
| scan_ecc_memories | ECC メモリスキャン | 533 |
| select_active_probe | アクティブプローブ選択 | 534 |
| serdes_lane_reset / serdes_read_register / serdes_write_register | SerDes レーンリセット・レジスタ | 535-537 |
| set_debug_device / set_debug_programmer | 対象デバイス / プログラマ選択 | 538, 539 |
| set_hsm_params | HSM パラメータ | 539 |
| set_live_probe / unset_live_probe | ライブプローブ割当て/解除 | 540, 551 |
| signal_integrity_export / _import / _write | SI 設定の入出力・書込み | 541-543 |
| smartbert_test | SmartBERT テスト | 545 |
| static_pattern_transmit | 固定パターン送信 | 547 |
| transceiver_lane_reset | トランシーバレーンリセット | 549 |
| tvs_monitor | 温度・電圧センサ監視 | 549 |
| ungroup | グループ解除 | 550 |
| update_fp6_programmers | FlashPro6 更新 | 552 |
| write_active_probe | アクティブプローブ書込み | 552 |
| write_lsram / write_usram | LSRAM / uSRAM 書込み | 554, 556 |
| xcvr_add_register / xcvr_export_register / xcvr_import_register / xcvr_read_register / xcvr_write_register | トランシーバレジスタ操作 | 557-560 |

## Ch.14 System Builder Tcl Commands（p.562-568, SmartFusion 2/IGLOO 2 用）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| sb_configure_page | System Builder ページのパラメータ設定 | 562 |
| sb_add_core | サブシステムへコア追加 | 562 |
| sb_configure_core | 追加済みコアの構成 | 563 |
| sb_configure_core_count | コアのインスタンス数 | 563 |
| sb_move_core | コアを別サブシステムへ移動 | 564 |
| sb_enable_core / sb_disable_core | コア有効/無効 | 565 |
| sb_enable_peripheral / sb_disable_peripheral | MSS 周辺有効/無効 | 566 |
| sb_configure_peripheral | MSS 周辺構成 | 566 |
| sb_set_fic_direct_mode | FIC ダイレクトモード | 567 |
| sb_configure_envm | ENVM クライアント .cfg 指定 | 567 |
| open_sb_component / generate_sb_component / close_sb_component | System Builder コンポーネントの開く/生成/閉じる | 568 |

## Ch.15 SSN Analyzer Tcl Commands（p.569-577）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| ssn_analyzer_noise_report | ノイズレポート出力 | 569 |
| ssn_analyzer_rerun_analysis | 再解析 | 570 |
| ssn_analyzer_set_dontcare | I/O を Don't care に | 571 |
| ssn_analyzer_set_pulse_width | パルス幅設定 | 573 |
| ssn_analyzer_set_static | I/O を静的に | 574 |
| ssn_analyzer_summary_report | サマリレポート出力 | 576 |

## Ch.16 HSM Tcl Commands（p.578-583）
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| list_all_hsm_tickets | HSM チケット一覧 | 578 |
| complete_prog_job | ジョブ完了・ステータス生成 | 578 |
| get_job_status | ジョブ状態出力 | 579 |
| process_job_request | Job Manager の要求処理 | 580 |
| remove_hsm_tickets | HSM チケット削除 | 581 |
| set_hsm_params | Job Manager 用 HSM パラメータ | 583 |

## Ch.17 Derive Constraints Tcl Commands（p.584-593）— 単体ツール `derive_constraints <x>.tcl` 用 ★03
| コマンド | 1 行説明 | PDF 頁 |
|---|---|---|
| （節）Working with the derive_constraints Utility | 実行手順（device → RTL → top → コンポーネント SDC → derive → write） | 584 |
| set_device | family（PolarFire / PolarFire SoC 等）・die・speed | 585 |
| read_verilog | Verilog 読込み（`-mode system_verilog` 等） | 586 |
| read_vhdl | VHDL 追加（`-mode vhdl_2008` 等） | 587 |
| set_top_level | トップモジュール指定 | 588 |
| read_sdc | コンポーネント SDC 読込み（`-component`） | 588 |
| read_ndc | NDC 読込み（RTG4） | 589 |
| derive_constraints | コンポーネント制約を設計全体へ展開 | 590 |
| write_sdc | 派生 SDC 出力 | 590 |
| write_pdc | 派生 PDC 出力 | 591 |
| write_ndc | 派生 NDC 出力 | 592 |
| add_include_path | include 検索パス | 593 |
