# Macro Library User Guide: Post-Layout マクロ（クロック網・I/O パッド・IOREG ほか）（PDF p.79-86）

## Core Idea
post-layout マクロは配置配線後のネットリスト（NLV = Netlist Viewer、タイミング解析、post-layout シミュレーション）に現れる実セル。ユーザーが直接インスタンスするものではないが、**差動 I/O が IOPADP_* / IOPADN_* / IO_DIFF に展開されること**、**I/O レジスタが IOREG に取り込まれること**、クロックが GB → RGB の 2 段で配られることを読み取るのに使う。

## Frameworks Introduced
- **差動 I/O の post-layout での姿（読み取り方）**: 差動入力は P 側パッドマクロ（IOPADP_IN、出力に N2PIN_P/IOUT_P）と N 側（IOPADN_IN）に分かれ、IO_DIFF が N 側の位置を予約するプレースホルダになる。入力バッファは IOIN_IB 系（ODT 入力付きは IOIN_IB_E_ODT）
- **クロック配信の 2 段構造**: GB（Global Buffer）が左右半チップのグローバルクロック（GCLK）を駆動 → 縦クロックストライプ上の RGB（Row Global Buffer）がロジックセクタを駆動。チップ全体に配るクロックは各半分から 1 個ずつ計 2 個の GB を使う

## Key Concepts
- **GB**: 全チップグローバル 24 本、半チップグローバル 48 本。半チップグローバルは最大 24 本のファブリック配線信号で駆動可。I/O 由来クロック、内部生成クロック、高ファンアウト信号（リセット等）を載せられる
- **RGB**: GCLK・リージョナルクロック・ファブリック配線クロックのいずれかを選んで左右のロジックセクタを駆動。SERDES 用など小領域のリージョナルクロックにも使う
- **HS_IO_CLK**: I/O バンクに組み込まれた高速 I/O クロック網。バンク全体に低スキューで分配し、高速インターフェースの I/O ロジックのデータ入出力に使う
- **ICB_INT / ICB_CLKINT**: ファブリック → Interface Clock Block / ICB → グローバルバッファ
- **CRN_INT**: ファブリック → コーナーセル（PLL, DLL）へのクロック
- **IOREG**: IOINFF / IOUTFF / IOEFF をまとめた単一 post-layout マクロ。IOD ブロック内の入力・出力・出力イネーブルのレジスタ（ファブリックの DFF 相当）
- **PLL_DELAY**: PLL の参照クロック経路またはフィードバック経路に入るプログラマブル遅延線。参照側=遅延、フィードバック側=前進（外部フィードバックモード必須）
- **CC_CONFIG**: クラスタ内 ARI1_CC のキャリーを生成（CC[n] = !Px!Y3+PxCC[n-1]）

## Reference Tables
**I/O 系 post-layout マクロ（p.81-83）**
| マクロ | 内容 |
|---|---|
| IOIN_IB / IOIN_IB_E | 入力バッファ（YIN → Y） |
| IOIN_IB_E_ODT | 上記 + ODT 入力 |
| IOTRI_OB_EB | 出力・イネーブルのフィードスルー（D/E → DOUT/EOUT） |
| IOBI_IB_OB_EB | 双方向のフィードスルー（D/E/YIN → DOUT/EOUT/Y） |
| **IO_DIFF** | 差動 I/O のプレースホルダ。**N 側の位置を予約** |
| IOPAD_IN / IOPAD_TRI / IOPAD_BI | 単端の入力 / トライステート / 双方向パッド（Y, Y_HW） |
| IOPADP_IN / IOPADN_IN | 差動入力の P 側（PADP → N2PIN_P, IOUT_P, IOUT_HW_P）/ N 側 |
| IOPADP_TRI / IOPADN_TRI | 差動トライステート出力の P / N 側（OIN_P, EIN_P → PAD_P） |
| IOPADP_BI / IOPADN_BI | 差動双方向の P / N 側 |
| IOPADP_IN_MIPI | MIPI 低電力エスケープ対応の差動入力パッド |
| IOPAD_FEEDBACK / IOPADP_FEEDBACK / IOPADN_FEEDBACK | 双方向入力フィードバック付き出力（N 版は D を内部反転） |
| IOREG | IOD 内レジスタ（IOINFF/IOUTFF/IOEFF） |

**クロック・その他 post-layout マクロ**
| マクロ | 内容 |
|---|---|
| GB / RGB | グローバル / 行グローバルバッファ（GB 真理値表: An, ENn → YNn, YSn） |
| CRN_INT, ICB_INT, ICB_CLKINT | コーナー / ICB へのクロック経路 |
| HS_IO_CLK | バンク内高速 I/O クロック |
| PLL_IP, PLL_DELAY | PLL 本体と遅延線 |
| CFG0 | 0 入力 LUT（INIT で 0/1） |
| CFG1A–D, CFG1A–D_TEST | 1 入力バッファ（Y=f(A..D)）。_TEST は NLV で見え BUFF に再マップ |
| CFG4A, CFG4_ROM, CFG4_IP_ABCD | 4 入力 LUT 系（_IP_ABCD は B/C/D の反転出力 IPB/IPC/IPD 付き） |
| RAM1K20_IP, RAM64x12_IP, MACC_IP | RAM/MACC の中核（パイプラインレジスタと組み合わせて元マクロを構成） |
| XCVR_APB_LINK | APB Link でトランシーバ Quad のメモリマップにアクセス |
| PFSOC_SCSM_IP | PFSOC_SCSM と同機能、入力反転（PFSOC_WAKE_N） |

**改訂履歴（p.85-86）の要点**: Rev S（03/2026）で APBM, BANKEN, BANKCTRL_GPIO, BANKCTRL_HSIO を追加し、**LANECTRL と LANECTRL_BYPASS の節を削除**（内部管理でユーザー設定不可のため）。Rev T（06/2026）で INIT を更新。

## Anti-patterns
- **post-layout マクロを RTL で直接インスタンス**: 原文は「post-layout netlist only」と明記するものが多い
- **N 側パッドに別信号を割り当てようとする**: 差動ペアの N 側は IO_DIFF で予約される（PDC で N 側に別ポートを置く運用は避ける。原文外の推論）
- **LANECTRL を手でインスタンス**: 2026.1 では内部管理（遅延値の調整は pdc-05 の edit_instance_delay）

## Key Takeaways
1. 差動 I/O は post-layout で P 側・N 側パッドマクロ + IO_DIFF（N 側予約）に展開される
2. I/O レジスタは IOREG（IOINFF/IOUTFF/IOEFF）。NDC の set_ioff で取り込みを指示
3. クロックは GB（全 24 / 半 48 本）→ RGB の 2 段
4. HS_IO_CLK がバンク内の高速 I/O クロック
5. LANECTRL はユーザー設定対象外になった

## Connects To
- **mlg-02-io-clocking**: ユーザーが使う INBUF_DIFF / OUTBUF_DIFF / CLKINT
- **pdc-04-location-ndc-floorplan**: set_ioff（IOREG への取り込み）
- **pdc-05-post-layout-edit**: edit_instance_delay（LANECTRL, ICB_CLKDIVDELAY, PLL_DELAY）
- **ioed-01-port-pin-view**: Pin View で P/N パッドの確認
