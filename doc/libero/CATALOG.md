# Libero SoC v2026.1 ドキュメント索引

- 取得元ページ: https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/fpga-software-downloads （Documentation / User Guides の表。ページ本体は bot を 403 で弾くので、ブラウザの Console でリンクを抜く）
- PDF の実体: `https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/swdocs/<libero|synopsys>/<ファイル名>` で、curl で直接取得できる（2026-11-06 確認）
- 列「DL」: ✅ = `doc/libero/` に保存済み ／ — = 未取得（必要になったら上の URL から取る）
- 内容欄は、DL 済みのものは PDF のタイトルから、未取得のものはファイル名・タイトルからの推定

## A. PolarFire SoC の設計で使うもの

| DL | ファイル | 頁 | MB | 内容 | 使う場面 |
|---|---|---|---|---|---|
| ✅ | Libero_Installation_Licensing_Setup_User_Guide_2026_1.pdf | 12 | 0.2 | Libero のインストールとライセンス（Silver 含む）の設定 | 初回セットアップ |
| ✅ | libero_soc_v2026_1_release_notes.pdf | 18 | 0.3 | v2026.1 の新機能、対応デバイス、既知の不具合 | バージョン選定、不具合の確認 |
| ✅ | pf_pdc_ug_2026_1.pdf | 42 | 0.5 | PolarFire/SoC 用 PDC コマンド（set_io、配置制約） | ピン・I/O 規格の制約 |
| ✅ | io_editor_ug_2026_1.pdf | 56 | 3.1 | I/O Editor（I/O 規格、ODT、差動ペアの割り当て） | LVDS などの I/O 設定 |
| ✅ | pf_mlg_2026_1.pdf | 88 | 1.4 | PolarFire/SoC のマクロライブラリ（差動バッファ、IOD、DDR I/O など） | RTL でプリミティブを直接使う |
| ✅ | smartdesign_ug_2026_1.pdf | 64 | 1.9 | SmartDesign（ブロック図でコンポーネントを接続） | トップの構築、IP の接続 |
| ✅ | smarttime_ce_ug_2026_1.pdf | 143 | 2.4 | タイミング制約エディタ（SDC：クロック、I/O 遅延、例外） | 制約の作成 |
| ✅ | smarttime_sta_ug_2026_1.pdf | 105 | 6.1 | SmartTime による静的タイミング解析 | タイミング違反の解析 |
| ✅ | libero_soc_tcl_cmd_ref_ug_2026_1.pdf | 599 | 5.0 | Libero の Tcl コマンド全集 | スクリプトでのビルド（リファレンスデザインは Tcl） |
| ✅ | flashpro_express_ug_2026_1.pdf | 71 | 1.7 | FlashPro Express（.job による書き込み） | 書き込み |
| ✅ | smartdebug_ug_2026_1.pdf | 281 | 11.9 | SmartDebug（実機で内部信号の観測、Live Probe、Active Probe、メモリ読み書き） | 実機でのデバッグ |
| — | pf_block_flow_ug_2026_1.pdf | — | 0.5 | PolarFire のブロックフロー（ブロック単位で設計・配置を固めて再利用） | 大規模設計、チーム開発 |
| — | polarfire_custom_flow_ug_2026_1.pdf | — | 0.7 | PolarFire のカスタムフロー（外部の合成ツール・シミュレータを使う流れ） | Libero 以外のツールと組み合わせる |
| — | libero_custom_flow_ug_2026_1.pdf | — | 0.8 | カスタムフロー（全ファミリ共通版） | 同上 |
| — | smartpower_ug_2026_1.pdf | — | 2.8 | SmartPower（消費電力の見積もり・解析） | 電源設計、発熱の確認 |
| — | chipplanner_ug_2026_1.pdf | — | 3.7 | Chip Planner（配置の確認・手動フロアプラン） | 配置の確認、タイミング改善 |
| — | nlv_interface_ug_2026_1.pdf | — | 3.3 | Netlist Viewer（Libero 内） | 合成結果の回路を確認する |
| — | nlv_interface_ug_2025_2.pdf | — | 3.3 | 同上の v2025.2 版 | 旧版 |
| — | stdalone_nlv_ug_2026_1.pdf | — | 1.0 | Netlist Viewer の単体版 | 同上 |
| — | PF_SRAM_AHBL_AXI_ug_2025.2.pdf | — | 1.0 | PolarFire の LSRAM/uSRAM を AHB-Lite/AXI で使う IP | ファブリック RAM をバスにつなぐ |
| — | PolarFire-System-Services-SgCore-User-Guide_2025_2.pdf | — | 1.3 | PolarFire System Services コア（シリアル番号、乱数、署名などのシステムサービス） | デバイスのセキュリティ機能を使う |
| — | libero_soc_secure_ip_ug_2026_1.pdf | — | 1.4 | 暗号化 IP（IEEE 1735 など）の扱い | 暗号化 IP を受け取る・配る |
| — | vhdlsim_ug.pdf | — | 0.5 | VHDL シミュレーションのライブラリ・手順 | VHDL でのシミュレーション |
| — | vlogsim_ug.pdf | — | 0.8 | Verilog シミュレーションのライブラリ・手順 | Verilog でのシミュレーション |
| — | ModelSim-Microchip-Edition-%28ME%29-to-QuestaSim-Microchip-Edition-%28ME%29-Migration-Guide-DS50004156.pdf | — | 0.3 | ModelSim ME から QuestaSim ME への移行 | シミュレータの移行 |
| — | Libero_SoC_Design_Suite_Software_Installation_Guide_2025_1.pdf | — | 0.3 | インストールガイドの旧版（v2025.1） | 旧版 |
| — | libero_soc_faq.pdf | — | 0.2 | Libero の FAQ | 困ったとき |

