# 早見表（Libero SoC v2026.1）

**GPIO LVDS（LVDS25 / LVDS33 / LVDS18G）の set_io 属性（PDC UG p.11-16）**
| 属性 | 方向 | 値 | 既定 |
|---|---|---|---|
| -io_std | — | LVDS25（vcci 2.5）/ LVDS33（3.3）/ LVDS18G（1.8） | — |
| 方向 | — | Input・Output 可、Tribuff・Inout 不可 | — |
| -ODT | In | OFF, ON, DYNAMIC（DYNAMIC=フェイルセーフ） | ON |
| -ODT_VALUE | In | 100 | 100 |
| -VCM_RANGE | In | MID, LOW（バンク内統一） | MID |
| -RES_PULL | In | Up, Down（出力は非対応） | Up |
| -OUT_DRIVE | Out | 3, 3.5, 4, 6 mA | 6 |
| -SOURCE_TERM | Out | OFF, 100 | OFF |
| -IN_DELAY / -OUT_DELAY | In / Out | OFF, 0–127, 128…254（偶数） / OFF, 0–127 | OFF |

**差動 I/O マクロ（Macro Library UG p.32-45）**
| マクロ | 入力 → 出力 | 用途 |
|---|---|---|
| INBUF_DIFF | PADP, PADN → Y | LVDS 受信 |
| OUTBUF_DIFF | D → PADP, PADN | LVDS 送信 |
| TRIBUFF_DIFF / BIBUF_DIFF | D, E(, PAD*) → PADP, PADN(, Y) | 差動トライステート/双方向（LVDS 不可） |
| CLKBUF_DIFF / GCLKBUF_DIFF | PADP, PADN(, EN) → Y | 差動クロック → グローバル網 |

**I/O 制約まわりのコマンド早見（PDC UG）**
| やりたいこと | コマンド |
|---|---|
| バンク電圧固定 | `set_iobank -bank_name <b> -vcci <V> -fixed true` |
| ピン・規格・属性 | `set_io -port_name <p> -pin_name <pin> -io_std <std> -fixed true ...` |
| ピン予約 | `reserve -pin_name "F2 B4"` |
| I/O FF 取り込み（NDC） | `set_ioff -port_name <p> -IN_REG 1` |
| 配置後に属性変更 | `edit_io -port_name <p> ...` |

### SmartTime SDC コマンド早見表（Timing Constraints Editor UG）
| 目的 | コマンド（要点） | 注意 |
|---|---|---|
| 一次 / virtual clock | `create_clock -name n -period p [-waveform {r f}] [src]` | src 省略で virtual。`-add` は別名必須 |
| 生成クロック | `create_generated_clock -name n -source ref [-multiply_by m] [-divide_by d] pin` | `-edges`/`-edge_shift` は乗除算と排他。`-phase` は遅延扱い |
| 入力遅延 | `set_input_delay d -clock c [-max\|-min] [-clock_fall] [-add_delay] ports` | DDR は fall 側に `-clock_fall -add_delay` |
| 出力遅延 | `set_output_delay d -clock c [-max\|-min] [-clock_fall] [-add_delay] ports` | max = 外部 setup 側、min = hold 側 |
| False path | `set_false_path [-from] [-through] [-to]` | `-through` は pin/net のみ |
| Max / Min delay | `set_max_delay d [-ignore_clock_latency] -from -to` | multicycle より優先 |
| Multicycle | `set_multicycle_path N [-setup\|-hold\|-setup_only] [-start\|-end] -from -to` | 無指定時 hold は 0 |
| クロック群 | `set_clock_groups -asynchronous -group {..} -group {..}` | フラグは 3 つのうち 1 つ |
| 不確かさ | `set_clock_uncertainty u -setup [get_clocks c]` | ジッタ用途は `-setup` を付ける |
| 入力ジッタ | `set_input_jitter j [get_clocks c]` | SDC ファイル専用 |
| 外部 latency | `set_clock_latency -source [-early\|-late] d c` | early と late 両方で setup に影響 |

### 例外の優先順位と PolarFire 解析既定
| 項目 | 内容 |
|---|---|
| 例外優先順位 | disable_timing > false_path > max/min_delay > multicycle |
| setup コーナー既定 | slow_lv_lt（他に slow_lv_ht, fast_hv_lt） |
| hold コーナー既定 | fast_hv_lt |
| inter-clock 解析 | 既定 ON |
| システムジッタ | 自動計算（v2022.1 以降） |

### Libero Tcl 早見表（Discovery Kit 前提）

**実行**: `libero script:x.tcl SCRIPT_ARGS:A+B logfile:x.log`（原文形式は `"SCRIPT_ARGS:a b"`）

**new_project（MPFS095T-1FCSG325E）**
`-family {PolarFireSoC} -die {MPFS095T} -package {FCSG325} -speed {-1} -die_voltage {1.0} -part_range {EXT} -hdl {VERILOG} -use_enhanced_constraint_flow 1 -adv_options {IO_DEFT_STD:LVCMOS 1.8V}`
（family 表記は原文では `PolarFire SoC`。export_script で確認）

**run_tool -name（実行順）**
| 名前 | GUI |
|---|---|
| `SYNTHESIZE` | Synthesize |
| `PLACEROUTE` | Place and Route |
| `VERIFYTIMING` | Verify Timing |
| `GENERATEPROGRAMMINGDATA` | Generate FPGA Array Data |
| `generate_design_initialization_data`（コマンド） | Design & Memory Init |
| `GENERATEPROGRAMMINGFILE` | Generate Bitstream |
| `PROGRAMDEVICE` | Run PROGRAM Action |

