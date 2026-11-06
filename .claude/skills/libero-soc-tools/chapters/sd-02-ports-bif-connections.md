# SmartDesign UG: ポート・BIF・結線・階層化（PDF p.18-40）

## Core Idea
ピン/ポートには方向（input / output / inout）と型（Scalar / Bus / Slice / BIF）がある。BIF（Bus Interface）は AXI・APB などの信号束で、BIF 同士を 1 本つなげば全メンバ信号が接続される。PAD 属性のピンは必ずトップレベルポートへ自動昇格し、最終的にパッケージピンに割り当たる。

## Frameworks Introduced
- **結線の 2 方法（p.27）**
  - **ドラッグ&ドロップ**: ピン/ポートにホバーすると十字カーソルになり、相手までドラッグ。階層ビュー、Zoom to Selection、Add Note、Filtered 表示中は無効
  - **右クリック Connect**: Ctrl で複数ピンを選び右クリック → Connect（論理的に可能なときだけ有効）
- **トップへの引き出し**: 非 PAD ピンを右クリック **Promote to Top-Level** → 同名ポート＋ネットを作成（重複時は一意名）。BIF 内ピンは `<BIF名>_<ピン名>` になる
- **未使用ピンの処理（DRC を通す）**
  - 入力: Tie High / Tie Low / Tie Constant（バスのみ、HEX 入力）
  - 出力: Mark Unused（DRC の警告対象から外す）
  - 反転: Invert（バブル表示）。Clear Attributes で解除
- **Smart Search and Connect（p.31-34）**: 大規模設計でクロック・リセット等をまとめて結線する
  - 起動: Edit > Find、ツールバーのトグル、Ctrl+K / Ctrl+F
  - 検索構文 `[Type]:[Instance].[Pin]`、キーワード `port:` `pin:` `instance:` `net:` `*:`（先頭に書く）
  - Pin 留め（pinning）で複数回の検索結果を保持
  - Filter で対象だけのコーン表示（読み取り専用）。点線ネットをダブルクリックで周辺を追加。解除は Undo Filter / ウィンドウを閉じる / Ctrl+Z
- **階層化（p.35-40）**
  - **Create Hierarchical SmartDesign**（Ctrl+H）: 選択（最低 1 インスタンス）を子 SmartDesign に切り出す。接続は保持
  - **Flatten**（Ctrl+J）: 子の中身を上位に展開。内部ネット/インスタンス名は `<階層インスタンス名>_<元の名前>`
  - **Expand / Fold Instance**: その場で 1 階層下を読み取り専用で表示。下位の変更はリアルタイムに反映されない（畳んで再展開）

## Key Concepts
- **Scalar / Bus / Slice**: 1 bit / `name[3:0]` / バスの一部（`name[2:0]`）
- **BIF（Bus Interface）**: 機能的意味を持つ信号束。AXI, AXI4, AXI4Stream, AHB, AHBLite, APB3 と Microchip 独自型
- **Initiator BIF / Target BIF / System BIF**: トランザクション発行側 / 応答側 / どちらでもない特殊接続（p.55）
- **PAD 属性**: トップレベルポート必須。自動昇格し、PAD ネットは削除・コピー不可
- **Driver**: ネットの信号源。トップの input / target BIF、インスタンスの output / initiator BIF が該当
- **Pin Group**: 表示用のピンまとめ。接続不可。畳むと未接続や Tie 済みピンが隠れる
- **Pins to Expose**: BIF のメンバピンを個別に表示/非表示。BIF が接続済みなら入力ピンは出せない
- **Go to Driver / Magnify Pin**: ドライバへジャンプ（同一階層内のみ）/ ファンアウト一覧

## Reference Tables
**バス・スライス操作（p.22-26）**
| 操作 | 要点 |
|---|---|
| Create Slice | 32bit なら 32×1, 16×2, 8×4, 4×8, 2×16 から選ぶ。既存スライスは削除される |
| Custom Slices | 任意範囲を列挙。既存スライスは残す。範囲外・入力の重複は Log にエラー |
| Edit Slice | 範囲変更・追加・削除 |
| Flip Bit Order | スライスのみ。`[10:0]`→`[0:10]` |
| Modify/Rename | バス範囲を変えると全スライスが消える |

**階層 SmartDesign 生成時の命名（p.36）**
- 既定名 `hier_[index]`、未接続ピンは `[instance]_pin_[pin]` のポートになる
- PAD ポートは規則外で自動的にトップへ

**コピー/カット（Appendix E, p.61）**
- PAD ポート/ネットはコピー不可。Cut&Paste は Undo 不可、同一 SmartDesign 内では不可、別プロジェクト間も不可
- Tcl でのコピー/カットではバス名を範囲なしで指定（`bus_name[10:1]` は無効）

## Anti-patterns
- **PAD ピンを手で別ネットにつなごうとする**: PAD ネットは削除できない。PAD はトップポートに出すもの
- **未使用入力を浮かせたまま Generate**: Undriven Pin で DRC エラー。Tie High/Low/Constant する
- **バスの範囲を Modify で変える**: 既存スライスが全消去される。先に接続を確認
- **Expanded 表示で編集しようとする**: 展開中は読み取り専用。子 SmartDesign を開いて直す
- **Filtered 表示のまま結線しようとする**: 読み取り専用。Undo Filter で戻す

## Key Takeaways
1. BIF 同士を 1 本つなぐのが基本。メンバ信号を個別につなぐ必要はない
2. PAD ピンは自動でトップポートになる（LVDS 用 I/O マクロ/IOD の pad ピンもこの扱いになるはず: 原文外）
3. 未使用入力は Tie、未使用出力は Mark Unused で DRC を黙らせる
4. 大規模な結線は Smart Search and Connect（Ctrl+K）
5. 塊を整理するなら Ctrl+H で子 SmartDesign 化、戻すなら Ctrl+J

## Connects To
- **sd-01**: HDL+ で BIF を定義する
- **sd-03**: DRC エラーの種類（Undriven Pin、BIF 幅不一致など）
