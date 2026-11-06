# MSS: Security・Protection / Debug・Trace・System Registers

> 出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（`C:/_Devs/PolarFire/doc/device/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`）。「p.NN」は PDF ページ番号。原文に無い補足は「(原文外)」。原文の不整合は SKILL.md の Errata を参照。

## 14. Security / Protection

| 機能 | 内容 | 参照 |
|---|---|---|
| PMP (Hart ごと) | 16 region、`pmpcfg0`/`pmpcfg2`（RV64 では pmpcfg1/3 は使わない）、`pmpaddr0–15` は PA[55:2]、A: 0=OFF, 1=TOR, 2=NA4, 3=NAPOT、最小 4 byte。**L bit を立てると M-mode にも制限がかかり、system reset まで解除できない** | §3.1.4 p.13–15 |
| MPU (マスタごと) | §7.1。reset 直後は領域を設定するまでアクセス不可。最小 4 KB | §3.4 p.44–46 |
| AMP 分離 | LOW / HIGH の 2 つのアドレス系統と PMP の組み合わせ。D0 / D1 ポートで帯域も分けられる | §3.8, Ch.10 |
| DDRC Blocker | SEG0 reg7 `UNBLOCK`（一度だけ 1 を書ける） | §3.5 p.47 |
| Secure Boot | Boot mode 2（UBL in sNVM）/ 3（ECDSA + SHA-384 の SBIC） | Ch.7 |
| Anti-tamper | MSS の anti-tamper mesh（`mesh_fail` 割り込み） | Table 1-1, Table 5-1 |
| Crypto | Athena F5200 TeraFire（200 MHz）、MPU4 で保護 | §3.10 |
| ECC 監視 | L1 → BEU、L2 → PLIC 1–4、ペリフェラル → `EDAC_SR` / `EDAC_INTEN_CR` / `EDAC_CNT_*`（PLIC 78/79） | §3.1.12, Table 5-1 Note |

---

## 15. Debug / Trace / System Registers

- **Debug** (§3.1.13, p.28–33): RISC-V External Debug Support v0.13、8-bit IR。JTAG は fabric JTAG ピンかオンチップ JTAG ピンを使う（MSS Configurator で選択）。Program Buffer 16 word、Debug Data RAM 1 word、abstract command は GPR アクセスのみ。デバッガは SoftConsole（マルチコアでも一度に 1 コア）、FPGA 側は SmartDebug。Debug Module のメモリ: 0x100–0x110 と 0x400–0x7FF が flags、0x300–0x3FF が RAM / Program Buffer、0x800–0xFFF が ROM。
- **Trace** (§3.1.14, p.33–39): 5 コアの命令トレース、AXI Switch の任意の 1 スレーブ I/F の完全な AXI(64) トレース、L2→DDR の AXI(128) アドレストレース、fabric 信号 40 本。JTAG 経由で取り出す。AXI Communicator と Virtual Console がある。サポートは Lauterbach。
- **System Registers** (Ch.4, p.117):
  - **SYSREG** (`PFSOC_MSS_TOP_SYSREG`, APB 0x2000_2000): クロック、割り込み、MSS I/O と IOMUX、reset（`SOFT_RESET_CR`、`SUBBLK_CLOCK_CR`）、ペリフェラル ECC、L2 低電力（`L2_SHUTDOWN_CR`）、ブリッジ（`SR_AHBAPB_CR`）、eNVM、`FAB_INTEN_U54_n` など。
  - **SCBSYSREG** (`SYSREGSCB`, 0x2000_3000): boot address、boot ROM、MPU、MSS soft reset、AXI Switch、trace / debug 接続。
  - 詳細は PolarFire SoC Device Register Map（`pfsoc_regmap.htm`）。
