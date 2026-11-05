# 組込みソフトウェア（HSS / Linux / ベアメタル）リファレンス

> 主出典: *MPFS Discovery Kit Embedded Software User Guide*（GitHub `polarfire-soc/polarfire-soc-documentation`、文書番号・Rev なし）
> ローカル: `C:\_Devs\PolarFire\doc\mpfs-discovery-kit-embedded-software-user-guide.md`
> 副出典: リファレンスデザイン README（`C:\_Devs\PolarFire\doc\polarfire-soc-discovery-kit-reference-design-README.md`）、QSG DS50003565J
>
> 注: Embedded SW UG 本文は短く、Yocto のビルド手順・`.wic` の書込み手順・HSS の詳細は**記載がない**。それらは外部文書（Updating MPFS Kit、Yocto BSP リポジトリ）への参照のみ。本書で補った部分は (原文外) と明記する。

---

## 1. 概要

- 搭載デバイス: **MPFS095T** PolarFire SoC（RISC-V 5 コア MSS + PolarFire FPGA ファブリック）
- 設計の分担: Libero SoC = RTL デザイン（MSS / ファブリック）、SoftConsole IDE = C/C++ アプリ
- 箱出し状態: **FIR フィルタデモが書き込み済み**（→ [an5165-fir-demo.md](an5165-fir-demo.md)）。**Linux を起動するにはリファレンスデザインへの書き換えが必要**

---

## 2. Linux 起動用の既定ジャンパ（Default Jumper Settings）

| Jumper | Setting | Description（原文どおり） |
|---|---|---|
| J45 | 1 & 2 | Bank 1 voltage. 1 & 2: 3.3V, 2 & 3: 2.5v |
| J46 | 1 & 2 | Bank 1 voltage. 1 & 2: 3.3V, 2 & 3: 2.5v |
| J47 | Closed | Power supply selection, closed: USB C power, open: jack power |
| J49 | 1 & 2 | Reference power for 7 segment display connector. 1 & 2: 3.3V, 2 & 3: 5V |

- FIR デモ（QSG / AN5165）では J49 = 2–3。資料間の比較は [quick-start.md §3.2](quick-start.md)。
- J46 の説明は原文で J45 と同文（AN5165 / AN5282 では J46 = VDDAUX1）。

---

## 3. Linux を起動するまでの手順（箱出し → Linux）

- [ ] 1. **FPGA を Linux 対応ビットストリームに書き換える**
  - リファレンスデザインリポジトリの **Releases** にある FlashPro Express ファイル（.job）を FlashPro Express で書き込む（Libero 不要）
  - 手順の詳細: 外部文書 "Updating MPFS Kit"（`https://mi-v-ecosystem.github.io/redirects/boards-mpfs-generic-updating-mpfs-kit`）
  - FlashPro Express の操作自体は AN5165 と同じ（New → .job 選択 → RUN → RUN PASSED。[an5165-fir-demo.md §6.3](an5165-fir-demo.md)）
  - 自分で生成する場合は Tcl 引数 `HSS_UPDATE` 付きで生成すると HSS が eNVM に入る（[reference-design.md](reference-design.md)）
- [ ] 2. **SD カードに Linux イメージを書く**
  - イメージ取得元: Meta PolarFire SoC Yocto BSP の **releases**（`https://mi-v-ecosystem.github.io/redirects/releases-meta-polarfire-soc-yocto-bsp`）
  - `.wic` 形式のイメージを SD カードにそのまま書き込むのが Yocto BSP の一般的な手順 (原文外)。例（Linux ホスト、デバイス名は必ず確認）:
    ```
    # (原文外) 例: 解凍後の .wic を SD カード全体（パーティションではなく）に書く
    sudo dd if=<image>.wic of=/dev/sdX bs=4M conv=fsync status=progress
    ```
    Windows では balenaEtcher 等のイメージ書込みツールを使う (原文外)。正式手順は Updating MPFS Kit / Yocto BSP の README に従うこと。
  - QSG: SD カードは **SanDisk 製推奨**
- [ ] 3. SD カードを microSD スロットに挿入（README: SD/SDIO は MSS I/Os BANK 4 → uSD card socket）
- [ ] 4. ジャンパを §2 に設定
- [ ] 5. USB C コネクタ **J4** をホストに接続
- [ ] 6. ターミナルソフト（PuTTY, ExtraPutty, minicom, screen など）で USB C 経由の **3 つの UART** を開く
- [ ] 7. シリアル設定: **115220 baud, no flow control, no parity**（原文どおり。一般には 115200 の誤記と考えられる (原文外)）
- [ ] 8. 電源を入れ直す → 2 つのシリアルポートに起動メッセージ
- [ ] 9. 2 番目の COM ポートで Linux にログイン: ユーザー **`root`**、パスワードなし。途中でパスワードを求められたら **`microchip`**

