# Libero SoC v2026.1 Release Notes: 動作環境・ライセンス・新機能（PDF p.1-8）

## Core Idea
v2026.1 は **64 bit の Windows 11 / RHEL・AlmaLinux 8.10, 9.6 / Ubuntu 22.04.3, 24.04.3** 向け。Windows 10 は 2025.1 で対象外になった。PolarFire SoC 関連の主な変更は Silver 対象の拡大（MPFS250TC）、MSS DDR まわり、eNVM の平文データクライアント、Remote Programming Server。

## Frameworks Introduced
- **導入前チェック（p.1-2）**
  1. OS が対応一覧にあるか（Windows は **Windows 11 のみ**）
  2. Windows ではインストールに**管理者権限**が要る
  3. ダウンロードしたインストーラのハッシュを確認（Windows `certutil -hashfile`、Linux `md5sum` / `sha256sum`）
  4. Floating 利用者はデーモンを **v11.16 → v11.19** に更新し、microchipDIRECT から更新済みライセンスを入れ直す
  5. RAM: MPF500T / MPFS460T / RT PolarFire は 32 GB 以上、それ以外は 16 GB 以上を推奨
- **既存コアの更新手順** → rn-02

## Key Concepts
- **FlexLM v11.19**: v2024.2 以降、Floating 利用者に必須の 64 bit デーモン。過去の Libero 版とも互換
- **saltd**: Siemens シミュレータ用デーモン（旧 mgcld を置き換え）
- **QuestaSim ME Pro 2025.3**: 既定シミュレータ。ModelSim ME Pro 2025.3 は任意。**ModelSim ME Pro は v2026.1 が最後**で、v2027.1 から QuestaSim ME Pro のみ
- **Visualizer**: QuestaSim の統合デバッグ環境。旧来の `+acc` ではなく `-access` / `-debug` を使う
- **Remote Programming Server (RPS)**: 別マシンにつないだプログラマを Libero / FlashPro Express から使う新機能（→ fpx-02, fpx-03）
- **Silver License Expansion**: v2026.1 で Silver 対象に追加されたデバイスとパッケージの組
- **Classic Constraint flow**: 非対応になった。IGLOO2 / SmartFusion2 / RTG4 の Classic フロー案件は開けない

## Commands & Syntax / Code Examples
```bat
REM Windows: ハッシュ確認（原文外: [HASH] には MD5 / SHA256 などアルゴリズム名を入れる）
certutil -hashfile [FILENAME] [HASH]
```
```sh
md5sum <path_to_installer>
sha256sum <path_to_installer>
```
```sh
# Visualizer フローの基本形
vsim -voptargs="-debug -designfile design.bin" -qwavedb=+signal+memory
```
シミュレーションの `vlog` / `vcom` にユーザー引数を渡せるようになった（1.3.1.2）。

## Reference Tables
**対応 OS（64 bit, p.2）**
| OS | 版 |
|---|---|
| Windows | 11.0（10 は 2025.1 で終了） |
| RHEL / AlmaLinux | 8.10, 9.6 |
| Ubuntu | 22.04.3 LTS, 24.04.3 LTS（`check_linux_req.sh` で依存パッケージを入れる） |

**v2026.1 で Silver 対象に追加（1.4.13, p.7）**
| デバイス | パッケージ |
|---|---|
| MPF300TS-1FCG1152I（PolarFire） | FCG1152I |
| MPFS250TC（PolarFire SoC） | FCG1152I, FCG1152E, FCVG484E |
- 注: v2026.1 の Welcome Page は誤って **MPF200TC** と表示する。正しくは MPFS250TC で、Silver で使える

**PolarFire / PolarFire SoC 系の新機能（抜粋, p.5-8）**
| 項目 | 内容 |
|---|---|
| PF_XCVR シミュレーション | 高速ロック、TX/RX 独立レーン |
| PF_IOD | 静的遅延 IOD のポストレイアウト機能シミュレーションと初期タップ値。動的トレーニングの IOD は対象外 |
| PF_XCVR クロック | Global（共有）クロックを選べる。fabric へのグローバル出力はクアッドあたり 1 本まで |
| PF_Crypto | AHB ≥125 MHz なら内蔵 DLL を有効、≤70 MHz なら無効。70〜125 MHz は非対応 |
| PF_DDR | メモリ IF のバンクで I/O Calibration が無効だと警告 |
| DRC | `CLK_FROM_RCOSC_160MHZ` が RC Oscillator 駆動かを検査 |
| MSS（PFSoC） | DDR インピーダンスを Ω 表示、eNVM に平文データクライアント、PMP/MPU の batch DRC、`tCCD_L` 対応、HDL エクスポート時に MSS Timing Shell も出力 |
| ツール | Synplify Pro ME / Identify ME Y-2026.03M、Netlist Viewer で CFG マクロの INIT を表示、`sd_create_pin_group` の修正 |

## Anti-patterns
- **Windows 10 に v2026.1 を入れる**: 2025.1 以降は非対応
- **Floating のデーモンを古いまま新版を起動する**: 先に v11.19 へ更新
- **Welcome Page の「MPF200TC」を見て MPFS250TC が Silver 対象外と判断する**: 表示の誤記
- **ModelSim 前提のスクリプトを長く使い続ける**: v2027.1 で ModelSim ME Pro は外れる

## Key Takeaways
1. Windows 11 64 bit、管理者権限でインストールする
2. MPFS095T 用の RAM は 16 GB 以上（32 GB 推奨は MPF500T / MPFS460T / RT 系のみ）
3. Silver のベース対象一覧はこのリリースノートに無い。載っているのは v2026.1 の追加分だけ。MPFS095T は Discovery Kit 文書で Silver 利用が前提（原文外: 最終確認は Microchip の License Selector Guide）
4. 既定シミュレータは QuestaSim ME Pro。ModelSim は今回が最後
5. FlashPro Express / SmartDebug でリモートプログラマが使えるようになった

## Connects To
- **inst-01 / inst-02**: ライセンス申請と環境変数
- **rn-02**: コア更新、修正済み不具合、既知の不具合
- **fpx-02 / fpx-03**: RPS の設定とトラブル対処
- **polarfire-soc-mss**: MSS DDR / PMP / eNVM の詳細
