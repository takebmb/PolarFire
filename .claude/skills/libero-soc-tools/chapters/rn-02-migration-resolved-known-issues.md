# Libero SoC v2026.1 Release Notes: コア更新・修正済み不具合・既知の不具合（PDF p.9-16）

## Core Idea
旧版で作った設計を v2026.1 で開いたら、**コアを Update Component Version で更新 → SmartDesign の Update Instance → 制約を再導出 → フロー再実行**の順に進める。PolarFire SoC で注意すべき修正は「JTAG 書き込みで I/O 状態が誤って適用される」（BSR ピン順）と「MSS BFM_SELECT の既定値変更」。**既知の不具合は本文に無く、Web ページへのリンクだけ**。

## Frameworks Introduced
- **コアのバージョン更新（2.2, p.9）**
  1. 新しいコアを Vault にダウンロードする（Update Component Version ダイアログからも自動で取得できる）
  2. 設計階層でコアを右クリック → **Update Component Version**（SmartDesign 上のインスタンスでも可）。コンポーネントは自動で再生成される
  3. ポートリストが変わったら SmartDesign のインスタンスを右クリック → **Update Instance**。未接続ピンを接続するかタイオフし、SmartDesign を再生成
  4. **Build Design Hierarchy** → Constraint Manager でタイミング制約を **Derive** し直す
  5. 設計フローを再実行

## Key Concepts
- **Update Component Version**: コアを新版に上げるメニュー
- **Update Instance**: ポートが変わったインスタンスを SmartDesign に反映するメニュー
- **BSR (Boundary Scan Register)**: JTAG 書き込み時に I/O 状態を決めるレジスタ。PFSoC でピン順が誤っていた
- **BFM_SELECT**: PFSoC MSS のパラメータ。既定値が 1 から 0 に変わった
- **UL_DEBUG Lock**: デバッグロック。MSS JTAG をファブリックに出している場合はこのロックで止められない（情報メッセージを追加）
- **Known Issues ページ**: 「Libero SoC Design Suite Release Notes - Known Issues and Limitations」（Web、p.14）

## Reference Tables
**v2026.1 で更新されたコア（p.9）**
| Core | 版 | 変更 |
|---|---|---|
| PF_CRYPTO | 1.0.111 | AHB クロック周波数のガイダンスを修正 |
| PF_IOD_GENERIC_RX | 2.1.117 | データレートと fabric クロックの関係のガイダンス。整数入力時の計算誤りを修正 |
| PF_RGMII_TO_GMII | 1.3.112 | 新しい PF_IOD_GENERIC_RX を同梱 |
| PFSOC_TAMPER | 2.0.101 | RT PolarFire SoC で非対応の Fabric Digest Check を無効化 |

**PolarFire SoC / 共通で関係する修正（p.10-13 から抜粋）**
| 症状（修正前） | 修正内容 |
|---|---|
| PFSoC の JTAG 書き込みで、設定した I/O 状態が一部ピンに誤って適用された（例: G17 を 0 に設定しても 1 のまま） | I/O State 設定ファイルの BSR ピン順を修正 |
| PFSoC MSS の `BFM_SELECT` 既定値 | 1 → 0 |
| PFSoC MSS の PMP 設定に DRC が無かった | DRC を追加（batch モードでも検査） |
| MSS DDR Debug の DQ/DQS 最適化結果が誤表示 | 正しいトレーニング状態レジスタを読むよう修正 |
| SmartDebug が Ubuntu 22.04 で落ちた | 修正。Embedded FlashPro でも動作 |
| SmartDebug の DDR4 トレーニング時間表示が実測と違った | ユーザー（fabric）クロック基準に修正 |
| PFSoC の IBIS 生成でモデル欠落、LVCMOS25 で unknown model | 必要モデルを出力するよう修正 |
| メモリマップの表示・エクスポートで Libero が落ちた | 修正 |
| 長い名前で block flow のコンパイルが落ちた | ポート名 1024 文字まで対応 |
| I/O Editor が MSS DDR のインピーダンスを mA で表示 | 新しい列 MSS Impedance (Ohm) に Ω で表示 |
| eNVM に平文データクライアントが置けなかった | 追加（sNVM の Boot Mode 2 クライアントとも併用可） |
| Configure Programming Options の Probe 設定が変えられなかった | 修正 |
| QuestaSim がネット非接続環境で 120 秒待つ | 起動を改善 |
| 合成エラー「Couldn't find binding for variable p」、c_vhdl.exe の内部エラー | 修正 |

## Anti-patterns
- **コアを更新しただけでフローを回す**: ポート変更の Update Instance と制約の再導出が抜けると古い制約が残る
- **既知の不具合はリリースノートを見れば分かると思う**: 本文は Web ページへのリンクだけ。最新版は Web で確認する
- **UL_DEBUG Lock で MSS JTAG のデバッグも塞がると思う**: ファブリックに出した MSS JTAG IF からはデバッグできてしまう
- **v2025.x 以前で書いた PFSoC の I/O 状態（BSR）を信用する**: 書き込み中に一部ピンの状態が誤っていた。v2026.1 で作り直す（原文外: 推奨）

## Key Takeaways
1. コア更新は Update Component Version → Update Instance → Derive Constraints → 再実行
2. PFSoC の JTAG 書き込み時の I/O 状態のバグ（BSR ピン順）は v2026.1 で修正済み
3. `BFM_SELECT` の既定値が 0 になった。シミュレーション設定を確認する
4. 既知の不具合の一覧は本文に無い。Known Issues and Limitations の Web ページを見る
5. 関連する別リリースノート: PolarFire SoC MSS Configurator v2026.1、Programming and Debug Tools v2026.1、SmartHLS 2026.1（p.15）

## Connects To
- **rn-01**: 新機能と動作環境
- **fpx-04**: 書き込み時のエラーコード
- **polarfire-soc-mss**: MSS DDR / PMP / eNVM / BFM