---

## 4. ブートの流れと UART への出力

```
電源投入 → HSS（eNVM, MMUART1 に出力） → U-Boot → Linux（MMUART4 に出力・ログインプロンプト）
```

（HSS → U-Boot → Linux の順序は UART 出力の記述から整理したもの。HSS が eNVM の boot mode 1 クライアントとして動作する点は README §Boot mode 1 による）

| COM ポート（ホスト側の並び） | MSS UART | 表示内容 |
|---|---|---|
| 1st COM port | **MMUART1** | **HSS（Hart Software Services）** のブートメッセージ |
| 2nd COM port | **MMUART4** | **U-Boot**、**Linux** ブートメッセージ、Linux プロンプト |
| 3 つ目 | （Embedded SW UG では未記載） | README の割当からは **MMUART0（Fabric 経由）** と推定 (原文外) |

---

## 5. UART / COM ポート割り当て

### 5.1 リファレンスデザインでの MSS UART の配線（README "MSS peripherals"）

| MSS Peripheral | Route to Device I/O | Board peripheral | Board Interface | 用途 |
|---|---|---|---|---|
| MMUART0 | Fabric | USB-UART | J4 (USB-C port) | （用途記載なし） |
| MMUART1 | MSS I/Os BANK 2 | USB-UART | J4 (USB-C port) | HSS コンソール |
| MMUART4 | MSS I/Os BANK 2 | USB-UART | J4 (USB-C port) | U-Boot / Linux コンソール |

### 5.2 USB 側インターフェース（udev ルールと QSG から）

USB ブリッジは "Embedded FlashPro5"（VID `1514` / PID `2008`）として見え、4 インターフェースを持つ。

| bInterfaceNumber | チャネル | 用途 | Linux の補助 symlink（任意ルール） |
|---|---|---|---|
| `00` | A | **JTAG**（ftdi_sio からアンバインド） | — |
| `01` | B | シリアル | `/dev/ttyUSB-FlashPro5B` |
| `02` | C | シリアル | `/dev/ttyUSB-FlashPro5C` |
| `03` | D | シリアル | `/dev/ttyUSB-FlashPro5D` |

- Windows では "USB FP5 Serial Converter D" の Location を持つ COM ポートが存在すればドライバ正常（QSG。FIR デモはこのポートを使う流れ）。
- B/C/D と MMUART0/1/4 の厳密な対応は doc/ 内の資料に明記がない。「1st COM = MMUART1, 2nd COM = MMUART4」の記述から B = MMUART1, C = MMUART4, D = MMUART0（ファブリック経由、FIR デモでは CoreUART）と推定される (原文外)。回路図の FT4232H Connection ページで確認済み（次行）。
- **回路図での確認結果**: FT4232H の A = JTAG（eFP5）、B = MMUART_1（Bank 2 F2/F4）、C = MMUART_4（Bank 2 G2/H4）、D = ファブリック Bank 0（W21/Y21、U3 レベルシフタ経由）。README の「MMUART0 = Fabric 経由」は、リファレンスデザインで MMUART0 をファブリック経由で D チャネルへ出す構成と整合する（ピン直結ではない） → [schematic-verification.md](schematic-verification.md)

---

## 6. 更新手順（Updating）

| 更新対象 | 入手先 | 備考 |
|---|---|---|
| 手順全体 | Updating MPFS Kit: `https://mi-v-ecosystem.github.io/redirects/boards-mpfs-generic-updating-mpfs-kit` | 最新リファレンスデザインと Linux イメージへの更新手順 |
| FPGA（+ HSS）ジョブファイル | Discovery Kit Reference Design の releases: `https://mi-v-ecosystem.github.io/redirects/repo-discovery-kit-reference-design` | 事前生成済み FlashPro Express ジョブ |
| Linux イメージ | Meta PolarFire SoC Yocto BSP の releases: `https://mi-v-ecosystem.github.io/redirects/releases-meta-polarfire-soc-yocto-bsp` | — |
| eMMC / SD 向けイメージとジャンパ | MPFS Discovery Kit User Guide: `https://mi-v-ecosystem.github.io/redirects/boards-mpfs-discovery-kit-user-guide`（README "Board configuration" より。この Embedded SW UG 自身を指す） | — |

推奨順序 (原文外): FPGA ジョブ（HSS 含む）とLinux イメージは同じリリース世代のものを組み合わせる。

---

## 7. Linux ホストから MSS UART に接続する（udev ルール）

Linux ホストでは、FTDI USB-to-UART ブリッジを認識させるため udev ルールを追加する。**設定しないと COM ポートが現れないことがある。**

- [ ] 1. `/etc/udev/rules.d/70-microchip.rules` をエディタ（vim, nano など）で作成し、以下を記述（原文どおり）:

