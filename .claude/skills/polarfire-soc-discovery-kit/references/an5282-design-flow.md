# AN5282 PolarFire SoC 設計フロー リファレンス

> 主出典: *PolarFire SoC Design Flow*, Microchip, **AN5282 / DS00005282B**（Revision B, 07/2024, © 2025）
> ローカル: `C:\_Devs\PolarFire\doc\AN5282_PolarFire_SoC_Design_Flow.pdf`
> 副出典（§9〜§11）: *PolarFire SoC Discovery Kit Reference Design Generation Tcl Scripts - Libero SoC v2025.2+*（GitHub README）
> ローカル: `C:\_Devs\PolarFire\doc\polarfire-soc-discovery-kit-reference-design-README.md`

> **範囲の注意**: AN5282 本文が扱うのは **ファブリックのみの AND ゲート設計**（プロジェクト作成 → SmartDesign → 合成 → ピン割当 → Run PROGRAM Action → 動作確認 → 修正）だけである。**MSS Configurator / SoftConsole / HSS / eNVM は AN5282 には記載がない**ため、§9〜§11 はリファレンスデザイン README の記述でまとめた。**SPI フラッシュへの書き込み手順は doc/ 内のどの資料にも記載がない**（§12）。

---

## 0. 全体フロー早見表

| # | フェーズ | ツール | 主な成果物 / 確認点 | 本書 |
|---|---|---|---|---|
| 1 | プロジェクト作成・デバイス選択 | Libero SoC | MPFS095T / FCSG325 / -1 / EXT | §2 |
| 2 | デザイン入力（SmartDesign） | Libero SoC | `andgate` コンポーネント生成、Set As Root | §3 |
| 3 | 合成 | Synplify Pro（Libero 内） | Synthesize 完了 | §4 |
| 4 | I/O 制約 | I/O Editor | A=T19, B=U18, Y=T18, LVCMOS18 | §5 |
| 5 | 基板準備 | — | ジャンパ、J4 接続 | §6 |
| 6 | 書き込み | Libero → FlashPro Express（batch） | `Chain programming PASSED` | §7 |
| 7 | 動作確認 | — | SW1/SW2 と LED1 | §8 |
| 8 | 修正と再書き込み | Libero | Update and Run | §8.2 |
| 9 | MSS を含む設計 | Tcl スクリプト + MSS Configurator（batch） | `.cxz` / `.xml` | §9（README） |
| 10 | ベアメタル SW / ブートモード / eNVM | SoftConsole, Libero | eNVM クライアント、boot mode 0/1 | §10（README） |
| 11 | HSS の組込み | Tcl 引数 `HSS_UPDATE` | HSS hex を eNVM クライアントに | §11（README） |

---

## 1. 要件（AN5282 Table 1 / Prerequisites）

| Requirements | Version |
|---|---|
| Operating System | Windows 10 |
| Hardware: PolarFire SoC Discovery Kit (MPFS-DISCO-KIT) | REV 1 |
| Software: FlashPro Express | Available with Libero SoC installation package. |
| Software: Libero SoC | （版の記載なし。本文の手順は Libero SoC **v2023.2**） |

- 事前に Libero SoC をインストール。評価には**無償 Silver ライセンス**を使用（取得手順は [quick-start.md §6](quick-start.md)）。
- AN5282 は REV 1 / Libero v2023.2 時点の記述。REV 2 基板・新しい Libero でも手順の骨格は同じと考えられるが未確認 (原文外)。

---

## 2. プロジェクト作成チェックリスト（§1.1）

- [ ] 1. Libero SoC Project Manager を起動（Start > Microchip Libero SoC v2023.2 > Libero SoC v2023.2、またはデスクトップのショートカット）
- [ ] 2. 新規プロジェクト: Start Page の **New**、または **Project > New Project**
- [ ] 3. Project Details:

| 項目 | 値 |
|---|---|
| Project Name | `PolarFire_SoC_fabric` |
| Project Location | `<C:, D:>/` |
| Preferred HDL Type | Verilog or VHDL |

- [ ] 4. **Next**
- [ ] 5. Device Selection → **Next**:

| 項目 | 値 |
|---|---|
| Family | PolarFireSoC |
| Die | MPFS095T |
| Package | FCSG325 |
| Speed | -1 |
| Range | EXT |

- [ ] 6. Device Settings → **Next**:

| 項目 | 値 |
|---|---|
| Core Voltage | 1.0 (default) |
| Default I/O Technology | LVCMOS1.8V |
| Reserve Pins for Probes | Checked (default) |
| System Controller Suspend Mode | Un-checked (default) |

- [ ] 7. **Finish**

---

