# MSS: Peripherals / MSS I/O

> 出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（`C:/_Devs/PolarFire/doc/device/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`）。「p.NN」は PDF ページ番号。原文に無い補足は「(原文外)」。原文の不整合は SKILL.md の Errata を参照。

## 9. Peripherals (§3.12, p.61–116)

全ペリフェラル共通: reset 状態から `SOFT_RESET_CR`（PFSOC_MSS_TOP_SYSREG）で個別に解除する。ドライバ例は PolarFire SoC GitHub にある。

| Peripheral | 数 | バス | 主要機能 | 制約・注意 | p. |
|---|---|---|---|---|---|
| GEM (Ethernet MAC) | 2 | AXI（データ, M5/M6）+ APB（設定） | 10/100/1000、IEEE 802.1AS/Qav/Qbv/CB/Qci、802.3br、802.1Qbb、VLAN、TCP/IP offload、1000BASE-X PCS、jumbo 10,240 B、WoL、IEEE 1588 TSU、Qav の credit-based shaping は上位 2 キュー（スケジューリングは Fixed / DWRR / ETS から選択） | SGMII は **I/O Bank 5** 専用。MII/GMII は fabric にしかつながらない。MDIO は MSSIO か fabric、MDC は 2.5 MHz 以下。TX 10/100/1000 = 1.25/12.5/125 MHz (SGMII)、tsu_clk 5–400 MHz | 63–81 |
| CAN | 2 | APB (CDC 付き) | CAN 2.0B、ISO 11898-1、RX buffer 32（それぞれフィルタ付き）、TX 32、RTR 自動応答、SST、Listen-only / Loopback / SRAM test | 8 MHz CAN clock で最大 1 Mbps。CAN clock は MSS PLL から 8–80 MHz（8 MHz 刻み）。**内部 SRAM は未初期化だと SECDED エラーになるので、SRAM test mode で初期化する** | 82–90 |
| eNVM | 128 KB | AHB | SECDED、C-Bus 32-bit（書き込み・設定）、R-Bus 64-bit（読み出し） | **CPU からは書けない**（JTAG でのデバイス書き込み時に書く）。eNVM clock は最大 26.3 MHz、80 ns アクセス。保持期間: 110 °C で 10 年 / 書き換え 10000 回、125 °C で 4 年 / 1000 回 | 90–91 |
| QSPI-XIP | 1 | AHB | Master のみ、SCK = HCLK/2, /4, /6、TX FIFO 16 B / RX FIFO 32 B、x1/x2/x4、3 または 4 byte アドレス、XIP、Direct mode | reset 時は XIP 無効。XIP 中はレジスタを読めない（write は可）。XIP の 1 read は約 80 clock（SPI = AHB/4 のとき）。XIP を抜けるときは QSPI 上で実行していないこと | 91–94 |
| MMUART | 5 | APB | 非同期/同期、5–8 bit、parity、1/1.5/2 stop、9-bit address、TX/RX FIFO 16 B、半二重 single-wire、LIN、ISO 7816、分数ボーレート、IrDA (RZI)、MSB/LSB first | MMUART_n の割り込みは Hart n の Local 11 に直結 | 94–95 |
| SPI | 2 | APB | Master/Slave、Motorola / TI SSP / MICROWIRE、FIFO 4×32 | SCK は APB clock から生成。信号は fabric にも出せる | 95–97 |
| I2C | 2 | APB | I2C v2.1、SMBus v2.0、PMBus v1.1、100 / 400 kbit/s、7-bit アドレス、multi-master、second slave address、SMBus timeout | グリッチフィルタは 3–21 APB clock。fabric I/O に出すときは BIBUF で双方向にする | 97–99 |
| GPIO | 3 | APB | 各 32 bit、入力/出力/双方向、edge (rise/fall/both) と level の割り込み | GPIO_0 は MSIO 最大 14 本、GPIO_1 は MSIO 最大 24 本、**GPIO_2 は fabric 経由で 32 本**。edge 割り込みは 1 を書いてクリア | 99–100 |
| RTC | 1 | APB | Calendar mode / Binary mode（43-bit, 0..2^43−1）、alarm（RTC_MATCH / RTC_WAKEUP） | Calendar では 26-bit prescaler を 1 Hz になるよう設定する。精度は基準クロック次第（例: 124.988 MHz で 1 日約 8.1 秒遅れる） | 100–102 |
| Timer | 1（32-bit ×2） | APB | 減算カウンタ、One-shot / Periodic、2 本を連結して 64-bit にできる | TIMER1INT / TIMER2INT（PLIC 82/83） | 102–103 |
| Watchdog | 5 | APB | 32-bit 減算、Debug 中は停止、WFI からの wake-up 割り込み | **MSS を reset できるのは watchdog_0 (E51) だけ**。WDT1–4 は E51 への割り込みのみ。カウンタがトリガ値を越えると non-maskable の timeout 割り込み、そのまま 0 まで行くと reset（PLIC には wdogN_mvrp と wdogN_tout の 2 本がある）。reset 解除直後は無効（電源投入時のタイムアウトは約 10.47 s @VDD=1.2 V） | 103–104 |
| USB OTG | 1 | AHB master/slave | USB 2.0 HS (480 Mbps) / FS (12 Mbps)、Host / Device / OTG (SRP / HNP)、DMA 4 ch、TX/RX EP 各最大 5（EP0 含む）、8 KB RAM (SECDED)、LPM | 外付け PHY を **ULPI** で接続（MSSIO 固定）。fabric I/O にはつなげない | 104–107 |
| eMMC / SD / SDIO | 1 | AXI (M8) | eMMC: DS / HS / HS DDR / HS200 / HS400 / HS400ES。SD: DS〜UHS-I SDR104 / DDR50。SDMA / ADMA2（32 / 64-bit） | **UHS-II 非対応**。**fabric にはつなげない**（MSSIO 固定）。eMMC と SD は排他 | 107–112 |
| FRQ Meter | 1 | APB | 最大 8 counter × 8 input、Frequency mode / Time mode、最大 625 MHz | 測定対象: clk_cpu, clk_axi, clk_ahb, clk_reference, clk_dfi, MAC, SGMII PLL, CAN, USB, eMMC など（Table 3-77）。基準は JTAG TCK（default）か MSS REFCLK。ドライバは `mpfs_hal/common/nwc/mss_cfm.c` | 112–115 |
| M2F INT Ctrl | 1 | APB | §8.6 参照 | — | 115–116 |
| Core Complex DMA | 4 ch | TileLink | §2.4 参照 | — | 23–25 |
| User Crypto | 1 | AXI S4 / M4、FIC4 | Athena F5200 TeraFire, 200 MHz | 詳細は PolarFire Family FPGA Security User Guide | 50 |

