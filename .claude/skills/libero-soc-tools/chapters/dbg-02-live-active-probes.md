# SmartDebug UG: Debug FPGA Array — Live Probe / Active Probe（PDF p.25-33, 61-64, 170-171）

## Core Idea
Debug FPGA Array は Live Probes / Active Probes / Memory Blocks / Probe Insertion の 4 タブと、FHB 系（Event Counter / Frequency Monitor / User Clock Frequencies）の操作を持つ。Live Probe と Active Probe はどちらも**フラッシュスイッチ書き込み用の回路を流用した専用プローブ回路**で、設計変更・再コンパイルなしで使える。
- **Live Probe**: 内部の 2 点（Channel A / B）を専用の I/O パッドへリアルタイムに出し、オシロで見る
- **Active Probe**: JTAG で任意の DFF を非同期に読み書きする（スナップショット / 強制）

## Frameworks Introduced
- **プローブ点の選び方（p.25-28）**
  - Hierarchical View: インスタンス階層。インスタンスを追加すると配下の全プローブ点が入る。同名信号は自動でバス化
  - Netlist View: フラットなネット一覧とセル種別（Type = フリップフロップの物理位置）
  - Filter は `*` `?` のワイルドカード可。検索結果は葉インスタンスだけ追加できる
- **Live Probe の使い方（p.28-29）**
  1. Live Probes タブでプローブ名を選ぶ
  2. 右クリック **Assign to Channel**（または Assign to Channel ボタン）で A / B に割当て
  3. ボタン右にプローブ名が出たら、チャネル I/O を外部オシロで観測
  4. 終了は **Unassign Channels**
  - Save / Load でプローブ一覧をファイル保存・復元
- **Active Probe の使い方（p.29-31, FAQ p.170-171）**
  1. Hierarchical / Netlist View で選び右クリック Add（または Add ボタン）
  2. **Read Active Probes** で現在値を読む（前回から変化した値は青）
  3. Write Value 列に値（単信号は 0/1、バス・グループは hex/bin）→ **Write Active Probes**（成功は緑）
  - タイミング絡みの不具合を見るときは、FHB で設計クロックを止めてから全 DFF を読むのが推奨
- **Pseudo Static Signal Polling（p.61-64）**: Active Probe のタイミングでは捕まえにくい「いずれ特定値になり留まる」信号用
  - Active Probes で信号/バス/グループを右クリック **Poll**
  - スカラーは Poll for 0 / Poll for 1、ベクタは値を入力（形式検証あり）→ Start Polling
  - **1 秒間隔**でポーリングし、一致で "User value matched"（緑）。実行中は値変更・ダイアログを閉じる操作不可

## Key Concepts
- **Channel A / B**: Live Probe の出力 2 系統。PolarFire / PolarFire SoC は 2 本、RTG4 は 1 本
- **Live Probe I/O**: チャネル出力用の専用パッド。デザインがこの I/O を**入力として使っていると Live Probe タブは無効**、Active Probe の Assign to Channel メニューも隠れる
- **Probe point**: プローブ可能な DFF 出力など。PolarFire は隣接配置された **18 DFF 単位**で読む
- **対応/非対応**: 順序素子と LSRAM・uSRAM・MATH の出力は可。**組合せ論理と I/O パッド関連レジスタは不可**（Live / Active 共通）
- **Bus と Probe Group**: Bus は同名ネットの自動グループ（Ungroup 不可・Move/Remove 不可、名前変更不可）。Probe Group は任意信号の手動グループ（Ungroup 可）
- **右クリック操作**: Create Group / Add・Move to Group / Remove from Group / Ungroup / Reverse Bit Order / Change Radix / Read / Write / Delete / Poll

## Reference Tables
**Live Probe と Active Probe の比較**
| 観点 | Live Probe | Active Probe |
|---|---|---|
| 観測 | 実時間の波形（外部オシロ） | JTAG でのスナップショット読み |
| 書き込み | 不可 | 可（DFF を強制） |
| 同時本数 | 2（A/B） | 多数（グループ・バス） |
| 必要な HW | 専用 Live Probe I/O ピン＋オシロ | JTAG のみ |
| 設計変更 | 不要 | 不要 |
| 相互影響 | Active Probe の読み書きで Live Probe 設定が乱れる（同じ回路を共有, p.253） | — |

**Live Probe の制約（UG 本文 + `set_live_probe` の Exception, p.28-29, 252-253）**
- アレイが書き込み済みで動作中であること
- ProbeA / ProbeB の I/O が Live Probe 用に予約されていること（予約手順は本 UG に記載なし。Tcl Command Reference のデバイス設定 `RESTRICTPROBEPINS`（1 = probe pins reserved）が該当: 原文外）
- 片チャネルだけ設定すると他方に影響する → 両方使うなら **1 回の `set_live_probe` で A と B を同時に指定**
- セキュリティロックで無効になり得る（Device Status の Security 欄に Live Probe 可否が出る）
- Live Probe I/O をデザインで入力に使うと Live Probe・FHB・Spatial Debug Widget が無効

## Anti-patterns
- **組合せ論理のネットを Live/Active Probe しようとする**: 対象外。その段の DFF 出力を見るか Probe Insertion（dbg-03）を使う
- **Live Probe 観測中に Active Probe を読み書き**: 同一回路のため Live Probe の設定が崩れる
- **プローブピンをユーザ I/O（入力）として流用**: Live Probe が使えなくなる
- **動作中のクロック域で Active Probe の値をそのまま信じる**: 非同期読み出し。整合した状態が欲しければ FHB で Halt してから読む

## Key Takeaways
1. 2 本までの波形を見るなら Live Probe、多数の DFF を読み書きするなら Active Probe
2. どちらも再コンパイル不要。ただし対象は DFF（と RAM/MATH 出力）に限られる
3. Live Probe は A/B を同時設定する。Active Probe 操作とは干渉する
4. Live Probe 専用ピンを入力として使う設計にしない
5. 「いずれ立つフラグ」は Poll（1 秒周期）で待つ

## Connects To
- **dbg-03**: Probe Insertion（任意ネットを空き I/O へ）、FHB / Event Counter / Frequency Monitor（Channel A を使う）
- **dbg-06**: `set_live_probe` / `unset_live_probe` / `select_active_probe` / `read_active_probe` / `write_active_probe`
- 姉妹 skill **polarfire-soc-discovery-kit**: J12（Live probes ヘッダ）
