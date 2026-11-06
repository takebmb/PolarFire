# 資料一覧と公式 URL

> ローカル資料フォルダ: `C:\_Devs\PolarFire\doc\board\`
> この skill の各ファイルがどの資料に基づくかの対応表。文書番号・Rev はローカル原本の表記による（不明なものは "—"）。

---

## 1. doc/board/ 内の資料

| # | ローカルファイル | タイトル | 文書番号 / Rev | skill 内の参照先 | 公式 URL |
|---|---|---|---|---|---|
| 1 | `PolarFire_SoC_FPGA_Discovery_Kit_User_Guide.pdf` | PolarFire SoC FPGA Discovery Kit User Guide | DS50003630E (Rev E) | `SKILL.md`, `chapters/` ほか skill 本体 | https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/UserGuides/PolarFire_SoC_FPGA_Discovery_Kit_User_Guide.pdf |
| 2 | `PolarFire_SoC_Discovery_Kit_Quick_Start_Guide.pdf` | PolarFire SoC Discovery Kit Quickstart Guide | DS50003565J | [quick-start.md](quick-start.md) | https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ProductBrief/50003565_MPFS-Disco-Kit_QuickStart_Guide.pdf |
| 3 | `PolarFire_SoC_Discovery_Kit_Rev2_Schematics.pdf` | PolarFire SoC Discovery Kit Schematics（Part Number: MPFS-DISCO-KIT, Rev2） | SCH_090423_01 | [schematic-power.md](schematic-power.md) ほか schematic-*.md | https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_Discovery_Kit_Rev2_SCH_090423_01.pdf |
| 4 | `PolarFire_SoC_Discovery_Kit_Rev2_PCB_091923.brd` | PolarFire SoC Discovery Kit Board File（Cadence Allegro ボードファイル、約 10MB） | Rev2, PCB_091923 | **skill では未解析** | https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/BoardDesignFiles/PolarFire_SoC_Discovery_Kit_Rev2_23_0954_PCB_091923.brd |
| 5 | `../device/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`（`doc/device/`） | PolarFire SoC MSS Technical Reference Manual | DS60001702S | skill `polarfire-soc-mss`（[SKILL.md](../../polarfire-soc-mss/SKILL.md)） | https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ProductDocuments/ReferenceManuals/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual_VC.pdf |
| 6 | `AN5282_PolarFire_SoC_Design_Flow.pdf` | PolarFire SoC Design Flow (AN5282) | DS00005282B (Rev B, 07/2024) | [an5282-design-flow.md](an5282-design-flow.md) | https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ApplicationNotes/ApplicationNotes/PolarFire_SoC_Design_Flow_AN5282.pdf |
| 7 | `AN5165_PolarFire_SoC_DSP_FIR_Filter.pdf` | PolarFire SoC FPGA DSP FIR Filter Demo Application Note (AN5165) | DS00005165D (Rev D, 08/2026, Draft) | [an5165-fir-demo.md](an5165-fir-demo.md) | https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/ApplicationNotes/ApplicationNotes/PolarFire_SoC_FPGA_DSP_FIR_Filter_AN5165.pdf |
| 8 | `mpfs-discovery-kit-embedded-software-user-guide.md` | MPFS Discovery Kit Embedded Software User Guide | — (GitHub md) | [embedded-software.md](embedded-software.md) | https://github.com/polarfire-soc/polarfire-soc-documentation/blob/master/reference-designs-fpga-and-development-kits/mpfs-discovery-kit-embedded-software-user-guide.md |
| 9 | `polarfire-soc-discovery-kit-reference-design-README.md` | PolarFire SoC Discovery Kit Reference Design Generation Tcl Scripts - Libero SoC v2025.2+ | — (GitHub README) | [reference-design.md](reference-design.md)、[an5282-design-flow.md §9〜§11](an5282-design-flow.md) | https://github.com/polarfire-soc/polarfire-soc-discovery-kit-reference-design |

- AN5165 / AN5282 の PDF はパスワード保護（閲覧制限）付きで、Read ツールの pages 指定では開けない。図表の確認は `pdftoppm` で画像化して行った。

---

## 2. 未取得（参照のみ）

| 資料 | URL / 入手先 | 備考 |
|---|---|---|
| AN5165 デモ ZIP（GUI、スクリプト、プログラミングジョブ、約 244MB） | AN5165 Web ページ: `www.microchip.com/en-us/application-notes/an5165` | フォルダ名 `mpfs_an5165_v2024p1_df`（AN 本文より） |
| PolarFire SoC Bare Metal Examples | https://github.com/polarfire-soc/polarfire-soc-bare-metal-examples | — |
| Meta PolarFire SoC Yocto BSP | https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp | Linux イメージは releases |
| Discovery Kit 製品ページ | https://www.microchip.com/en-us/development-tool/MPFS-DISCO-KIT | QSG 表記は `www.Microchip.com/DiscoveryKit` |

---

## 3. 資料中に出てくるその他の URL

| 用途 | URL | 出典 |
|---|---|---|
| Updating MPFS Kit（キット更新手順） | https://mi-v-ecosystem.github.io/redirects/boards-mpfs-generic-updating-mpfs-kit | Embedded SW UG |
| リファレンスデザインリポジトリ（redirect） | https://mi-v-ecosystem.github.io/redirects/repo-discovery-kit-reference-design | Embedded SW UG |
| Yocto BSP releases（redirect） | https://mi-v-ecosystem.github.io/redirects/releases-meta-polarfire-soc-yocto-bsp | Embedded SW UG |
| MPFS Discovery Kit User Guide（redirect、Embedded SW UG を指す） | https://mi-v-ecosystem.github.io/redirects/boards-mpfs-discovery-kit-user-guide | README |
| Bare Metal Examples（redirect） | https://mi-v-ecosystem.github.io/redirects/repo-polarfire-soc-bare-metal-examples | README |
| AXI4 Stream デモガイド | https://mi-v-ecosystem.github.io/redirects/demo-guides_mpfs-axi4-stream-demo | README |
| QSG オンライン版 | https://onlinedocs.microchip.com/v2/literature/DS50003565?version=latest&redirect=true | Embedded SW UG |
| リファレンスデザイン Tcl（タグ 2024.04） | https://github.com/polarfire-soc/polarfire-soc-discovery-kit-reference-design/releases/tag/2024.04 | QSG |
| Libero SoC ダウンロード（v2025.2+） | https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/fpga/libero-software-later-versions#Documents%20and%20Downloads | README |
| Libero ライセンス情報 | https://www.microchip.com/en-us/products/fpgas-and-plds/fpga-and-soc-design-tools/fpga/licensing | README |
| ライセンス取得・設定クイックスタート | https://coredocs.s3.amazonaws.com/Libero/2023_1/Tool/libero_download_license_quickstart.pdf | QSG |
| Mi-V RV32 構成の説明 | https://github.com/Mi-V-Soft-RISC-V/PolarFire-Eval-Kit/tree/main/Libero_Projects#pf_eval_kit_miv_rv32_basedesign | README |
| AN5165 ページ | https://www.microchip.com/en-us/application-notes/an5165.html | README |
| Windows 用 wget 導入ガイド | https://www.addictivetips.com/windows-tips/install-and-use-wget-in-windows-10/ | README |
| PolarFire SoC ナレッジベース | https://microchip.my.site.com/s/article/KB-of-PFSoC-Knowledge-Base-articles | QSG |
| FTDI ドライバ | `FTDIChip.com/Drivers` | QSG |
| Libero / Mi-V / サポート | `www.microchip.com/libero`, `www.microchip.com/mi-v`, `www.microchip.com/support` | QSG |

---

## 4. 資料間で食い違う点（作業時の注意）

| 項目 | 食い違い | 詳細 |
|---|---|---|
| J49 設定 | QSG / AN5165: 2–3（5V）、AN5282 / Embedded SW UG: 1–2（3.3V） | [quick-start.md §3.2](quick-start.md) |
| J45 / J46 の説明 | Embedded SW UG は両方 "Bank 1 voltage"、AN5165 / AN5282 は J46 = VDDAUX1 | 同上 |
| Libero バージョン | AN5282: v2023.2、QSG・AN5165: v2024.1、README: v2025.2+ | [an5282-design-flow.md §13](an5282-design-flow.md) |
| 基板 Rev | AN5282: REV 1、AN5165: REV 2（回路図・.brd は Rev2） | — |
| ボーレート | Embedded SW UG: "115220 baud"（115200 の誤記と思われる (原文外)） | [embedded-software.md §3](embedded-software.md) |
| 内蔵プログラマ表記 | README: "embedded FlashPro6"、QSG / udev: FP5 / "Embedded FlashPro5" | [an5282-design-flow.md §10.3](an5282-design-flow.md) |

### 4.1 回路図で確認した追加の食い違い

| 項目 | 食い違い | 詳細 |
|---|---|---|
| J49 既定 | 回路図・UG: 2–3（5V）。Embedded SW UG のみ 1–2 | [schematic-verification.md](schematic-verification.md) |
| MIC22405 電流 | UG Table 2-5 の説明は 3A、回路図 Sheet 14 は 2 個とも 4A（UG の列ずれと思われる） | 同上 |
| RPi コネクタ品番 | UG: 61204021621、回路図: 61304021121 | 同上 |
| J12 ネット名 | ネット名は GPIO52、FPGA パッドは GPIO4 | 同上 |
| Ethernet の VDDI1_5 | UG・回路図の注記は 2.5V（J45/J46 = 2-3）。3.3V 既定で動くかは資料から判断不可 | 同上 |

### 4.2 MSS TRM 自体の不整合

RTC/Timer の HIGH アドレス、ファブリック→DDR 経路（FIC0 / FIC2）、LIM サイズ、PLIC アドレス → skill `polarfire-soc-mss` の SKILL.md「Errata」