## 3. SmartDesign でのデザイン入力（§1.1 手順 8〜17）

- [ ] 8. SmartDesign キャンバスを開く: **File > New > SmartDesign**、または Design Flow タブ Create Design の **Create SmartDesign** をダブルクリック
- [ ] 9. 名前 **`andgate`** を入力 → OK（Verilog では大文字小文字を区別）
- [ ] 10. IP カタログの **Macro Library** を展開
- [ ] 11. **AND2** をキャンバスへドラッグ＆ドロップ
- [ ] 12. A, B, Y ピンを選択 → 右クリック → **Promote to Top Level**（外部端子化）
- [ ] 13. 接続後のキャンバスを確認（Figure 1-9）
- [ ] 14. 保存: **File > Save top**
- [ ] 15. 生成: **SmartDesign > Generate Component**（またはツールバーの Generate Component）
- [ ] 16. **Design Hierarchy** タブ → **Build Hierarchy**
- [ ] 17. `andgate` を右クリック → **Set As Root**（太字表示になればルート）

---

## 4. 合成（§2）

- [ ] Design Flow ウィンドウ → **Implement Design**（原文表記 "Impelement Design"）を展開 → **Synthesize** を右クリック → **Run**（Synplify Pro）

---

## 5. ピン割当（§3）

- [ ] 1. Design Flow → **Constraints** → **Manage Constraints** を右クリック → **Open Manage Constraints View**
- [ ] 2. Constraint Manager の **I/O Attributes** タブ → **Edit with I/O Editor**
- [ ] 3. I/O Editor の **Port View** タブ
- [ ] 4. 各ピンの I/O Standard が **LVCMOS18** であることを確認（Rev B で LVCMOS33 → LVCMOS18 に訂正）
- [ ] 5. Pin Number 列の "Unassigned" をクリックして割当（直接入力またはプルダウン）:

| Port | Direction | I/O Standard | Pin Number | 基板上の対象 |
|---|---|---|---|---|
| A | INPUT | LVCMOS18 | **T19** | SWITCH1 |
| B | INPUT | LVCMOS18 | **U18** | SWITCH2 |
| Y | OUTPUT | LVCMOS18 | **T18** | LED1 |

- [ ] 6. **File > Commit and Check**。I/O Editor Log にエラーがないこと
- [ ] 7. **File > Exit** で閉じる
- [ ] （任意）Design Flow 上部の緑矢印で Place and Route まで一括実行

### 5.1 押しボタン（Table 3-1）と LED（Table 3-2）

| Discovery Board | FPGA Pin Number | FPGA Pin Name |
|---|---|---|
| SWITCH1 | T19 | HSIO12NB0 |
| SWITCH2 | U18 | HSIO15NB0/DQS |
| LED1 | T18 | HSIO12PB0/CLKIN_N_9/CCC_NE_CLKIN_N_9 |

- 押しボタンは FPGA に接続されたデバッグ用 2 個。LED は FPGA に接続された 8 個（原文: "eight active low LEDs"）。
- 原文 §6 では「スイッチを離すとロジック 1 → LED 点灯」と記載。図 3-6 の LED 回路はトランジスタ（MBT2222ADW1T1G）経由で駆動されているように見え、FPGA 出力 High で点灯する動作と整合する (原文外: 図からの読み取り)。正確な極性は [schematic-pins.md](schematic-pins.md) で確認。

---

## 6. 基板準備（§4, Table 4-1）

- [ ] 1. 基板の電源 OFF を確認
- [ ] 2. ジャンパを確認:

| Jumper | Location | Purpose | Settings |
|---|---|---|---|
| J45 | Near the Raspberry Pi connector | I/O bank 5 電圧（Pin 1–2 で 3.3V） | 1–2 installed |
| J46 | Near the Raspberry Pi connector | VDDAUX1 電圧（1–2: 3.3V / 2–3: 2.5V） | 1–2 installed |
| J47 | Near the power block | 電源選択（1–2: USB Type C コネクタ J4 給電） | 1–2 installed |
| J49 | Near the RJ45 connector | 外付け 8 桁 7 セグ表示の電圧（1–2: 3.3V / 2–3: 5.0V） | **1–2 installed** |

- [ ] 3. USB Type-C ケーブルで **J4** とホスト PC を接続

（FIR デモでは J49 = 2–3。差異は [quick-start.md §3.2](quick-start.md)）

---

## 7. 書き込み（§5）

FlashPro Express が **batch モード**で MPFS095T を書き込む。

