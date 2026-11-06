# MSS: Memory Map

> 出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（`C:/_Devs/PolarFire/doc/device/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`）。「p.NN」は PDF ページ番号。原文に無い補足は「(原文外)」。原文の不整合は SKILL.md の Errata を参照。

## 6. Memory Map (Ch.10, p.151–153)

### 6.1 CPU Core Complex Address Space (Table 10-1)

| Start | End | Attr | Description |
|---|---|---|---|
| 0x0000_0000 | 0x0000_00FF | — | Reserved（アドレス 0 は Debug の safe zero：読むと 0） |
| 0x0000_0100 | 0x0000_0FFF | RWX | Debug |
| 0x0100_0000 | 0x0100_1FFF | RWXA | E51 DTIM (8 KB) |
| 0x0170_0000 | 0x0170_4FFF | RW | Bus Error Unit 0–4（4 KB ずつ：0x0170_0000, _1000, _2000, _3000, _4000） |
| 0x0180_0000 | 0x0180_1FFF | RWXA | E51 Hart 0 ITIM |
| 0x0180_8000 | 0x0180_EFFF | RWXA | U54 Hart 1 ITIM |
| 0x0181_0000 | 0x0181_6FFF | RWXA | U54 Hart 2 ITIM |
| 0x0181_8000 | 0x0181_EFFF | RWXA | U54 Hart 3 ITIM |
| 0x0182_0000 | 0x0182_6FFF | RWXA | U54 Hart 4 ITIM |
| 0x0200_0000 | 0x0200_FFFF | RW | CLINT |
| 0x0201_0000 | 0x0201_0FFF | RW | Cache Controller (L2) |
| 0x0202_0000 | 0x0202_0FFF | RW | WCB |
| 0x0300_0000 | 0x030F_FFFF | RW | DMA Controller |
| 0x0800_0000 | 0x081F_FFFF | RWX | L2-LIM |
| 0x0A00_0000 | 0x0BFF_FFFF | RWXC | L2 Zero Device (Scratchpad) |
| 0x0C00_0000 | 0x0FFF_FFFF | RW | PLIC |
| 0x1000_0000 | 0x1FFF_FFFF | — | Reserved |

属性: R/W/X、C = Cacheable、A = Atomics。ITIM と DTIM は全 Hart からアクセスできる。

### 6.2 Memory Address Space (Table 10-3)

| Start | End | Attr | Description / 経路 |
|---|---|---|---|
| 0x2000_0000 | 0x27FF_FFFF | RWX | ペリフェラル LOW（CPU D0, M10） |
| 0x2800_0000 | 0x2FFF_FFFF | RWX | ペリフェラル HIGH / AMP（CPU D1, M11） |
| 0x3000_0000 | 0x5FFF_FFFF | RWX | CPU D0 (M10) |
| 0x3000_0000 | 0x3FFF_FFFF | RWX | IOSCB-DATA |
| 0x3708_0000 | 0x3708_0FFF | RWX | IOSCB-CONFIGURATION |
| 0x4000_0000 | 0x5FFF_FFFF | RWX | FIC3 – 512 MB（D0） |
| 0x6000_0000 | 0x7FFF_FFFF | RWX | FIC0 – 512 MB（F0, M12） |
| **0x8000_0000** | **0xBFFF_FFFF** | **RWXC** | **DDR Cached Access – 1 GB** |
| **0xC000_0000** | **0xCFFF_FFFF** | RWX | **DDR Non-Cached Access – 256 MB** |
| **0xD000_0000** | **0xDFFF_FFFF** | RWX | **DDR Non-Cached WCB Access – 256 MB**（NC, M14） |
| 0xE000_0000 | 0xFFFF_FFFF | RWX | FIC1 – 512 MB（F1, M13） |
| 0x01_0000_0000 | 0x0F_FFFF_FFFF | — | Reserved |
| **0x10_0000_0000** | **0x13_FFFF_FFFF** | **RWXC** | **DDR Cached Access – 16 GB** |
| **0x14_0000_0000** | **0x17_FFFF_FFFF** | RWX | **DDR Non-Cached Access – 16 GB** |
| **0x18_0000_0000** | **0x1B_FFFF_FFFF** | RWX | **DDR Non-Cached WCB Access – 16 GB** |
| 0x1C_0000_0000 | 0x1F_FFFF_FFFF | — | Reserved |
| 0x20_0000_0000 | 0x2F_FFFF_FFFF | RWX | FIC0 – 64 GB |
| 0x30_0000_0000 | 0x3F_FFFF_FFFF | RWX | FIC1 – 64 GB |

