# Timing Constraints Editor UG: エディタ概要と Required Constraints（PDF p.5-23）

## Core Idea
Timing Constraints Editor は SDC タイミング制約を GUI で作成・編集するツール。制約は **Requirements（クロック・I/O 遅延）→ Exceptions（false path・multicycle・max/min delay）→ Advanced（latency・clock groups 等）** の 3 分類で管理する。最初に置くべきは「クロック」と「I/O 遅延」で、これが無いと STA は何も検査できない。

## Frameworks Introduced
- **画面構成（1.1）**
  - Constraint Browser: 種類別ツリー（Requirements / Exceptions / Advanced）
  - Constraint List: 制約のスプレッドシート。セルを直接編集できる
  - Constraint Adder: List の先頭行。右クリック → Add Constraint でダイアログ、またはダブルクリックで直接入力（上級者向け）
- **制約の追加 4 経路（1.2）**: ツールバーのアイコン / Browser で種類をクリック / Constraints メニュー / 先頭行を右クリック
- **保存**: 作成後 File > Save で DB に保存。保存しないと反映されない
- **Required Constraints の手順（2.1.1 例: Clock）**
  1. Add Clock Constraint を開く
  2. Clock Source を選ぶ（Browse で Pin Type: Explicit clocks / Potential clocks / All Ports / All Pins / All Nets / Pins on clock network / Nets in clock network）
  3. Period(ns) か Frequency(MHz) を入力（片方を入れると他方は自動計算）
  4. 名前・Duty cycle・Offset・初期エッジを必要なら変更
  5. OK → File > Save

## Key Concepts
- **Clock 制約**: 順序回路のタイミング検証の基本。周期・波形・ソースを定義
- **Virtual clock**: ソースピンを指定しないクロック。FPGA 外部のクロックで I/O が同期している場合に使う
- **Generated clock**: 内部で生成されるクロック（PLL/CCC 出力、分周レジスタ出力）を基準クロックから乗除算で定義
- **Input delay**: 入力ポートに到着するデータの、クロックに対する到着時刻（FPGA 外のタイミング予算）
- **Output delay**: 出力ポートからキャプチャエッジまでに必要な外部遅延（FPGA 外のタイミング予算）
- **External Check**: FPGA 内部側の入力 setup/hold 予算を直接指定（set_external_check）
- **Clock to Out**: FPGA 内部側の出力 clock-to-out 予算を直接指定（set_clock_to_output）
- **Add ... to existing one with same source**: 同じソース/ポートに 2 つ目の制約を追加（名前が違う場合のみ有効）

## Reference Tables
**Create Clock Constraint（Table 2-1）**
| Option | 内容 |
|---|---|
| Clock Name | 制約名（既定は最初のソース名） |
| Clock Source | Input Ports / All Pins / All Nets から選択。Pattern 既定 `*` |
| Period / Frequency | ns または MHz。小数 3 桁まで |
| Starting Clock Edge | 最初のエッジ（立上り/立下り） |
| Offset | 時刻 0 からの最初のエッジのずれ(ns)。正の実数、既定 0 |
| Duty cycle | High の割合(%)。既定 50 |
| Add this clock to existing one | 同じソースに別名クロックを追加 |

- 複数ソースピンを 1 クロックに指定できる（1 つのクロックが遅延の異なる複数入力から入る場合）

**Create Generated Clock Constraint（Table 2-2）**
| Option | 内容 |
|---|---|
| Clock Pin | 生成クロックの定義点（Output Ports / All Register Output Pins / All Pins / All Nets / Input Ports） |
| Reference Pin | 基準クロックのピン（Input Ports / All Pins） |
| Generated Frequency | multiply / divide 係数（正の整数） |
| Generated Clock Edges | 基準クロックのエッジ番号 3 つ（立上り, 立下り, 次の立上り） |
| Edge Shift | 各エッジのシフト量 3 つ（正 = 遅らせる、負 = 早める） |
| Generated Waveform | 基準と同相 / 反転 |
| Phase | CCC 構成から自動生成時の情報。有効値 0/45/90/135/180/225/270/315 |
| PLL Output / PLL Feedback | 外部フィードバック構成の CCC 用 |
| Master Clock | 複数クロックがマスタピンに入る場合に選択（Add と併用） |

- エッジ指定例: 100 MHz から 200 MHz を作るなら edge `{1 2 3}`、edge shift `{0 -2.5 -5.0}`。分周 2 なら `[1 3 5]`、分周 4 なら `[1 5 9]`

**Add Input Delay / Output Delay（Table 2-3, 2-4）**
| Option | Input Delay | Output Delay |
|---|---|---|
| Port | 入力ポート（複数可） | 出力ポート（複数可） |
| Clock Name / Edge | 基準クロックと launch エッジ（rise/fall） | 同左 |
| Max Rise/Fall | 入力に到着する最長パスの遅延 | 外部レジスタまでの組合せ遅延 + setup |
| Min Rise/Fall | 最短パスの遅延 | 外部レジスタまでの組合せ遅延 + hold |
| Falling same as Rising / Min same as Max | 値を共通化 | 同左 |
| Add ... to existing one | 別クロック・別エッジの制約を同じポートに追加（= `-add_delay`） | 同左 |

**External Check / Clock to Out（Table 2-5, 2-6）**: Port、Clock Name、Setup/Hold（External Check）または Maximum/Minimum Delay（Clock to Out）、Comment のみ。

## Anti-patterns
- **Virtual clock にソースピンを指定する**: virtual clock はソース無しで定義する
- **同じ名前で "Add to existing" する**: 名前が同じだと既存制約を上書きする（チェックしても上書き）
- **作成後 File > Save を忘れる**: DB に保存されない
- **Input/Output delay を max だけ入れる**: 「Min same as Max」がオフで min を空にすると hold 側の検査が意図どおりにならない。min/max を両方意識する（原文外）

## Key Takeaways
1. 制約は Requirements → Exceptions → Advanced の順に考える
2. 外部クロック基準の I/O は virtual clock（ソース無し create_clock）で表せる
3. 生成クロックは乗除算か、エッジ番号 + エッジシフトのどちらかで定義する
4. 同一ポートに複数クロック・複数エッジの I/O 遅延を載せるときは「Add to existing」（`-add_delay`）
5. External Check / Clock to Out は FPGA 内部予算を直接書く非標準の方式。標準 SDC は input/output delay

## Connects To
- **tce-03**: create_clock / create_generated_clock の SDC 構文
- **tce-04**: set_input_delay / set_output_delay / set_clock_to_output / set_external_check の構文
- **tce-02**: Exceptions と Advanced の GUI
- **sta-01**: 作った制約がどの path set（External Setup, Clock to Out 等）で検査されるか
