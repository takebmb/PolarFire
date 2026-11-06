---
name: polarfire-soc-discovery-kit
description: Board-level knowledge base for the Microchip PolarFire SoC Discovery Kit (MPFS-DISCO-KIT, MPFS095T-1FCSG325E) from the kit User Guide, Rev2 schematics, Quick Start Guide, AN5282, AN5165, and the GitHub reference-design / embedded-software docs. Use for bring-up, jumpers and bank voltages, power, FPGA ball/net names and PDC constraints for on-board LEDs/switches/connectors, UART/COM ports, HSS/Linux boot on this kit, reference design generation, and FlashPro Express programming. For MSS internals (memory map, FIC, PLIC, peripherals) use polarfire-soc-mss.
when_to_use: MPFS-DISCO-KIT, PolarFire SoC Discovery Kit, ピン配置, PDC set_io, 回路図 ネット名, J45 J46 J47 J49, Bank 電圧, LED SW8 push button, Raspberry Pi J10, mikroBUS J5 J6, MIPI J11, microSD J17, 7セグ J48, VSC8221 Ethernet, FT4232H COM ポート, FlashPro Express .job, Libero Silver, リファレンスデザイン Tcl, Linux wic SD 起動, FIR demo AN5165, AN5282
allowed-tools: Read Grep
argument-hint: [topic, connector/jumper name, pin, or chapter number]
---

# PolarFire SoC Discovery Kit（ボード）