- [ ] 1. 求められたら FlashPro5 ドライバをインストール。場所: `<Libero SoC v20232.2 Installation Directory>\Designer\fp_drivers`（原文どおり）
- [ ] 2. Design Flow → **Program Design** → **Run PROGRAM Action** を右クリック → **Run**（プログラミングファイル生成から書き込みまで実行）
- [ ] 3. **書き込み中は中断しない**
- [ ] 4. Reports View の Run PROGRAM Action に次のメッセージ（プログラマ番号は環境で異なる）:
  ```
  programmer 'E2007ZZUNS': Chain programming PASSED.
  ```
- [ ] 5. Design Flow の Program Design と Run PROGRAM Action に緑のチェック

---

## 8. 動作確認と設計修正

### 8.1 動作確認（§6）

| SW1 | SW2 | LED1（Y = A AND B） |
|---|---|---|
| 離す | 離す | 点灯 |
| 押す | 離す | 消灯 |
| 離す | 押す | 消灯 |
| 押す | 押す | 消灯 |

スイッチを離すとロジック 1。両方押したときに点灯させたい場合は入力を反転する（§8.2）。

### 8.2 設計修正と再書き込み（§7）

- [ ] 1. Design Hierarchy の `andgate` をダブルクリックしてキャンバスを再オープン
- [ ] 2. `andgate` タブを選択
- [ ] 3. CTRL を押しながら `AND2_0` の入力 A, B を選択 → 右クリック → **Invert**（SmartDesign の I/O 反転 / Tie off 機能）
- [ ] 4. **Generate Component**（後段のコンパイル結果はリセットされるが I/O 制約は保持）
- [ ] 5. **Run PROGRAM Action** を右クリック → **Update and Run**（途中工程を全実行して再書き込み）
  - 時間がかかる場合、非タイミングドリブンモードの Warning ダイアログがウィンドウの裏に隠れていないか確認
- [ ] 6. 両スイッチ同時押しで LED1 点灯を確認

---

## 9. MSS を含む設計（README より。AN5282 範囲外）

Discovery Kit 用の MSS 込みデザインは、リファレンスデザインの Tcl スクリプトで生成するのが公式の流れ。詳細は [reference-design.md](reference-design.md)。

- [ ] 1. リポジトリを clone / download
- [ ] 2. Libero を開き、Execute Script（**CTRL + U**）で `MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.tcl` を実行（必要なら引数）
  - スクリプトが **PolarFire SoC MSS Configurator を batch モードで実行して MSS コンポーネントを生成**し、コンポーネント生成・インスタンス化・接続・トップレベル I/O 作成、I/O / フロアプラン制約（Place and Route 用）とタイミング制約（Synthesis / Place and Route / Timing Verification 用）の関連付けまで行う
- [ ] 3. 必要に応じてデザインを変更
- [ ] 4. Design Flow のステージ（例: Run PROGRAM Action）をダブルクリック → 必要な前段（Synthesize, Place and Route 等）を自動実行して書き込み

### MSS Configurator の成果物

| 成果物 | 拡張子 | 用途 |
|---|---|---|
| MSS 構成ファイル | `.xml` | PolarFire SoC Configuration Generator がベアメタル用構成ヘッダを生成（MPFS HAL のスタートアップコードでコンパイルされ、電源投入時にシステムを構成） |
| MSS コンポーネント | `.cxz` | Libero デザインにインポートして FPGA 設計フローで使用 |

- 保存済み MSS Configurator 構成は `script_support` フォルダ。引数付き生成時は `script_support/additional_configurations/[configuration name]` に生成。
- 全構成の XML はリポジトリの `XML` フォルダ。

---

## 10. ベアメタルアプリ・ブートモード・eNVM（README より）

### 10.1 SoftConsole のブートモードプログラマを Discovery Kit 向けに設定

既定は Icicle Kit 向けなので変更が必要。

- [ ] 1. **Run -> External Tools -> External Tools Configurations...**
- [ ] 2. "PolarFire SoC program idle boot mode..." 等、ブートモードごとの各構成の引数を **die `MPFS095T`**、**package `FCSG325`** に変更
- [ ] 3. Apply

boot mode 0 の引数例（原文どおり）:

```
-jar ${eclipse_home}/../extras/mpfs/mpfsBootmodeProgrammer.jar 
--bootmode 0
--die MPFS095T
--package FCSG325
```

### 10.2 ブートモード一覧

| Boot mode | 動作 | 設定に使えるツール |
|---|---|---|
| 0 | 電源投入時、全 MSS コアが WFI（Wait For Interrupt）。デバッガ接続までブートしない | SoftConsole **v6.5 以降** |
| 1 | 全 MSS コアが **eNVM** からブート。**eNVM クライアント必須**（HSS などのゼロステージブートローダ、またはベアメタルアプリ） | SoftConsole **v6.4 以上** / Libero SoC **v12.6 以上** |

