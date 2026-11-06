# SmartDesign UG: DRC・Generate・テストベンチ・メモリ/割込みマップ（PDF p.41-61）

## Core Idea
結線が終わったら DRC → Generate Component で HDL を作る。Generate は DRC を自動で走らせ、エラーが 1 つでもあると失敗する。Generate すると、そのコンポーネントを使う設計の合成結果は無効になる。BIF 接続からはメモリマップ（Initiator→Bus→Bridge→Target）と、PolarFire SoC MSS の割込みマップ（PLIC 番号付き）を出力できる。

## Frameworks Introduced
- **生成フロー（p.41, 45）**
  1. ツールバー Design Rule Check（または Generate Component で DRC も自動実行）
  2. Log / Message のエラーを潰す（下表）
  3. Generate Component → HDL 生成。言語は Project > Project Settings > Design flow で選ぶ
- **Recursive / Non-recursive 生成（p.45）**: Project > Preferences > Design Flow > SmartDesign generation options
  | モード | 挙動 | 既定 |
  |---|---|---|
  | Non-recursive | アクティブな SmartDesign だけ生成。サブが未生成でも成功扱い（Design Hierarchy に青丸 ! が付く） | **既定** |
  | Recursive | 未生成の SmartDesign / IP をボトムアップで生成。子が全部成功したときだけ親を生成 | |
- **テストベンチ（p.46-47）**
  - SmartDesign Testbench: File > New > SmartDesign Testbench。既定で root design を instantiate し active stimulus に設定。`component\work\` に保存
  - HDL Testbench: File > New > HDL Testbench。Clock Period 既定 100 ns。`hdl\` に保存
  - Instantiate Root Design を外すと Set As Active Stimulus も自動で外れる
- **Memory Map（p.48-50）**: ツールバー View Memory Map
  - 経路: `Initiator > Bus > Bridge > Bus > Targets` または `Initiator > Bus > Targets`。下位 SmartDesign 内の要素も含む
  - 各 Target に Offset / Range / High Address / DRC。Initiator のアドレス空間から一部でも届かない Target は DRC フラグ
  - Memory Map DRC のエラーは Generate 失敗、警告なら通過。Project Settings の SmartDesign ページでエラー→警告に格下げ可能（届かない Target はそのまま届かない）
  - Show Targets / Search / Refresh / Expand / Collapse / Zoom and Center / DRC Filters
- **レポート出力**
  - File > Export > Memory Map Report → `.json` または `.html`。Initiator/Target BIF と Start Address・Range（p.51-52）
  - File > Export > Export Interrupt Map Report → `.json`。MSS / MIV 割込みピンとドライバの階層パス、**PLIC 番号**（PFSOC_MSS と MIV プロセッサ対応, p.53-54）

## Reference Tables
**DRC エラーと対処（Table 9-1, p.41-42）**
| DRC | 対処 |
|---|---|
| Unused Instance | 削除するか出力を 1 本以上つなぐ |
| Out-of-date Instance | Update Instance（下位のポート/パラメータ変更を反映） |
| Undriven Pin | ドライバ接続、または Tie High/Low/Constant |
| Floating Driver | 使わないなら Mark Unused |
| Unconnected / Required Bus Interface | 必須 BIF をつなぐ（シリコン接続規則） |
| Exceeded Allowable Instances | 個数制限のあるコアを減らす |
| Incompatible Family Configuration | プロジェクトのファミリに合わせて再インスタンス化 |
| No RTL / Obfuscated / Evaluation License | ライセンス取得 |
| No Top level ports | ポートを作る |
| Self-Instantiation | 自分自身を入れられない |
| Bus interface data width mismatch | Initiator の write 幅 ≤ Target の write 幅、Target の read 幅 ≤ Initiator の read 幅に再設定 |
| Bus interface ID width mismatch | Initiator の AWID/WID/ARID ≤ Target、Target の RID/BID ≤ Initiator |

**SmartDesign で設定できる合成属性（Table 10-1, p.43）**: `syn_insert_buffer`, `syn_keep`, `syn_maxfan`, `syn_no_compile_point`, `syn_noclockbuf`, `syn_noprune`, `syn_preserve`, `syn_hier`（Synplify Pro。BIF ピン/BIF ネット/グループ/スライスには不可、ユーザ定義属性も不可）。ツールバー Manage Synthesis Attributes（Ctrl+T）

**メモリマップ対応の主なコア（Table 17-1, p.56-57 抜粋）**
| 分類 | コア |
|---|---|
| Initiator | PolarFire SoC MSS（AXI4）, MIV_RV32 系, CORERISCV_AXI4, COREAXI4DMACONTROLLER, COREABC, PF_PCIE |
| Bus | COREAXI4INTERCONNECT, CoreAHBLite, CoreAPB3（v2.5.100 以上）, CoreAXI |
| Bridge | COREAXITOAHBL（v3.5.100 以上）, COREAHBLTOAXI, COREAHBTOAPB3 |
| 未対応（将来） | PF_DRI, COREAHBL2AHBL_BRIDGE, COREAXITOAXICONNECT |

**キーボードショートカット（Table 18-1, p.58）**: Ctrl+H 階層化 / Ctrl+J Flatten / Ctrl+K・Ctrl+F Smart Search / Ctrl+T 合成属性 / Ctrl+Z・Ctrl+Y Undo・Redo / Ctrl+C・X・V / Ctrl+Shift+ドラッグでピン移動

## Anti-patterns
- **Non-recursive のままサブを直してトップだけ Generate**: サブが未生成でもトップは成功扱いになる。青丸 ! を確認するか Recursive にする
- **Memory Map DRC を警告に格下げして放置**: 該当 Target は Initiator から実際にアクセスできない
- **Generate 後に合成をやり直さない**: Generate で合成状態は無効化される
- **割込み番号を手計算する**: Interrupt Map Report に MSS_INT_F2M[n] ごとの PLIC 番号が出る（例 F2M[0]=118）

## Key Takeaways
1. Generate = DRC + HDL 生成。エラー 0 が条件
2. 既定は Non-recursive。階層を触ったら Recursive か下位から順に Generate
3. BIF 幅不一致は Initiator/Target どちらかのコアを再設定して直す
4. FIC 経由のファブリック周辺アドレスは View Memory Map / Memory Map Report で確認できる
5. MSS 割込みの PLIC 番号は Interrupt Map Report（.json）で取れる

## Connects To
- **sd-01 / sd-02**: Update Instance、Tie / Mark Unused
- **dbg-01**: Generate 後に合成〜配置配線〜書き込みを経て SmartDebug へ
- 姉妹 skill **polarfire-soc-mss**: FIC アドレス窓、PLIC ID