- 各 DDR ウィンドウが物理 DDR のどこに対応するかは Segmentation Block（SEG0/SEG1）のオフセットで決まる（§7.2）。
- AMP 用途: low-speed ペリフェラルは LOW (0x2000_xxxx、D0 経由) と HIGH (0x2800_xxxx、D1 経由) のどちらのアドレスでもアクセスできる。2 つのソフトウェアコンテキストを PMP で分けるのに使う。

### 6.3 Peripherals Address Space (Table 10-2 + Table 3-49)

| Peripheral | LOW | HIGH | Size | Dual AHB |
|---|---|---|---|---|
| MMUART0 | 0x2000_0000 | 0x2800_0000 | 4 KB | Yes |
| Watchdog0 | 0x2000_1000 | 0x2800_1000 | 4 KB | Yes |
| SYSREG-PRIV | 0x2000_2000 | — | 4 KB | No |
| SYSREG-SCB | 0x2000_3000 | — | 4 KB | No |
| AXISW-CFG | 0x2000_4000 | — | 4 KB | No |
| MPUCFG | 0x2000_5000 | — | 4 KB | No |
| FRQ METER | 0x2000_6000 | — | 4 KB | No |
| DFI-CFG | 0x2000_7000 | — | 4 KB | No |
| MMC-CFG | 0x2000_8000 | — | 8 KB | No |
| DDRC-CFG | 0x2008_0000 | — | 512 KB | No |
| MMUART1–4 | 0x2010_0000 / _2000 / _4000 / _6000 | 0x2810_0000 / _2000 / _4000 / _6000 | 4 KB | Yes |
| Watchdog1–4 | 0x2010_1000 / _3000 / _5000 / _7000 | 0x2810_1000 / _3000 / _5000 / _7000 | 4 KB | Yes |
| SPI0 / SPI1 | 0x2010_8000 / 0x2010_9000 | 0x2810_8000 / 0x2810_9000 | 4 KB | Yes |
| I2C0 / I2C1 | 0x2010_A000 / 0x2010_B000 | 0x2810_A000 / 0x2810_B000 | 4 KB | Yes |
| CAN0 / CAN1 | 0x2010_C000 / 0x2010_D000 | 0x2810_C000 / 0x2810_D000 | 4 KB | Yes |
| MAC0 (GEM0) | 0x2011_0000 | 0x2811_0000 | 8 KB | Yes |
| MAC1 (GEM1) | 0x2011_2000 | 0x2811_2000 | 8 KB | Yes |
| GPIO0 / GPIO1 / GPIO2 | 0x2012_0000 / _1000 / _2000 | 0x2812_0000 / _1000 / _2000 | 4 KB | Yes |
| RTC (MSRTC) | 0x2012_4000 | 原文表記 0x2012_4FFF（Errata 参照） | 4 KB | Yes |
| Timer (MSTIMER) | 0x2012_5000 | 原文表記 0x2012_5FFF（Errata 参照） | 4 KB | Yes |
| H2F Interrupts (M2FINT) | 0x2012_6000 | 0x2812_6000 | 4 KB | Yes |

AHB スレーブ（Table 3-48、すべて AXI-D0/AHB0 経由）:

| Slot | Device | Address |
|---|---|---|
| 0 | APB Slaves | 0x2000_0000–0x201F_FFFF（AHB0）、0x2800_0000–0x281F_FFFF（AHB1） |
| 1 | QSPI | 0x2100_0000–0x21FF_FFFF |
| 2 | eNVM | 0x2020_0000–0x2020_0FFF（C-Bus）、0x2022_0000–0x2023_FFFF（R-Bus） |
| 3 | IOSCB | 0x3000_0000–0x3FFF_FFFF |
| 4 | USB | 0x2020_1000–0x2020_1FFF |

その他: User Crypto Processor 0x2200_0000–0x2201_FFFF (S4)、Trace 0x2300_0000–0x2303_FFFF (S9)、MPU 設定 0x2000_5000 + MPU offset。

### 6.4 eNVM の区画 (Table 3-71)

| Bus | 区画 | Size | Access | Address |
|---|---|---|---|---|
| C-Bus | Configuration | 512 B | RW | 0x2020_0000 |
| R-Bus | Sector 2 | 8 KB | RO | 0x2022_0000 |
| R-Bus | Sector 0 | 56 KB | RO | 0x2022_2000 |
| R-Bus | Sector 1 | 56 KB | RO | 0x2023_0000 |
| R-Bus | Sector 3 | 8 KB | RO | 0x2023_E000 |
