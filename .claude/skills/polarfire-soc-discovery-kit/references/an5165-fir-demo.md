# AN5165 DSP FIR Filter デモ リファレンス

> 出典: *PolarFire SoC FPGA DSP FIR Filter Demo Application Note*, Microchip, **AN5165 / DS00005165D**（Revision D, 08/2026, "Draft Application Note", © 2026）
> ローカル: `C:\_Devs\PolarFire\doc\board\AN5165_PolarFire_SoC_DSP_FIR_Filter.pdf`
> デモ ZIP（約 244MB）は未取得。ZIP 内のファイル名は AN 本文の記載のみに基づく。

Discovery Kit の**工場出荷時（箱出し）に書き込まれているデザイン**がこの FIR デモ（Embedded SW UG / リファレンスデザイン README による）。

---

## 1. デモの概要

| 要素 | 内容 |
|---|---|
| FIR フィルタ | **127 tap**、係数リロード可能（re-loadable coefficients） |
| FFT | フィルタ出力に対する **256 point FFT**（スペクトル表示用） |
| ホスト I/F | UART。フィルタ係数と入力信号（low-pass, high-pass, band-pass, band-stop）をロード |
| ホスト GUI | 係数・入力信号を生成して FPGA とやり取りし、入出力波形とスペクトルをプロット |
| 実装 | ホスト I/F と FIR フィルタはすべて**ファブリック**に実装（MSS は使わない） |

---

## 2. 要件（Table 1-1 Design Requirements）

| Requirements | Version |
|---|---|
| Operating System | Windows 11 |
| Hardware: PolarFire SoC Discovery Kit (MPFS-DISCO-KIT with FPGA: MPFS095T-1FCSG325E) | REV 2 |
| Software: FlashPro Express | Available with Libero SoC installation package. |
| Software: Libero SoC | Note: Refer to the `readme.txt` file provided in the design files for the software versions used with this design. |

---

## 3. デザインファイル構成（ZIP 展開後）

```
mpfs_an5165_v2024p1_df/
├── Programming_Job/     ← 書き込み用 .job（FlashPro Express で使用）
├── GUI_Installer/
│   └── Volume/          ← setup.exe（DSP FIR Demo GUI インストーラ）
└── TCL_Scripts/         ← script.tcl, readme.txt（デザイン再生成用）
```

- 取得元: `www.microchip.com/en-us/application-notes/an5165`
- 同等デザインはリファレンスデザインリポジトリの `FIR_DEMO` 引数でも生成できる（[reference-design.md](reference-design.md)）。

---

## 4. デザイン構成

### 4.1 トップレベル SmartDesign のインスタンス（Figure 2-2 より）

| インスタンス | 役割 |
|---|---|
| `PF_CCC_C0_0` | クロック生成（50 MHz → 200 MHz / 25 MHz） |
| `CLKINT_0` | REF_CLK_0 をグローバルクロックへ（図より） |
| `CORERESET_PF_C0_0` | リセット生成（FABRIC_RESET_N） |
| `PFSOC_INIT_MONITOR_C0_0` | DEVICE_INIT_DONE などの初期化完了信号 |
| `Debounce_0` + `INV_0` | SW1（`switch_i`）のデバウンスと反転 → 外部リセット入力 |
| `PFSOC_DSP_FLOW_TOP_0` | DSP フロー本体（下表） |
| `COREUART_C0_0` | ホスト PC との UART（RX/TX がトップレベル端子） |

図から読める COREUART の定数（原文の図より、テキスト化は不完全）: `BAUD_VAL[12:0] = 13'h006B`, `BAUD_VAL_FRACTION[2:0] = 3'h4`。
200 MHz 動作として 200e6 / (16 × (0x6B + 1 + 0.5)) ≈ 115200 baud 相当 (原文外: CoreUART の一般的なボーレート式による推定)。

### 4.2 `PFSOC_DSP_FLOW_TOP` のブロック（Figure 2-3 / Table 2-1）

