# FlashPro Express User Guide v2026.1: プログラマ操作・アクション・モード・チェーン・ハード仕様（PDF p.16-33, 55-65）

## Core Idea
書き込み前の切り分けは **Ping（プログラマ）→ Scan and Check Chain（JTAG チェーン）→ アクション（デバイス）** の 3 段。アクションは PROGRAM / VERIFY / ERASE / DEVICE_INFO / READ_IDCODE / ENC_DATA_AUTHENTICATION。モードは量産用の **Operator** とジョブを編集できる **Developer** の 2 つ。

## Frameworks Introduced
- **切り分けの順序（5.3-5.5, p.22-23）**
  1. **Ping**: プログラマ本体だけと通信する。ターゲットには触れない。失敗なら USB・ドライバ・電源（リモートならネットワーク・Host）の問題で、ジョブやチェーンの問題ではない
  2. **Self Test**: 付属のセルフテスト基板でプログラマ自体を検査。**FlashPro5/4 は非対応**（工場で検査済み）。eFP5 でも使えない（原文外: FlashPro5 相当のため）
  3. **Scan and Check Chain**: チェーンの各デバイスの IDCODE を読み、ジョブの構成（種類・数・順序）と照合する。読み取り専用。IDCODE 違い、デバイスの欠落・余分、順序違い、TDI/TDO の断線・短絡を報告
  4. **アクションを RUN**
- **Enable/Disable と Remove の使い分け（5.6-5.8）**
  - 一時的に外す → チェックを外す（設定は残る。リモートの予約も残る）
  - セッションから外す → 右クリック **Remove**（名前・TCK 上書き等の設定を破棄、予約も解放）。Refresh/Rescan で既定設定のまま戻ってくる
- **Developer モードでできること（7, p.28-31）**: ジョブの更新、デバイスの有効・無効（bypass）、設計ファイルと SPI Flash ファイルの差し替え、デバイスごとに別アクション（**Run selected action for all devices** を外す）。切替は **Project > Preferences**。設定は Windows ではユーザー×マシンごと、Linux ではユーザーごとに保存
- **SPI Flash の書き込み（PolarFire / PolarFire SoC, p.17, 30）**: Libero のエクスポートは常に **`.bin`**。STP で書くときは先に `.bin` を Unload してから **Load Programming File** で `.stp` を読む（ログ上は設計の書き込み扱い）
- **リモートプログラマ（5.1, 5.10）**: **Tools > Programming and Debug Server Settings > Configure Remote Server** でホストを追加し、Reserve → Enable で使う。サーバ側は Libero インストーラで **Custom** → **RPS** にチェック、既定ポート **57605 / 57606**（インストーラがファイアウォールを開ける）。同一ネットワークか VPN 内が条件

## Key Concepts
- **Programming Action**: 選んだアクションを、有効なプログラマすべてと、ジョブのダイと一致するチェーン上の全デバイスに実行
- **Force TCK Frequency**: STAPL/PDB の `FREQUENCY` 文を無視して TCK を固定する。既定はオフ（ファイルの指定に従う）
- **Default TCK**: ファイルもチェーンも無いときは 4 MHz。複数デバイスでは `freq` と `MAX_FREQ` の最小値を使う
- **Discrete Clocking**: FlashPro4/3X の TCK モード。JTAG 非準拠デバイスが混ざるときに使う。書き込みが大幅に遅くなる
- **Vpump**: FlashPro5/4/3/3X の Vpump は 1 デバイス分しか供給できない。チェーンでは Vpump / Vcc / Vjtag を基板から供給する
- **Drive TRST**: TRST を弱プルダウンで GND に落としている基板で、JTAG をリセットから出すためのフラグ。普通の基板では不要
- **Disable Core Check During Scan and Check Chain**: Tools メニュー。スキャン時のコア検査を省く

## Reference Tables
**Programming Action（5.9, p.24-25）**
| Action | 内容 |
|---|---|
| PROGRAM | ビットストリームを書く。ログに書き込み時のコンポーネント digest、読み戻した digest、ビットストリーム全体の digest が出る（前 2 つは値も個数も一致しない） |
| VERIFY | 書き換えずに読み戻してファイルと比較 |
| ERASE | ファブリックと構成メモリを消去。PROGRAM に消去が含まれないジョブで再書き込みする前に使う |
| DEVICE_INFO | 種類・書き込まれた設計・書き込み回数・セキュリティ・各 digest を表示。消去が中断されると回数は失われる |
| READ_IDCODE | ジョブ対象デバイスの IDCODE だけを読む（Scan and Check Chain より狭い） |
| ENC_DATA_AUTHENTICATION | 全デバイスが暗号化ビットストリームのときだけ出る。各ファイルを認証 |

**選べる TCK（5.2）**
| Programmer | TCK (MHz) |
|---|---|
| FlashPro6 | 1〜20（1 MHz 刻み） |
| FlashPro5 | 1, 2, 3, 4, 5, 6, 10, 15, 30 |
| FlashPro4 | 1〜6 |
| FlashPro3/3X | 1, 2, 3, 4, 6 |
SPI Target の SCK: 1.00, 2.00, 2.50, 3.33, 4.00, 5.00, 6.67, 8.00, 10.00, 13.33, 20.00 MHz

**チェーンの順序（6, p.27）**: `TDI > Device N > ... > Device 1 > TDO`。Device 1 から順に書く。チェーン長の上限は無いが、信号品質とタイミングは保つ

**ハード仕様の要点（p.55-65）**: 10 ピン 2×5、0.1 インチピッチ。FlashPro5/6 の JTAG ピンの消費電流は約 30 mA。VJTAG はターゲットから供給する入力専用ピン。参考の書き込み時間（PPD 形式）: MPF100 は FP5 で 39 秒、MPF300 は FP5 で 1 分 33 秒。20 MHz で書くときは信号品質を確保する

## Anti-patterns
- **RUN 中に中断・抜線する**: デバイスを壊すおそれがある（p.25）。FlashPro6 の内部デザイン更新中に抜くと OS から見えなくなる
- **FlashPro5（eFP5）で Self Test を探す**: 非対応
- **TCK を上げただけで速くなると思う**: Force TCK Frequency を付けないとファイルの `FREQUENCY` が勝つ
- **Disable で予約も解放されたと思う**: リモートの予約は Reserve/Release で解放する

## Key Takeaways
1. 切り分けは Ping → Scan and Check Chain → アクション
2. 再書き込み前は ERASE が要るかをジョブで確認し、書いた後は VERIFY で確かめる
3. ジョブの中身を変えるなら Developer モード
4. PolarFire SoC の SPI Flash は `.bin` で書く
5. RPS はポート 57605/57606、信頼できる LAN か VPN 内だけで使う

## Connects To
- **fpx-01**: ジョブ作成
- **fpx-03**: 同じ操作を Tcl やコマンドラインで行う方法、RPS のトラブル対処
- **fpx-04**: 失敗時の終了コード
