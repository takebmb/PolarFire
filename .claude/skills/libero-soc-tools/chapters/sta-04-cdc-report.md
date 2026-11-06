# SmartTime STA UG: Clock Domain Crossing (CDC) Report（PDF p.37-46）

## Core Idea
CDC Report は、送信クロックドメインから受信クロックドメインへ渡る全パスを **クロック対ごと** に集計し、データ欠落やメタステーブルの恐れがある箇所を探す。表のセル 1 つが 1 つのクロック対で、状態（Timed / Timed Unsafe / Synchronized / User Ignored / No Paths）で色分けされる。**制約の無いクロックは表に現れない** ので、まずすべてのクロックを定義すること。

## Frameworks Introduced
- **生成手順**: Tools > Reports > Clock Domain Crossing (CDC) → 「CDC Table」（色付き表）か「Comma Separated Values」（CSV 出力）→ OK
- **読み方**
  1. 表（CDC table）でクロック対の状態を見る。Clock Selection で表示クロックを選択（既定は全表示）
  2. CDC カテゴリボタンで状態別にフィルタ
  3. 下の path table に各クロック対の最悪 slack パスが 1 行ずつ出る。セルをクリックするとその対だけ、再クリックで全体に戻る
  4. セルを右クリック → Copy / **Add a Clock Group Constraint** / **Go to Timing Explorer**
- **判断の流れ（原文外の整理）**
  - Timed（共通の一次クロックを持つ同期関係）→ 通常の STA で十分
  - Synchronized（全パスに同期化回路を検出）→ 非同期クロック間なら clock group / false path で外す候補
  - Timed Unsafe → 理由は合成後の `*_cdc.csv` に記載（例: CDC 上に組合せ論理）。回路を直すか、意図的なら例外を設計
  - User Ignored → false path か clock group で全パスが無効化済み。意図どおりか確認

## Key Concepts
- **Source Clock / Destination Clock**: 送信側 / 受信側クロック
- **Worst Slack (ns) / Worst Slack Edge**: その CDC の最悪パスの slack と、送受信エッジ
- **Max Delay Cstr / Multicycle Path Cstr**: Yes（全パスに適用）/ No（なし）/ Partial（一部に適用）
- **Clock Group Cstr**: Yes / No（set_clock_groups の有無）
- **Clock Uncertainty Cstr**: 最悪パスの uncertainty 値（無ければ空）
- **Num of Paths**: CDC のパス数（User Ignored は 0）
- **Clock Pair Status**: Timed Safe / Timed Unsafe / User Ignored
- **Corner Info**: どの operating condition の結果かを示す列

## Reference Tables
**CDC の状態（Table 8-7、色は原文の表組みが崩れており対応が判読しにくいため省略）**
| 状態 | 意味 |
|---|---|
| User-Ignored Paths | 全パスが false path または clock group で無効化 |
| No Paths | 送信 → 受信のパスが存在しない |
| Timed | 両クロックが共通の一次クロックを持つ同期関係で、パスあり |
| Timed Unsafe | 合成後 CDC レポート（`*_cdc.csv`）に理由あり（例: CDC 上の組合せ論理） |
| Synchronized | 送受信間の全パスで同期化回路を検出 |

**SmartTime と Timing Report Explorer の差（コーナーケース）**
| シナリオ | 現象 | 理由 |
|---|---|---|
| 1. 同一ポートに 2 クロック（`-add`） | SmartTime は対を CDC として出すが、Explorer は `-add` 側を destination として拾わない | Explorer は始点・終点の組ごとに最悪 slack の 1 本だけ拾う |
| 2. operating condition の違い | Explorer 97.958 ns、SmartTime 98.154 ns のように差が出る | Explorer は全コーナーの最悪、SmartTime/CDC は選択中の 1 コーナー（例: fast_hv_lt）。slow_lv_lt に合わせると一致 |

```tcl
# シナリオ 1 の制約（原文 p.43）
create_clock -name {reg_com_clk_10mhz} -period 100 -waveform {0 50 } [ get_ports
 { reg_com_clk_10mhz } ]
create_clock -name {reg_com_clk_5mhz} -period 200 -waveform {0 100 } -add [ get_ports
 { reg_com_clk_10mhz } ]
```

**注意事項（p.45）**
- 同一クロック（C1→C1）は有効な CDC ではないため path table に出ず、CDC 表では既定で timed safe 表示（Explorer への cross-probe は可能）
- パスの無い CDC は path table に出ない
- User Ignored の行では Worst Slack 等の列と Corner Info が空
- CSV 出力では CDC 種別が数値で表され、数値の説明も CSV に含まれる。Tcl の `report -type cdc` は CSV のみ

## Anti-patterns
- **クロックを定義せずに CDC Report を見る**: 制約の無いクロックは表に出ないので「CDC なし」に見える
- **Timed Unsafe を例外で黙らせる**: 原因（CDC 上の組合せ論理など）を `*_cdc.csv` で確認してから判断
- **SmartTime と Explorer の slack 差をバグと思う**: コーナーの違い（Corner Info 列で確認）
- **User Ignored を確認しない**: 広すぎる false path / clock group で本来検査すべき同期パスまで消していないか見る

## Key Takeaways
1. CDC Report はクロック対ごとの状態表 + 最悪パス表。表示されるのは制約のあるクロックだけ
2. 状態は Timed / Timed Unsafe / Synchronized / User Ignored / No Paths
3. Timed Unsafe の理由は合成後の `*_cdc.csv` にある
4. セル右クリックから clock group 追加や Timing Explorer への移動ができる
5. SmartTime/CDC は選択中の 1 コーナー、Timing Report Explorer は全コーナー最悪

## Connects To
- **sta-02**: inter-clock domain 解析の仕組み
- **tce-03**: set_clock_groups（Clock Group Cstr）
- **tce-05**: set_false_path / set_max_delay（Max Delay Cstr）
- **tce-06**: `report -type cdc`、`set_options -max_opcond/-min_opcond`