```console
# Bind ftdi_sio driver to all input
ACTION=="add", ATTRS{idVendor}=="1514", ATTRS{idProduct}=="2008", \
ATTRS{product}=="Embedded FlashPro5", ATTR{bInterfaceNumber}!="00", \
RUN+="/sbin/modprobe ftdi_sio", RUN+="/bin/sh -c 'echo 1514 2008 > /sys/bus/usb-serial/drivers/ftdi_sio/new_id'"

# Unbind ftdi_sio driver for channel A which should be the JTAG
SUBSYSTEM=="usb", DRIVER=="ftdi_sio", ATTR{bInterfaceNumber}=="00", ATTR{interface}=="Embedded FlashPro5",\
RUN+="/bin/sh -c 'echo $kernel > /sys/bus/usb/drivers/ftdi_sio/unbind'"

# Helper (optional)
KERNEL=="ttyUSB[0-9]*", SUBSYSTEM=="tty", SUBSYSTEMS=="usb", \
ATTRS{interface}=="Embedded FlashPro5", ATTRS{bInterfaceNumber}=="01", \
SYMLINK+="ttyUSB-FlashPro5B" GROUP="dialout" MODE="0666"

KERNEL=="ttyUSB[0-9]*", SUBSYSTEM=="tty", SUBSYSTEMS=="usb", \
ATTRS{interface}=="Embedded FlashPro5", ATTRS{bInterfaceNumber}=="02", \
SYMLINK+="ttyUSB-FlashPro5C" GROUP="dialout" MODE="0666"

KERNEL=="ttyUSB[0-9]*", SUBSYSTEM=="tty", SUBSYSTEMS=="usb", \
ATTRS{interface}=="Embedded FlashPro5", ATTRS{bInterfaceNumber}=="03", \
SYMLINK+="ttyUSB-FlashPro5D" GROUP="dialout" MODE="0666"
```

- [ ] 2. ルールを反映 (原文外): `sudo udevadm control --reload-rules && sudo udevadm trigger`、またはケーブルを挿し直す
- [ ] 3. 確認 (原文外): `ls -l /dev/ttyUSB*`

| ルール | 意味 |
|---|---|
| Bind | VID 1514 / PID 2008 / product "Embedded FlashPro5" の **インターフェース 00 以外**に `ftdi_sio` をロード・ID 登録 |
| Unbind | **インターフェース 00（チャネル A = JTAG）** から `ftdi_sio` を外す（書き込み用に確保） |
| Helper (optional) | インターフェース 01/02/03 に `ttyUSB-FlashPro5B/C/D` の固定名、`dialout` グループ、`0666` を付与 |

---

## 8. ベアメタル開発（README / QSG より）

| 項目 | 内容 |
|---|---|
| サンプル | PolarFire SoC Bare Metal Examples: `https://github.com/polarfire-soc/polarfire-soc-bare-metal-examples`（README 内では `https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-bare-metal-examples`） |
| 必要なハード構成 | 標準リファレンスデザインでベアメタル例と Linux の両方が動く。I2C ループバック等は `I2C_LOOPBACK` 引数のデザインが必要（[reference-design.md](reference-design.md)） |
| MPFS HAL の構成 | リファレンス XML（リポジトリの `XML` フォルダ、または `script_support/components/[target]` に生成される XML）を使って MPFS HAL を構成しビルド |
| 開発環境 | SoftConsole（Windows / GNU/Linux、無償） |
| デバッグ実行 | Boot mode 0（全コア WFI）に設定してデバッガで起動。SoftConsole v6.5 以降 |
| スタンドアロン起動 | Boot mode 1 でアプリを eNVM クライアントとして書込み。SoftConsole v6.4 以上 / Libero SoC v12.6 以上 |
| DDR | 1GB DDR4 は MSS DDR コントローラ接続。使用前に HSS またはベアメタルアプリで DDR トレーニングが必要（ファブリックから使うなら FIC ターゲットも有効化） |

ブートモード設定・eNVM 書込みの手順チェックリストは [an5282-design-flow.md §10](an5282-design-flow.md) を参照。

---

## 9. 参照

| 資料 | URL |
|---|---|
| PolarFire SoC Discovery Kit Quick Start Guide | `https://onlinedocs.microchip.com/v2/literature/DS50003565?version=latest&redirect=true` |
| PolarFire SoC Discovery Kit Schematics | `https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_Discovery_Kit_Rev2_SCH_090423_01.pdf` |
| PolarFire SoC Discovery Kit Product Page | `https://www.microchip.com/en-us/development-tool/mpfs-disco-kit` |
| Discovery Kit Reference Design Repository | `https://github.com/polarfire-soc/polarfire-soc-discovery-kit-reference-design` |
| Yocto BSP | `https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp` |
