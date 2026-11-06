# MSS: Overview / CPU Core Complex / L2・LIM・Scratchpad

> 出典: PolarFire SoC MSS Technical Reference Manual DS60001702S Rev S（`C:/_Devs/PolarFire/doc/device/PolarFire_SoC_FPGA_MSS_Technical_Reference_Manual.pdf`）。「p.NN」は PDF ページ番号。原文に無い補足は「(原文外)」。原文の不整合は SKILL.md の Errata を参照。

## 1. MSS Overview (Ch.1–2, p.5–7)

| Feature | 内容 |
|---|---|
| E51 Monitor Core ×1 | RV64IMAC, 625 MHz, 16 KB L1 iCache or 8 KB ITIM, 8 KB DTIM, M/U mode |
| U54 Application Cores ×4 | RV64GC (G=IMAFD), 625 MHz, 32 KB L1 iCache or 28 KB ITIM, 32 KB dCache, Sv39 MMU, M/S/U mode |
| L2 Cache | 2 MB L2 cache or 1.875 MB LIM, ECC 付き |
| BootFlash | 128 KB eNVM |
| PMP | コアごとに 16 region、粒度 4 byte |
| Interrupts | コアごとに 48 local（M/S）、169 external (platform level)、コアごとに Software/Timer local（M mode） |
| DMA Engine | 独立 4 チャネル |
| BEU | コアごとに 1 個（L1 ECC と TileLink バスエラー） |
| TileLink | B64 / D128 switch（I/O とメモリのコヒーレンシ） |
| Debug / Trace | JTAG デバッグ（全コア）、命令トレース（全コア） |
| MPU | 外部 AXI マスタごとに 1 個 |
| FIC | 64-bit AXI4 FIC ×3、32-bit APB FIC ×1（＋Crypto 専用 AHB の FIC4） |
| User Crypto Processor | Athena F5200 TeraFire ×1, 200 MHz |
| Secure Boot / Anti-tamper | E51 と全 U54 でサポート / MSS 用の anti-tamper mesh |
| DDR Controller ×1 | DDR3, DDR3L, DDR4, LPDDR3, LPDDR4、ECC 付き |
| Peripherals | GEM ×2, USB OTG 2.0 ×1, QSPI-XIP ×1, SPI ×2, eMMC 5.1 / SD / SDIO ×1, MMUART ×5, I2C ×2, CAN ×2, GPIO ×3, RTC ×1, FRQMeter, Watchdog ×5, Timer (2×32-bit) |
| MSS I/Os | 38 本 |

MSS を構成するブロック: CPU Core Complex, AXI Switch, FICs, MPU, Segmentation Blocks, AXI-to-AHB, AHB-to-APB, AMP APB Bus, MSS I/Os, User Crypto Processor, MSS DDR Controller, Peripherals（Figure 2-1, p.7）。

---

## 2. CPU Core Complex (§3.1, p.8–39)

### 2.1 E51 と U54 の比較

| 項目 | E51 (Hart0) | U54 (Hart1–4) |
|---|---|---|
| ISA | RV64IMAC | RV64GC |
| iCache / ITIM | 16 KB 2-way / ITIM 最大 8 KB | 32 KB 8-way VIPT / ITIM 最大 28 KB |
| データ側 | 8 KB DTIM（dCache なし） | 32 KB 8-way write-back VIPT dCache |
| Line size | 64 B | 64 B |
| Pipeline | single-issue, in-order, 5-stage | 同左（fully bypassed） |
| Modes | M, U | M, S, U |
| MMU | なし | Sv39（39-bit VA → 48-bit PA）、I/D TLB 各 32 entry fully assoc、2 MB / 1 GB ページ |
| FPU | なし | IEEE 754-2008 単精度/倍精度、FMA、subnormal 対応 |
| LR/SC | **非サポート** | サポート（キャッシュ可能領域のみ。DTIM や非キャッシュ領域では access exception） |
| ECC | iCache と DTIM に SECDED | iCache, ITIM, dCache に SECDED |
| 主な用途 | ブートローダ、ベアメタル、U54 の監視 | ベアメタル、OS |

補足:
- CPU Core Complex の物理アドレス空間は **38-bit**（§3.1.3）。
- U54 は PTE の A/D ビットをハードウェアで立てない。PTE.A=0 の read、PTE.D=0 の write は page fault になる（§3.1.2.8）。
- iCache はプラットフォームのメモリ系とコヒーレントではない。自己書き換えコードでは `FENCE.I` が必要。
- 非 executable 領域（属性 X なし）から命令をフェッチすると trap。misaligned アクセスも trap（DTIM/dCache）。
- L1 ECC はハードウェア内部で処理され、ユーザからは制御できない。報告は BEU 経由（§2.6）。

### 2.2 ITIM / DTIM の使い方

