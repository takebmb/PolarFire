# 用語集（Libero SoC v2026.1）

`用語 — 定義（章ファイル）`。章ファイルは `chapters/` 配下。

**-add_delay** — 同じポートに別クロック・別エッジの I/O 遅延を追加する。無いと上書き。重複時 -max は最大、-min は最小が残る（tce-04-sdc-io-delays.md）  
**-adv_options** — new_project / set_device の `KEY:VALUE` 形式の詳細設定。IO_DEFT_STD、RESTRICTPROBEPINS、TEMPR、VCCI_x_VOLTR、VOLTR、SYSTEM_CONTROLLER_SUSPEND_MODE 等（tcl-02-project-device.md）  
**-clock_fall** — I/O 遅延の基準を基準クロックの立下りにする。DDR の立下り側制約に使う（tce-04-sdc-io-delays.md）  
**-ignore_clock_latency** — max/min delay の slack 計算からスキューとジッタを除きデータパスのみを評価（tce-05-sdc-exceptions-objects.md）  
**-io_pdc / -fp_pdc** — import_files / create_links の引数。I/O 属性・ピン配置の PDC と、フロアプラン（配置・領域）の PDC。旧 `-pdc` は廃止（tcl-03-files-constraints.md）  
**-phase（generated clock）** — PLL 位相。SmartTime では波形シフトでなくクロック網の遅延として扱われ、setup/hold のエッジ対応は変わらない（tce-03-sdc-clocks.md）  
**.cxz** — PolarFire SoC MSS Configurator が出力する MSS コンポーネント。`import_mss_component` で取り込む（tcl-04-smartdesign-components.md）  
**0x8020 / -22** — PolarFire SoC 固有のエラー: Invalid Public Key for Boot Mode 3 eNVM client (fpx-04)  
**Active Probe** — JTAG で DFF を非同期に読み書きする機能。PolarFire は 18 DFF 単位で読む（dbg-02-live-active-probes.md）  
**actlmgrd / snpslmd / saltd** — Libero / Synopsys / Siemens 用ベンダーデーモン。v2024.2 以降は FlexLM v11.19 の 64 bit 版が必須 (inst-02, rn-01)  
**BANKCTRL_GPIO** — GPIO バンクのキャリブレーション制御マクロ（P/N/差動 PVT コード）（mlg-03-special-txpll）  
**BIF (Bus Interface)** — AXI/APB 等の信号束。BIF 同士 1 本の結線で全メンバが接続される。Initiator / Target / System の役割を持つ（sd-02-ports-bif-connections.md）  
**Bottleneck Report** — 多くの違反パスが通るインスタンスを path count または path cost で順位付け（sta-03-timing-reports.md）  
**BSR pin order fix** — PFSoC の JTAG 書き込みで I/O 状態が誤って適用されたバグ（例 G17）。v2026.1 で修正 (rn-02)  
**CDC Report** — クロック対ごとに Timed / Timed Unsafe / Synchronized / User Ignored / No Paths を表示（sta-04-cdc-report.md）  
**CLKBUF_DIFF / GCLKBUF_DIFF** — 差動クロック入力をグローバル網へ（GCLK 版は EN でゲート）（mlg-02-io-clocking）  
**CLKINT / RCLKINT** — ファブリック信号をグローバル網 / 行グローバルバッファへ（mlg-02-io-clocking）  
**Clock groups** — logically_exclusive / physically_exclusive / asynchronous。指定グループ間を両方向とも解析しない（tce-03-sdc-clocks.md）  
**Clock skew** — 受信側と送信側のクロック挿入遅延の差。setup は最小、hold は最大を使う（sta-02-interclock-timing-concepts.md）  
**Clock source latency** — 外部クロック源から定義ピンまでの遅延。-early と -late を両方与えたときだけ setup に影響（tce-03-sdc-clocks.md）  
**Clock uncertainty** — required time 側だけに加わる不確かさ。ジッタ用途では `-setup` を明示（tce-03-sdc-clocks.md）  
**configure_envm** — eNVM クライアントを .cfg で設定。HSS を eNVM に入れる入口（tcl-06-memory-init.md）  
**CONFIGURE_PROG_OPTIONS** — design_version / back_level_version / silicon_signature を設定する configure_tool のツール名（tcl-07-program-export.md）  
**configure_tool** — ツールのパラメータを `-params {KEY:VALUE}` で設定。run_tool より前に書く（tcl-05-design-flow.md）  
**Constraints Coverage Report** — 検査を Met / Violated / Untested に分類し、未制約箇所と追加すべき制約を示す（sta-03-timing-reports.md）  
**DDC (Design Debug Data Container)** — Libero の Export SmartDebug Data で作る Standalone 用デバッグデータ（dbg-01-getting-started-jtag.md）  
**Debug FPGA Array** — Live Probes / Active Probes / Memory Blocks / Probe Insertion タブと FHB 操作を持つ画面（dbg-02-live-active-probes.md）  
**Debug IOD** — PF_IOD_GENERIC_RX（Dynamic/Fractional Dynamic）の CORERXIODBITALIGN トレーニング結果（タップ・Eye Width）を表示する機能（dbg-05-xcvr-iod-ddr-tvs.md）  
**Debug MSS DDR I/O Margin** — MSS DDR PHY のトレーニング各段の結果と失敗理由を表示する機能（dbg-05-xcvr-iod-ddr-tvs.md）  
**derive_constraints_sdc** — コンポーネント（PF_CCC 等）の SDC から設計全体のタイミング制約を派生させる Libero 内コマンド（tcl-03-files-constraints.md）  
**DEVRST_N** — デバイスリセット。書き込み失敗の多くで「書き込み前に HIGH に固定」が対処になっている (fpx-04)  
**Direct Instantiation** — `sd_instantiate_core` でコンポーネントを作らずに IP を SmartDesign へ直接置く方式（tcl-04-smartdesign-components.md）  
**Edge_Anchor** — メモリインターフェースの配置位置（NORTH_NE 等 6 箇所）（ioed-02-interface-views）  
**edit_instance_delay** — LANECTRL の RX/TX_DQS_DELAY、ICB_CLKDIVDELAY、PLL_DELAY の遅延値を変更（pdc-05-post-layout-edit）  
**edit_io** — post-layout で I/O 属性だけを変更。ピン・IOSTD・fixed は不可。VCM 混在の手段（pdc-05-post-layout-edit）  
**Enhanced Constraint Flow** — `-use_enhanced_constraint_flow 1`。制約を I/O PDC・FP PDC・SDC に分けてツールごとに関連付けるフロー（tcl-02-project-device.md）  
**Event Counter / Frequency Monitor** — Live Probe Channel A の信号の立上り回数 / 周波数を測る FHB 依存機能（dbg-03-probe-insertion-fhb.md）  
**Exit 4 (FlashPro5)** — General device I/O error。USB パケット消失が原因。組込み FlashPro5（eFP5）も対象 (fpx-04)  
**export_prog_job** — FlashPro Express 用 .job を出力。リファレンスデザインの EXPORT_FPE に相当（tcl-07-program-export.md）  
**export_script** — 現セッションの操作を Tcl として書き出すコマンド（GUI: File > Export Script File）。GUI 操作を Tcl 化する最短路（tcl-01-scripting-basics.md）  
**False path** — 感応し得ないパス。setup と hold の両検査を外す（tce-05-sdc-exceptions-objects.md）  
**FHB (FPGA Hardware Breakpoint)** — クロックドメインごとのクロックゲート。Halt / Play / Step / トリガ停止 / 波形キャプチャ。PolarFire は NDC に `auto_instantiate_fhb` が必要（dbg-03-probe-insertion-fhb.md）  
**Floating license** — ライセンスサーバ（lmgrd）経由で共有するライセンス。Windows / UNIX / Linux (inst-02)  
**Force TCK Frequency** — STAPL の FREQUENCY 文より Programmer Settings の TCK を優先させる。FlashPro5 は 1〜6, 10, 15, 30 MHz (fpx-02)  
**FPExpress.exe SCRIPT:** — FlashPro Express の Tcl バッチ実行。`logfile:` / `console_mode:` / `SCRIPT_ARGS:` を付けられる (fpx-03)  
**Function 列 (Pin View)** — PPAT と同じピン機能名。GPIO/HSIO、バンク番号、差動 P/N パッドを示す（ioed-01-port-pin-view）  
**Generate Component** — DRC を実行して SmartDesign の HDL を生成。既定は Non-recursive（sd-03-drc-generate-memmap.md）  
**Generated clock** — 基準ピンのクロックから乗除算またはエッジ指定で導く内部クロック。`create_generated_clock -source` で定義（tce-03-sdc-clocks.md）  
**GENERATEPROGRAMMINGDATA** — Generate FPGA Array Data。ビットストリーム生成の前段（tcl-06-memory-init.md）  
**generate_component** — SmartDesign/コアの HDL を生成するコマンド。忘れると古い HDL で合成される（tcl-04-smartdesign-components.md）  
**generate_design_initialization_data** — メモリ初期化クライアント（eNVM/sNVM/SPI/Fabric RAM）を確定させるコマンド。configure_envm 等の後に必須（tcl-06-memory-init.md）  
**get_tool_state** — ツールの状態文字列（Tool successfully run / Tool failed / Tool has timing violations 等）を返す（tcl-05-design-flow.md）  
**GPIO / HSIO バンク** — PolarFire の 2 種類のファブリック I/O バンク。LVDS25/LVDS33/LVDS18G は GPIO、LVDS18 は HSIO（入力専用）（pdc-01-syntax-iobank）  
**GPIO_ACTIVE (INIT)** — 各 GPIO バンクのキャリブレーション完了を示す INIT マクロ出力（mlg-03-special-txpll）  
**HDL+ core** — ユーザ HDL から Create Core from HDL で作るコア定義。BIF ピンを持てる（sd-01-canvas-components.md）  
**Hierarchical SmartDesign** — 選択範囲から作る子 SmartDesign（Ctrl+H）。Flatten（Ctrl+J）で戻す（sd-02-ports-bif-connections.md）  
**I/O Editor** — post-synthesis で開く I/O 属性編集 GUI。Edit with I/O Editor で PDC に commit、View は read-only（ioed-01-port-pin-view）  
**INBUF_DIFF** — 差動入力バッファ。PADP, PADN → Y。LVDS 受信に使う（mlg-02-io-clocking）  
**Input delay** — クロックエッジ後に入力ポートへデータが現れるまでの外部遅延（`set_input_delay`）（tce-04-sdc-io-delays.md）  
**Inter-clock domain 解析** — 2 クロック周期の最小公倍数上で最も厳しい launch-capture 関係を検査。既定で有効（sta-02-interclock-timing-concepts.md）  
**IOD View** — FPGA I/O を使う非メモリインターフェース（CDR、RX_DDR_L_A/TX_DDR_G_A）のバンク/レーン配置ビュー（ioed-02-interface-views）  
**IOD レーンコントローラ** — レーンクロックとバンククロックを I/O FIFO で橋渡しし、DLL で 90° シフト、CDR の位相選択も行う（ioed-02-interface-views）  
**IOREG** — IOD 内の入力/出力/出力イネーブルレジスタ（IOINFF/IOUTFF/IOEFF）の post-layout マクロ（mlg-05-post-layout-macros）  
**IO_DIFF** — 差動 I/O の N 側位置を予約する post-layout プレースホルダ（mlg-05-post-layout-macros）  
**Job file (.job)** — Libero の Export FlashPro Express Job が出す、書き込み設定とビットストリームのまとめ (fpx-01)  
**Job Project (.pro)** — FlashPro Express の作業プロジェクト。`.job` から作る (fpx-01)  
**Live Probe** — 内部 DFF 2 点を Channel A/B の専用 I/O へ実時間出力する機能。再コンパイル不要（dbg-02-live-active-probes.md）  
**lmgrd** — FlexLM のライセンスマネージャ。`lmgrd -c License.dat [-l log]` で起動 (inst-02)  
**LM_LICENSE_FILE** — FlexLM のライセンス探索パス。ノードロックではファイルパス、Floating では `port@hostname`（既定 1702） (inst-01, inst-02)  
**LVDS fail-safe** — 入力が浮く場合の対策。`set_io -ODT DYNAMIC` と `-RES_PULL` を使う（pdc-03-set-io-attributes）  
**LVDS18G** — GPIO バンクで使う 1.8V 系 LVDS。入力の CLAMP_DIODE は OFF のみ（pdc-03-set-io-attributes）  
**LVDS25 / LVDS33** — GPIO バンクの LVDS 規格。vcci 2.5V / 3.3V。差動専用、Input/Output 可、Tribuff/Inout 不可（pdc-02-set-io-iostd）  
**Mark Unused / Tie High・Low・Constant** — 未使用出力/入力を DRC に通すためのピン属性（sd-02-ports-bif-connections.md）  
**MegaVault** — オフライン PC に IP コアを入れるための Vault。`.cpz` を Catalog の Add Core to Vault で取り込む (inst-02)  
**Memory Blocks** — ファブリック RAM（LSRAM/uSRAM/TPSRAM）を論理(L)・物理(P)ブロックで読み書きするタブ（dbg-04-memory-nvm-mss-registers.md）  
**Memory Map Report / Interrupt Map Report** — BIF 接続のアドレス表と MSS 割込みピンの PLIC 番号を .json で出力（sd-03-drc-generate-memmap.md）  
**MSS Register Access** — PolarFire SoC の MSS レジスタを読み書き・.csv 入出力する画面。MPU 設定が前提（dbg-04-memory-nvm-mss-registers.md）  
**Multicycle path** — 2 サイクル以上を許すパス。-setup/-hold を分けて指定、無指定時 hold は既定 0（tce-05-sdc-exceptions-objects.md）  
**NDC** — Compile 前に読む Netlist Design Constraint（*.ndc）。set_ioff, set_preserve（pdc-04-location-ndc-floorplan）  
**Node-locked license** — 1 台に紐付くライセンス。Windows 専用 (inst-01)  
**ODT (On-Die Termination)** — チップ内終端。LVDS 入力は既定 ON、ODT_VALUE は 100Ω のみ、DYNAMIC はフェイルセーフ用（pdc-03-set-io-attributes）  
**Operating condition（PolarFire）** — slow_lv_ht / slow_lv_lt / fast_hv_lt。既定は max = slow_lv_lt、min = fast_hv_lt（tce-06-sdc-scenarios-reporting.md）  
**Operator / Developer mode** — FlashPro Express のモード。Operator は量産用（実機がつながっていないとジョブを開けない）、Developer はジョブの編集やデバイスごとのアクションができる (fpx-01, fpx-02)  
**organize_tool_files** — どのツール（SYNTHESIZE / PLACEROUTE / VERIFYTIMING 等）がどのファイルを使うかを指定するコマンド。import だけでは制約は効かない（tcl-03-files-constraints.md）  
**OUTBUF_DIFF** — 差動出力バッファ。D → PADP, PADN。LVDS 送信に使う（mlg-02-io-clocking）  
**Output delay** — 外部レジスタまでの遅延 + setup（max）/ hold（min）。`set_output_delay`（tce-04-sdc-io-delays.md）  
**OUT_DRIVE (LVDS)** — LVDS25/33/18G の駆動電流 3, 3.5, 4, 6 mA（既定 6）（pdc-03-set-io-attributes）  
**PAD port** — パッケージピンに割り当たるポート属性。自動でトップへ昇格し、PAD ネットは削除・コピー不可（sd-02-ports-bif-connections.md）  
**part_range** — 温度グレード。PolarFire 系は EXT / IND / MIL。パート番号末尾の文字（MPFS095T-1FCSG325**E** → EXT）（tcl-02-project-device.md）  
**Path set** — Register to Register / External Setup(Hold) / Clock to Out など、ドメインごとのパス分類（sta-01-overview-analyzer.md）  
**PDC (Physical Design Constraints)** — I/O 属性・配置・配線を制約する Place-and-Route 用制約ファイル。コマンドと引数は大小区別（pdc-01-syntax-iobank）  
**Ping / Self Test** — Ping はプログラマとの疎通確認。Self Test は FlashPro5/4 では使えない (fpx-02)  
**prgsrvdmn / prgservapp / dbgservapp** — プログラミング・デバッグのサーバプロセス。Windows でプログラマが見えないときはファイアウォールで許可する (fpx-03)  
**Probe Group** — Active Probe の任意信号グループ。同名自動グループの Bus と違い Ungroup 可（dbg-02-live-active-probes.md）  
**Probe Insertion** — 内部ネットを空き I/O へ incremental P&R で配線し再書き込みする post-layout 手法。Integrated 専用（dbg-03-probe-insertion-fhb.md）  
**Programming Action** — PROGRAM / VERIFY / ERASE / DEVICE_INFO / READ_IDCODE / ENC_DATA_AUTHENTICATION (fpx-02)  
**program_fabric_snvm / program_envm** — PolarFire SoC の GENERATEPROGRAMMINGFILE パラメータ。PolarFire（非 SoC）の program_fabric / program_snvm とは名前が違う（tcl-07-program-export.md）  
**prog_action** — PROGRAMDEVICE のアクション。PROGRAM / VERIFY / ERASE / DEVICE_INFO / READ_IDCODE / VERIFY_DIGEST 等（tcl-07-program-export.md）  
**Promote to Top-Level** — 非 PAD ピンからトップポートとネットを作る操作（sd-02-ports-bif-connections.md）  
**PRPF_xxx** — IOD/CDR/RGMII/SGMII/XCVR/PLL の配置ルール違反エラー（pdc-04-location-ndc-floorplan）  
**Pseudo Static Signal Polling** — Active Probe 信号を 1 秒周期で監視し指定値一致を待つ機能（dbg-02-live-active-probes.md）  
**Remote Programming Server (RPS)** — 別マシンのプログラマを LAN 越しに使う v2026.1 の新機能。既定ポート 57605/57606 (rn-01, fpx-02, fpx-03)  
**RPS (Remote Programming Server)** — v2026.1 のリモートプログラマ機能のサーバ。既定 TCP 57605/57606（dbg-01-getting-started-jtag.md）  
**run_tool** — Design Flow のツールを名前（SYNTHESIZE, PLACEROUTE, PROGRAMDEVICE 等）で実行。上流は自動実行（tcl-05-design-flow.md）  
**Scan and Check Chain** — JTAG チェーンの IDCODE を読んでジョブの構成と照合する。読み取り専用 (fpx-02)  
**SCRIPT_ARGS** — `libero script:<x>.tcl` に続けてスクリプトへ渡す引数。原文は `"SCRIPT_ARGS:a b c"`、Discovery Kit リファレンスデザインは `+` 区切り。スクリプト側では `$argv` で受ける（tcl-01-scripting-basics.md）  
**set_as_target** — GUI で追加した制約を書き込む先の sdc / io_pdc / fp_pdc ファイルを指定（tcl-03-files-constraints.md）  
**set_clock_to_output / set_external_check** — FPGA 内部のタイミング予算を直接書く非標準コマンド。Libero の STA のみが参照（tce-04-sdc-io-delays.md）  
**set_input_jitter / set_system_jitter** — SDC ファイル内でのみ有効（Tcl コマンドとしては不可）（tce-03-sdc-clocks.md）  
**set_io** — ポートに IOSTD・ピン・属性を割り当てロックするコマンド（pdc-02-set-io-iostd）  
**set_iobank** — バンクの vcci/vref を設定し固定するコマンド。未指定だと最初に置いた I/O の IOSTD でバンク電圧が決まる（pdc-01-syntax-iobank）  
**set_ioff** — NDC で I/O ポートにレジスタを取り込む（IN_REG/OUT_REG/EN_REG のいずれか 1 つ、fanout 1）（pdc-04-location-ndc-floorplan）  
**set_plain_text_client** — eNVM.cfg / sNVM.cfg に書く平文クライアント定義（tcl-06-memory-init.md）  
**Silver license** — Libero の無償ライセンス。microchipDIRECT の Request Free License で Floating か Node-locked を申請する。v2026.1 で MPF300TS-1FCG1152I と MPFS250TC（FCG1152I/E, FCVG484E）が追加された (inst-01, rn-01)  
**Slack** — Required − Arrival。Required = T + capture クロック遅延 − setup（sta-02-interclock-timing-concepts.md）  
**Smart Search and Connect** — `[Type]:[Instance].[Pin]` 構文で検索・一括結線するツール（Ctrl+K）（sd-02-ports-bif-connections.md）  
**SmartBERT** — トランシーバレーンの PRBS テスト。CoreSmartBERT IP 使用時はエラー注入可（dbg-05-xcvr-iod-ddr-tvs.md）  
**SmartDebug** — JTAG 経由の実機デバッグツール。SPI 非対応。Integrated / Standalone / Demo モード（dbg-01-getting-started-jtag.md）  
**SmartDesign** — IP・HDL・他 SmartDesign をキャンバスで結線し、DRC 済み合成可能 HDL を生成するブロック図エディタ（sd-01-canvas-components.md）  
**SmartTime** — Libero の静的タイミング解析ツール。timing-driven P&R も内部で同じエンジンを使う（sta-01-overview-analyzer.md）  
**SNPSLMD_LICENSE_FILE** — Synopsys ツール用のライセンス変数。Floating のクライアントで設定する (inst-02)  
**SOURCE_TERM** — 差動出力の近端終端。OFF/100、既定 OFF（pdc-03-set-io-attributes）  
**SPI Target** — SPI での書き込みモード。FlashPro5 は SF2/IGLOO2 のみ。PolarFire SoC は JTAG で書く (fpx-01, fpx-02)  
**System jitter（自動計算）** — PolarFire/PolarFire SoC で v2022.1 から自動計算。ユーザーは `set_input_jitter` で入力ジッタだけ与える（tce-02-exceptions-clock-adjust.md）  
**Target file** — Constraints Editor で追加した制約の保存先 SDC。無ければ user.sdc が自動作成される（sta-05-tutorials.md）  
**Timing Constraints Editor** — SDC タイミング制約を GUI で作成・編集するツール。Requirements / Exceptions / Advanced に分類（tce-01-editor-required-constraints.md）  
**Timing exception 優先順位** — set_disable_timing > set_false_path > set_max/min_delay > set_multicycle_path（tce-02-exceptions-clock-adjust.md）  
**Timing scenario** — 制約のセット。解析用と TDPR 用を別に指定でき、既定は Primary（tce-06-sdc-scenarios-reporting.md）  
**TRUSTED_FACILITY** — 既定のビットストリーム種別（カスタムセキュリティなし）。export_prog_job / export_bitstream_file の既定（tcl-07-program-export.md）  
**TVS Monitor** — TVS IP でダイ温度と VDD/VDD18/VDD25 を監視する機能（dbg-05-xcvr-iod-ddr-tvs.md）  
**Update Component Version** — コアを新版に上げるメニュー。続けて Update Instance と制約の再導出を行う (rn-02)  
**Update Instance** — 下位コンポーネントのポート変更を上位インスタンスに反映する操作。赤三角 ! の解消に使う（sd-01-canvas-components.md）  
**user.pdc** — I/O Editor の commit で生成される `<project>/constraints/io/user.pdc`（ioed-02-interface-views）  
**USK (User Secret Key)** — sNVM 保護ページ用の鍵（dbg-04-memory-nvm-mss-registers.md）  
**VCM_RANGE** — 差動入力の同相電圧レンジ MID/LOW（既定 MID）。同一バンクの真差動入力では混在不可（pdc-03-set-io-attributes）  
**Virtual clock** — ソースピンを持たない `create_clock`。FPGA 外部のクロックに同期した I/O の基準に使う（tce-03-sdc-clocks.md）  
**VLNV** — Vendor:Library:Name:Version。カタログ IP の識別子（例 `Actel:SgCore:PF_CCC:1.0.115`）（tcl-04-smartdesign-components.md）  
**`set_live_probe`** — Live Probe チャネル割当ての Tcl。A/B は 1 回の呼び出しで同時に指定する（dbg-06-tcl-commands.md）  
