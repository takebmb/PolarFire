# SmartDebug UG: Probe Insertion と FPGA Hardware Breakpoint（PDF p.45-61, 171-174）

## Core Idea
- **Probe Insertion**: 内部ネットを**空きユーザ I/O**へ配線し直す post-layout デバッグ。Libero が incremental Place and Route と再書き込みを自動で行う。Live/Active Probe が扱えない組合せ論理のネットも外に出せる
- **FHB（FPGA Hardware Breakpoint）**: クロックドメインごとにクロックゲートを自動挿入し、Halt / Play / Step、Live Probe Channel A をトリガにした停止、波形キャプチャを可能にする。Event Counter・Frequency Monitor・User Clock Frequencies も FHB 前提

## Frameworks Introduced
- **Probe Insertion 手順（p.46-47）**: Integrated モード専用（FlashPro 接続必須）
  1. Design Flow で SmartDebug Design → Debug FPGA Array → **Probe Insertion** タブ
  2. Hierarchical / Netlist View からネットを Add（Net と Driver 名で識別）
  3. Port Name は自動生成（変更可）
  4. **Package Pin** 列で未使用ピン（spare I/O）を選ぶ
  5. **Run** → incremental P&R → 自動再書き込み。進捗は Log
- **元に戻す（p.47-48）**: Delete All → **Run** → 青い回転アイコンが消えるまで待つ → SmartDebug を閉じる。Run しない Delete は設計から消えない
- **FHB 自動挿入の有効化（p.55）**
  1. Project > Project Settings > Design flow > **Enable FPGA Hardware Breakpoints Auto Instantiation**
  2. PolarFire / PolarFire SoC は NDC 制約ファイル（Constraint Manager）にクロックドメインごとに記述:
     ```
     auto_instantiate_fhb -inst_name {<clkint name here>}
     ```
     対象は CLKINT / GCLKINT / RCLKINT / RGCLKINT（FCCC に限らない）。全ドメインに入れるならドメイン数だけ書く
  3. 合成済みなら**合成をやり直す**
- **FHB 操作（p.56-59）**: Active Probes タブ側に FHB ウィジェットが出る
  - モード: All Clock Domains / Selected Clock Domain（切替で前の設定は保持されない）
  - **Force Halt**: Halt ボタンで即停止
  - **Live Probe Halt**: Live Probe **Channel A（PROBE_A）** に信号を割当て → Edge（PolarFire は Rising / Falling 選択可）と Delay Cycles Before Halt（0-255）→ **Arm Trigger**
  - **Play** で再開、**Step** で 1 クロック進める（All モードは Select Clock Domain の順に逐次）
  - **Capture Waveform**: Active Probe に入れた信号を 1-8192 サイクル分 `.vcd` に保存
  - **Trigger Input**: DUT 側の信号で FHB をトリガ。Arm 時点で High なら即停止
  - **FHB Status** タブで全ドメインの状態を一覧（Tcl 等価なし）
- **Event Counter（p.48-51）**: Channel A に割当て → Activate Event Counter → 立上りエッジ数と経過秒を表示 → Stop。実行中は Active Probes / Memory Blocks / Probe Insertion が無効
- **Frequency Monitor（p.51-55）**: Channel A に割当て → Frequency Monitor タブ → 監視時間を選び Activate → MHz 表示。時間が長いほど精度が上がる
- **User Clock Frequencies（p.61）**: FCCC で設定したクロックの周波数一覧。計算中は Live Probe 割当てが一時解除→復帰。Refresh で再計算

## Key Concepts
- **incremental mode P&R**: 既存配置配線を保ったまま probe ネットだけ追加配線
- **Delay 実効値**: 実 HW の遅延 = 設定値 × (DUT クロック / FHB クロック)。PolarFire の FHB クロックは **160 MHz**
- **CLKINT_PRESERVE**: これを含む設計には FHB が自動挿入されない
- **System Controller Suspend Mode**: ON だと FHB を入れられない
- **FHB SDC**: PolarFire 系は自動生成されるが Constraint Manager には表示されず管理不可

## Reference Tables
**デバッグ手段の使い分け**
| 目的 | 手段 | 前提 |
|---|---|---|
| 2 本の内部 DFF 波形を見る | Live Probe | Live Probe I/O 予約 |
| 組合せ論理/多数ネットを外に出す | Probe Insertion | Integrated、空き I/O、再 P&R |
| 条件で止めて全 DFF を読む | FHB + Active Probe | FHB 自動挿入＋NDC＋再合成 |
| 立上り回数を数える | Event Counter | FHB、Channel A |
| 信号周波数を測る | Frequency Monitor | FHB、Channel A |
| PLL 出力周波数一覧 | User Clock Frequencies | FHB |

**FHB の制限（p.60-61）**
- DUT クロック ≤ 160 MHz なら 1 クロック以内に停止。超えると停止位置は保証されない
- 1 ドメインだけ止めても他ドメインは走り続ける
- EDIF + 制約フローは非対応
- PolarFire 系は暗号化/セキュア部品を含む設計にも挿入可（SmartFusion2/IGLOO2/RTG4 は不可、CCC の GL のみ）
- Event Counter の既知不具合: Channel A が DC '0' や遷移なしだと停止時に 1 と表示される

## Anti-patterns
- **Probe Insertion を残したまま本番ビルドに進む**: Delete All → Run で必ず元に戻す
- **リストから Delete しただけで消えたと思う**: Run するまで設計には残る
- **FHB を有効にしたのに再合成しない**: FHB 機能が出てこない
- **NDC に `auto_instantiate_fhb` を書かない（PolarFire 系）**: 他ファミリと違い自動生成されない
- **160 MHz 超のクロックで正確な停止サイクルを期待**: 保証外

## Key Takeaways
1. 組合せ論理や多数の信号を外部測定器で見るなら Probe Insertion（空き I/O＋再 P&R が必要）
2. Probe Insertion は Integrated モード専用。終わったら Delete All → Run
3. FHB は Project Settings のチェック＋NDC の `auto_instantiate_fhb`＋再合成で入る
4. Event Counter / Frequency Monitor / Live Probe Halt はすべて Channel A を使う
5. FHB の基準は 160 MHz。それより速いドメインの停止位置はずれる

## Connects To
- **dbg-02**: Channel A の割当て、Active Probe の読み書き
- **dbg-06**: `add_probe_insertion_point` / `program_probe_insertion` / `fhb_control` / `event_counter` / `run_frequency_monitor` / `get_user_clock_frequencies`
