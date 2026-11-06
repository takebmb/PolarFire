# Libero SoC License Installation Guide v2026.1: ライセンス取得とノードロック設定（PDF p.1-5, 10）

## Core Idea
Libero SoC を使うまでの流れは 3 段: **ダウンロード → インストーラ実行 → ライセンス取得・設定**。ライセンスは microchipDIRECT で発行され、メールで `License.dat` が届く。Windows 単体 PC なら**ノードロック**を選び、`LM_LICENSE_FILE` にファイルパスを通せば終わる。

## Frameworks Introduced
- **ライセンス種別の選択**
  - When: 最初のライセンス申請時
  - How:
    - Windows の 1 台で使う → **Node-locked**（Windows 専用。DiskID で紐付け）
    - Linux/UNIX、または複数人で共有 → **Floating**（Windows/UNIX/Linux。MAC ID で紐付け、ライセンスサーバが必要 → inst-02）
    - 迷ったら License Selector Guide を見る
- **Silver（無償）ライセンスの申請手順（p.3）**
  1. microchipDIRECT にサインイン
  2. `www.microchipdirect.com/fpga-software-products` の License ページへ
  3. **Request Free License** を選ぶ
  4. Libero Silver の Floating か Node-locked の行で **Register**
  5. 種別に応じて **DiskID** または **MACID** を入力 → **Register**
  6. ライセンスが microchipDIRECT のメールで届く。15〜30 分来なければポータルに再ログインしてダウンロード
- **有償（Gold / Platinum）・有償 IP（p.3-4）**: 購入 → Software ID がメールで届く → License ページの **Registered Purchased Software ID** → Software ID 入力 → **Submit** → DiskID/MACID 入力 → **Register**。IP ライセンスは届いた内容を、同じ DiskID/MACID の Libero ライセンスファイルに追記して 1 ファイルにまとめる
- **ノードロックライセンスのインストール（p.5）**
  1. ルート直下に `FlexLM` フォルダを作り（例 `c:\FlexLM`）、`License.dat` を置く
  2. スタートメニューで **Environment Variables** を検索して開く
  3. **User Variables** で:
     - `LM_LICENSE_FILE` が既にある → **Edit** で末尾に `;` 区切りでパスを追記
     - 無い → **New**、名前 `LM_LICENSE_FILE`、値 `<drive directory>\Folder\License.dat`
  4. Libero を起動して動作確認

## Key Concepts
- **microchipDIRECT**: ライセンス申請・ダウンロードのポータル
- **Silver**: 無償ライセンス。Floating と Node-locked の両方を申請できる
- **Gold / Platinum**: 有償ライセンス。Software ID を受け取ってから登録
- **DiskID / MACID**: ライセンスを紐付けるホスト識別子。種別で使い分ける（原文は「depending on the license type」とだけ書く）
- **License.dat**: 発行されるライセンスファイル。メール添付または License ページから取得
- **LM_LICENSE_FILE**: FlexLM がライセンスを探す環境変数。複数値は `;` 区切り（Windows）
- **FlexLM**: ライセンス管理（FlexNet）。フォルダ名の慣例にも使われる

## Commands & Syntax / Code Examples
```text
REM ノードロック: ユーザー環境変数（GUI で設定）
LM_LICENSE_FILE = c:\FlexLM\License.dat
REM 既存値がある場合は ; で連結
LM_LICENSE_FILE = <既存値>;c:\FlexLM\License.dat
```
（原文外）コマンドラインで設定するなら `setx LM_LICENSE_FILE "c:\FlexLM\License.dat"`。既存値を上書きするので、先に `echo %LM_LICENSE_FILE%` で中身を確かめる。

## Reference Tables
| 種別 | 対応 OS | 紐付け | 用途 |
|---|---|---|---|
| Node-locked | Windows のみ | DiskID（原文外: 一般的な対応） | 1 台の PC |
| Floating | Windows / UNIX / Linux | MAC ID（原文外: 一般的な対応） | サーバ経由で共有 |

## Anti-patterns
- **Linux でノードロックを申請する**: ノードロックは Windows 専用。Linux は Floating しか使えない
- **環境変数の既存値を上書きする**: 他ツールの FlexLM 設定が消える。`;` で追記する
- **メールが来ないまま待ち続ける**: 15〜30 分で来なければポータルから直接ダウンロード
- **IP ライセンスを別ファイルのまま置く**: 同じホスト ID の Libero ライセンスファイルへ追記する

## Key Takeaways
1. Windows 11 の 1 台で Discovery Kit を使うなら **Silver + Node-locked** が最短
2. 設定は `License.dat` を `C:\FlexLM` に置き、ユーザー環境変数 `LM_LICENSE_FILE` を通すだけ
3. Silver の対象デバイス一覧はこのガイドには無い。v2026.1 での追加分は rn-01、Discovery Kit 用ドキュメントでは Silver の使用が前提になっている
4. このガイド（Rev G）にはインストーラの GUI 手順が載っていない。Rev C（2024/02）時点にあった GUI/Console/Silent インストールや「VM 上でのライセンス」の節は Rev E の再構成で消えている（p.10）
5. インストーラの実行には管理者権限が要る（rn-01）

## Connects To
- **inst-02**: Floating ライセンスのサーバ設定とクライアント側の環境変数
- **rn-01**: 対応 OS（Windows 11）、Silver の対象拡大、Floating 用デーモン v11.19
- **polarfire-soc-discovery-kit / ch02, references/quick-start.md §6**: キット側の Silver 取得手順
