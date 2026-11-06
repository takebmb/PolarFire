# FlashPro Express User Guide v2026.1: 概要とジョブプロジェクト（PDF p.1-15）

## Core Idea
FlashPro Express は、**`.job`（Libero の Export FlashPro Express Job が出すジョブファイル）から `.pro`（ジョブプロジェクト）を作って書き込む**量産・ラボ向けツール。Libero と一緒に入るほか、単体でもインストールできる。対象は PolarFire / PolarFire SoC / SmartFusion 2 / IGLOO 2 / RTG4 で、Windows と Linux の両方で動く。

## Frameworks Introduced
- **ジョブ作成から書き込みまで（p.8-15）**
  1. Libero で **Export FlashPro Express Job** を実行 → `.job` ができる
  2. FlashPro Express を起動（`<Libero SoC installation folder>\bin` の `FPExpress`）
  3. **Project > New Job Project from FlashPro Express Job**（Start ページなら **New**）
  4. `.job` と保存先を指定 → **OK**。プロジェクト名はジョブ名で固定され、変えられない
  5. プログラマが出ていなければ **Refresh/Rescan**
  6. Programming Action で **PROGRAM** を選ぶ → **RUN**
  7. 状態バーに PASSED / FAILED。ログは Messages / Errors / Warnings / Info のタブで確認
- **ハードが無い状態でのジョブ作成（p.11）**: プログラマ未接続でも任意の `.job` からジョブを作れる。作った後でデバイスの追加・削除やファイル読込ができる
- **Construct automatically（p.11）**: 選んだプログラマにつながる実チェーンをスキャンしてチェーンを組む。**JTAG モード専用**。プログラマが複数あればドロップダウンで選ぶ
- **既存ジョブを開く（p.11）**: **Project > Open Job Project** → `.pro`。**Operator モードではデバイスが 1 個以上つながった実チェーンが無いと開けない**。Developer モードは制限なし
- **旧 FlashPro の `.pro` を移行する（p.4-5）**: single モードの `.pro` は非対応。FlashPro で chain モードに切り替え（**Tools > Mode > Chain Programming**）、**File > Configuration > Add Microchip Devices from Files** で STAPL を読み直して保存。別 PC へはフォルダごとアーカイブして移す

## Key Concepts
- **Job file (`.job`)**: 書き込み設定と暗号化ビットストリームを 1 つにまとめたファイル。量産先へ安全に渡せる（Secure Job Programming）
- **Job Project (`.pro`)**: FlashPro Express の作業プロジェクト。旧 FlashPro の `.pro` も開ける
- **Programmer table / Chain table**: メイン画面。左がプログラマ、右が JTAG チェーン（または SPI Target の 1 デバイス）
- **Device/Programmer state**: IDLE / DISABLED / PASSED / FAILED
- **HighZ**: 書き込み無効（bypass）のデバイスで列見出しに出る表示
- **SPI Target**: JTAG の代わりに SPI で書き込むモード。FlashPro5 は SmartFusion 2 / IGLOO 2 のみ、FlashPro6 は SmartFusion 2 / IGLOO 2 / PolarFire。RTG4 は非対応。**既定は JTAG**
- **Parallel programming**: 同じ STAPL を複数プログラマで同時に書く。最大 16 台（FlashPro5/4/3/3X）

## Reference Tables
**メイン画面の操作（Table 3-1）**
| したいこと | 操作 |
|---|---|
| プログラマの詳細を見る | Info アイコンにホバー |
| プログラマ名を変える | Name 欄をクリック |
| プログラマを有効・無効にする | チェックボックス |
| Ping / Self-Test / Scan / Check Chain / Remove | プログラマを右クリック |
| デバイスと書き込みファイルの情報を見る | デバイスの Info アイコンにホバー |

**Project メニュー（Table 13-1, p.62）**
| コマンド | 機能 |
|---|---|
| New Job Project | ジョブ名のフォルダを作って新規作成 |
| Open / Close / Save Job Project | `.pro` を開く・閉じる・保存 |
| Set Log File / Export Log File | ログの保存先指定・書き出し |
| Preferences | Operator / Developer モードの切替 |
| Execute Script / Export Script File | Tcl の実行・セッション操作の Tcl 書き出し |

## Anti-patterns
- **Operator モードのまま、ボード未接続でジョブを開こうとする**: 開けない。ボードをつなぐか Developer モードにする
- **ジョブプロジェクト名を変えようとする**: ジョブ名で固定
- **並列書き込みでバスパワーのハブを使う**: FlashPro5/4/3/3X にはセルフパワーのハブが必要。USB 1.1 ポートでは遅くなる
- **single モードの旧 `.pro` をそのまま開く**: 非対応。chain モードに変換してから開く

## Key Takeaways
1. 書き込みの単位は `.job` → `.pro`。Libero のプロジェクトは要らない
2. 実行ファイルは `<Libero>\bin\FPExpress`
3. プログラマが見えなければ Refresh/Rescan。結果は状態バーとログで確認
4. PolarFire SoC は JTAG で書く（SPI Target の対象一覧に PolarFire SoC は無い）
5. 量産では暗号化ビットストリーム入りの `.job` を渡す

## Connects To
- **fpx-02**: プログラマの設定と操作、アクション、モード、チェーン
- **fpx-03**: Tcl とコマンドラインでのバッチ書き込み
- **fpx-04**: 失敗時の終了コード
- **polarfire-soc-discovery-kit / ch05**: eFP5 での GUI 書き込み手順
