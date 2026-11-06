# I/O Editor User Guide: 起動・Port View・Pin View・Package View（PDF p.1-17）

## Core Idea
I/O Editor は全 I/O マクロと属性を表形式で表示・編集し、変更を PDC に書き出す GUI。**post-synthesis 状態でないと開けない**。Port View（ポート名順）・Pin View（ピン番号順）・Package View（パッケージ図）が基本で、列名は set_io の属性に 1 対 1 で対応する。

## Frameworks Introduced
- **起動経路（Constraint Manager から）**
  - 編集して保存: Design Flow > Manage Constraints > Open Manage Constraints View > Constraint Manager > I/O Attributes > Edit > **Edit with I/O Editor**（PDC へ commit 可）
  - 閲覧のみ: 同 > I/O Attributes > **View**（post-P&R の配置と属性を read-only で表示。保存不可）
  - どちらでも File > Export Physical Constraint (PDC) で書き出しは可能
- **どのビューを使うか**: ポート単位で属性を揃える → Port View / ピンの空き・P/N・専用ピンを見る → Pin View / 物理位置を見る → Package View / DDR・XCVR・IOD → 専用ビュー（ioed-02）

## Key Concepts
- **I/O Standard 列**: プルダウンはそのバンクが対応できる規格だけに絞られる
- **Bank Name**: 読み取り専用。北側のバンクは HSIO のみ。同一バンクの I/O は VDDI と VREF を共有
- **Function（Pin View）**: PPAT（Public Pin Assignment Table）と同じ機能名。GPIO/HSIO 種別、専用機能（XCVR 等）、バンク番号、**差動の Positive/Negative パッド**、VSS を含む
- **User Reserved（Pin View）**: 別デザイン用にピンを予約（割り当て不可になる）
- **Dedicated**: UJTAG・電源・XCVR REFCLK・リセット・クロック等の専用ピン
- **Vref（Pin View）**: 任意の GPIO/HSIO パッドをバンクの外部 VREF にできる。そのパッドのバッファ・終端は無効化
- **User I/O Lock Down**: Tamper マクロの IO_Disable で I/O を無効化（入力はロジック Low、出力はトライステート。プルは維持）。PolarFire/PolarFire SoC のみ
- **Use I/O Calibration from the Lane**: I/O を再キャリブレーション対象から外す（既定 OFF。ES/XT 非対応。古いデザインは再 Compile で列が出る）

## Reference Tables
**属性列と PDC 引数の対応（Port View 2.x / Pin View 3.x）**
| I/O Editor 列 | set_io 引数 | 要点 |
|---|---|---|
| I/O Standard | -io_std | バンク対応規格のみ表示 |
| Pin Number / Locked | -pin_name / -fixed | Locked=レイアウトで動かさない |
| Clamp Diode | -CLAMP_DIODE | GPIO は ON/OFF 設定可、HSIO は常に ON。受信側信号レベルが VDDIx を超える（ホットソケット）なら OFF |
| Resistor Pull | -RES_PULL | None/Up/Down/Hold。既定は Port View 節で Up、Pin View 節で None と記述が食い違う |
| Schmitt Trigger | -SCHMITT_TRIGGER | 既定 OFF |
| Vcm Input Range | -VCM_RANGE | MID/LOW、既定 MID（PolarFire/SoC のみ） |
| On-Die Termination / ODT Static | -ODT | on=チップ内終端 / off=基板上終端 |
| ODT Dynamic | （-ODT DYNAMIC） | 原文では「RTG4 量産品のみ」と注記 |
| ODT Value | -ODT_VALUE | ODT on 時に選択。IOSTD で変わる |
| ODT Imp (Ohm) | — | Port Configuration ビット。値域はデータシート参照 |
| Low Power Exit | — | 単端のみ Off/Wake On Change/Wake On 0/Wake On 1。差動は Off 固定 |
| Input Delay / Output Delay | -IN_DELAY / -OUT_DELAY | 既定 OFF。DDR など動的遅延 I/O では非表示 |
| Slew | -SLEW | 非差動出力のみ。PolarFire HSIO は slew 制御なし（PVT 補償内蔵） |
| Pre-Emphasis | — | NONE（既定）/MIN/MEDIUM/MAX |
| Output Drive | -OUT_DRIVE | 1–20 mA、規格依存 |
| Impedance | -IMPEDANCE | 22–240 Ω、規格依存 |
| Output Load | -OUT_LOAD | pF。SmartTime・タイミング駆動レイアウトが使う |
| Source Termination | -SOURCE_TERM | 差動出力の近端終端、既定 OFF |

**ODT Static × ODT Dynamic（Table 2-2）**
| 組合せ | 動作 |
|---|---|
| Static=On, Dynamic=On | 不正 |
| Static=On, Dynamic=Off | 常時 ON |
| Static=Off, Dynamic=Off | 常時 OFF |
| Static=Off, Dynamic=On | バンクの ODT Dynamic 設定に従う |
対象規格: LVDS, RSDS, MINILVDS, LVPECL, HSTLI/II, SSTL15I/II, SSTL18I/II, HSTL18I/II, LPDDRI/II

**Flash*Freeze 関連列**（I/O State in Flash*Freeze Mode / I/O Available in Flash*Freeze Mode）は SmartFusion 2 / IGLOO 2 専用。PolarFire では無関係。

**Package View 色**: 通常ピン=緑、特殊ピン=青、予約ピン=赤、未接続=灰（Display Options で変更可）

## Anti-patterns
- **pre-synthesis で I/O Editor を開く**: 警告が出る。先に Synthesize
- **View で開いて編集して保存しようとする**: read-only。Edit with I/O Editor で開く
- **差動ペアの P/N で異なる RES_PULL を期待する**: 既知の制限で両側同じ値しか持てない。表示が Up/Up でも N 側は Down としてプログラムされる（逆も同様）
- **Vref に設定したパッドを信号に使う**: バッファが無効化される

## Key Takeaways
1. 編集は「Edit with I/O Editor」、確認は「View」（post-P&R、read-only）
2. 列は set_io の引数とほぼ 1 対 1。迷ったら pdc-03 の値表を見る
3. Pin View の Function 列で差動 P/N パッドとバンク番号を確認できる
4. 差動入力の RES_PULL は P/N 共通値。N 側は自動で逆極性
5. Resistor Pull の既定値は原文内で記述が食い違う（PDC 側は差動入力既定 Up）

## Connects To
- **pdc-02-set-io-iostd / pdc-03-set-io-attributes**: 列の値の根拠
- **ioed-02-interface-views**: Memory / XCVR / IOD View
- **ioed-03-floorplanner-export-mss**: PDC 書き出し、MSS I/O 属性
