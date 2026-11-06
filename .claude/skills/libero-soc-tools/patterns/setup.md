# パターン: セットアップ・書き込み

(原文外) は資料外から補った部分。

## Windows 単体 PC で Silver ノードロックを使う
**When to use**: Windows 11 の 1 台で Libero を評価・学習に使う（Discovery Kit など）
**How**: microchipDIRECT → Request Free License → Libero Silver Node-locked の Register → DiskID を入力 → 届いた `License.dat` を `C:\FlexLM` に置く → ユーザー環境変数 `LM_LICENSE_FILE` にパスを設定（既存値があれば `;` で追記） → Libero を起動して確認（inst-01）
**Trade-offs**: 他の PC では使えない。Linux はノードロック非対応なので Floating が必要。Silver の対象デバイスは限られる（v2026.1 の追加分は rn-01）

## Floating ライセンスサーバで共有する
**When to use**: Linux で使う、または複数人・複数台で共有する
**How**: Daemons タブから v11.19 のデーモンを取得 → `License.dat` の SERVER 行のホスト名、ポート（既定 1702）、DAEMON/VENDOR 行のパスを編集 → `lmgrd -c License.dat -l <log>` で起動 → クライアントで `LM_LICENSE_FILE` と `SNPSLMD_LICENSE_FILE` に `1702@server` を設定（Linux は `SYNPLCTYD_LICENSE_FILE` も）（inst-02, rn-01）
**Trade-offs**: サーバの運用が要る。デーモンの版上げ（v11.16 → v11.19）を忘れると新しい Libero が起動しない

## 書き込み失敗を段階的に切り分ける
**When to use**: FlashPro Express の RUN が FAILED になる、またはプログラマが見えない
**How**: Refresh/Rescan → Ping（プログラマとの通信）→ Scan and Check Chain（IDCODE と構成の照合）→ DEVICE_INFO（セキュリティ・既存設計）→ PROGRAM → VERIFY。エラーコードは fpx-04 の表で引き、電源・DEVRST_N・JTAG 信号品質から確認する（fpx-02, fpx-04）
**Trade-offs**: eFP5（FlashPro5）では Self Test が使えないので、プログラマの故障を直接は切り分けられない

## GUI で一度やってから Tcl でバッチ化する
**When to use**: 同じ `.job` を繰り返し書く、CI や量産で自動化する
**How**: GUI でジョブを開いて RUN → File > Export Script File（Include commands from current project only）→ `<Libero>/bin/FPExpress.exe SCRIPT:prog.tcl logfile:prog.txt console_mode:brief`。引数は `SCRIPT_ARGS:"..."` で渡し、Tcl 側では `argv` で受ける（fpx-03）
**Trade-offs**: コマンド名や引数はガイドに全部は載っていない。Export で出た実物を基準にする。終了コードの扱いはスクリプト側で作る必要がある

## Windows でローカルのプログラマが見えないときにファイアウォールを開ける
**When to use**: ケーブルもドライバも問題ないのに、Windows で FlashPro（eFP5 を含む）が検出されない
**How**: 管理者の PowerShell で `New-NetFirewallRule` を実行し、`Designer\bin` の prgsrvdmn.exe / prgservapp.exe / dbgservapp.exe の Inbound を許可する。アンチウイルスの除外も確認する（fpx-03）
**Trade-offs**: セキュリティポリシーに触れる変更。RPS 用の 57605/57606 を外部に公開してはいけない
