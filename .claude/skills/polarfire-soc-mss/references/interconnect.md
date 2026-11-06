# MSS: AXI Switch / FIC / MPU・Segmentation・AHB-APB

> 出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（`C:/_Devs/PolarFire/doc/device/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`）。「p.NN」は PDF ページ番号。原文に無い補足は「(原文外)」。原文の不整合は SKILL.md の Errata を参照。

## 4. AXI Switch (§3.2, p.40–44)

15 master port、9 slave port。S5 と S6 は AXI-to-AHB / AHB-to-APB ブリッジ経由でペリフェラルにつながる。

### 4.1 Master / Slave ポート

| Master | 接続元 | QoS の出どころ |
|---|---|---|
| M1 | FIC0 Fabric Master | fabric から入力 |
| M2 | FIC1 Fabric Master | fabric から入力 |
| M3 | FIC2 Fabric Master | fabric から入力 |
| M4 | Crypto Processor | System register（一度設定すると固定） |
| M5 / M6 | GEM0 / GEM1 | Ethernet MAC が設定 |
| M7 | USB | System register（固定） |
| M8 | eMMC | System register（固定） |
| M9 | SCB Bridge | System Controller の SCB I/F（最優先） |
| M10–M14 | CPU D0 / D1 / F0 / F1 / NC | System register（固定） |
| M15 | Trace Module | Trace SMB（優先度調停） |

| Slave | 接続先 | Region 1 | Region 2 | Region 3 |
|---|---|---|---|---|
| S1 | FIC0 Fabric Slave | 0x20_0000_0000–0x2F_FFFF_FFFF | 0x6000_0000–0x7FFF_FFFF | — |
| S2 | FIC1 Fabric Slave | 0x30_0000_0000–0x3F_FFFF_FFFF | 0xE000_0000–0xFFFF_FFFF | — |
| S3 | FIC3 Fabric Slave (APB) | 0x4000_0000–0x5FFF_FFFF | — | — |
| S4 | Crypto Processor | 0x2200_0000–0x2201_FFFF | — | — |
| S5 | AHB0 | 0x2000_0000–0x21FF_FFFF | 0x3000_0000–0x3FFF_FFFF | — |
| S6 | AHB1 (AMP APB) | 0x2800_0000–0x2FFF_FFFF | — | — |
| S7 | DDR Non-Cacheable | 0x14_0000_0000–0x1B_FFFF_FFFF | 0xC000_0000–0xDFFF_FFFF | — |
| S8 | CPU Core Complex MMIO | 0x10_0000_0000–0x13_FFFF_FFFF | 0x8000_0000–0xBFFF_FFFF | 0x0000_0000–0x1FFF_FFFF |
| S9 | Trace Module | 0x2300_0000–0x2303_FFFF | — | — |

### 4.2 Connectivity（Table 3-34、✓ = 接続あり）

| | S1 | S2 | S3 | S4 | S5 | S6 | S7 | S8 | S9 |
|---|---|---|---|---|---|---|---|---|---|
| M1 FIC0 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| M2 FIC1 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| M3 FIC2 | — | — | — | — | — | — | ✓ | ✓ | — |
| M4 Crypto | ✓ | ✓ | ✓ | — | ✓ | — | ✓ | ✓ | — |
| M5–M8 GEM0/GEM1/USB/eMMC | ✓ | ✓ | — | ✓ | — | — | ✓ | ✓ | — |
| M9 SCB | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| M10 D0 | — | — | ✓ | ✓ | ✓ | — | — | — | ✓ |
| M11 D1 | — | — | — | — | — | ✓ | — | — | — |
| M12 F0 | ✓ | — | — | — | — | — | — | — | — |
| M13 F1 | — | ✓ | — | — | — | — | — | — | — |
| M14 NC | — | — | — | — | — | — | ✓ | — | — |
| M15 Trace | ✓ | ✓ | — | — | — | — | ✓ | — | — |

FIC2 (M3) から届くのは DDR 非キャッシュ (S7) と CPU MMIO (S8) だけ。S8 経由ならキャッシュ DDR にも届く（Figure 3-18）。

