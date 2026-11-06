# Quick Start Guide リファレンス

> 出典: *PolarFire SoC Discovery Kit Quickstart Guide*, Microchip, **DS50003565J**（© 2026, Online Reference）
> ローカル: `C:\_Devs\PolarFire\doc\board\PolarFire_SoC_Discovery_Kit_Quick_Start_Guide.pdf`
> 補足出典は各所に明記（AN5165 = DS00005165D、AN5282 = DS00005282B、Embedded SW UG = GitHub md）。

---

## 1. 同梱物（Discovery Kit Contents）

| 数量 | 内容（原文どおり） |
|---|---|
| 1 | PolarFire SoC Discovery Kit Board with MPFS095T-1FCSG325E |
| 1 | USB 2.0 Cable, USB-C Male to USB-C Male, 3.28ft (1.00m), Shielded |
| 1 | Quickstart card |

- SD カードは同梱されない。Linux / AMP 運用には別途必要（SanDisk 製推奨。§7 参照）。
- 基板の部品配置図は原本 Figure 1（Top-view）/ Figure 2（Bottom-View）。テキストには含まれないので原本 PDF を参照。

---

## 2. 箱出しから FIR デモまでの手順チェックリスト

> 前提: **DSP FIR Filter Demonstration は Windows ホスト PC のみ対応**。デモの実行だけなら **Libero SoC は不要**。

- [ ] **1. ジャンパ確認**（§3 の表）。ESD 対策をしてから基板を扱う。
- [ ] **2. ドライバとソフトの導入**
  - [ ] FTDI ドライバ: `FTDIChip.com/Drivers` から取得・インストール（基板は FTDI チップで USB-to-UART を実現）
  - [ ] Microchip **FlashPro Express** をインストール（AN5165 によると Libero SoC インストールパッケージに同梱）
- [ ] **3. 付属 USB Type-C ケーブルで基板とホスト PC を接続**
  - ケーブル 1 本で給電と通信（FPGA 書き込み・デモ通信）の両方を行う
  - ノート PC / デスクトップの USB ポートに**直結**する。**ドッキングステーションの USB ポートは未検証**
  - AN5165 補足: 基板側は USB Type-C ポート **J4**、ホスト側は **5V, 3A** 対応の Type-C ポート。電源 LED **5P0V, VDD, 1P8V** が点灯すれば給電 OK
- [ ] **4. ホスト PC を再起動し、ドライバ初期化を確認**（§4 の手順）
- [ ] **5. デモ資料の取得**
  - [ ] AN5165 の Web ページを開く
  - [ ] "Download Application Note" で AN（PDF）を取得（GUI のインストールと実行手順の全体はこちら）
  - [ ] "Download" で .zip（GUI、スクリプト、プログラミングジョブ）を取得・展開
- [ ] **6. 以降は AN5165 の手順で書き込み・GUI 実行** → [an5165-fir-demo.md](an5165-fir-demo.md)

補足: YouTube 動画「Using the Discovery Kit and DSP FIR Filter Demo」が QSG §2 からリンクされている（URL は原本の画像リンクのみ）。

---

## 3. FIR デモ用ジャンパ設定

### 3.1 QSG Table 1-1（USB Type-C ポート使用時）

| Jumper | Setting |
|---|---|
| J45 | 1 and 2 closed |
| J46 | 1 and 2 closed |
| J47 | 1 and 2 closed |
| J49 | **2 and 3 closed** |

### 3.2 資料間の比較（作業時の注意）

