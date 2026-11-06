# MSS: DDR Controller / Clocking・Reset / Boot

> 出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（`C:/_Devs/PolarFire/doc/device/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`）。「p.NN」は PDF ページ番号。原文に無い補足は「(原文外)」。原文の不整合は SKILL.md の Errata を参照。

## 10. MSS DDR Controller (§3.11, p.50–61)

| 項目 | 内容 |
|---|---|
| 対応メモリ | DDR3, DDR3L, DDR4, LPDDR3, LPDDR4 |
| 構成ブロック | DDR controller、training logic、I/O lane、PLL。PHY と DFI 4.0 は MSS の外の **I/O Bank 6** にある |
| 入口 | CPU: 128-bit AXI4 → **SEG0**（キャッシュ経路, DDRC Port 1）。fabric / 非キャッシュ: 64-bit AXI4 → AXI Switch **S7** → **SEG1**（DDRC Port 2） |
| 初期化 | **E51 が DDR サブシステムを初期化する**。SYS_RESET_N / PLL_LOCK の解除 → E51 で初期化 → 完了すると `MSS_RESET_N_M2F` がアサートされる（fabric から監視できる） |
| メモリテスト | HSS がブート時のメモリテストを含む。キャッシュ有効のままテストすると、古いキャッシュデータのために初回は失敗することがある |
| ECC | **DDR3 / DDR4 のみ**。32-bit ごとに 4-bit の ECC (SECDED)。未初期化領域を読むとエラーになるので、先に全域を書いておく。MSS Configurator の DDR Topology タブで有効化 |
| データ幅 | 16 / 32-bit（ECC 付きは 18 / 36）。Lane 0–3 がデータ 8 bit ずつ、Lane 4 が ECC（4 bit 幅） |
| 最大容量 | アドレス 18 本 + CS 2 本（dual rank）で 8 GB（ECC を除く） |
| 動作モード | Half-rate controller、AL (0, CL-1, CL-2)、2T timing、multi-burst、bank management 最大 64 bank |
| アドレスマッピング | Address Ordering（例: Chip-Row-Bank-Col）を Controller Options タブで選ぶ |
| クロック | 外部 100 / 125 MHz → DDR PLL（MSS の外）。DDR PHY clock は最大 800 MHz。REF_CLK : SYS_CLK = 4:1 |
| QoS | `CFG_STARVE_TIMEOUT_P[0–7]`（§4.3） |
| 性能 | PolarFire SoC FPGA Datasheet の "MSS DDR Speed Grades" を参照 |

サポートしているデバイス（ECC を除く、x32 時の最大容量）:

| Type | Device | Max Devices | Max Size |
|---|---|---|---|
| DDR4 | 8 Gb 512Mx16 | 2 | 2 GB |
| DDR4 | 8 Gb 1024Mx8 | 4 | 4 GB |
| DDR4 | 16 Gb 2×512Mx16 twin die | 2 | 4 GB |
| DDR4 | 16 Gb 2×1024Mx8 twin die | 4 | 8 GB |
| DDR4 | 16 Gb 2048Mx8 | 4 | 8 GB |
| DDR3 | 8 Gb 1024Mx8 / 512Mx16 | 4 / 2 | 4 GB / 2 GB |
| LPDDR4 | 32 Gb 1Gx32 (dual ch) / 16 Gb 512Mx32 (single ch) | 1 | 4 GB / 2 GB |
| LPDDR3 | 16 Gb 512Mx32 / 8 Gb 256Mx32 | 1 | 2 GB / 1 GB |

- DDR3 / DDR4 は dual rank にするとデバイス数と容量が倍になる。LPDDR3 / LPDDR4 は single channel・single rank のみ。ピン数の少ないデバイスは 16-bit のみ。
- 主な信号: CK/CK_N, RESET_N (DDR3/4), A[15:0], BA[2:0]（DDR4 は 2 bit）, BG[1:0] (DDR4), CS_N, CKE, ODT, PAR / ALERT_N (DDR4), ACT_N (DDR4), DQ, DQ_ECC[35:32], DM / DM_N, DQS / DQS_N（x16 で [1:0]、x32 で [3:0]）。
- PFSOC_MSS の汎用ポート: REF_CLK / REF_CLK_N（100/125 MHz 入力）、**MSS_SYS_RESET_N_F2M**（PFSOC_INIT_MONITOR の DEVICE_INIT_DONE につなぐ）、MSS_RESET_N_M2F、PLL_CPU_LOCK_M2F、PLL_DDR_LOCK_M2F（Table 3-57）。

---

## 11. Clocking / Reset (Ch.8–9, p.149–150)

### 11.1 Clocking

