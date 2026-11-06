# Macro Library User Guide: Special マクロ（バンク制御・INIT・SC_STATUS・PFSOC_SCSM ほか）と TX_PLL（PDF p.47-56）

## Core Idea
特殊用途マクロ群。I/O 観点で重要なのは **BANKCTRL_GPIO / BANKCTRL_HSIO（バンクのキャリブレーション制御）**、**BANKEN（バンク有効化）**、**INIT（GPIO_ACTIVE 等の初期化完了信号）**。PolarFire SoC 固有では **PFSOC_SCSM**（System Controller Suspend Mode 中の MSS リブート）がある。APBM / BANKEN / BANKCTRL_* は post-layout マクロで、ポートは設定した時間後にアサートされる（シミュレーション用パラメータ付き）。

## Frameworks Introduced
- **I/O の準備完了を待つ**: INIT マクロの `GPIO_ACTIVE`（バンクごとのキャリブレーション完了）や `FABRIC_POR_N` をユーザーロジックのリセット解除条件に使う。`HSIO_ACTIVE` は VDDI 喪失の監視にも使える（INIT_MONITOR IP の出力）
- **GPIO バンクのキャリブレーション手動制御（BANKCTRL_GPIO）**: CALIB_START で開始（CALIB_STATUS クリア）→ 完了で CALIB_STATUS=High・CALIB_INTERRUPT パルス → コード調整は CALIB_MOVE_PCODE/NCODE/DIFFR_PVT と CALIB_DIRECTION → CALIB_LOAD（CALIB_STATUS=High のときのみ）→ CALIB_LOCK で確定
- **SCSM を有効にした PolarFire SoC で MSS リブートを可能にする**: MSS Configurator の Misc タブで「Expose Feedback ports to Fabric」を有効化 → MSS の `REBOOT_REQUESTED_M2F` を PFSOC_SCSM の `SC_WAKE` に**直結**（他の接続は不可）

## Key Concepts
- **APBM**: post-layout の APB マスタ。PRESETN, PSEL, PENABLE, PWRITE, PSTRB[3:0], PADDR[28:0], PWDATA[31:0], PSELFAST, PSTRBCLK, PCLK を出力し、PRDATA[31:0], PREADY, PSLVERR, PINTERRUPT を受ける
- **BANKEN**: BANK_EN 出力をシミュレーション遅延後に High にして I/O/HSIO バンクを有効化。パラメータ BANK_EN_SIMULATION_DELAY, BANK_NUMBER
- **FCEND_BUFF / FCINIT_BUFF**: キャリーチェーン末尾の FCO を受けるバッファ / 先頭 FCI を初期化するバッファ
- **PF_SPI**: System Controller の専用 SPI ポートをファブリックから使う（SPI-Initiator モード。IO_CFG_INTF と SC_SPI_EN を両方 High）
- **SC_STATUS**: SUSPEND_EN（Suspend 中 1）と ACTIVE（System Controller 動作中 1）。シミュレーション非対応
- **OSC_RC160MHZ / OSC_RC2MHZ**: OSC_160MHZ_ON / OSC_2MHZ_ON を High に固定すると CLK に 160 MHz / 2 MHz のフリーラン RC クロック
- **LIVE_PROBE_A**: SmartDebug 用の専用プローブ
- **TX_PLL**: トランシーバの送信 PLL。TXPLL_SSC（拡散スペクトラム付き）と TXPLL。ハーフレート Frac-N。レーンごとに 1, 2, 4, 8, 11 で分周。例: 5 Gb/s には 2.5 GHz