| Jumper | QSG Table 1-1（FIR デモ） | AN5165 Table 3-1（FIR デモ） | AN5282 Table 4-1（AND ゲート設計） | Embedded SW UG（Linux 起動） |
|---|---|---|---|---|
| J45 | 1–2 closed | 1–2 closed: Bank 1 / Bank 5 電圧（VDDI1_5）を 3.3V（RPi GPIO・mikroBus 用） | 1–2 installed: I/O bank 5 電圧（1–2 で 3.3V） | 1 & 2: "Bank 1 voltage. 1 & 2: 3.3V, 2 & 3: 2.5v" |
| J46 | 1–2 closed | 1–2 closed: VDDAUX1 を 3.3V | 1–2 installed: VDDAUX1（1–2: 3.3V / 2–3: 2.5V） | 1 & 2: "Bank 1 voltage. 1 & 2: 3.3V, 2 & 3: 2.5v" |
| J47 | 1–2 closed | 1–2 closed: USB Type-C（J4）から給電 | 1–2 installed: J4 USB Type-C 給電 | Closed: USB C 給電 / open: ジャック給電 |
| J49 | **2–3 closed** | **2–3 closed: 7 セグ表示電圧 5V** | **1–2 installed**（1–2: 3.3V / 2–3: 5.0V） | **1 & 2**（7 セグコネクタ基準電源 1&2: 3.3V, 2&3: 5V） |

- **J49 は資料によって異なる**。FIR デモ（QSG / AN5165）は 2–3（5V）、AN5282・Linux 用既定値は 1–2（3.3V）。7 セグ表示アダプタを使わない場合の影響は原文に記載なし。
- J45/J46 の説明文は資料ごとに表現が揺れている（Embedded SW UG は J46 も "Bank 1 voltage" と記載）。正確な電源系統は User Guide 本体・回路図（[schematic-power.md](schematic-power.md) ほか schematic-*.md）で確認すること。

---

## 4. ドライバ確認手順（FP5 Serial Converter）

1. Windows のデバイスマネージャーを開く（スタートを右クリック、または `Windows key + X` → Device Manager）
2. **Ports (COM & LPT)** を展開
3. 各ポートを右クリック → **Properties**
4. いずれかのポートのプロパティに **`Location: on USB FP5 Serial Converter D`** と表示されれば、ドライバ導入済み・接続正常

| 結果 | 対処 |
|---|---|
| "on USB FP5 Serial Converter D" のポートがある | OK。そのポートの COM 番号を控える（GUI で使用、(原文外)） |
| どのポートにも表示されない | 手順 2（ドライバ導入）と 3（ケーブル接続）をやり直す |

- COM ポート番号は環境依存（原本 Figure 1-2 の番号と一致しなくてよい）。
- AN5282 補足: Libero から書き込む際に FlashPro5 ドライバを求められた場合、ドライバは `<Libero SoC v20232.2 Installation Directory>\Designer\fp_drivers` にある（パス表記は原文どおり）。

---

## 5. Linux に関する注意（QSG §3）

- QSG が案内する GitHub のリファレンスデザインは、**FIR デモとは機能が異なる**ベースデザイン。
- **Linux を起動するには**、Discovery Kit Reference Design Repository のリファレンスデザインで基板を書き換える必要がある。
- Libero を使わずに書き込める **FlashPro Express ファイルはリポジトリの Releases** にある。
- 詳細: [reference-design.md](reference-design.md)、[embedded-software.md](embedded-software.md)

---

## 6. Libero SoC 無償 Silver ライセンスの取得（QSG §4）

- [ ] 1. **Microchip Direct** にアクセス
- [ ] 2. Microchip Direct に登録 / サインイン
- [ ] 3. Products リストで **Field-Programmable Gate Arrays (FPGAs)** をクリック
- [ ] 4. **FPGA Licenses** の下の **Generate Free and Paid Licenses** をクリック
- [ ] 5. **Request Free License** をクリック

- AN5165 / AN5282 とも「Discovery Kit での評価には無償の Libero SoC Silver ライセンスを使う」と記載。
- リファレンスデザイン README: 無償・有償どちらの Libero ライセンスでも生成可能。
- ライセンス取得・設定の資料: `https://coredocs.s3.amazonaws.com/Libero/2023_1/Tool/libero_download_license_quickstart.pdf`

---

## 7. 新規プロジェクト開発の前提（QSG §5）

PolarFire SoC はプログラマブルロジックファブリックと 4 コア RISC-V マイクロプロセッササブシステムを持ち、Linux / RTOS / BareMetal を SMP・AMP で動かせる。

| 必要物 | 内容（原文の要旨） |
|---|---|
| SD Card | Linux または AMP 運用に必要。SanDisk 製推奨 |
| SoftConsole | 無償の開発環境（Windows / GNU/Linux）。ベアメタル・RTOS の C/C++ 開発とデバッグ |
| Libero SoC Design Suite **v2024.1** | RTL 入力〜プログラミングまでの統合ツール |

