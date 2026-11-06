# SmartTime STA UG: チュートリアル（クロック制約と false path）（PDF p.56-79）

## Core Idea
2 本のチュートリアルで「制約を書く → P&R → SmartTime で確認 → 制約を変える → 再確認」の一巡を示す。題材は SmartFusion2 だが、Constraint Manager・Constraints Editor・SmartTime の操作は PolarFire でも同じ流れ（原文外の判断）。要点は **制約ファイルを P&R と Timing Verification の両方に関連付ける** ことと、**制約を変えると P&R が無効化される** こと。

## Frameworks Introduced
- **Tutorial 1: 32-bit シフトレジスタ（クロックイネーブル付き）（10.1）**
  1. 新規プロジェクト（Verilog）、HDL を作成して Synthesize
  2. Manage Constraints → Timing タブ → Edit with Constraints Editor > Edit Place and Route Constraints
  3. Requirement の Clock を右クリック → Add Clock Constraint。Clock Source = CLK、名前 my_clk、250 MHz
  4. File > Save → `user.sdc` が作られ **Target** に指定される（Editor 起動時に SDC が無ければ自動作成）
  5. Constraint Manager で Place and Route と Timing Verification の両チェックを入れる
  6. Place and Route > Configure Options で **Timing-driven** を有効にして実行
  7. Open SmartTime > Open Interactively → Max Delay Analysis: Summary に Fmax、External setup/hold、clock-to-out（例: CLK の Fmax 609.75 MHz）
  8. my_clk を展開 → Register to Register / External Setup / Clock to Output を順に確認。Expanded Path で arrival/required と回路図
  9. Tools > Minimum Delay Analysis で hold を確認（Register to Register, External Hold, Clock to Output, Register to Asynchronous, External Removal, Asynchronous to Register）
  10. 制約を 800 MHz に変えて再解析 → slack が減り、負なら赤表示
- **Tutorial 2: False path（10.2）**
  1. `false_path.v` を取り込み、100 MHz のクロック制約を作り、P&R（timing-driven）
  2. Max Delay Analysis で Fmax 442.48 MHz を確認
  3. Register to Register のパスを展開して回路を読む: D0_reg と D0_inv_reg は動作モードで常に反転関係 → XOR2 は常に 1 → MUX2 の select は常に 1。リセット時の遅延に関心がなければ D0_reg/D0_inv_reg → Q_reg:D は false path
  4. Constraints Editor（Edit Timing Verification Constraints）を起動したまま SmartTime でパスを右クリック → Add False Path Constraint（ダイアログが背後に隠れることがある）
  5. MUX の "0" 側（D1_reg から AND2 経由）も false path に
  6. 保存 → P&R が無効化 → Open SmartTime の **Update and Open Interactively** で P&R が自動再実行
  7. Fmax が 586.17 MHz に向上。Register to Register には D2_reg:CLK → Q_reg:D の 1 本だけが残る（実際に信号が伝わる唯一のパス）

## Key Concepts
- **Target file**: Constraints Editor で新しく追加した制約の保存先 SDC。未指定時は `user.sdc` が自動作成され Target になる
- **Constraint Manager の関連付け**: 各 SDC を Synthesis / Place and Route / Timing Verification のどれに使うかをチェックボックスで指定
- **Edit Place and Route Constraints / Edit Timing Verification Constraints**: 同じ Editor を、どのツール用の制約セットとして開くかの違い
- **Timing-driven layout**: P&R オプション。SmartTime をバックグラウンドで使い制約を満たすよう配置配線
- **Update and Open Interactively**: 無効化された上流ステップ（P&R）を再実行してから SmartTime を開く

## Code Examples
```verilog
// Tutorial 1 の HDL（原文 p.58、整形のみ）
module shift32 (Q, CLK, D, EN, RESET);
  input D, EN, CLK, RESET;
  output [31:0] Q;
  reg [31:0] Q_int;
  assign Q = Q_int;
  always @(posedge CLK) begin
    if (RESET) Q_int <= 0;
    else if (EN) Q_int <= {Q_int[30:0], D};
  end
endmodule
```
- `false_path.v`（p.78-79）は、反転関係の 2 FF の XOR を MUX の select にした回路。`syn_keep=1` で中間ネットを残している

## Anti-patterns
- **SDC を作っただけで P&R / Timing Verification に関連付けない**: 制約が使われない
- **P&R を timing-driven にしない**: 制約に向けた最適化がされない
- **P&R と Verify Timing で同じ SDC を使っていることを忘れて例外を追加**: P&R が無効化される。解析専用にしたいなら別ファイルにすれば P&R 再実行は不要（原文 p.77）
- **論理的に起こり得ないと検証せずに false path にする**: チュートリアルは HDL を読んで「動作モードでは select が常に 1」と確認してから設定している

## Key Takeaways
1. 制約は Constraint Manager で P&R と Timing Verification に関連付ける。新規制約は Target file（既定 `user.sdc`）に入る
2. P&R は timing-driven を有効にして回す
3. SmartTime から右クリックで例外を追加できる（Constraints Editor 起動が前提）
4. 共有 SDC を変更すると P&R が無効化され、Update and Open Interactively で再実行される
5. false path は回路の論理的根拠を確認してから設定する（例では Fmax 442 → 586 MHz）

## Connects To
- **sta-01**: Analysis View の操作
- **sta-02**: cross-probe による例外追加
- **tce-01 / tce-02**: Create Clock / Set False Path ダイアログの詳細