### 4.3 調停と QoS

- Slave 側: S0 (Default Slave、不正アドレスに応答) は FCFS、S1–8 は QoS Arbiter / Fair among equals、S9 (Trace) は Priority。
- Master 側: M1–8 と M10–14 は Fair among equals、M9 (SCB) と M15 (Trace) は Priority（最優先）。
- QoS は 4-bit で、大きいほど優先度が高い。
- **Bandwidth Tuning**（各 master port M1–M15 の QoS regulator, Table 3-39）: Transaction Rate（例 1/64 = 64 clock に 1 要求）、Burstiness（溜められる token 数。0 で常時レート制限）、Peak Rate（transaction rate より速くする）、Slave-Ready Dependency。
- **DDR Controller QoS**（`CFG_STARVE_TIMEOUT_P[0–7]`）: Port 0 = 内部用、**Port 1 = 128-bit キャッシュ DDR 経路**、**Port 2 = 64-bit 非キャッシュ DDR 経路**、Port 3–7 = 未使用。AWQOS/ARQOS を見て、同じ優先度どうしは round-robin。低レイテンシが必要なポートは timeout を短くする。
- DDR4 の switch 構成では `QOS_VALUE` レジスタは使えない。rate / burstiness / peak-rate / slave-ready の各制御で調整する。
- AXI exclusive / locked: MSS 内のマスタは発行しない。FIC マスタからの exclusive は通常転送として OKAY を返す。ただし FICx→FICy の場合は FICy 側に Ax_LOCK がそのまま出る。

---

## 5. Fabric Interface Controllers (§3.3, Ch.6, p.44, 136–138)

| FIC | バス / 幅 | 方向 | MSS 側アドレス | 用途 |
|---|---|---|---|---|
| FIC0 | AXI4 64-bit | MSS master と fabric master の双方向 | 0x6000_0000–0x7FFF_FFFF (512 MB)、0x20_0000_0000–0x2F_FFFF_FFFF (64 GB) | fabric との汎用データ転送 |
| FIC1 | AXI4 64-bit | 双方向 | 0xE000_0000–0xFFFF_FFFF (512 MB)、0x30_0000_0000–0x3F_FFFF_FFFF (64 GB) | fabric と PCIe Controller hard block |
| FIC2 | AXI4 64-bit | fabric master → MSS slave のみ | MSS メモリマップには出ない | **非キャッシュ DDR へのアクセス専用**（MSS 内 DDR controller） |
| FIC3 | APB 32-bit | MSS master → fabric slave のみ | 0x4000_0000–0x5FFF_FFFF (512 MB) | fabric の APB スレーブ、PCIe/XCVR hard block の設定 |
| FIC4 | AHB-Lite 32-bit ×2 | fabric → Crypto（slave）、Crypto DMA → fabric slave | メモリマップには出ない | User Crypto Processor 専用 |

- CPU から見た経路: FIC0 の 512 MB 窓は F0 (M12)、FIC1 の 512 MB 窓は F1 (M13)、FIC3 は D0 (M10) を通る。
- 各 FIC は MSS main clock の比で決まる個別のクロックで動く。**FIC クロックが 125 MHz 以上なら DLL を有効、125 MHz 未満なら DLL を bypass**。fabric から MSS へは 5 本のクロックが入り、FIC の DLL に使われる（Ch.9）。
- FIC を使うには、`SUBBLK_CLOCK_CR`（clock）と `SOFT_RESET_CR`（reset）の該当ビットで有効にする（§6.2）。
- 設定は Standalone MSS Configurator で行う。Figure 6-2 に FIC_0 を 250 MHz で動かしたときの非キャッシュ DDR R/W 波形がある。

---

## 7. MPU / Segmentation / AHB-APB (§3.4–3.8, p.44–49)

### 7.1 MPU (非 CPU マスタ用の PMP)

