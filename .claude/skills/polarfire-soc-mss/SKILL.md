---
name: polarfire-soc-mss
description: Device-level reference for the PolarFire SoC Microprocessor Subsystem (MSS), from the PolarFire SoC MSS Technical Reference Manual (DS60001702S Rev S). Use when writing firmware/HSS/Linux drivers or MSS Configurator settings for any PolarFire SoC board — E51/U54 cores, L2/LIM/scratchpad, AXI switch, FIC0-4 address windows, memory map (DDR cached/non-cached/WCB windows, peripheral base addresses), MPU/PMP, PLIC/CLINT interrupt IDs, MSS peripherals (MMUART, SPI, QSPI, I2C, CAN, GEM, USB, eMMC/SD, GPIO, timers, WDT), DDR controller, clocking/reset, boot modes, MSSIO.
when_to_use: PolarFire SoC MSS, MSS memory map, ベースアドレス, FIC0 FIC1 FIC2 FIC3 FIC4, PLIC 割り込み ID, CLINT mtime, E51 U54 hart, L2-LIM scratchpad, ITIM DTIM, PMP MPU, MMUART base address, GEM SGMII, QSPI XIP, eNVM, DDR cached non-cached WCB, boot mode 0 1 2 3, MSSIO Bank 2 Bank 4, SUBBLK_CLOCK_CR SOFT_RESET_CR, M2F interrupt
allowed-tools: Read Grep
argument-hint: [topic, peripheral, register, or address]
---

# PolarFire SoC MSS（デバイス）

出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（06/2026, 164 p）。原本 `C:\_Devs\PolarFire\doc\PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`（パスワード保護付き。図は `pdftoppm` で画像にして確認）。

## 使い方

- 下の Core で足りなければ、**Where to Look** から該当ファイルを 1 つだけ Read する
- 数値・アドレスは原文どおり。原文に無い補足には「(原文外)」が付いている
- ペリフェラルのレジスタ詳細は TRM に無く、別配布の PolarFire SoC Device Register Map（`pfsoc_regmap.htm`）にある。手元に無いので、レジスタのビット定義を聞かれたら「未収録」と答える
- ボード固有のこと（ピン、ジャンパ、COM ポート）は skill **polarfire-soc-discovery-kit**

## Core

- **CPU**: E51 ×1（RV64IMAC、モニタ用）+ U54 ×4（RV64GC、Sv39 MMU）、625 MHz。L2 は 2 MB（一部を LIM として使える）。eNVM 128 KB
- **FIC**: FIC0 / FIC1 = AXI4 64-bit・双方向（窓は 0x6000_0000 / 0xE000_0000、各 512 MB）、FIC2 = ファブリック → MSS の非キャッシュ DDR 専用、FIC3 = APB 32-bit・MSS → ファブリック（0x4000_0000、512 MB）、FIC4 = Crypto 専用。FIC のクロックが 125 MHz 以上なら DLL を有効にする
- **DDR の窓**: Cached 0x8000_0000（1 GB）/ Non-Cached 0xC000_0000（256 MB）/ Non-Cached WCB 0xD000_0000（256 MB）。64-bit 側は 0x10_0000_0000 からそれぞれ 16 GB
- **主要アドレス**: CLINT 0x0200_0000、L2-LIM 0x0800_0000、PLIC 0x0C00_0000、MMUART0 0x2000_0000、MMUART1-4 0x2010_0000 / _2000 / _4000 / _6000（HIGH 側は 0x28xx_xxxx）
- **割り込み**: コアごとにローカル割り込み 48 本、PLIC の外部割り込み 169 本。ファブリック → MSS が 64 本、MSS → ファブリックが 16 本。MMUART_n の割り込みは Hart n の Local 11 に直結
- **Discovery Kit（FCSG325 パッケージ）**: DDR の ECC は使えず、DDR のバス幅は 16-bit のみ

## Where to Look

| 知りたいこと | ファイル |
|---|---|
| MSS の機能一覧、E51/U54、ITIM/DTIM、分岐予測、DMA、WCB、BEU、L2/LIM/スクラッチパッド | [core-complex.md](references/core-complex.md) |
| AXI スイッチのポートと接続表、QoS、FIC0-4、MPU、Segmentation、AHB/APB | [interconnect.md](references/interconnect.md) |
| メモリマップ（コア空間、ペリフェラルの LOW/HIGH、DDR/FIC の窓、eNVM の区画） | [memory-map.md](references/memory-map.md) |
| 割り込み（mie/mcause/mtvec、PLIC の ID 表、ローカル割り込みの経路、CLINT、M2F） | [interrupts.md](references/interrupts.md) |
| ペリフェラル一覧（個数・機能・制約）、MSSIO の割り当て規則 | [peripherals-io.md](references/peripherals-io.md) |
| DDR コントローラ、クロック・リセット要因、ブートモード 0-3 | [ddr-clock-boot.md](references/ddr-clock-boot.md) |
| PMP・セキュアブート・Anti-tamper・Crypto、Debug/Trace、System Registers | [security-debug.md](references/security-debug.md) |
| トピックから原本の章・表・ページを引く | [page-index.md](references/page-index.md) |

## Errata（原文の不整合。回答時に注意）


| 箇所 | 内容 |
|---|---|
| Table 10-2 (p.152) | RTC / Timer の High Address が `0x2012_4FFF` / `0x2012_5FFF` と書かれている。他の行の規則と Table 3-49 からは、これは終端アドレスの誤記と思われる。(原文外) 規則どおりなら HIGH は 0x2812_4000 / 0x2812_5000 |
| Table 3-50 (p.52) | fabric マスタの 64-bit AXI を「FIC0 経由」と書いている。一方 Ch.6 は FIC2 を非キャッシュ DDR 専用としている。実際には FIC0 / FIC1 / FIC2 のどれからでも S7 に届く（Table 3-34） |
| Table 1-1 / Figure 1 | L2 LIM が 1.875 MB（表）と 1.8 MB（図）で食い違う |
| Table 5-16 / 5-22 (p.130–133) | PLIC の source 186 priority が `0x0C00_02D0` と書かれているが、`0x0C00_0000 + 4×ID` の式とは合わない。enable 6 のアドレスも `0x0C00_201C`（Table 5-22）と `0x0C00_2014`（Table 5-16）で食い違う。式と Table 5-16 を優先した |
| §3.11.7 | Functional Timing Diagrams は "To be updated" のまま |