## Reference Tables
**BANKCTRL_GPIO ポート（Table 5-4、PDF 画像で確認）**
| ポート | 方向 | 内容 |
|---|---|---|
| CALIB_STATUS | Out | キャリブレーション完了/ロックで High、実行中 Low |
| CALIB_INTERRUPT | Out | 完了時に High パルス |
| CALIB_DIRECTION | In | コード調整方向 |
| CALIB_LOAD | In | 新コードをバンクへロード（CALIB_STATUS=High 時のみ） |
| CALIB_LOCK | In | 値をロック（完了扱い、CALIB_STATUS=High） |
| CALIB_MOVE_NCODE / PCODE | In | N 側 / P 側コードを動かす |
| CALIB_START | In | 開始。CALIB_STATUS をクリア |
| CALIB_MOVE_DIFFR_PVT | In | 差動 PVT キャリブレーションコードを動かす |
主なパラメータ: PC_REG_CALIB_PCODE/NCODE（6bit 初期値）, PC_REG_CALIB_DIFFR（4bit 差動初期値）, PC_CALIB_D/N/P_OFFSET（4/6/6bit）と各 _DIRECTION, PC_REG_DIFFR_VSEL（2bit 差動電圧選択）, BANK_NUMBER, CALIB_STATUS_SIMULATION_DELAY, CALIB_CALIB_INTERRUPT_DELAY。
BANKCTRL_HSIO は DIFFR 系の代わりに CALIB_MOVE_SLEWR / SLEWF（スルーレート立ち上がり/立ち下がり）と PC_SLEWR/SLEWF, PC_CALIB_ROFFSET/FOFFSET を持つ。

**INIT マクロ出力（Table 5-16）**
| ポート | 内容 |
|---|---|
| FABRIC_POR_N | ファブリックの Low アクティブ POR |
| PCIE_INIT_DONE | PCIe 初期化完了 |
| SRAM_INIT_DONE / USRAM_INIT_DONE | LSRAM / µSRAM 初期化完了 |
| GPIO_ACTIVE | GPIO サブシステム準備完了。**各 I/O バンクのキャリブレーション完了判定に使える**（# はバンク番号 0, 1, 7, 8, 9） |
| HSIO_ACTIVE | HSIO 準備完了。特定バンクの VDDI 喪失監視 |
| UIC_INIT_DONE | デバイスレベル初期化完了 |
| RFU[11:0] | 予約（RFU[0]=XCVR 初期化完了、RFU[7]=自動キャリブ完了、RFU[11]=自動 IO キャリブ完了 等の記述あり） |
パラメータ: *_SIMULATION_DELAY 各種、SUSPEND_MODE（1 で初期化後に全出力をデアサート）。

**PFSOC_SCSM の DRC（p.55）**
| 状況 | 結果 |
|---|---|
| SC_WAKE を REBOOT_REQUESTED_M2F 以外が駆動 | エラーで停止 |
| Suspend 無効なのにインスタンス | 警告（Export Design Initialization Data and Memory Report） |
| Suspend 有効・MSS 使用・未インスタンス | 警告（MSS リブート要求を処理できない） |
| ES デバイス / PolarFire（非 SoC） | CMPPF_026 エラー |
| RT PolarFire SoC | ピンを静的にできないエラー |
未使用の PolarFire SoC デザインでは Libero が入力を静的 1 に固定。

## Anti-patterns
- **GPIO_ACTIVE を待たずに LVDS 受信ロジックを動かす**: キャリブレーション前の I/O を使うことになる（リセット設計への適用は原文外の推奨）
- **CALIB_STATUS=Low の間に CALIB_LOAD**: 原文は High 時のみと規定
- **SC_STATUS をシミュレーションで使う**: 非対応。テストベンチで t=0 に SUSPEND_EN=0/ACTIVE=1、DEVICE_INIT_DONE か AUTOCALIB_DONE の遅い方から 0.625 µs 後に SUSPEND_EN=1/ACTIVE=0 を与える
- **Suspend 中にプログラミング/デバッグ**: Suspend でないことを先に確認

## Key Takeaways
1. GPIO バンクの準備完了は INIT の GPIO_ACTIVE（バンク 1 も対象）
2. GPIO キャリブレーションを手動で動かすなら BANKCTRL_GPIO（P/N/差動 PVT コード）
3. PFSOC_SCSM は REBOOT_REQUESTED_M2F → SC_WAKE の直結のみ
4. OSC_RC160MHZ / OSC_RC2MHZ は ON 端子を High 固定で使う
5. TX_PLL の出力はビットレートの半分（ハーフレート）

## Connects To
- **pdc-01-syntax-iobank**: set_iobank -auto_calib / -auto_calib_ramp_time（自動キャリブの PDC 側）
- **ioed-01-port-pin-view**: Use I/O Calibration from the Lane
- **pdc-04-location-ndc-floorplan**: TxPLL の set_location
- 姉妹 skill `polarfire-soc-mss`: MSS Configurator、リブート