> 版差の注意: 現行リファレンスデザイン README は **Libero SoC v2025.2 以上**を要求（[reference-design.md](reference-design.md)）。QSG の v2024.1 は FIR デモ（`mpfs_an5165_v2024p1_df`）や旧タグ `2024.04` に対応する版と考えられる (原文外)。

---

## 8. Linux ホストでの udev 設定（QSG §5.1 → Embedded SW UG）

- Linux ホスト PC で使う場合、**FTDI USB-to-UART ブリッジを Linux に認識させる udev ルールを追加**する。設定しないと COM ポート（`/dev/ttyUSB*`）が現れないことがある。
- ルール本文・配置先（`/etc/udev/rules.d/70-microchip.rules`）は [embedded-software.md §7](embedded-software.md) を参照。

---

## 9. 参照リンク（QSG Table 6-1 ほか）

| 項目 | リンク（原文どおり） |
|---|---|
| Discovery Kit 製品ページ | `www.Microchip.com/DiscoveryKit`（Embedded SW UG では `https://www.microchip.com/en-us/development-tool/mpfs-disco-kit`） |
| 基本設計フローの AN | AN5282: PolarFire SoC Design Flow Application Note → [an5282-design-flow.md](an5282-design-flow.md) |
| ハードウェア UG | PolarFire SoC FPGA Discovery Kit Hardware User Guide（skill 本体） |
| 回路図 | PolarFire SoC Discovery Kit Schematics → [schematic-power.md](schematic-power.md) ほか schematic-*.md |
| 基板設計ファイル | PolarFire SoC Discovery Kit Board File（.brd） |
| 組込みソフト UG | Discovery Kit Embedded Software User Guide → [embedded-software.md](embedded-software.md) |
| MSS（プロセッササブシステム） | PolarFire SoC MSS Technical Reference Manual → skill `polarfire-soc-mss`（[SKILL.md](../../polarfire-soc-mss/SKILL.md)） |
| Microchip FPGA 製品ページ | FPGA Project Pages |
| 設計ツール | `www.microchip.com/libero` |
| ライセンス取得・設定 | `https://coredocs.s3.amazonaws.com/Libero/2023_1/Tool/libero_download_license_quickstart.pdf` |
| RISC-V パートナーエコシステム | `www.microchip.com/mi-v` |
| リファレンスデザイン生成 Tcl | `https://github.com/polarfire-soc/polarfire-soc-discovery-kit-reference-design/releases/tag/2024.04` |
| ベアメタルドライバ・例 | `www.github.com/polarfire-soc/polarfire-soc-bare-metal-examples` |
| Yocto Linux ビルドシステム | `https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp` |
| ナレッジベース | `https://microchip.my.site.com/s/article/KB-of-PFSoC-Knowledge-Base-articles` |
| QSG オンライン版 | `https://onlinedocs.microchip.com/v2/literature/DS50003565?version=latest&redirect=true`（Embedded SW UG の References より） |

### サポート窓口（QSG Table 7-1）

| Support | URL/Contact | 内容 |
|---|---|---|
| Technical Support | `Microchip.com/Support` | サポート、フォーラム、wiki、トレーニング、コード例 |
| Technical Support Line | (888) 624-7435 | Press 2 for technical support |
| GitHub Repository | `GitHub.com/PolarFire-SOC` | ドキュメント、リファレンスデザイン、ソフト |
| Microchip FPGAs & SOCs | `Microchip.com/FPGA` | FPGA、SoC、設計ソフト、開発ハード、IP |
| My Microchip | `Microchip.com/MyMicrochip` | 個人ポータル |
| Microchip Direct | `MicrochipDirect.com` | 直販 |
| Product Alerts | `Microchip.com/PCN` | 製品変更通知 |
| Microchip University | `Microchip.com/MU` | トレーニングコース |

テクニカルサポートのケース作成時は、FPGA デバイス型番を明記し、適切なカテゴリを選び、デザインファイルをアップロードする（QSG "Microchip FPGA Support"）。