MPFS-DISCO-KIT REV 2（シルク SEP 2023）、デバイス MPFS095T-1FCSG325E。原本は `C:\_Devs\PolarFire\doc\board\`。

## 使い方

- 下の Core で足りなければ、**Where to Look** から該当ファイルを 1 つだけ Read して答える（全部は読まない）
- 資料どうしが食い違うときは **回路図 > UG > その他** の順で信頼し、食い違いがあることを回答に書く。食い違いの一覧は [references/sources.md §4](references/sources.md)
- `[推定]` の付いた信号名（chapters/）は回路図と照合済み（[schematic-verification.md](references/schematic-verification.md)）
- MSS の内部（アドレス、FIC、割り込み、ペリフェラルのレジスタ）は skill **polarfire-soc-mss** を使う
- Libero の操作（PDC の構文、I/O 規格の属性、SDC、Tcl、SmartDebug、FlashPro Express の詳細）は skill **libero-soc-tools** を使う

## Core

### 1. I/O 電圧プロファイル（J45 + J46）
- **3.3V（既定、両方 1-2）** = Raspberry Pi GPIO・mikroBUS ／ **2.5V（両方 2-3）** = MIPI RX・Ethernet。J45 と J46 は必ず同じ側にする
- VDDI1_5 は **Bank 1（GPIO）と Bank 5（MSS SGMII＝Ethernet）で共用**。Ethernet は回路図の注記でも 2.5V 指定。3.3V のままで動くかは資料からは判断できない
- Bank 0 は **1.8V 固定**（LVCMOS18）

### 2. MSS 側かファブリック側か
| MSS Configurator で設定 | RTL + PDC で設定 |
|---|---|
| DDR4（Bank 6）、microSD（Bank 4）、Ethernet SGMII（Bank 5）、125 MHz REFCLK、MMUART_1/4（Bank 2）、RPi の SPI + ID I2C（Bank 2）、LED8（E1） | LED1-7、SW1/SW2、DIP SW8、7 セグ、UART_D（Bank 0）、RPi の残り、mikroBUS、MIPI（Bank 1）、50 MHz（R18） |

### 3. 電源・書き込み・UART
- USB-C J4 から給電（J47 1-2）。PC 本体のポートを使う。DC ジャック J7 を使うときは J47 を開放する。正常なら LED 5P0V / VDD / 1P8V が点灯する
- J49（7 セグの電圧）の既定は 5V（2-3）
- 書き込みはオンボードの eFP5 を使う。FlashPro Express で **New Job Project → .job を指定 → Programmer を確認（無ければ Refresh/Rescan）→ RUN → RUN PASSED**
- FT4232H の割り当て: **A = JTAG / B = MMUART_1（HSS）/ C = MMUART_4（U-Boot/Linux）/ D = ファブリック Bank 0（W21/Y21）**。115200 baud

### 4. 論理レベルとよく引くピン
- LED は Active High、SW1/SW2 は Active Low（10k プルアップ）、DIP SW8 は ON = 0
- LED1-7: T18 V17 U20 U21 AA18 V16 U15 ／ SW1 T19、SW2 U18 ／ SW8[1:8]: U17 Y16 R17 AA15 AA20 Y20 V21 AA19
- 50 MHz → R18（Bank 0）、125 MHz → P5/P4（Bank 5）、DEVRST_n = E8
- 上記以外のピンは [cheatsheet.md](cheatsheet.md) か schematic-*.md で引く

## Where to Look

| 知りたいこと | ファイル |
|---|---|
| ジャンパ・Bank 電圧・主要ピンの 1 枚表 | [cheatsheet.md](cheatsheet.md) |
| 作業手順（初回起動、Blinky、Click、MIPI、トラブル） | [patterns.md](patterns.md) |
| キット内容、電源 LED、電圧の測定点 | [ch01](chapters/ch01-getting-started.md) |
| ソフトのインストール、J45/J46/J47/J49、レギュレータ | [ch02](chapters/ch02-installation-settings.md) |
| DDR4・SD・Ethernet・UART・発振器・リセット（UG） | [ch03](chapters/ch03-board-components.md) |
| シルク図、FIR デモの概要 | [ch04](chapters/ch04-placement-demo.md) |
| FlashPro Express での書き込み | [ch05](chapters/ch05-programming-flashpro5.md) |
| UG のピン表と PDC の例 | [ch06](chapters/ch06-hardware-pinout.md) |
| UG の改訂履歴、サポート窓口 | [ch07](chapters/ch07-revision-support.md) |
| 電源ツリー、Bank の電源ボール、部品の型番、ジャンパの回路、クロック・リセット回路 | [schematic-power.md](references/schematic-power.md) |
| コネクタ（RPi / mikroBUS / MIPI / SD / 7 セグ / JTAG / Ethernet）のネット名とボール番号 | [schematic-connectors.md](references/schematic-connectors.md) |
| Bank 0/1 の全ピン表（PDC 用）、LED・スイッチの回路、DDR4 のピン | [schematic-pins.md](references/schematic-pins.md) |
| UG の未記載・推定項目の回路図での照合結果、要確認の項目 | [schematic-verification.md](references/schematic-verification.md) |
| 箱出しから Linux 起動まで、ブートの流れ、COM ポート、udev、ベアメタル | [embedded-software.md](references/embedded-software.md) |
| リファレンスデザインの Tcl 生成、引数、出力物、デザイン内容、eNVM | [reference-design.md](references/reference-design.md) |
| Libero の設計フロー（新規作成から書き込みまで） | [an5282-design-flow.md](references/an5282-design-flow.md) |
| FIR デモ（構成、GUI、書き込み） | [an5165-fir-demo.md](references/an5165-fir-demo.md) |
| QSG（同梱物、デモ用ジャンパ、ドライバ確認、ライセンス） | [quick-start.md](references/quick-start.md) |
| 資料一覧と URL、資料間の食い違い | [sources.md](references/sources.md) |
| 用語 | [glossary.md](glossary.md) |

## Scope

- カバーする資料: キットの UG、Rev2 回路図、QSG、AN5282、AN5165、GitHub のリファレンスデザインと組込みソフトの文書
- カバー外: ファブリックのデバイス仕様（DC 特性、タイミング）は PolarFire SoC Datasheet と PPAT。HSS・Yocto・ベアメタルドライバの詳細は各 GitHub リポジトリ（URL は sources.md）。PCB の .brd は未解析
- 動作温度はどの資料にも記載がない
- PDF はパスワード保護付きのため、図を確認するときは `pdftoppm` で画像にする
