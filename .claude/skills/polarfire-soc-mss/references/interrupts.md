# MSS: Interrupts (Local / PLIC / CLINT / M2F)

> 出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（`C:/_Devs/PolarFire/doc/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`）。「p.NN」は PDF ページ番号。原文に無い補足は「(原文外)」。原文の不整合は SKILL.md の Errata を参照。

## 8. Interrupts (Ch.5, p.118–135)

### 8.1 全体像

- Local: ペリフェラルから 48 本が各コアの Local Interrupt 0–47 に直結している。CLINT の Software / Timer も local。Local は Global より速い。
- Global: ペリフェラル 169 本 + Core Complex 内部（DMA 8、L2 4、BEU 5）16 本 → PLIC（**186 sources、priority 7 段階**）。
- **PLIC ID = Global_int + 13**（ID 0 = 割り込みなし。ID 1–12 は L2 と DMA）。Global 割り込みは正論理のレベルトリガ。
- 優先順位（高い順）: Local 47 … Local 0 > M Global > M Software > M Timer > S Global > S Software > S Timer。Local 47 はベクタテーブルの最後のスロットなので、ハンドラを jump なしで直接置ける。最も重要な割り込みを割り当てるとよい。
- レイテンシ: 4 cycle。PLIC 経由はさらに 3 cycle（PLIC は user_clock で動く）。ハンドラがキャッシュ済みか ITIM 上にあるとき、Global は **4 + 3 × (core clock / user_clock)** cycle。
- Local 割り込みは S-mode に委譲できない。`mideleg` で委譲できるのは bit1 SSI、bit5 STI、bit9 SEI。

### 8.2 CSR（M-mode）

| CSR | 要点 |
|---|---|
| `mie` | bit3 MSIE、bit7 MTIE、bit11 MEIE、bit1 SSIE、bit5 STIE、bit9 SEIE、**bit16+n = LIEn（Local n、n=0..47）** |
| `mcause` | bit63 = 1 なら割り込み。3 = M software、7 = M timer、11 = M Global（external）、1/5/9 = S 版、**16+n = Local n**（16–63）。例外コード: 0–7, 8/9/11 = ecall U/S/M, 12/13/15 = page fault |
| `mtvec` | MODE [1:0]: 0 = Direct、1 = Vectored（PC = BASE + 4 × cause）。Vectored のときは **BASE を 256 byte 境界に置く**。M timer → BASE+0x1C、Global → BASE+0x2C |

### 8.3 PLIC Interrupt ID（Table 5-17 / 5-1）

| PLIC ID | Source | M2F-Int | Local 接続 |
|---|---|---|---|
| 1 / 2 / 3 / 4 | L2: metadata 訂正 / metadata 訂正不能 / data 訂正 / data 訂正不能 | — | — |
| 5–12 | DMA Ch0 Done, Ch0 Error, Ch1 Done, … Ch3 Error | — | — |
| 13–26 | gpio0/2 [13:0] | 0 | — |
| 27–50 | gpio1/2 [37:14] | 0 | — |
| 51 / 52 / 53 | gpio0 / gpio1 / gpio2 `_non_direct` | 0 | — |
| 54 / 55 | spi0 / spi1 | 1 | — |
| 56 / 57 | can0 / can1 | 1 | — |
| 58–60 | i2c0_main / alert / sus | 2 | — |
| 61–63 | i2c1_main / alert / sus | 2 | — |
| 64–69 | mac0_int, queue1–3, emac, mmsl | 3 | Hart1 / Hart2 Local 8,7,6,5,4,3 |
| 70–75 | mac1_int, queue1–3, emac, mmsl | 4 | Hart3 / Hart4 Local 8,7,6,5,4,3 |
| 76 | ddrc_train | 9 | — |
| 77 | scb_interrupt | 7 | Hart0 L15 |
| 78 / 79 | peripheral_ecc_error / peripheral_ecc_correct | 6 | Hart0 L14 / L13 |
| 80 / 81 | rtc_wakeup / rtc_match | 11 | — |
| 82 / 83 | timer1 / timer2 | 12 | — |
| 84 | envm | 13 | Hart0 L12 |
| 85 | qspi | 13 | — |
| 86 / 87 | usb_dma / usb_mc | 14 | — |
| 88 / 89 | mmc_main / mmc_wakeup | 15 | — |
| 90–94 | mmuart0–4 | 1 | mmuartN → HartN の L11（mmuart0 は Hart0） |
| 95 / 96 | g5c_devrst / g5c_message | 10 / 8 | Hart0 L4 / L3 |
| 97 / 98 | usoc_vc_interrupt / usoc_smb_interrupt | 11 | Hart0 L2 / L1 |
| 99 | pll_event, mpu_fail, decode_error, lp_state_enter/exit, ff_start/end, fpga_on/off, scb_error/fault, mesh_fail, io_bank_b2/b4/b5/b6_on/off（共通 1 本） | 6 | Hart0 L0 |
| 100–104 | wdog0–4_mvrp | 5 | wdogN → HartN の L10 |
| 105–109 | wdog0–4_tout | 5 | HartN の L9、さらに Hart0 の L9(wdog0)・L8–L5(wdog1–4) |
| 110 | g5c_mss_spi | 13 | — |
| 111 | volt_temp_alarm | No | — |
| 112–114 | athena_complete / alarm / buserror | No | — |
| 115 / 116 | usoc_axic_us / usoc_axic_ds | 11 | — |
| 117 | reserved/spare | — | — |
| 118–149 | **MSS_INT_F2M[31:0]**（fabric → MSS） | — | Hart1–4 Local 16–47（U54-Mask = MASKED） |
| 150–181 | **MSS_INT_F2M[63:32]** | — | **Hart0 (E51) Local 16–47** |
| 182–186 | BEU Hart0–4 | — | （BEU local は cause 128） |