| 項目 | 内容 |
|---|---|
| 基準クロック | 外部 100 MHz または 125 MHz → MSS PLL / DDR PLL / SGMII PLL（デバイスの NW コーナー） |
| MSS PLL | CPU 最大 625 MHz（STD）、**"-1" デバイスは最大 667 MHz**。Standby 80 MHz |
| 分周 | CPU /1、**L2 と AXI /2**、**AHB/APB /4**（大部分は CPU 600 MHz 以下 / AMBA 300 MHz 以下で動く） |
| 起動直後 | MSS は **内蔵 80 MHz RC 発振器（SCB clock）** で動く。E51 のファームウェアがブート中に MSS PLL へ切り替える |
| DDR PLL | 400 MHz（実際の値は DDR の種類で決まる） |
| SGMII PLL | SGMII PHY と GEM 用 |
| fabric → MSS | 5 本（FIC の DLL 用。125 MHz 以上で DLL を使う） |
| その他 | JTAG ポートは内部 50 MHz、ピン 25 MHz。CLINT mtime は 1 MHz。CAN は 8–80 MHz |

詳細は PolarFire Family Clocking Resources User Guide。

### 11.2 Reset Sources (Table 8-1)

| Reason (bit) | Asserted By | 効果 |
|---|---|---|
| SCB_PERIPH_RESET (0) | SCB | POR。MSS 全体を reset（SOFT-RESET の追加ビットで SCB レジスタも reset できる） |
| SCB_MSS_RESET (1) | SCB, CPU, MSS | MSS 全体（CPU、ペリフェラル、AXI）を reset。SCB レジスタは残る |
| SCB_CPU_RESET (2) | SCB, CPU, MSS | CPU Core Complex だけ reset。未完了の AXI トランザクションが残るので注意 |
| DEBUGER_RESET (3) | Debugger | SCB_MSS_RESET と同じ効果 |
| FABRIC_RESET (4) | Fabric (`MSS_RESET_N_F2M`) | SCB_MSS_RESET と同じ効果。電源投入時にアサートしておけば MSS を reset に保持できる |
| WDOG_RESET (5) | Watchdog | WDOG0 による reset |
| GPIO_RESET (6) | Fabric | fabric GPIO reset（その設定にした GPIO だけ。MSS は reset しない） |
| SCB_BUS_RESET (7) | Fabric | SCB バスの reset |
| CPU_SOFT_RESET (8) | MSS | soft reset レジスタによる CPU reset |

MSS reset の後、ペリフェラルは `SOFT_RESET_CR` で個別に reset 解除する。

---

## 12. Boot Process (Ch.7, p.139–148)

ブート用 NVM: **eNVM 128 KB**、**sNVM 56 KB**。System Controller が pNVM 内の `U_MSS_BOOTMODE` / `U_MSS_BOOTCFG` を読んで Core Complex を起動する（pNVM / sNVM は bitstream でしか書き換えられず、CPU からは見えない）。

| Mode | 用途 | シーケンス |
|---|---|---|
| **0** (IDLE) | ブランクデバイス、デバッグ | 全 Hart が pseudo BOOTROM（system register 上のコードループ）を回り続け、デバッガが PC を書き換えるのを待つ（TEMP1=0 の間ループする） |
| **1** (User non-secure) | eNVM（または fabric LSRAM）から非セキュアに起動 | `U_MSS_BOOTCFG` の 5 Hart 分の reset vector を system register に設定してから reset 解除 |
| **2** (User secure) | ユーザ定義のセキュアブート | pseudo BOOTROM ループ中に、System Controller が sNVM から UBL を on-chip RAM にコピー → TEMP1 に UBL のベクタアドレスを書いて実行させる。UBL が eNVM を認証する。UBL の長さは **32-bit word 数**。sNVM は Libero で ROM 指定にする。ハッシュが合わないと boot mode 0 になり boot_fail tamper が立つ |
| **3** (Factory secure) | Microchip 提供の eNVM 認証 | eNVM 上の SBIC（Secure Boot Image Certificate）を ECDSA で検証し、UBL の SHA-384 も照合する。DSN バインド（任意）、Version による失効（任意）。失敗すると Hart は起動せず、fabric に tamper を出す。暗号化はしない |

- いずれのモードも、MSS / Core Complex のデフォルトクロックは SCB clock の 80 MHz。
- Secure boot は "S" 版と非 "S" 版の両方で使える。
- (原文外) HSS (Hart Software Services) は通常 boot mode 1 で eNVM に置かれ、E51 で動く zero-stage bootloader / モニタ。DDR 初期化、クロック切り替え、U54 への payload 配布を行う。TRM 本文で HSS に触れているのは DDR メモリテストの注記（Table 3-50）だけ。詳細は PolarFire SoC Software Development and Tool Flow User Guide と HSS のドキュメントを参照。
