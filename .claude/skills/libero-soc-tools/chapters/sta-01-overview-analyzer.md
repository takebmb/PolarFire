# SmartTime STA UG: 概要・設定・Timing Analyzer の使い方（PDF p.4-21）

## Core Idea
SmartTime は Libero の静的タイミング解析（STA）ツール。入力ベクタ不要で全パスを検査し、setup/hold 違反を報告する。P&R のタイミングドリブン配置配線も裏で SmartTime を使うので、**解析と P&R の制約は常に一致** する。弱点は false path を自動検出しないこと。ユーザーが制約で教える必要がある。

## Frameworks Introduced
- **SmartTime が動く場面（2 章）**
  - P&R 中: timing-driven P&R でバックグラウンド実行
  - P&R 後: Implement Design > Verify Post-Layout Implementation > **Verify Timing**（レポート生成）または **Open SmartTime > Open Interactively**
  - Back-Annotation 時: タイミングシミュレーション用 SDF 生成
  - 対話的に開くには P&R 完了が前提（未完了なら Libero が先に実行）
- **解析時の優先順位（6.2）**: 1. False path → 2. Max/Min delay → 3. Multi-cycle path → 4. Clock。同優先度が複数なら最も厳しいものを使う
- **Max Delay Analysis View の読み方（6.3）**
  1. Domain Browser でクロックドメインを選ぶ（緑チェック = 満足、x = 違反あり）
  2. Paths List は slack 昇順（最悪が先頭）。違反は負の slack で赤表示
  3. Path Details で arrival / required の計算内訳を確認
  4. ダブルクリックで Expanded Path View（回路図 + Path Profile: セル遅延とネット遅延の割合）
  - 最小パルス幅が最大周波数を制限している場合、Summary にアイコンが出て制限ピンを示す
- **フィルタ（6.8）**: From/To にパターンを入れ Apply Filter → Store Filter で名前付き保存。セットの 1 階層下に保存され、カスケードできる

## Key Concepts
- **Domain Browser / Path List / Path Details / Analysis View Filter / Path Slack Histogram**: Analyzer の 5 構成要素。ヒストグラムの棒を選ぶと Path List を絞れる
- **Clock domain**: パスはクロックドメインごとに整理。既定は explicit clock のドメインのみ表示。非 explicit クロックは Manage Clock Domains の New Clock で追加
- **Path set**: 各ドメインに最低 3 つ。Register to Register、External Setup（Max ビュー）/ External Hold（Min ビュー）、Clock to Out
- **Custom Path Set（User Sets）**: 任意の source/sink ピンで作るセット（Add Set）
- **Parallel paths**: 同じ始点・終点を持つ並列パス。既定は 1 本だけ展開
- **Max / Min Delay Analysis**: setup 検査 / hold 検査
- **Operating Conditions**: 解析コーナー。PolarFire は slow_lv_ht / slow_lv_lt / fast_hv_lt（tce-06）

## Reference Tables
**SmartTime Options（4 章 / 11.7）**
| カテゴリ | 設定 | 既定 |
|---|---|---|
| General | Operating conditions（max/min 解析のコーナー） | max = worst 側、min = best 側 |
| General | Include inter-clock domains in calculations | ON |
| General | Enable recovery and removal checks | OFF（ON で各ドメインにセット追加） |
| Analysis | パスセットあたりの最大パス数 | 100 |
| Analysis | Filter the paths by slack value | OFF |
| Analysis | Show clock network details in expanded path | - |
| Analysis | Expanded path の並列パス数 | 1 |
| Advanced | bibuf loopback / 非同期ピンでパスを切る / clockpath の non-unate アーク無効化 | - |
| Advanced | Scenarios（解析用と TDPR 用） | Primary |

**Path List の既定列（6.6）**
| セット | 既定列 |
|---|---|
| Register to Register | Source Pin, Sink Pin, Delay, Slack, Arrival, Required, Setup, Minimum Period, Skew |
| External Setup | Source Pin, Sink Pin, Delay, Slack, Arrival, Required, Setup, External Setup |
| Clock to Out | Source Pin, Sink Pin, Delay, Slack, Arrival, Required, Clock to Out |
| Input to Output / Custom | Source Pin, Sink Pin, Delay, Slack |

- 追加可能な列の例: Source/Destination Clock Edge、Logic Stage Count、Max Fanout、Clock Constraint、Input Delay Constraint、Output Delay Constraint、Maximum Delay Constraint、Multicycle Constraint（Customize table から）

**ツールバー（Table 5-1）**: Save / Undo / Redo / Max Delay Analysis View / Min Delay Analysis View / Manage Clock Domains / Path Set Manager / Recalculate All

## Anti-patterns
- **P&R 前の結果で判断する**: 対話解析は P&R 完了が前提。post-layout の数値で判断する
- **Max ビューだけ見る**: hold は Min Delay Analysis View（External Hold セット等）で確認
- **I/O パスが Path List に出ないのを「問題なし」と読む**: External Setup / Clock to Out セットは input/output delay 等の制約があって初めて意味を持つ（Constraints Coverage で Untested を確認: sta-03）
- **false path が自動で除外されると期待する**: STA は全パスを報告する

## Key Takeaways
1. 解析優先順位は False path > Max/Min delay > Multicycle > Clock
2. 各ドメインは Register to Register / External Setup(Hold) / Clock to Out の 3 セットが基本
3. Paths List は slack 昇順。負の slack は赤
4. Expanded Path View で arrival/required の内訳と回路図を確認
5. inter-clock domain 解析は既定 ON

## Connects To
- **sta-02**: inter-clock domain、slack・skew の計算式、cross-probe
- **sta-03**: レポート生成
- **tce-06**: `set_options` / `list_paths` / `expand_path`（同じ操作の Tcl 版）