| 規則 | 内容 |
|---|---|
| ITIM の割り当て | ITIM 領域の n バイト目に store すると、iCache の先頭 (n+1) バイトが ITIM に再割り当てされる（cache line 単位で切り上げ） |
| 割り当て単位 (U54) | 内部的には 4 KB の way 単位。1 line だけ要求しても 4 KB way を 1 本消費する |
| 最低 1 way | iCache の 1 way は必ずキャッシュとして残る |
| 解放 | ITIM 領域の直後のバイト（U54 は base+28 KB）に 0 を store すると解放されて iCache に戻る。way の一部だけの解放は不可 |
| 内容の保持 | 解放→再割り当てで内容が保持されるかは不定。割り当て後に必ず初期化する |
| 他 Hart からのアクセス | 可能だが multi-wait-state で遅い。E51 DTIM も他 Hart からアクセス可能 |
| DTIM レイテンシ (E51) | full-word 2 cycle、sub-word 3 cycle |
| 推奨用途 | ITIM: ISR、ブート時ルーチン、短い hot path。DTIM: スタック、mailbox、ディスクリプタ、小さな制御構造 |
| 配置方法 | linker script で ITIM/DTIM のメモリ領域を定義し、セクションを配置する（PolarFire SoC Bare-Metal Examples を参照） |

### 2.3 その他のコア機能

| 機能 | 要点 | 参照 |
|---|---|---|
| Branch Prediction | BTB 28 entry、BHT 512 entry、RAS 6 entry。予測ミスは 3 cycle。`bpm` CSR (0x7C0) の bit0 `bdp` に非 0 を書くと static-taken になる（bdp に書くと BTB はクリアされる） | §3.1.7 p.22 |
| HPM | `mcycle`, `minstret`, `mhpmevent3/4` + `mhpmcounter3/4`（40-bit）。event class: 0=Instruction Commit, 1=Micro-architectural, 2=Memory System（L1 のみ）。例: `mhpmevent3=0x4200` → load または条件分岐の retire を数える | §3.1.1.5 p.9 |
| TileLink | 固定の round-robin 調停。PLIC/CLINT 等の共有資源へのアクセスに使う。調停を変えるレジスタはない | §3.1.8 p.23 |
| External Bus I/F | AXI 128 to DDR、D0、D1、F0、F1、NC の 6 ポート。非 CPU マスタは AXI Switch の S8 から Core Complex に入る | §3.1.9 p.23 |

### 2.4 DMA Engine (Core Complex 内 PDMA, §3.1.10, p.23–25)

- 4 チャネル。チャネルごとに Done / Error の 2 割り込み（PLIC ID 5–12）。
- Channel base = **DMA Controller base (0x0300_0000) + 0x1000 × Channel ID**。

| Offset | Register | 内容 |
|---|---|---|
| 0x000 | Control | bit0 `claim`、bit1 `run`、bit14 `doneIE`、bit15 `errorIE`、bit30 `done`、bit31 `error` |
| 0x004 | NextConfig | bit2 `repeat`、bit3 `order`、[27:24] `wsize`、[31:28] `rsize`（log2 サイズ: 0=1B, 3=8B, 5=32B）。1 トランザクションは最大 64 B |
| 0x008 | NextBytes | 転送バイト数 (WARL) |
| 0x010 | NextDestination | 転送先物理アドレス |
| 0x018 | NextSource | 転送元物理アドレス |
| 0x104–0x118 | ExecConfig / ExecBytes / ExecDestination / ExecSource | 実行中転送の状態 (RO) |

- 手順: `claim` をセット（Next 系がクリアされる）→ Next 系を設定 → `run` をセット（Next が Exec にコピーされて開始）。
- 一度に転送できるのは連続した 1 ブロックだけ。`repeat` を立てると完了時に自動で再開する。
- `claim` は `run`=0 のときだけクリアできる。

### 2.5 Write Combining Buffer (WCB, §3.1.11, p.25–26)

- 連続する single-beat の書き込みを TileLink/AXI バーストにまとめる。read と multi-beat 転送はバイパスする。
- 対象のアドレス範囲は非キャッシュ DDR の WCB ウィンドウ（System Port 4 / AXI4-NC 経由）:
  - `0xD000_0000 – 0xDFFF_FFFF`
  - `0x18_0000_0000 – 0x1B_FFFF_FFFF`
- 例: 8 byte ずつ 2048 byte を昇順に書くと、64-bit × 256 の 1 バーストにまとまる。
- `idle` レジスタ（offset 0、[7:0]、reset 16）: この cycle 数だけ書き込みがないとバッファを吐き出す。0 で WCB 無効。
- 同じアドレスへの write → read ではバッファがフラッシュされ、read は write 完了まで待たされる。

### 2.6 Bus Error Unit (BEU, §3.1.12, p.26–28)

| Offset | Register | 内容 |
|---|---|---|
| 0x000 | cause | 最初に記録したイベント（0 = エラーなし。0 に戻すまで次は記録しない） |
| 0x008 | value | エラーの物理アドレス |
| 0x010 | enable | 記録するイベントのマスク |
| 0x018 | plic_interrupt | PLIC 経由で割り込みを出すイベント |
| 0x020 | accrued | 発生したイベント（enable と無関係にセットされる） |
| 0x028 | local_interrupt | Hart に直接割り込みを出すイベント |