- F2M のローカル割り込みを U54 で使うには `SYSREG->FAB_INTEN_U54_n` を設定する（例: `SYSREG->FAB_INTEN_U54_1 = 0xffffffff;`）。E51 では MSS_INT_F2M[63:32] がデフォルトで有効（p.122）。
- mac* と MSS_INT_F2M[31:0] の行は、Table 5-1 の U54-Mask 列が "MASKED" になっている。列の意味は原文に説明がない。

### 8.4 PLIC Register Map (base 0x0C00_0000)

| Address | 内容 |
|---|---|
| 0x0C00_0000 + 4 × ID | Source priority（[2:0]、0 = 無効、1 = 最低 … 7 = 最高。同じ優先度なら ID の小さい方が勝つ） |
| 0x0C00_1000–0x0C00_1014 | Pending array（6 word。ID N は word N/32 の bit N mod 32） |
| 0x0C00_2000 | Hart0 M-mode enables（6 word） |
| 0x0C00_2080 / 0x0C00_2100 | Hart1 M / S enables |
| 0x0C00_2180 / 0x0C00_2200 | Hart2 M / S enables |
| 0x0C00_2280 / 0x0C00_2300 | Hart3 M / S enables |
| 0x0C00_2380 / 0x0C00_2400 | Hart4 M / S enables |
| 0x0C20_0000 / +4 | Hart0 M threshold / claim-complete |
| 0x0C20_1000 / 0x0C20_2000 | Hart1 M / S threshold（各 +4 が claim/complete） |
| 0x0C20_3000 / 0x0C20_4000 | Hart2 M / S |
| 0x0C20_5000 / 0x0C20_6000 | Hart3 M / S |
| 0x0C20_7000 / 0x0C20_8000 | Hart4 M / S |

- Threshold: その値以下の priority をマスクする（0 = すべて許可、7 = すべてマスク）。
- Claim: 読むと最も優先度の高い pending ID が返り、その pending がクリアされる（0 = なし）。Complete: 処理の最後に同じ ID を書き戻す。
- E51 (Hart0) には S-mode のコンテキストがない。(原文外) コンテキスト番号は 0 = H0M、1 = H1M、2 = H1S … 8 = H4S の順に並ぶ。

### 8.5 CLINT (base 0x0200_0000)

| Address | Register |
|---|---|
| 0x0200_0000 + 4 × hart | `msip` Hart0–4（bit0 だけ有効。IPI に使う） |
| 0x0200_4000 + 8 × hart | `mtimecmp` Hart0–4（reset しない） |
| 0x0200_BFF8 | `mtime`（64-bit、**1 MHz = 1 µs ごとに増える**、reset で 0） |

デフォルトでは timer と software 割り込みはすべて M-mode に trap する。S-mode で受けるには委譲が必要。

### 8.6 M2F Interrupt Controller (§3.12.16, p.115–116)

- MSS 側の 43 本の割り込みを、**MSS_INT_M2F[15:0]**（MSS → fabric、16 本、正論理レベル）にまとめる。どの番号にまとまるかは §8.3 の M2F-Int 列。
- fabric から MSS へは **MSS_INT_F2M[63:0]**（64 本、正論理レベルトリガ）。アサート時間は PLIC のクロック周期より長くする。ペリフェラル側は処理されるまでアサートを保持する。
- レジスタは M2FINT（0x2012_6000）。