### 10.3 Boot mode 0 の設定（SoftConsole）

- [ ] 1. 内蔵 FlashPro6 または外付け FlashPro で基板と PC を接続し、電源 ON（原文は "embedded FlashPro6"。QSG 等では FP5 / FlashPro5 と表記）
- [ ] 2. "External Tool" ドロップダウン
- [ ] 3. "PolarFire SoC idle boot mode 0" を実行

### 10.4 Boot mode 1: SoftConsole で eNVM に書き込む

- [ ] 0. ベアメタルアプリを、提供されたリファレンス XML で MPFS HAL を構成してビルド（MSS コンポーネント生成時に `script_support/components/[target]` にも XML が出力される）
- [ ] 1. Project Explorer で eNVM クライアントにするアプリのプロジェクトフォルダを選択
- [ ] 2. FlashPro6 / 外付け FlashPro で接続し電源 ON
- [ ] 3. "External Tool" ドロップダウン
- [ ] 4. "PolarFire SoC non-secure boot mode 1" を実行

### 10.5 Boot mode 1: Libero で eNVM クライアントを追加

- [ ] 1. Design Flow を **Generate FPGA Array Data** 完了まで実行 → **Configure Design Initialization Data and Memories** を開く
- [ ] 2. **eNVM** タブ
- [ ] 3. **Add ...** → **Add Boot Mode 1** client
- [ ] 4. クライアントにするバイナリファイルを選択 → OK
- [ ] 5. 保存してクライアントを追加
- [ ] 6. 残りの Design Flow を実行して書き込み

---

## 11. HSS（Hart Software Services）の組込み（README より）

| 方法 | 内容 |
|---|---|
| Tcl 引数 **`HSS_UPDATE`** | このリファレンスデザインのリリースに対応する HSS リリース hex をダウンロードし、Libero の **eNVM クライアント**として追加。**wget が必要**（Windows では wget をインストールして PATH に追加） |
| Boot mode 1 + eNVM クライアント | HSS は「ゼロステージブートローダ」の eNVM クライアント例として挙げられている（§10.2） |
| FlashPro Express ジョブ | リリース済みジョブ（Releases）を書けば HSS 込みで Linux 起動可能な状態になる（Embedded SW UG）。詳細は [embedded-software.md](embedded-software.md) |

- DDR を soft CPU / ファブリックイニシエータから使う場合、**HSS またはベアメタルアプリで DDR トレーニングと MSS の FIC ターゲット有効化**が必要（README "MSS DDR4 configuration"）。

---

## 12. ビットストリーム出力と SPI フラッシュについて

| 出力 | 方法 | 出典 |
|---|---|---|
| デバイスへ直接書き込み | Run PROGRAM Action（Libero） / Tcl 引数 `PROGRAM` | AN5282 §5 / README |
| プログラミングデータ生成 | Tcl 引数 `GENERATE_PROGRAMMING_DATA` | README |
| FlashPro Express ジョブ（.job） | Tcl 引数 `EXPORT_FPE` / `EXPORT_FPE:PATH` | README |
| ビットストリームの design version / silicon signature | `DESIGN_VERSION:VERSION`（16 bit）/ `SILICON_SIGNATURE:SIGNATURE`（既定 `D15C0417`） | README |
| eNVM | §10.5（Libero）/ §10.4（SoftConsole） | README |
| **SPI フラッシュ** | **doc/ 内の AN5282・AN5165・QSG・Embedded SW UG・README のいずれにも手順の記載なし** | — |

> SPI フラッシュ経由のプログラミング（Auto Update / IAP 等）が必要な場合は、User Guide 本体（skill の chapters/）、回路図（[schematic-power.md](schematic-power.md) ほか）、MSS TRM（skill polarfire-soc-mss）、または Libero の Configure Design Initialization Data and Memories の SPI Flash タブ等の公式資料を別途確認すること (原文外)。

---

## 13. ツールのバージョン対応メモ

| 資料 | Libero | 基板 Rev | その他 |
|---|---|---|---|
| AN5282（DS00005282B） | v2023.2 | REV 1 | Windows 10 |
| QSG（DS50003565J） | v2024.1 | — | — |
| AN5165（DS00005165D） | v2024.1（`v2024p1`） | REV 2 | Windows 11 |
| リファレンスデザイン README | **v2025.2 以上** | — | SoftConsole v6.4+/v6.5+（ブートモード） |

## 14. 改訂履歴（AN5282）

| Rev | 日付 | 内容 |
|---|---|---|
| B | 07/2024 | Pin Assignments 手順の I/O Standard を "LVCMOS33" → "LVCMOS18" に更新 |
| A | 01/2024 | 初版 |
