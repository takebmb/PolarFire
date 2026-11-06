# FlashPro Express User Guide v2026.1: Tcl・コマンドライン・リモートプログラマの対処（PDF p.33-36, 51-53, 62-63）

## Core Idea
GUI の操作は **File > Export Script File** で Tcl に書き出せる。それを `FPExpress.exe SCRIPT:<file>` に渡せばバッチで書き込める。リモートホストの追加・削除も Tcl で書ける。ローカルのプログラマが Windows で見えないときは、**ファイアウォールやアンチウイルスがプログラミングサーバのプロセスを止めている**ことがある。

## Frameworks Introduced
- **GUI 操作を Tcl にしてバッチ化する（8.4, p.35-36）**
  1. GUI で一度ジョブを開いて RUN する
  2. **File > Export Script File** → ファイル名 → **Export Script Options** で **Include commands from current project only**、Relative / Qualified filenames を選ぶ → OK
  3. コマンドラインで `FPExpress.exe SCRIPT:<file>` を実行する
- **GUI から Tcl を実行（8.2）**: **File > Execute Script** → Browse → **Arguments** に空白区切りで引数 → **Run**
- **Tcl の基本ルール**: コマンド名は大文字小文字を区別し、引数は区別しない。全コマンドは Tcl Commands Reference Guide を参照
- **FlashPro6 の内部デザインを更新する（7.1, p.31-33）**: Developer モードでは RUN 時に確認ダイアログが出る。Operator モードでは警告だけが出るので、Developer に切り替えて RUN するか、Tcl で `update_fp6_programmers` を使う。更新中は抜かない
- **リモート接続のトラブル対処（9.7, p.51-53）**
  1. ホスト名・IP が正しいのに失敗 → **リモート側**で TCP 57605-57606 を開ける
  2. 届くのにプログラマが見えない → Media Installer のインストール後チェックを確認し、だめなら RPS を入れ直す
  3. 異常終了や挙動不良 → 最後の手段として `bin/CleanUpRPS` を管理者で実行。手動なら共有メモリを消し、`prgsrvdmn` / `prgservapp` / `dbgservapp` を kill する
  4. ローカルのプログラマが Windows で見えない → 3 プロセスにファイアウォールの許可ルールを足す
  - リモート側のプロセスにはユーザー向けのログファイルが無い

## Key Concepts
- **SCRIPT: / script:**: `FPExpress.exe` に Tcl ファイルを渡す引数（大文字小文字どちらの例も原文にある）
- **logfile:**: ログを書き出すファイル
- **console_mode:**: `hide`（既定・未指定）/ `brief`（ログを一時表示）/ `show`（コンソールを開いたまま）
- **SCRIPT_ARGS:**: Tcl に渡す引数。Tcl 側では `argc` / `argv` / `argv0` で受ける
- **add_programmer_server / remove_programmer_server**: リモートホストの追加・削除（接続・切断も兼ねる）。`-host_name` を省くとローカル接続とみなす
- **prgsrvdmn / prgservapp / dbgservapp**: プログラミング・デバッグのサーバプロセス（Designer\bin）
- **LOG_WINDOW_BUFFER_SIZE**: ログウィンドウのバッファサイズを決める def 変数

## Commands & Syntax / Code Examples
コマンドライン実行（原文どおり）:
```text
<location of Microchip software>/bin/FPExpress.exe SCRIPT:<filename>
<location of Microchip software>/bin/FPExpress.exe script:foo.tcl
<location of Microchip software>/bin/FPExpress.exe script:foo.tcl logfile:foo.txt
<location of Microchip software>/bin/FPExpress.exe script:foo.tcl console_mode:brief logfile:foo.txt
<location of Microchip software>/bin/FPExpress.exe script:foo.tcl console_mode:show logfile:foo.txt
<location of Microchip software>/bin/FPExpress.exe SCRIPT:<filename> SCRIPT_ARGS:"param1 param2 param3"
FPExpress script:"FPExpress tcl/foo 1.tcl"
```
Tcl で引数を受ける（原文外: PDF では改行が崩れているので、動く形に直した）:
```tcl
puts "Script name: $argv0"
puts "Number of arguments: $argc"
set i 0
foreach arg $argv { puts "Arg $i : $arg"; incr i }
```
リモートホストと各種 Tcl:
```tcl
add_programmer_server -host_name {hostname}
remove_programmer_server -host_name {hostname}
update_fp6_programmers
run_selected_actions -prog_spi_flash 0 -disable_prog_design 0
```
（原文外）ジョブを開いて PROGRAM する Tcl の例（`open_project` / `set_programming_action` / `run_selected_actions` など）は Export Script File で実物を出して確かめる。コマンド名をこのガイドから推測しない。

RPS のポートを開ける（リモート側）:
```sh
sudo firewall-cmd --permanent --add-port 57605-57606/tcp   # AlmaLinux / RHEL
sudo ufw allow 57605:57606/tcp                              # Ubuntu
```
Windows は **Windows Defender Firewall with Advanced Security** → Inbound Rules → New Rule → Port → TCP → `57605, 57606` → Allow → Domain/Private/Public すべて。

ローカルのプログラマが見えない（Windows, 管理者で実行）:
```powershell
New-NetFirewallRule -DisplayName "Programming and Debug Server" -Direction Inbound -Program "<installation path>\Libero_SoC_<version>\Libero_SoC\Designer\bin\prgsrvdmn.exe" -Action Allow
New-NetFirewallRule -DisplayName "Programming Server" -Direction Inbound -Program "<installation path>\Libero_SoC_<version>\Libero_SoC\Designer\bin\prgservapp.exe" -Action Allow
New-NetFirewallRule -DisplayName "Debug Server" -Direction Inbound -Program "<installation path>\Libero_SoC_<version>\Libero_SoC\Designer\bin\dbgservapp.exe" -Action Allow
```
（原文外）原文は「command prompt as administrator」と書くが、`New-NetFirewallRule` は PowerShell のコマンドレット。管理者の PowerShell で実行する。

共有メモリの手動クリア: Linux は `rm /dev/shm/*`、Windows は `C:/ProgramData/boost_interprocess/` の中身を削除。

## Anti-patterns
- **バッチのログを残さない**: `logfile:` を付けないと結果を後から追えない
- **スペースを含むスクリプトパスをクォートしない**: `script:"..."` で囲む
- **RPS のポートをインターネットに公開する**: RPS は信頼できる LAN か VPN 前提で、守りは顧客側のネットワーク対策頼み
- **最初から CleanUpRPS や `/dev/shm/*` 削除に走る**: 最後の手段。先にポートと RPS の状態を確認する

## Key Takeaways
1. バッチ書き込みは Export Script File で作った Tcl を `FPExpress.exe SCRIPT:` に渡す
2. `console_mode` は hide / brief / show、ログは `logfile:`、引数は `SCRIPT_ARGS:"..."`
3. Windows 11 で eFP5 が見えないときは、ケーブルとドライバの次にファイアウォールとアンチウイルスを疑い、prgsrvdmn / prgservapp / dbgservapp を許可する
4. RPS の既定ポートは 57605/57606（ライセンスの 1702 とは別）
5. FlashPro6 の更新は Developer モードか `update_fp6_programmers` で行う（eFP5 には関係しない）

## Connects To
- **fpx-01 / fpx-02**: GUI 側の同じ操作
- **fpx-04**: バッチ実行で返る終了コード
- **rn-01**: Remote Programming Server が新機能として入った
- **polarfire-soc-discovery-kit / references/an5282-design-flow.md**: FlashPro Express の batch 書き込み例