| ブロック | 内容 | 図から読めるポート幅 |
|---|---|---|
| `COREFIR_PF_C0_0`（CoreFIR_PF） | 係数リロードモード。LPF / HPF / BPF / BSF に対応 | `DATAI[15:0]`, `COEFI[15:0]`, `FIRO[47:0]`（`FIRO[29:14]` を後段へ） |
| `COREFFT_C0_0`（CoreFFT） | フィルタ出力の周波数スペクトルを生成。`SLOWCLK` 入力あり | `DATAI_RE/IM[15:0]`, `DATAO_RE/IM[15:0]` |
| `FILTERCONTROL_FSM_0` | バッファ読出し、CoreFIR への書込み、FIR 出力を出力バッファへ、FFT 出力を実部/虚部バッファへ | `FIR_WR_ADDR[9:0]`, `FFT_WADDR[7:0]`, `COEF_RADDR[6:0]` |
| `UART_IF_0` | UART とファブリック間の制御 FSM（係数ロード、入力データ格納、UART 送受信）。16-bit パケット化 | `DATA_IN/OUT[7:0]`, `WDATA[15:0]` |
| `PF_TPSRAM` ×5 | 係数・FIR 入力・FIR 出力・FFT 実部・FFT 虚部の格納 | 下表 |

| バッファ（PF_TPSRAM） | データ幅 | アドレス幅（図より） |
|---|---|---|
| `PF_COEF_BUF_0`（係数） | 16 bit | `[6:0]`（128 語） |
| `PF_FIR_IN_BUF_0`（FIR 入力） | 16 bit | `[9:0]`（1024 語） |
| `PF_FIR_OUT_BUF_0`（FIR 出力） | 16 bit | `[9:0]`（1024 語） |
| `PF_FFT_RE_BUF_0`（FFT 実部） | 16 bit | `[7:0]`（256 語） |
| `PF_FFT_IM_BUF_0`（FFT 虚部） | 16 bit | `[7:0]`（256 語） |

（語数はアドレス幅からの換算 (原文外)）

### 4.3 データ経路（原文の手順 1〜6）

1. UART ハンドシェイク（既知パターンを送受信して経路を事前確認）後、GUI がフィルタ係数 → 入力データの順に送信
2. `UART IF` が 16-bit パケットを作り、各入力バッファ（FFT Imaginary / FFT Real / FIR IN / FIR OUT）と係数バッファ（Coef Buffer）に格納
3. `Filter Control FSM` がバッファ読出しと CoreFIR への書込みを制御
4. CoreFIR の出力応答を **FIR OUT buffer** に格納
5. CoreFIR 出力を CoreFFT へ送り、CoreFFT が実部・虚部を **FFT Real / FFT Imaginary buffer** に格納
6. `UART IF` が FIR 出力と FFT 出力を読み出して UART で GUI へ送信 → GUI がプロット

```
GUI(Host PC) ⇄ CoreUART ⇄ UART IF ─→ Coef Buffer ─┐
                              │      FIR IN Buffer ─┴→ CoreFIR ─→ FIR OUT Buffer ─→ UART IF
                              │                          └──→ CoreFFT ─→ FFT Real / Imag Buffer ─→ UART IF
                       Filter Control FSM（全体制御）
```

### 4.4 クロック構成（§2.3）

| ソース | 出力 | 周波数 | 駆動先 |
|---|---|---|---|
| オンボード 50 MHz 水晶発振器 → `PF_CCC_C0` | `OUT_FABCLK_0`（図では `OUT0_FABCLK_0`） | **200 MHz** | `top_0` と `PFSOC_DSP_FLOW_TOP` の全ブロック（COREUART_C0, CORERESET_PF_0, DEBOUNCE, 各バッファ, CoreFIR, UART IF, Filter Control FSM 等） |
| 同上 | `OUT_FABCLK_1`（図では `OUT1_FABCLK_0`） | **25 MHz** | CoreFFT の 2 次クロック（slow clock） |