**ファイル**: `import_files -hdl_source|-sdc|-io_pdc|-fp_pdc` → `build_design_hierarchy` → `set_root` → `derive_constraints_sdc` → `organize_tool_files -tool {PLACEROUTE} -file a -file b -input_type {constraint}`

**SmartDesign**: `sd_instantiate_hdl_module` → `sd_connect_pins -pin_names {inst:PIN PORT}` → `save_smartdesign` → `generate_component`

**書込み（PolarFire SoC）**
| 目的 | コマンド |
|---|---|
| eNVM | `configure_envm -cfg_file {ENVM.cfg}` |
| 構成 | `configure_tool -name {GENERATEPROGRAMMINGFILE} -params {program_fabric_snvm:true} -params {program_envm:true}` |
| 書込み | `configure_tool -name {PROGRAMDEVICE} -params {prog_action:PROGRAM}` → `run_tool -name {PROGRAMDEVICE}` |
| .job | `export_prog_job -bitstream_file_components {FABRIC SNVM ENVM}` |

**確認**: `get_tool_state -name {VERIFYTIMING}`

### SmartDebug のデバッグ手段（PolarFire / PolarFire SoC）
| 手段 | 見る/やること | 設計変更 | 前提・制約 |
|---|---|---|---|
| Live Probe | DFF 2 本を Ch A/B の専用ピンへ実時間出力 | 不要 | Probe I/O 予約、入力に流用しない、組合せ論理不可、Active Probe と干渉 |
| Active Probe | DFF を JTAG で読む/書く（多数） | 不要 | 非同期読み。整合が要るなら FHB で Halt |
| Probe Insertion | 任意ネットを空き I/O へ | 再 P&R＋再書込み（自動） | Integrated 専用、終了後 Delete All → Run |
| FHB | Halt / Step / トリガ停止 / .vcd | 再合成 | Project Settings ＋ NDC `auto_instantiate_fhb`、160 MHz 超は停止位置不定 |
| Event Counter / Freq Monitor | Ch A 信号の立上り数 / 周波数 | FHB 必要 | Ch A に割当て |
| Memory Blocks | LSRAM/uSRAM/TPSRAM 読み書き | 不要 | アクセス中はユーザ側から RAM 不可 |
| MSS Register Access | MSS レジスタ R/W・.csv | 不要 | MSS あり、MPU 設定済み |
| Debug IOD | IOD タップ・Eye Width | debug ピン Promote | PF_IOD_GENERIC_RX Dynamic |
| Debug MSS DDR | DDR トレーニング各段 | 不要 | MSS DDR 使用時のみ |

### SmartDesign: DRC エラー早見
| DRC | 対処 |
|---|---|
| Out-of-date Instance | 右クリック Update Instance |
| Undriven Pin | 接続 or Tie High/Low/Constant |
| Floating Driver | Mark Unused |
| Unconnected / Required BIF | 必須 BIF を接続 |
| BIF data/ID width mismatch | Initiator ≤ Target（write/ID）、Target ≤ Initiator（read/RID・BID）に再設定 |
| Unused Instance | 削除か出力を接続 |

### ライセンス種別と設定（inst-01, inst-02, rn-01）
| 種別 | OS | 紐付け（原文外） | 設定 |
|---|---|---|---|
| Silver Node-locked | Windows のみ | DiskID | `LM_LICENSE_FILE=C:\FlexLM\License.dat` |
| Silver / 有償 Floating | Win / UNIX / Linux | MAC ID | サーバ: `lmgrd -c License.dat`（v11.19）／クライアント: `LM_LICENSE_FILE` と `SNPSLMD_LICENSE_FILE` に `1702@host` |
| v2026.1 で Silver に追加 | -- | -- | MPF300TS-1FCG1152I、MPFS250TC（FCG1152I/E, FCVG484E）。Welcome Page の「MPF200TC」は誤記 |

### FlashPro Express の操作一覧（fpx-02, fpx-03）
| 操作 | 対象 | 用途 |
|---|---|---|
| Refresh/Rescan | プログラマ | 検出し直す |
| Ping | プログラマ | 通信確認（USB・ドライバ・電源） |
| Scan and Check Chain | JTAG チェーン | IDCODE と構成を照合（読み取り専用） |
| DEVICE_INFO / READ_IDCODE | デバイス | 状態・セキュリティ・IDCODE を見る |
| PROGRAM / VERIFY / ERASE | デバイス | 書き込み / 照合 / 消去 |
| `FPExpress.exe SCRIPT:x.tcl logfile:x.txt` | バッチ | Tcl での自動書き込み |

### eFP5 / PolarFire SoC でよく出るエラー（fpx-04）
| Code | 意味 | まず試すこと |
|---|---|---|
| Exit 4 | FlashPro5 の USB I/O エラー | ケーブルを挿し直し、ノイズ源から離す |
| 0x8003 / 5 | 書き込みモードに入れない | 電源、DEVRST_N を HIGH |
| 0x8004 / 6 | IDCODE 不一致 | ファイルとデバイスの組を確認、TRST |
| 0x8020 / -22 | PFSoC Boot Mode 3 の公開鍵が不正 | 鍵を入力し直す |
| 0x8032 / -32 | 命令タイムアウト | 電源、信号品質、DEVRST_N |

