# Patterns — PolarFire SoC Discovery Kit

## 初回の電源投入と確認
**When to use**: 開封直後、またはボードの異常を疑うとき
**How**: J47 が 1-2 クローズであることを確認 → 付属 C-C ケーブルで PC 本体の 5V/3A Type-C ポートへ → 5P0V / VDD / 1P8V の LED を確認 → 出荷時の FIR デモ（AN5165）の動作を見る → 異常があれば Table 1-3 の測定点（例 VDD: C144、1.0V ±3%）を実測
**Trade-offs**: ドッキングステーション経由は未検証。供給が足りなければ J7（DC ジャック）+ J47 開放に切り替える

## I/O 電圧プロファイルの選択（J45 + J46）
**When to use**: 使う拡張機能が決まったとき（最初に決める）
**How**: RPi HAT・mikroBUS Click → 3.3V（J45・J46 とも 1-2、既定）。MIPI カメラ・Ethernet → 2.5V（J45・J46 とも 2-3）。PDC の io_std も合わせる（Bank 1: LVCMOS33 または 2.5V 系）
**Trade-offs**: VDDI1_5 は Bank 1 と Bank 5 で共有なので、3.3V ペリフェラルと MIPI/Ethernet は同時には UG 推奨の電圧にできない。UG の記述と参照デザインが食い違ったら回路図を優先する

## FlashPro Express で書き込む
**When to use**: Libero から `.job` を出力したあと
**How**: New Job Project → `.job` とプロジェクト保存先を指定 → OK → Programmer 欄を確認（空なら Refresh/Rescan Programmers）→ RUN → RUN PASSED
**Trade-offs**: Libero の GUI から直接書き込むこともできるが、`.job` なら Libero の無い PC でも書き込める

## UART コンソールを見つける
**When to use**: HSS / Linux / ベアメタルの出力を見たいとき
**How**: FT4232H ドライバを入れる → デバイスマネージャで COM ポートを確認 → MSS の UART は UART_B/C（Bank 2）、ファブリックの UART は UART_D（Bank 0）
**Trade-offs**: FT4232H は 4 チャネルだが UART は 3 本。どの COM 番号がどのチャネルかは OS が付ける順番によるので、出力の有無で確かめる

## ファブリックで Blinky
**When to use**: 新しいプロジェクトの最初の動作確認
**How**: 50 MHz（R18）を入力にしてカウンタを作り、LED1-7（T18, V17, U20, U21, AA18, V16, U15）を駆動する。PDC は LVCMOS18。LED は High で点灯
**Trade-offs**: LED8（E1）は MSS 側なので、ファブリックから直接は駆動しない

## MSS の周辺と RPi SPI を使う
**When to use**: RPi HAT の SPI や ID EEPROM を MSS のドライバで扱うとき
**How**: MSS Configurator で SPI・I2C を Bank 2 の該当ピン（A4/B4/D6/D5/B2、D1/C2）に割り当てる。それ以外の 21 本は Bank 1 のファブリック IO として RTL + PDC で扱う
**Trade-offs**: HAT が使う信号が MSS 側とファブリック側にまたがるので、片方だけでは完結しない

## mikroBUS Click の追加
**When to use**: センサー・通信モジュールを手早く足すとき
**How**: 3.3V プロファイル（既定）→ J5（AN/RST/CS/SCK/MISO/MOSI）・J6（PWM/INT/RX/TX/SCL/SDA）のピンをファブリックの CoreUART / CoreSPI / CoreI2C 等に接続 → PDC は LVCMOS33
**Trade-offs**: 信号はすべてファブリック Bank 1 にあるので、MSS から使う場合は Fabric Interface 経由になる

## MIPI CSI-2 カメラ
**When to use**: 底面 J11 に RPi カメラを繋ぐとき
**How**: J45・J46 を 2-3（2.5V）にする → 2 レーン（B11/C11、A10/B10）+ CLK（B12/A12）、I2C は T21/T20、制御 GPIO は Y14/Y17
**Trade-offs**: 2.5V にすると RPi GPIO や mikroBUS の 3.3V 動作は使えなくなる

## 7 セグ 8 桁基板
**When to use**: 状態表示を外に出したいとき
**How**: 表示基板の電源電圧に合わせて J49 を設定（既定 5V、3.3V なら 1-2）→ J48 の 8 本（Y15, T17, V15, Y19, AA17, W20, W16, V18）をファブリックから駆動（Bank 0、LVCMOS18）
**Trade-offs**: 信号の割り当ては基板側の仕様書で確認する。FPGA 側 1.8V と VCCB の間は電圧変換される構成

## 書き込みできないときの切り分け
**When to use**: Programmer 欄が空、または RUN が失敗するとき
**How**: 電源 LED → ケーブルが J4 に刺さっているか（データ対応の C-C ケーブル）→ Libero（ドライバ同梱）のインストール → Refresh/Rescan → 必要なら Table 1-3 で電圧確認
**Trade-offs**: 電源だけの USB-C ケーブルでは eFP5 が認識されない