### 4.5 リセット構成（§2.4）

- `DEVICE_INIT_DONE`（PFSOC_INIT_MONITOR_C0）、`PLL Lock`（PF_CCC_C0）、`switch_i` から、`CORERESET_PF_C0` が `FABRIC_RESET_N` を生成（`OUT_FABCLK_0` に同期）
- `switch_i` は基板の **SW1** に接続 = デザインの外部リセット入力（GUI 操作中に SW1 を押す手順あり、§7）

---

## 5. シミュレーション（§2.5）

- テストベンチ: **`top.v`**。係数（LPF/HPF/BPF/BSF）と入力データの選択、UART_IF の状態信号、`DATAOUT`、FFT 出力状態（DATA Valid, output ready）を監視
- 波形をアナログ表示する手順:
  1. `DATAI` を右クリック → radix → **decimal**
  2. `DATAI` を右クリック → format → **Analog (automatic)**
- 信号: `COEFFI` = 入力係数、`DATAI` = FIR 入力、`FIRO` = FIR 出力、`DATAO_RE` / `DATAO_IM` = FFT 出力の実部 / 虚部

---

## 6. ボードセットアップとプログラミング（§3）

### 6.1 ジャンパ（Table 3-1）

| Jumper | Setting | Description |
|---|---|---|
| J45 | 1 and 2 closed | To set Bank 1 and Bank 5 voltages (VDDI1_5) to 3.3V for Rpi GPIO and mikroBus operation |
| J46 | 1 and 2 closed | To set the VDDAUX1 voltage to 3.3V |
| J47 | 1 and 2 closed | To receive power from the USB Type-C port (J4) for the board |
| J49 | **2 and 3 closed** | To set the 7 Segment display voltage to 5V |

他資料との差異は [quick-start.md §3.2](quick-start.md) を参照（J49 は資料により 1–2 / 2–3 が異なる）。

### 6.2 電源投入

- [ ] J4（USB Type-C）とホスト PC の Type-C（**5V, 3A**）ポートを C-C ケーブルで接続
- [ ] 電源 LED **5P0V, VDD, 1P8V** の点灯を確認

### 6.3 FlashPro Express での書き込み（§3.2）

.job の場所: `mpfs_an5165_v2024p1_df/Programming_Job`

- [ ] 1. Board Setup（§6.1, §6.2）完了
- [ ] 2. Windows Start メニューから **FlashPro Express** を起動
- [ ] 3. **New** で新規ジョブプロジェクト作成
- [ ] 4. Create New Job Project ダイアログ:
  - "Import FlashPro Express job file" → Browse で .job を選択
  - "FlashPro Express job project location" → Browse で保存先を選択
- [ ] 5. **OK** → FlashPro Express ウィンドウが開く
- [ ] 6. **Programmer** 欄にプログラマ番号が出ることを確認。出なければ接続を確認し **Refresh/Rescan Programmers**
- [ ] 7. **RUN** → 成功時 **`RUN PASSED`** 表示

---

## 7. GUI 操作手順（§4）

### 7.1 インストール

- [ ] `mpfs_an5165_v2024p1_df/GUI_Installer/Volume` の **`setup.exe`** を実行し、ウィザードに従う

### 7.2 係数と入力信号の生成

| # | 操作 | 結果 |
|---|---|---|
| 1 | GUI 起動 → **Filter Settings** | 係数生成画面 |
| 2 | Filter Generation パラメータは既定のまま → **Generate Filter** | — |
| 3 | — | **Filter Response** と **Filter Coefficient** のプロット表示 |
| 4 | 入力信号パラメータも既定のまま → **Generate Signal** | — |
| 5 | — | **Filter Input** タブに入力信号と周波数スペクトル表示 |

### 7.3 フィルタ出力の生成

