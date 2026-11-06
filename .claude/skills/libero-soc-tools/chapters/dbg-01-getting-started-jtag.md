# SmartDebug UG: 起動モード・JTAG 接続・デバイス状態（PDF p.1-24）

## Core Idea
SmartDebug は JTAG 経由で FPGA ファブリック・トランシーバ・メモリ・MSS レジスタを実機デバッグするツール。内蔵ロジアナ IP は不要で、ファブリックに作り込まれた専用プローブ回路を使うため、プローブ点の変更に再コンパイルが要らない。**JTAG 専用（SPI ポートは非対応）**。使う前に FlashPro プログラマを PC とターゲットに接続し、デバイスを書き込んでおく。

## Frameworks Introduced
- **3 つの起動モード（p.4-6）**
  | モード | 起動 | 使える機能 |
  |---|---|---|
  | Integrated | Libero Design Flow > Debug Design > **SmartDebug Design** をダブルクリック | 全機能。**Probe Insertion はこのモードだけ** |
  | Standalone | `<Libero>/Designer/bin/sdebug.exe` など。プロジェクト作成＋DDC 取込み | DDC ありでほぼ全機能、DDC なしは限定。Probe Insertion 不可（incremental routing が必要なため） |
  | Demo | HW 未接続で起動 | Active/Live Probe、Memory Blocks、Transceiver、sNVM、UPROM、Fabric DDR、IOD の画面確認のみ。途中で他モードへ切替不可 |
- **Standalone プロジェクトの作り方（p.4-5）**: Project > New Project
  - **Import DDC File**: Libero の Export SmartDebug Data で作った DDC（Design Debug Data Container）を取込む。デバッグデータと JTAG チェーン情報を継承
  - **Construct Automatically**: 選んだプログラマのチェーン上の全デバイスでプロジェクトを作る（FlashPro の Construct Chain Automatically 相当）。作成直後は全デバイス disabled
  - DDC とソフトのバージョン非互換なら作れない → Libero で Generate SmartDebug FPGA Array Data → Export SmartDebug Data をやり直す
- **JTAG チェーン上のデバイスをデバッグ（p.6-9）**
  1. Libero: Handoff Design for Debugging > Export SmartDebug Data
  2. Standalone を起動し Construct Automatically
  3. Programming Connectivity and Interface でデバイスを Enable、DDC をロードして書き込み
  4. Device ドロップダウンで対象を選ぶ。**同時にデバッグできるのは 1 デバイス**
- **基本の最初の一手（p.4）**: SmartDebug を開いたら **View Device Status** で異常がないか見る
- **リモートプログラマ（v2026.1 新規, p.18-24）**: 開発 PC → Ethernet/VPN → Remote Programming Server（RPS）→ USB → プログラマ → FPGA
  1. プロジェクトを開き Configure Remote Server
  2. Name or IP address に入力（非既定ポートは `host:7898` 形式）→ Add → State が緑
  3. Status が Available のプログラマを Select → OK（予約される）
  - RPS は Libero Media Installer の Custom で RPS を選んで入れる。既定ポート **57605 / 57606**（TCP）。1 サーバあたり 40 台以下推奨
  - 信頼済み LAN/VPN 専用。ポートをインターネットに晒さない

## Key Concepts
- **DDC**: Libero から書き出すデバッグ用データ（Export SmartDebug Data）
- **Programming Connectivity and Interface**: TDI→TDO の物理チェーン表示。Construct Chain Automatically / Scan and Check Chain / Run Programming Action
- **Set as Libero Design Device**: 同一デバイスがチェーンに複数あるとき対象を指定
- **Configure Device**: 同じ IDCODE の派生品（同一ダイ）で Family/Die を明示する
- **Debug context**: Debug FPGA Array 等の選択状態。Auto Construct・DDC 取込み・Configure Device・Load Programming File でリセットされる
- **Device Status Report**: IDCode、Device Certificate、Design Name/Checksum/Version、Fabric/sNVM Digest、Security（Live Probe 可否含む）、Programming 情報
- **Cycle Count**: 工場出荷後の書き込み回数。上限なしだが目安の下限閾値は約 2000 回

## Reference Tables
**Programmer Settings 注意（p.12）**: Discrete Clocking TCK を選ぶと書き込み時間が大幅に延びる警告が出る

**リモート接続トラブル（p.22-24）**
| 症状 | 対処 |
|---|---|
| ホストに繋がらない | リモート側で TCP 57605-57606 を開放（Windows Defender Firewall 受信規則 / `sudo ufw allow 57605:57606/tcp` / `sudo firewall-cmd --permanent --add-port 57605-57606/tcp`） |
| 繋がるがプログラマが出ない | Media Installer の post-install チェック、RPS 再インストール |
| 異常終了後に動かない | 最終手段: `bin/CleanUpRPS`、または prgsrvdmn / prgservapp / dbgservapp を kill |
| Linux で FlashPro6 が消える | 他の SmartDebug セッションが掴んでいる。終了を待つ |

## Anti-patterns
- **SPI 経由でデバッグしようとする**: SmartDebug は JTAG のみ
- **Standalone で Probe Insertion を探す**: Integrated（Libero から起動）でしか出ない
- **設計変更後に書き込まずにデバッグ**: 先に Programming Connectivity and Interface で書き込む
- **Demo モードの値を実機と取り違える**: Demo は一時ファイルの値を表示しているだけ

## Key Takeaways
1. 普段は Libero から SmartDebug Design（Integrated）で起動する
2. ラボ PC で使うなら Export SmartDebug Data → Standalone で DDC 取込み
3. 最初に View Device Status で IDCode・デザイン名・セキュリティを確認
4. JTAG チェーンでも同時デバッグは 1 デバイス
5. v2026.1 からリモートプログラマ（RPS, TCP 57605/57606）が使える
6. Discovery Kit はオンボード eFP5（FlashPro5）が JTAG プログラマになる（姉妹 skill polarfire-soc-discovery-kit より。原文外）

## Connects To
- **dbg-02**: Debug FPGA Array（Live/Active Probe）
- **dbg-06**: `new_project` / `import_ddc_file` / `read_device_status` 等の Tcl
- 姉妹 skill **polarfire-soc-discovery-kit**: eFP5 と FlashPro Express