## B. 合成ツール（Synplify Pro ME、Synopsys）

URL のフォルダは `swdocs/synopsys/`。

| DL | ファイル | MB | 内容 |
|---|---|---|---|
| — | fpgasyn_ug_2026_1.pdf | 10.1 | Synplify の User Guide（合成の手順、最適化） |
| — | fpgasyn_ref_2026_1.pdf | 5.9 | Synplify の Reference Manual |
| — | fpgasyn_att_ref_2026_1.pdf | 4.8 | 合成属性・ディレクティブ（syn_keep、syn_preserve など） |
| — | fpgasyn_cmd_ref_2026_1.pdf | 5.6 | Synplify の Tcl コマンド |
| — | fpgasyn_hdl_ref_2026_1.pdf | 2.3 | HDL（Verilog/VHDL/SystemVerilog）の対応範囲 |
| — | fpgasyn_id_debug_ug_2026_1.pdf | 2.2 | Identify（RTL レベルの実機デバッグ）の User Guide |
| — | fpgasyn_id_debug_ref_2026_1.pdf | 2.0 | Identify の Reference |
| — | h_f_message_reference_2026_1.pdf | 7.0 | 合成メッセージ（警告・エラー）の説明 |
| — | oem_message_reference_2024_2.pdf | 5.9 | 合成メッセージの説明（OEM 版、v2024.2） |
| — | fpga_rn_2026_1.pdf | 0.2 | Synplify のリリースノート |
| — | synplify_synthesis_faq.pdf | 0.4 | 合成の FAQ |

## C. 他ファミリ用・一般資料（PolarFire SoC では基本的に不要）

| DL | ファイル | MB | 内容 |
|---|---|---|---|
| — | sf2_mlg_2026_1.pdf | 1.4 | SmartFusion2/IGLOO2 のマクロライブラリ |
| — | rtg4_mlg_2026_1.pdf | 1.8 | RTG4 のマクロライブラリ |
| — | RTG4_SRAM_AHBL_AXI_ug_2025_2.pdf | 1.7 | RTG4 の SRAM の AHB-Lite/AXI IP |
| — | pdc_ug_2026_1.pdf | 0.5 | PDC コマンド（SmartFusion2/IGLOO2/RTG4 用。PolarFire は pf_pdc を使う） |
| — | libero_ecf_ug_2026_1.pdf | 19.0 | Enhanced Constraint Flow（SmartFusion2/IGLOO2/RTG4 の制約フロー） |
| — | design_sep_methodology_ug_2023_2.pdf | 3.3 | 設計分離（Design Separation）の手法。安全・セキュリティ用途 |
| — | msvt_ug_2023_1.pdf | 1.1 | 設計分離の検証ツール（MSVT） |
| — | Libero-SoC-Design-Suite-DS00005786.pdf | 3.3 | Libero の製品パンフレット |
| — | FPGA_Power_From_Architectural_Planning_to_Sign-off.pdf | 10.0 | ホワイトペーパー：FPGA の電力設計 |
| — | Leveraging_Microchip_IP_Cores_for_Deterministic_and_Low_Power_Edge_Systems.pdf | 0.8 | ホワイトペーパー：Microchip IP の活用 |
| — | Microchip+Non-Exclusive+Software+License+for+Libero+SoC+Design+Suite_v10+12+12+25.pdf | 0.3 | Libero のソフトウェア使用許諾 |
