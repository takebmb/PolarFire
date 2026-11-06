# Libero SoC License Installation Guide v2026.1: Floating ライセンスとクライアント接続、オフライン IP（PDF p.6-9）

## Core Idea
Floating ライセンスは **サーバで `lmgrd` とベンダーデーモン 3 種を動かし**、クライアントは `port@hostname` を環境変数に書いて接続する。既定ポートは **1702**。ノードロックならこの章は不要。

## Frameworks Introduced
- **Floating サーバ構築（Windows: p.6 / Linux: p.6-7）**
  1. Microchip Libero ページ → Download Software → **Daemons** タブから OS 用デーモンを取得
  2. `License.dat` をサーバに置く（Windows 例 `C:\FlexLM`、Linux 例 `<user_home>/FlexLM/`）
  3. `License.dat` を編集:
     - `SERVER` 行の `<put.hostname.here>` を実ホスト名に（山括弧は消す）
     - 必要ならポート番号を変更（既定 1702）
     - `VENDOR` / `DAEMON` 行をデーモンの実パスに
  4. 保存して `lmgrd` を起動（ログを取るなら `-l`）
- **クライアント接続（p.8）**
  - Windows: ユーザー環境変数 `LM_LICENSE_FILE` と `SNPSLMD_LICENSE_FILE` に `port@hostname` を設定（既存なら追記）
  - RHEL/CentOS（csh 系）: `setenv`、Ubuntu（sh 系）: `export`。`SYNPLCTYD_LICENSE_FILE` も設定する
- **オフライン PC への IP コア導入（MegaVault, p.8-9）**
  - When: インターネット不可、またはセキュリティ制限で Libero Catalog から IP を落とせない
  - How:
    1. 対応 Libero 版の MegaVault インストーラを取得（v12.0 以降用 / v11.9 以前用）
    2. MegaVault を入れて Vault の場所を設定
    3. 必要な IP の `.zip`、または特定版なら `.cpz` を取得（索引: DirectCores / SgCores / Firmware cores）
    4. Libero で **View > Catalog** → **Option** → **Add Core to Vault** → `.cpz` を指定

## Key Concepts
- **lmgrd**: FlexLM のライセンスマネージャ本体
- **actlmgrd**: Libero 用ベンダーデーモン
- **snpslmd**: Synopsys（Synplify Pro / Identify）用デーモン
- **saltd**: Siemens（ModelSim / QuestaSim）用デーモン。旧 mgcld の後継（rn-01）
- **port@hostname**: クライアントがサーバを指す書式。例 `1702@license_server`
- **SNPSLMD_LICENSE_FILE / SYNPLCTYD_LICENSE_FILE**: Synopsys ツール用のライセンス変数
- **MegaVault**: Microchip IP コアをまとめたオフライン用 Vault
- **.cpz**: Vault に取り込む IP コアパッケージ

## Commands & Syntax / Code Examples
Windows サーバの `License.dat`（パス例）:
```text
DAEMON actlmgrd C:\flexlm\actlmgrd
DAEMON saltd C:\flexlm\saltd
VENDOR snpslmd C:\flexlm\snpslmd
```
Windows サーバの起動とログ出力:
```bat
C:\flexlm\lmgrd -c C:\flexlm\License.dat
C:\flexlm\lmgrd -c <path_to_License.dat>/License.dat -l /license.log
```
Linux サーバの `License.dat` と起動:
```sh
VENDOR snpslmd /tools/Libero/snpslmd
DAEMON saltd /tools/Libero/saltd
DAEMON actlmgrd /tools/Libero/actlmgrd

<user_home>/FlexLM/lmgrd -c <path_to_License.dat>\License.dat
<user_home>/FlexLM/lmgrd -c <user_home>/FlexLM/License.dat -l <user_home>/FlexLM / license.log
```
Linux クライアント（原文どおり）:
```sh
# RHEL / CentOS
setenv LM_LICENSE_FILE 1702@sage
setenv SNPSLMD_LICENSE_FILE port@hostname (Ex: 1702@license_server)
setenv SYNPLCTYD_LICENSE_FILE port@hostname (Ex: 1702@license_server)
# Ubuntu
export LM_LICENSE_FILE port@hostname (Ex: 1702@license_server)
```
（原文外）bash の `export` は `=` が必要。実際には `export LM_LICENSE_FILE=1702@license_server` と書く。Linux 例の `<path_to_License.dat>\License.dat` の `\`、ログ例の `FlexLM /` と ` license.log` の間の空白も PDF の改行・誤記と思われる。

## Reference Tables
| 変数 | 設定先 | 値 |
|---|---|---|
| `LM_LICENSE_FILE` | 全 OS のクライアント | `port@hostname`（ノードロックならファイルパス） |
| `SNPSLMD_LICENSE_FILE` | Windows / Linux クライアント | `port@hostname` |
| `SYNPLCTYD_LICENSE_FILE` | Linux クライアント | `port@hostname` |

## Anti-patterns
- **`SERVER` 行の `<put.hostname.here>` をそのまま、または山括弧付きで残す**: 山括弧ごと実ホスト名に置き換える
- **v11.16 の古いデーモンを使い続ける**: v2024.2 以降は FlexLM v11.19 の 64 bit デーモンが必須（rn-01）
- **ノードロックなのにクライアント接続を設定する**: ノードロックでは不要（p.8）

## Key Takeaways
1. Floating = サーバで `lmgrd -c License.dat`、クライアントで `LM_LICENSE_FILE=port@hostname`
2. 既定ポートは 1702。デーモンは actlmgrd / snpslmd / saltd の 3 種
3. Synopsys ツール用に `SNPSLMD_LICENSE_FILE` も設定する
4. オフライン環境の IP は MegaVault と `.cpz` を Catalog の **Add Core to Vault** で取り込む
5. 原文の Linux 用コマンド例は書式が崩れている。そのまま貼らずに直して使う

## Connects To
- **inst-01**: ライセンス種別の選び方とノードロック
- **rn-01**: Floating License Daemon Support（v11.19 への更新手順）
- **fpx-03**: RPS の 57605/57606 ポート（ライセンスの 1702 とは別物）