| # | 操作 | 結果 |
|---|---|---|
| 1 | **Connect** アイコンをクリック（入力周波数と係数を設定） | 通信確立ダイアログ |
| 2 | GUI の指示に従い基板の **SW1 を押して** から **OK** | 通信確立（SW1 = デザインの外部リセット、§4.5） |
| 3 | **Start** アイコンをクリック | フィルタ処理実行 |
| 4 | — | **Operation Completed** 表示。**Filter Output** タブにフィルタ後データと FFT データ。LPF 既定なら高域成分が抑圧され低域が残る |
| 5 | ズームイン / アウトで波形確認 | — |
| 6 | **Text Viewer** | 係数、入力信号、出力信号、FFT 出力の数値を表示 |
| 7 | GUI を閉じる | デモ終了 |

- 周波数プロットの精度は周波数分解能（サンプリングレートと FFT 長）に依存する。

---

## 8. Tcl スクリプトでのデザイン再生成（Appendix 1）

- [ ] 1. Libero SoC を起動
- [ ] 2. **Project > Execute Script**
- [ ] 3. Browse で `TCL_Scripts` ディレクトリの **`script.tcl`** を選択
- [ ] 4. **Run**
- → `TCL_Scripts` ディレクトリ内に Libero プロジェクトが作成される。Design Implementation（Synthesis, Place and Route, Verify Timing）から .job 生成まで再現可能
- 詳細は `mpfs_an5165_v2024p1_df/TCL_Scripts` の `readme.txt`

> **既知の問題（長いパス）**: 長いパスで実行すると失敗することがある。`script.tcl` は C ドライブ直下付近など**短いパス**に置いて実行する。

---

## 9. 制約・既知の問題・注意点

| 項目 | 内容 | 出典 |
|---|---|---|
| ホスト OS | デモは **Windows ホスト PC のみ**（Table 1-1 は Windows 11） | QSG §1 / AN Table 1-1 |
| Libero | デモの実行だけなら不要。再生成・改造には必要（Silver ライセンスで可） | QSG §1 / AN §1 |
| 基板リビジョン | Discovery Kit **REV 2**（Rev B で Rev 1 → Rev 2 に更新） | AN Table 1-1 / 改訂履歴 |
| CoreFIR の既知問題 | `FIRO_VALID` を使わず、**初期レイテンシ 128 サイクル**で代用（次リリースで修正予定と記載） | AN §2.1 |
| IP 構成 | デザインと GUI は**所定の CoreFIR / CoreFFT 構成のみ**サポート | AN §2.1 |
| Tcl 実行パス | 長いパスで失敗 → 短いパスで実行 | AN Appendix 1 |
| USB 給電 | PC / ノートの USB ポートに直結。ドッキングステーションは未検証 | QSG §1 |
| Linux | FIR デモでは Linux は起動しない。Linux はリファレンスデザインに書き換えが必要 | QSG §3 |

---

## 10. 参照（Appendix 2）

| 対象 | 参照先 |
|---|---|
| PF_TPSRAM | PolarFire Family Fabric User Guide |
| PF_CCC | PolarFire Family Clocking Resources User Guide |
| CoreFIR / CoreFFT / CoreUART | 各 IP ページ |
| Libero, ModelSim, Synplify Pro | Libero SoC Documentation |
| Tcl コマンド | Tcl Commands Reference Guide |

## 11. 改訂履歴（Table 7-1）

| Rev | 日付 | 主な変更 |
|---|---|---|
| D | 08/2026 | Table 1-1 のホスト PC バージョン更新 |
| C | 10/2024 | Appendix 1 に Tcl スクリプトの Note（長いパス）追加 |
| B | 04/2024 | Libero v2024.1 対応、キット Rev 1 → Rev 2、GUI 項目追加、既知問題の Note、`top.v` 明記、"Output data is valid when FIRO_VALID is high." 削除、フォルダ名 `v2023p2` → `v2024p1` |
| A | 12/2023 | 初版 |
