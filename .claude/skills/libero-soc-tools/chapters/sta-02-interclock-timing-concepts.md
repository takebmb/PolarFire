# SmartTime STA UG: Inter-Clock Domain 解析とタイミング概念（PDF p.22-26, 47-55, 101-102）

## Core Idea
異なるクロックで launch と capture が行われるパスは、両クロックを正確に定義して初めて正しく検査できる。SmartTime は 2 クロックの **周期の最小公倍数（共通周期）** 上で全アクティブエッジの組を調べ、setup は最も厳しい launch-capture 関係、hold は次の launch で上書きされないことを検査する。slack は `Required − Arrival`、スキューは setup と hold で逆向きに悲観側を取る。

## Frameworks Introduced
- **Inter-clock domain 解析（7.1-7.4）**
  - 既定で有効（Options > General > Include inter-clock domains）。有効時、false path か asynchronous clock group が無い限り setup/hold を検査
  - 同じクロックから派生したドメイン（入力クロック → 生成クロック等）は、各ドメインに generated clock 制約を定義しないと計算されない
  - 表示: 受信側ドメイン CK1 を展開すると「CKn to CK1」のセットが自動で並ぶ
  - **特定のクロック対だけ外す手順**: Constraints Editor の False Path → From の Browse で Pin Type「Registers by clock names」→ `Clk2` でフィルタ → `[get_clocks {Clk2}]` が入る → To も同様に `Clk1` → OK → Recalculate All → clk2→clk1 セットが空になったことを確認
- **Slack 計算（9.6）**
  - `Arrival_Time(FF2:D) = d1 + d2`（d1: launch クロック網遅延、d2: データパス遅延）
  - `Required_Time(FF2:D) = T + d3 - d4`（T: 周期、d3: capture クロック網遅延、d4: FF2 の setup）
  - `Slack = Required_Time - Arrival_Time`。負なら setup 違反
- **Clock skew（9.8）**
  - skew = 受信側と送信側のクロック挿入遅延の差
  - setup 検査: 最小スキュー（送信側に最大挿入遅延、受信側に最短挿入遅延）
  - hold 検査: 最大スキュー（送信側に最短、受信側に最大）
- **最大クロック周波数（9.4）**: 各レジスタ間パスの最小周期 = 最大データパス遅延 − 最小スキュー + 受信 setup（極性も考慮）。ドメイン内の最大値の逆数が Fmax。それを決めるパスが critical path
- **SmartTime → Constraints Editor の cross-probe（9.9）**
  1. SmartTime と Constraints Editor（Constraint Manager > Timing > Edit with Constraints Editor）を両方開く
  2. Max/Min ビューでパスを右クリック → False Path / Max Delay / Min Delay / Multicycle を選択（Editor が閉じているとグレーアウト）
  3. From/To が自動入力される → OK → Editor で Save
  4. その制約ファイルが P&R と Verify Timing に使われているなら P&R を再実行
- **SmartTime → Chip Planner の cross-probe**: P&R 後、両方開いてマクロ/ポート/ネット・パスを右クリック → Show in Chip Planner / Show Path in Chip Planner。ポートなら配置座標、I/O Bank、I/O Standard、ピン割当が見られる。逆方向は不可

## Key Concepts
- **Timing path の 4 分類**: Register to Register / 入力→レジスタ（External Setup・External Hold）/ レジスタ→出力（Clock to Out）/ 入力→出力（Input to Output）
- **Delay model**: セル遅延はライブラリで事前特性化（入力スルー・容量負荷依存）。配線遅延は pre-layout で Wire Load Model、post-layout で RC tree
- **Launch edge / Capture edge**: データを送り出すエッジ / 取り込むエッジ
- **Recovery / Removal time**: 非同期信号（リセット解除等）がアクティブエッジの前 / 後に変化してはいけない時間
- **Potential clock**: STA がイネーブルかクロックか判別できない、レジスタのクロックピンにつながるピン/ポート。ゲーテッドクロックに多い
- **Explicit clock**: クロックピンから一意に辿れるクロック源（DLL/PLL 出力を含む）
- **Virtual clock**: ソースの無いクロック。FPGA 外のクロックが内部解析に影響する場合（I/O が外部クロック同期）に使う
- **Minimum period**: レジスタ間パスで、そのパスが最も critical なときのクロック周期。スキューと受信 setup を含む
- **Output 容量（7.5）**: 出力遅延は基板容量と I/O 規格に依存。Chip Planner の I/O Attribute Editor で負荷（既定 5 pF）を実際値に変えると SmartTime が自動で反映（例: Clock to Out 6.603 ns → 15 pF 設定後の値が変化）

## Anti-patterns
- **派生クロックに generated clock を定義しない**: inter-clock パスが計算されない
- **非同期クロック間を解析させたまま違反に悩む**: 本当に非同期なら clock group（asynchronous）か false path で外す。ただし外す前に同期化回路があるか確認（sta-04 CDC）
- **Constraints Editor を開かずに SmartTime で例外追加しようとする**: メニューがグレーアウト
- **例外を追加したのに P&R を再実行しない**: 同じ制約ファイルを P&R でも使っていれば P&R は無効化され、再実行が必要
- **出力負荷を既定 5 pF のままにする**: 基板の容量と違えば Clock to Out がずれる

## Key Takeaways
1. inter-clock 解析は共通周期（最小公倍数）上で最も厳しい launch-capture 関係を検査する
2. Slack = Required − Arrival。Required = T + capture クロック遅延 − setup
3. スキューは setup で最小、hold で最大を取る（どちらも悲観側）
4. 特定のクロック対は false path（`-from [get_clocks A] -to [get_clocks B]`）で外す
5. Path List から右クリックで例外を追加でき、From/To は自動入力される

## Connects To
- **sta-01**: Domain Browser とパスセット
- **sta-04**: CDC レポートで inter-clock の安全性を判定
- **tce-03**: create_generated_clock / set_clock_groups
- **tce-05**: set_false_path の構文