---

## 13. MSS I/O (§3.9, p.49–50)

- MSS I/O は **38 本**。これとは別に SGMII I/O（Bank 5）、外部基準クロック 2 本、DDR I/O（Bank 6）がある。どのパッケージでも全ピンがボンディングされている。
- MSS I/O は **Bank 2 と Bank 4** の 2 つの電圧バンクに分かれる。
- MSSIO に出せるペリフェラル: eMMC/SD/SDIO、USB、QSPI-XIP、CAN ×2、UART ×5、SPI ×2、I2C ×2、MSS GPIO。どのペリフェラルにどのピンを割り当てるかは system register（IOMUX）で選ぶ。

| ルール | 内容 |
|---|---|
| 固定ペリフェラル | **USB、eMMC、SD/SDIO は決まった MSSIO にしか出せず、fabric I/O にはつなげない** |
| eMMC と SD | 排他。Bank 2 と Bank 4 のどちらにも置ける |
| その他 | I/O MUX を通して複数の MSSIO 候補、または fabric I/O に出せる |
| 2 つのバンクが同じ I/O 規格 | 1 つのペリフェラルを Bank 2 / Bank 4 / fabric にまたがって割り当てられる |
| 異なる I/O 規格 | 1 つのペリフェラルは 1 つのバンクにまとめるか、fabric I/O に出す |
| 3 種類以上の I/O 規格が必要 | fabric I/O に出す |
| 外部 Flash | MSSIO に直接つなげる（QSPI Example 1/2） |

- GEM の SGMII は Bank 5、MDIO は MSSIO か fabric。USB の ULPI は MSIO。
- (原文外) Discovery Kit で実際にどのピンを使っているかは、skill 本体のピン割り当て資料を参照。