| MPU | Master | Offset (0x2000_5000 +) | PMP 数 |
|---|---|---|---|
| MPU1 | FIC0 | 0x0000 | 16 |
| MPU2 | FIC1 | 0x0100 | 16 |
| MPU3 | FIC2 | 0x0200 | 8 |
| MPU4 | User Crypto Processor | 0x0300 | 4 |
| MPU5 | Ethernet 0 | 0x0400 | 8 |
| MPU6 | Ethernet 1 | 0x0500 | 8 |
| MPU7 | USB | 0x0600 | 4 |
| MPU8 | MMC | 0x0700 | 4 |
| MPU9 | SCB | 0x0800 | 8 |
| MPU10 | TRACE | 0x0900 | 2 |
| (SEG0 / SEG1) | Segmentation | 0x0d00 / 0x0e00 | NA（レジスタ構成が異なる） |

- アドレスの bit[11:8] で MPU を、bit[7:0] でレジスタを選ぶ。PMPCFG0–15 が 0x00–0x78、STATUS が 0x80。
- PMPCFG (64-bit): [35:0] PMP（38-bit アドレスを 2 bit シフトした値。bit[8:0] は 0x1FF 固定なので**最小ブロックは 4 KB**）、[63:56] MODE: bit63 LOCKED、[60:59] Match（00=OFF、11=NAPOT のみ有効。TOR/NA4 は OFF 扱い）、bit58 X、bit57 W、bit56 R。
- **Reset 直後は、MPU のアクセス領域を作るまで MSS にアクセスできない**。SCB だけは System mode で MPU を通らない（MPU9 は AXI ID=1 をバイパスさせる）。
- 違反した場合: トランザクションを止めて割り込みを出し、後で DECERR を返す。STATUS: [37:0] 失敗したアドレス、bit38 Write、[42:39] AXI ID、bit43 Failed（system register でクリアするまで次の違反は記録しない）。

### 7.2 Segmentation Blocks (SEG0 / SEG1)

- CPU から見た DDR アドレスを物理 DDR アドレスに変換するため、オフセットを加減算する。SEG0 は CPU Core Complex の 128-bit 経路、SEG1 は AXI Switch S7 の 64-bit 経路に入る。
- 各レジスタ: bit31 LOCKED、[14:0] Address Offset（アドレス bit[37:24] に足す 2 の補数）。

| Reg | 機能 | SEG0 | SEG1 |
|---|---|---|---|
| 0 | Cached access at 0x00_8000_0000 | Configurable | Reserved |
| 1 | Cached access at 0x10_0000_0000 | Configurable | Reserved |
| 2 | Non-Cached at 0x00_C000_0000 | Reserved | Configurable |
| 3 | Non-Cached at 0x14_0000_0000 | Reserved | Configurable |
| 4 | Non-Cached WCB at 0x00_D000_0000 | Reserved | Configurable |
| 5 | Non-Cached WCB at 0x18_0000_0000 | Reserved | Configurable |
| 6 | Trace access | Reserved | Configurable |
| 7 | DDRC Blocker Control | Configurable | Reserved |

- **DDRC Blocker**（SEG0 reg7, 0x1c, bit0 `UNBLOCK`）: reset 時は 0 で、L2 から DDR controller へのアクセスを止めている。1 を書くと解除され、MSS reset まで 0 に戻せない。

### 7.3 AHB / APB ブリッジ

- S5 → AHB-Lite（AHB0）、S6 → AHB-Lite（AHB1）。AHB は 5:1 の mux で 5 つのスレーブにつながる。AHB クロックは AXI に同期した /2, /4, /8。**eNVM を動かすには AHB クロックが 1 MHz 以上必要**。
- APB クロックは AHB と同じ。AHB-to-APB はデフォルトで non-posted write（遅い APB デバイスが AXI まで止める）。`SR_AHBAPB_CR.APBx_POSTED` で posted write にできるが、書き込み完了前に CPU が次へ進む点に注意。
- AMP APB bus: Dual AHB = Yes のペリフェラルは、起動時の設定で S6（0x2800_0000–0x2FFF_FFFF）側に付け替えられる。PMP の設定と配置アドレスが合わないと PSLVERR → AXI エラーになる。
