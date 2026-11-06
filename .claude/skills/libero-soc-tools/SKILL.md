---
name: libero-soc-tools
description: Libero SoC v2026.1 の公式 UG 11 本（PDC、I/O Editor、Macro Library、SmartDesign、SmartTime、Tcl、SmartDebug、FlashPro Express、ライセンス、リリースノート）の知識ベース。PolarFire / PolarFire SoC の I/O 制約、SDC、Tcl によるビルド、書き込み、実機デバッグで使う。
when_to_use: Libero, Libero Tcl, PDC set_io, -io_std, LVDS, ODT, I/O Editor, INBUF_DIFF, SDC, set_input_delay, SmartTime, SmartDesign, SmartDebug, Live Probe, FlashPro Express, Silver ライセンス
---

# Libero SoC v2026.1 ツール

対象は PolarFire / PolarFire SoC。利用者のボードは Discovery Kit（MPFS095T-1FCSG325E）。ボードのピンは skill **polarfire-soc-discovery-kit**、MSS の内部は **polarfire-soc-mss** を使う。

## 読み方
- 下の **Where to Look** で章ファイルを 1 本選んで Read する。2 本で足りなければ、もう 1 本だけ足す
- **LVDS の質問**は、まず `chapters/topic-lvds-gpio.md` を読む（資料横断のまとめ）
- 用語は `glossary.md` を、Tcl の個々のコマンドは `chapters/tcl-08`・`tcl-09`（索引）を、**Read せずに Grep** で引く
- 手順は `patterns/<分野>.md`（io / timing / sd-debug / tcl / setup）。1 枚表は `cheatsheet.md`。原文の誤記は `errata.md`
- `(原文外)` の付いた記述は資料に無い補足や推定。断定せず、そう伝える
- 未取得の Libero 資料 38 本（SmartPower、Chip Planner、Synplify、シミュレーションなど）は `C:\_Devs\PolarFire\doc\libero\CATALOG.md` に URL がある。必要になったら curl で取得する。原本 PDF も同じフォルダにある

## Core: 間違えやすい点
- PDC の規格名の引数は `-io_std`（`-iostd` は誤り）。コマンドと引数は大文字小文字を区別する
- 制約は `import_files -io_pdc / -fp_pdc / -sdc` で取り込む（旧 `-pdc` は廃止）。**取り込んだだけでは効かない**。`organize_tool_files` で関連付ける。このコマンドは既存の一覧を**置き換える**ので、全ファイルを列挙する
- PLL/CCC のクロック制約は手で書かず、Constraint Manager の **Derive Constraints** で取り込む
- `set_input_delay` / `set_output_delay` を同じポートに重ねるときは `-add_delay`。忘れると上書きされる
- タイミングの確認は **Constraints Coverage で Untested が 0** になってから Violations（max と min）を見る
- Tcl のフローは `configure_tool` → `run_tool`（上流は自動で実行される）。結果は `catch` で判定する
- Discovery Kit の `new_project`: `-die {MPFS095T} -package {FCSG325} -speed {-1} -die_voltage {1.0} -part_range {EXT}`。family の表記は資料で揺れているので `export_script` の出力で確認する
- v2026.1 の Windows 対応は Windows 11（64 bit）だけ。RAM は MPFS095T なら 16 GB 以上を推奨

## Where to Look（`chapters/`）

| 分野 | ファイル: 中身 |
|---|---|
| PDC | pdc-01: 構文、set_iobank、reserve、バンクごとの規格 ／ pdc-02: set_io、-io_std と差動・方向の制限 ／ pdc-03: ODT・VCM・RES_PULL・OUT_DRIVE・遅延の値表 ／ pdc-04: set_location、set_ioff、set_preserve、リージョン ／ pdc-05: edit_io、edit_instance_delay |
| I/O Editor | ioed-01: Port/Pin/Package View、ODT ／ ioed-02: Memory/XCVR/IOD View、IOD レーンの DRC ／ ioed-03: Floorplanner、PDC 書き出し、MSS I/O |
| マクロ | mlg-01: 論理・DFF・SLE ／ mlg-02: I/O バッファ（*_DIFF）、UJTAG、CLKINT ／ mlg-03: BANKCTRL_GPIO、INIT、SC_STATUS、TX_PLL ／ mlg-04: RAM1K20、RAM64x12、MACC_PA ／ mlg-05: GB/RGB、IOPADP/N、IO_DIFF、IOREG |
| SDC 制約 | tce-01: エディタと必須制約 ／ tce-02: 例外の優先順位、ジッタ、clock groups ／ tce-03: create_clock、generated clock ／ tce-04: input/output delay、ソース同期 ／ tce-05: false path、multicycle、get_* ／ tce-06: check・read/write_sdc・report・set_options |
| SmartTime | sta-01: 画面とパスセット ／ sta-02: slack・skew・Fmax、inter-clock ／ sta-03: 各種レポート ／ sta-04: CDC Report ／ sta-05: チュートリアル、Target file |
| SmartDesign | sd-01: キャンバス、HDL+、Update Instance ／ sd-02: ポート、BIF、Promote to Top ／ sd-03: DRC、Generate、Memory/Interrupt Map |
| SmartDebug | dbg-01: 起動、JTAG、Device Status ／ dbg-02: Live Probe、Active Probe ／ dbg-03: Probe Insertion、FHB ／ dbg-04: RAM・NVM・MSS レジスタ ／ dbg-05: Debug IOD、DDR、TVS、XCVR ／ dbg-06: SmartDebug の Tcl |
| Tcl | tcl-01: 実行方法、SCRIPT_ARGS、export_script ／ tcl-02: new_project、set_device ／ tcl-03: import_files、organize_tool_files ／ tcl-04: コア生成、SmartDesign の Tcl ／ tcl-05: run_tool、configure_tool ／ tcl-06: eNVM・sNVM・SPI Flash ／ tcl-07: 書き込みファイル、export_prog_job ／ tcl-08, 09: 全コマンド索引（Grep で引く） |
| 書き込み | fpx-01: .job、チェーン ／ fpx-02: アクション、TCK、SPI Flash ／ fpx-03: コマンドライン、ファイアウォール ／ fpx-04: 終了コード |
| 導入 | inst-01: ライセンス種別、Silver、ノードロック ／ inst-02: Floating ／ rn-01: 動作環境、新機能 ／ rn-02: コア更新、不具合 |
| 横断 | topic-lvds-gpio: LVDS（規格、マクロ、PDC、SDC、実機確認） |

## 資料に無いもの
デバイスの電気的特性（LVDS の最大データレート、DC/AC 特性）は PolarFire SoC Datasheet、I/O と IOD の詳しい仕様は User I/O User Guide、IP コア（PF_IOD_GENERIC、PF_CCC など）は各コアの UG にある。いずれも未取得。
