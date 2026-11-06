# パターン: タイミング制約

(原文外) は資料外から補った部分。

## ソース同期 DDR 入力の制約（LVDS 受信）
**When to use**: 送信側デバイスがデータと一緒にクロックを転送してくる DDR 入力（LVDS の RX 等）。
**How**: 転送クロックの入力ポートに `create_clock`（外部クロック基準なら virtual clock）。データポートに立上り基準の `set_input_delay -max` / `-min` を書き、立下り基準の 2 本を `-clock_fall -add_delay` で追加する。`-add_delay` 無しだと立上り側が上書きされる（tce-04 p.123-125）。組み合わせ方そのものは原文に例が無い（原文外）。
**Trade-offs**: -min を省くと max と同値扱いになり hold 検査が非現実的。PLL で位相をずらして取り込む場合、`-phase` は波形シフトではなく遅延として扱われる点に注意（tce-03 p.69）。

## 転送クロック出力（ソース同期 DDR 出力）の制約
**When to use**: FPGA がクロックを出力ポートへ転送し、データと一緒に送る場合（LVDS TX 等）。
**How**: 出力ポートに `create_generated_clock -divide_by 1 -source <PLL出力ピン> [get_ports clk_out]`（原文 p.112 の clk_out 例と同形）。データ出力に、その生成クロック基準で `set_output_delay -max/-min`、立下り側を `-clock_fall -add_delay` で追加（DDR への当てはめは原文外）。
**Trade-offs**: `set_clock_to_output` を使うと STA は見るが P&R と Synplify は無視するため、配置配線は最適化されない。P&R に効かせるなら標準の `set_output_delay` を使う（tce-04 p.114）。

## 生成クロックの定義（PLL/CCC・分周）
**When to use**: CCC/PLL 出力、FF 分周クロック、同一ソースから派生した複数ドメインを解析したいとき。
**How**: `create_generated_clock -name <名前> -multiply_by/-divide_by <整数> -source <基準ピン> <定義ピン>`。分周比で表せない波形は `-edges {r f r}` + `-edge_shift`。CCC/IP が生成する SDC は Constraint Manager の Derive Constraints で取り込むのが推奨（tce-03, tce-04 p.115）。
**Trade-offs**: 派生ドメインに generated clock を定義しないと inter-clock パスが計算されない（sta-02 p.22）。`-divide_by` 等は整数に警告なしで切り捨てられる。

## 非同期クロック間の除外（clock groups / false path）
**When to use**: 周波数・位相関係の無いクロック間で、同期化回路を入れて CDC を設計済みのとき。
**How**: クロック単位でまとめて外すなら `set_clock_groups -asynchronous -group {A ...} -group {B ...}`（同一グループ内は同期として解析）。特定の向きだけなら `set_false_path -from [get_clocks Clk2] -to [get_clocks Clk1]`。データパスの遅延だけを縛りたいなら `set_max_delay -ignore_clock_latency`（tce-03 p.110-112, tce-05, sta-02 p.24）。
**Trade-offs**: 外すと STA は一切検査しない。CDC Report で Synchronized / Timed Unsafe を先に確認し、User Ignored に意図外のペアが無いかを見る（sta-04）。

## 多サイクルパスの setup/hold 指定
**When to use**: クロックイネーブル等で、設計上 N サイクルに 1 回しかデータを取り込まないパス。
**How**: `set_multicycle_path N -setup -from ... -to ...` と、hold を元の位置に戻す `set_multicycle_path N-1 -hold ...` を組で書く（原文 p.131 の 4 / 2 の例。N-1 という一般化は原文外）。setup だけでよければ `-setup_only`。
**Trade-offs**: `-setup`/`-hold` 無指定では hold は既定 0。false path や max delay が同じパスにあると multicycle は無視される（優先順位が最下位）。

## 制約の抜け漏れ確認フロー
**When to use**: 制約を書いた後、違反ゼロが「本当に検査された結果か」を確かめたいとき。
**How**: `check_constraints`（構文とオブジェクト照合、`*_sdc.log`）→ Constraints Coverage Report で Untested を確認 → Timing Violations を max と min の両方で生成 → CDC Report（tce-06, sta-03, sta-04）。
**Trade-offs**: SmartTime ビューと CDC は選択中の 1 コーナーのみ。Timing Report Explorer は全コーナーの最悪なので値がずれることがある。
