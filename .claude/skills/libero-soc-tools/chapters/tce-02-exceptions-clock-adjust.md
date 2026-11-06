# Timing Constraints Editor UG: Timing Exceptions / Adjusting Clock Information / Advanced Constraints（PDF p.24-61）

## Core Idea
タイミング例外は既定の「1 サイクル」関係を上書きする。同じパスに複数の例外が掛かったら **優先順位の高いものだけが効き、他は無視** される。クロック側の補正（jitter・uncertainty・latency・clock groups）は、PolarFire/PolarFire SoC では **システムジッタが v2022.1 から自動計算** されるので、ユーザーは外部入力クロックのジッタだけを入れれば基本は足りる。

## Frameworks Introduced
- **例外の優先順位（Table 3-1/3-3/3-8）**
  1. `set_disable_timing`
  2. `set_false_path`
  3. `set_max_delay` / `set_min_delay`
  4. `set_multicycle_path`
  - false path は multicycle と max/min delay を常に上書き。max/min delay は multicycle を上書き
  - 同じ優先度の制約が複数掛かる場合は最も厳しいものを使う（STA UG p.13）
- **例外の使い分け（3 章冒頭）**
  - 設計上 2 サイクル以上かかるパス → multicycle
  - 解析・最適化から外すべきパス → false path
  - 特定パスの要求を緩める/締める → max/min delay
- **クロックジッタの扱い（4 章）**
  - ジッタ源: Global（system）jitter、PLL/DLL jitter、External jitter
  - PolarFire/PolarFire SoC（v2022.1 以降）: システムジッタ自動計算。ユーザーは FPGA 一次入力のクロックジッタを `set_input_jitter` で指定するだけ
  - 推定ジッタにマージンを足すなら `set_clock_uncertainty`
  - 自動計算値を上書きするのは稀なケースだけ（`set_system_jitter`）。推定はレジスタ数と RAM ブロック数に基づく
  - P&R と Verify Timing 完了時にジッタ推定レポートが出る。Expanded path にはクロックジッタが 1 行として現れる
  - 外部クロック源から一次入力までの遅延は `set_clock_latency` で与える

## Key Concepts
- **Max Delay / Min Delay**: 指定パスの要求遅延(ns)。起点が順序素子ならクロックスキューを含め、起点に input delay があれば加算、終点が順序素子ならスキュー + setup、終点に output delay があれば加算
- **Ignore clock latency**: スキューとジッタを除きデータパスだけで slack を計算。非同期クロック間のレジスタパス解析向け。Synplify Pro は非対応（Libero フローではスキップ）
- **有効な起点/終点**: 起点 = クロック、一次入力、inout ポート、順序セルのクロックピン。終点 = クロック、一次出力、inout ポート、順序セルのデータピン。Through = ポート/ピン/ネット/インスタンス
- **Multicycle**: Setup Path Multiplier（2 以上）と Hold Path Multiplier。「Setup Check Only」か「Setup and Hold Checks」を選ぶ
- **False path**: どの入力ベクトルでも感応しないパス。setup と hold の両方の検査を外す
- **Clock Uncertainty**: Simple（単一クロック）と Clock-to-Clock（from/to クロック間、edge 指定可）。setup / hold / 両方を選べる
- **Clock Source Latency**: 外部クロック源からクロック定義ピンまでの遅延。Early/Late × Rise/Fall の 4 値。値 1 つなら両エッジ同値。負値も可だが楽観的解析になりうる
- **Clock Groups**: Logically Exclusive（同時に存在しうるが論理的に排他: MUX クロック）/ Physically Exclusive（同時に存在しない: 同一ピンに複数定義）/ Asynchronous（周波数・位相関係なし → 解析から除外）
- **Disable Timing**: インスタンス内のタイミングアークを無効化。全アーク除外か From/To ポート指定。P&R と Verify Timing 用で、合成では無視
- **External Delay**: PLL 出力が From（出力ポート）で外に出て To（入力ポート）から PLL フィードバックに戻る外部遅延。Min/Max/Both。Synplify Pro 非対応

## Reference Tables
**GUI ダイアログと SDC コマンドの対応**
| ダイアログ | SDC | 章 |
|---|---|---|
| Set Maximum / Minimum Delay | `set_max_delay` / `set_min_delay` | 3.1-3.2 |
| Set Multicycle Constraint | `set_multicycle_path` | 3.3-3.4 |
| Set False Path Constraint | `set_false_path` | 3.5-3.6 |
| Set Clock Uncertainty（Simple / Clock To Clock） | `set_clock_uncertainty` | 4.2, 5.3 |
| Set Clock Source Latency | `set_clock_latency -source` | 4.3, 5.2 |
| Add Clock Groups | `set_clock_groups` | 4.4, 5.4 |
| Set Constraint to Disable Timing Arcs | `set_disable_timing` | 5.1 |
| Add Set External Delay | `set_external_delay` | 5.5 |
| （GUI なし: SDC のみ） | `set_input_jitter` / `set_system_jitter` | 4.1, 4.5 |

**ピン選択ダイアログの Type（共通）**
| 欄 | 選べる Type |
|---|---|
| From | Clock Pins / Input Ports / All Register Clock Pins |
| Through | All Ports / All Pins / All Nets / All Instances |
| To | Clock Pins / Output Ports / All Register Data Pins |

- Through/To の Browse は、先に選んだ From から到達可能なピンだけを表示する

**Families Supported（Table 3-5/3-7）**: multicycle と false path は PolarFire / RTG4 / IGLOO2 / SmartFusion2 で SDC・Constraints Editor の両方に対応。

## Anti-patterns
- **multicycle を掛けたパスに false path も掛けて「multicycle が効かない」と悩む**: false path が常に勝つ
- **PolarFire で手計算のシステムジッタを uncertainty に積む**: 自動計算と二重計上になる。外部クロックのジッタは `set_input_jitter`、追加マージンだけを uncertainty で
- **`set_system_jitter` を常用する**: 原文は「稀なケースまたは UG の指示があるときのみ」
- **Ignore clock latency / External Delay を Synplify Pro 側でも効くと思う**: 合成では無視される
- **Clock source latency に負値を入れる**: 楽観的解析になりうる

## Key Takeaways
1. 優先順位は disable_timing > false_path > max/min_delay > multicycle
2. PolarFire はシステムジッタ自動計算。ユーザーは入力クロックジッタ（`set_input_jitter`）だけ入れる
3. 非同期クロック同士は Clock Groups の Asynchronous で一括除外できる
4. Max/Min delay は input/output delay とスキューを含めて評価される。データパスだけで見たいときは Ignore clock latency
5. Disable Timing と External Delay は P&R / STA 用で、合成ツールは無視する

## Connects To
- **tce-05**: set_false_path / set_max_delay / set_multicycle_path の構文
- **tce-03**: set_clock_groups / set_clock_uncertainty / set_clock_latency / jitter の構文
- **sta-01 / sta-02**: SmartTime 側の優先順位（sta-01）と、false path で inter-clock domain を無効化する手順（sta-02）
- **sta-04**: CDC レポートで例外の掛かり具合を確認
