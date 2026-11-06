# SmartTime STA UG: タイミングレポート（PDF p.27-36, 89-98）

## Core Idea
SmartTime は 7 種のレポートを出せる。日常的に見るのは **Timer（Timing）Report = ドメイン別の全体像**、**Timing Violations Report = 違反だけを slack 順に並べたフラットな一覧**、**Constraints Coverage Report = 制約が掛かっていない（Untested）検査の洗い出し** の 3 つ。I/O 制約を書いたら Coverage で Untested が残っていないか確認するのが効く。

## Frameworks Introduced
- **生成手順（共通）**: Max/Min Delay Analysis View → Tools > Reports > {Timer / Timing Violations / Constraints Coverage / Bottleneck / Datasheet / Combinational Loops / Clock Domain Crossing (CDC)} → オプションを選んで OK → 別ウィンドウに表示
- **レポートを読む順（原文外の推奨）**: Coverage（Untested が無いか）→ Violations（max と min の両方）→ Timer の Summary（Fmax、External Setup/Hold、Clock to Out）→ 必要なら Bottleneck

## Reference Tables
**レポート種類（Table 8-1）**
| レポート | 内容 | Tcl `-type` |
|---|---|---|
| Timer Report | クロックドメイン別のタイミング情報 | `timing` |
| Timing Violations | 制約違反のフラット slack 一覧 | `violations` |
| Bottleneck | 多くの違反に寄与する箇所 | `bottleneck` |
| Datasheet | ピン・I/O 規格・外部タイミング特性（**PolarFire 非対応**、p.92） | `datasheet` |
| Constraints Coverage | 制約のカバレッジ | `constraints_coverage` |
| Combinational Loop | 初期化時に見つかったループと切断位置 | `combinational_loops` |
| Clock Domain Crossing | クロック間パスの安全性（sta-04） | `cdc` |

**Timer Report の構成（Table 8-2, 8.3.x）**
| セクション / セット | 内容 |
|---|---|
| Header | レポート種別、バージョン、日時、設計情報 |
| Summary | ドメインごとのタイミング情報（既定は explicit clock のドメイン） |
| Path | パス種別ごとの情報。既定 1 セット 5 本 |
| Register to Register | レジスタのクロックピン → データピン |
| External Setup | トップ入力ポート → レジスタ |
| Clock to Output | レジスタのクロックピン → トップ出力ポート |
| Register to Asynchronous | レジスタ → 非同期制御（set/reset） |
| External Recovery | 非同期制御の外部 recovery |
| Asynchronous to Register | 非同期制御 → レジスタ |
| Inter-clock Domain | 他ドメインのクロックピン → 当該ドメインのデータピン（既定で出力） |
| Pin to Pin / Input to Output | 入力 → 出力パスとユーザーセット |
| Expanded Paths | セットごとの展開パス（既定 1 本） |

- Max Delay Timing Report は「入力 I/O → 出力 I/O」「入力 I/O → 内部レジスタ」「内部レジスタ → 出力 I/O」「各クロック網」「クロック網間」の最大遅延を含む

**Timing Violations Report（Table 8-3, 11.12）**
- 全ドメイン混在、slack の小さい順。既定 slack 閾値 0、最大 100 本、展開 0 本、並列 1 本
- 既定は Max Delay Analysis。hold 違反は Analysis を Min にして別途出す

**Constraints Coverage Report（Table 8-4）**
| セクション | 内容 |
|---|---|
| Results by Clock Domain | Setup / Hold / Recovery / Removal / Output の各検査を Met（満足）/ Violated（違反）/ Untested（該当制約なし）に分類 |
| Coverage Summary | 全ドメイン合計（複数ドメインに属す検査は重複計上） |
| Enhancement Suggestions | カバレッジ向上のために追加すべき制約・変更すべきオプション |
| Detailed Stats | 制約が必要なクロック・I/O ポートをピン単位で提示 |

**Bottleneck Report（8.9, 11.9）**
- Cost type: Path count（通過する違反パス数、既定）/ Path cost（違反量の合計）
- 出力: Instance Name（インスタンスの出力ピン）と Path Count
- 既定: slack 閾値 0、1 セクション 100 パス、並列 1、インスタンス 10 個
- Sets: Entire design / Clock Domain（+Type）/ 既存ユーザーセット / Input to Output

**Timing Report Options（11.11）の既定**
| 項目 | 既定 |
|---|---|
| Format | Plain Text（CSV 可） |
| Summary | 含める |
| Analysis | Maximum Delay |
| Slack フィルタ | なし |
| User sets | 含めない（Input to Output セットは含める） |
| Clock domains | 現在の解析ドメイン |

## Anti-patterns
- **Violations が空 = 完了と判断する**: 制約が無い検査は違反にならない。Coverage の Untested を確認する
- **Max だけで Violations を出す**: 既定は Max。hold は Min で再生成
- **PolarFire で Datasheet Report を頼る**: 原文で PolarFire 非対応と明記（ただし `report -type datasheet` の例は残っている）
- **Summary を全体の合否と思う**: 既定は explicit clock のドメインのみ。非 explicit クロックは Manage Clock Domains で追加

## Key Takeaways
1. 合否判定は Violations（max と min）、抜け漏れ確認は Constraints Coverage
2. Coverage は Met / Violated / Untested の 3 分類。Untested は「制約が無い」
3. Timer Report は既定でドメインあたり 5 本、展開 1 本
4. Bottleneck は違反が多いときに共通原因インスタンスを探すのに使う
5. Datasheet Report は PolarFire 非対応

## Connects To
- **tce-06**: `report` コマンドで同じレポートをスクリプト生成
- **sta-04**: CDC レポート
- **sta-01**: Analysis View とパスセット