cause の値: 2 = Instruction cache/ITIM correctable ECC、3 = ITIM uncorrectable、5 = Load/store TileLink bus error、6 = Data cache correctable ECC、7 = Data cache uncorrectable ECC（0 = No Error、1/4 = Reserved）。
BEU のローカル割り込みは **cause 128**。`mie`/`mideleg` にビットがないので常に有効で、M-mode からは委譲できない。PLIC 側は ID 182–186（Hart0–4）。

---

## 3. L2 Cache / L2-LIM / Scratchpad (§3.1.5–3.1.6, p.15–22)

### 3.1 構成

- 2 MB を 4 bank に分割（アドレスインタリーブ）。各 bank 512 KB = 512 set × 16 way × 64 B block。
- 各 bank は専用の 128-bit TL-C inner port を持つ。outer port は 128-bit で DDR controller につながる。
- dCache のコヒーレンシは L2 内のディレクトリ方式 coherence manager が管理する。

### 3.2 WayEnable と L2-LIM

| 規則 | 内容 |
|---|---|
| Reset 直後 | way0 だけがキャッシュとして有効、残りは L2-LIM として見える |
| 有効化 | `WayEnable`（offset 0x008、[7:0]）: この値以下の way index をキャッシュとして使う。**値は増やす方向にしか書けない** |
| 無効化 | 一度有効にした way は Core Complex を reset しないと LIM に戻らない |
| アドレス対応 | 番号の大きい way ほど低い LIM アドレスに対応する（way15 → 0x0800_0000、way14 → 0x0802_0000 … way1 → 0x081C_0000）。キャッシュ way を増やすと LIM 空間は小さくなる |
| 優先度 | 同じ bank への L2-LIM アクセスは cache way へのアクセスより優先される |
| ECC 書き戻し | LIM モードでは SECDED 訂正はするが、訂正データの自動書き戻しはしない（cache / scratchpad モードでは書き戻す） |

### 3.3 Way Masking / Scratchpad

- `WayMaskX`（0x800 + 8 × Master ID）: bit を 0 にした way は、そのマスタが**割り当て（evict）できなくなる**。読み出しは可能。すべての WayMaskX でマスクすると、その way はロックされる。
- AXI front port は 4 つの内部 TileLink ID（Master 1–4）に割り振られ、どの ID になるかは予測できない。**front port の 4 つの way mask は同じ値に設定すること**。
- Scratchpad = **L2 Zero Device**（0x0A00_0000–0x0BFF_FFFF）。裏にメモリがないので、evict されるとデータが失われる。L1 にキャッシュできる点が LIM より有利。
- Scratchpad の手順（§3.1.6.1.4 p.18–19）: ①WayEnable で way を有効化 → ②書き込み役のマスタ S を決める → ③他のマスタ X の WayMask で scratchpad 用 way をマスク → ④S は scratchpad 用 way 以外をマスク → ⑤S が Zero Device に書く → ⑥way ごとに ④⑤を繰り返す → ⑦S 自身も scratchpad 用 way をマスクする。

### 3.4 L2 Controller Register Map（base 0x0201_0000、Table 3-9）

| Offset | Register | 説明 |
|---|---|---|
| 0x000 | Config (RO) | [7:0] Banks=4, [15:8] Ways=16, [23:16] lgSets=9, [31:24] lgBlockBytes=6 |
| 0x008 | WayEnable (RW) | 上記参照 |
| 0x040 | ECCInjectError | [7:0] 反転するビット位置、bit16 Target（1 = metadata SRAM）。注入は次の write 時に起きる |
| 0x100 / 0x108 | ECCDirFixAddr / ECCDirFixCount | metadata 訂正。Count を読むと DirError 割り込みがクリアされる |
| 0x120 / 0x128 | ECCDirFailAddr / ECCDirFailCount | metadata 訂正不能 |
| 0x140 / 0x148 | ECCDataFixAddr / ECCDataFixCount | data 訂正。Count を読むと DataError がクリアされる |
| 0x160 / 0x168 | ECCDataFailAddr / ECCDataFailCount | data 訂正不能。Count を読むと DataFail がクリアされる |
| 0x200 | Flush64 (WO) | 書いたアドレスを含むブロックをフラッシュ |
| 0x240 | Flush32 (WO) | 書いた値を 4 bit 左シフトしたアドレスのブロックをフラッシュ |
| 0x800–0x870 | WayMask0–14 | Master ID: 0=DMA、1–4=AXI4 front port ID#0–3、5/6=Hart0 dCache MMIO/iCache、7/8=Hart1 d/i、9/10=Hart2、11/12=Hart3、13/14=Hart4 |

- 電源制御: `L2_SHUTDOWN_CR` (SYSREG, 0x174) [3:0] で L2 RAM ブロックを停止する。512 KB / 1 MB / 1,512 KB 構成にできる（Table 3-7/3-8）。
- L2 ECC の割り込みは PLIC ID 1–4（§8.3）。
