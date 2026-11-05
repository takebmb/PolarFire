# Discovery Kit リファレンスデザイン（Tcl 生成スクリプト）リファレンス

> 出典: *PolarFire SoC Discovery Kit Reference Design Generation Tcl Scripts - Libero SoC v2025.2+*（GitHub `polarfire-soc/polarfire-soc-discovery-kit-reference-design` の README。文書番号・Rev なし）
> ローカル: `C:\_Devs\PolarFire\doc\polarfire-soc-discovery-kit-reference-design-README.md`
> リポジトリ: `https://github.com/polarfire-soc/polarfire-soc-discovery-kit-reference-design`
> 注: QSG（DS50003565J）はタグ `releases/tag/2024.04` を案内しているが、取得した README は **v2025.2+** 版。タグにより対応 Libero と引数が異なる可能性がある (原文外)。

---

## 1. 概要

- Discovery Kit 用リファレンスデザインを Libero SoC で生成する **Tcl スクリプト**と、デバイス固有の I/O 制約を提供
- **箱出し（工場出荷）デザイン = FIR フィルタデモとは機能が異なる**。本デザインはベアメタルアプリ実行・**Linux 起動**・キット上の周辺機器の有効化用
- FIR デモは引数 `FIR_DEMO` で生成可能

| 項目 | 内容 |
|---|---|
| 対応 Libero | **Libero SoC v2025.2 以上**。テスト済みバージョンは各リリースのリリースノートに記載 |
| ライセンス | 無償・有償いずれの Libero ライセンスでも生成可（Silver ライセンス可、[quick-start.md §6](quick-start.md)） |
| メインスクリプト | `MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.tcl` |
| 事前生成済みジョブ | リポジトリの **Releases** に FlashPro Express ジョブ（Libero なしで書込み可） |

---

## 2. 生成方法

### 2.1 標準デザイン（GUI）

- [ ] 1. リポジトリを clone または download
- [ ] 2. Libero を開く
- [ ] 3. Execute Script ダイアログを開く（**CTRL + U**）
- [ ] 4. `MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.tcl` を実行
- [ ] 5. 必要ならデザインを変更
- [ ] 6. Libero の Design Flow を実行してデバイスを書き込む

### 2.2 引数付き生成（GUI）

- [ ] 1〜4. 上と同じ
- [ ] 5. 引数を入力（例: `I2C_LOOPBACK HSS_UPDATE PROGRAM`）
  - **GUI では引数をクォートで囲まない。区切りはスペース**
- [ ] 6. 必要ならデザインを変更
- [ ] 7. Design Flow を実行して書き込み

### 2.3 コマンドライン（GUI なし）

GUI を起動しないため生成時間を大幅に短縮できる。

- [ ] 1. リポジトリを clone または download
- [ ] 2. 生成スクリプトのあるディレクトリでターミナルを開く
- [ ] 3. 実行（引数は例、原文どおり）:

```
[path to Libero]/libero script:MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.tcl SCRIPT_ARGS:I2C_LOOPBACK+HSS_UPDATE+PROGRAM logfile: MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.log
```

- **コマンドラインでは引数を `+` で区切る**

### 2.4 引数区切りのまとめ

| 実行方法 | 区切り | 例 |
|---|---|---|
| GUI（Execute Script） | スペース、クォートなし | `I2C_LOOPBACK HSS_UPDATE PROGRAM` |
| コマンドライン | `+`（`SCRIPT_ARGS:` に続けて） | `SCRIPT_ARGS:I2C_LOOPBACK+HSS_UPDATE+PROGRAM` |

### 2.5 スクリプトが自動で行うこと

1. **PolarFire SoC MSS Configurator を batch モード**で実行し MSS コンポーネントを生成
2. コンポーネントの生成・インスタンス化・接続、トップレベル I/O 作成
3. トップレベル I/O とフロアプランの制約をインポートし Place and Route に関連付け
4. タイミング制約を生成し Synthesis / Place and Route / Timing Verification に関連付け
5. 引数によっては、スクリプト実行中に設計フローの一部を実行

---

## 3. 引数一覧

### 3.1 デザイン構成引数（**排他**: 同時に 1 つだけ）

| Argument | 内容 |
|---|---|
| `I2C_LOOPBACK` | I2C0 と I2C1 をファブリックへ回し、Raspberry Pi I/O に接続した bibuf（双方向バッファ）につなぐ。I2C ループバック用制約も関連付け。I2C ループバックのベアメタル例で使用でき、**Linux も起動する**。ピン: **I2C0_SCL = RPi ヘッダ pin 29、I2C1_SCL = pin 31、I2C0_SDA = pin 35、I2C1_SDA = pin 37**。**pin 29–31、pin 35–37 を短絡**するとループバック |
| `BFM_SIMULATION` | リファレンスデザインに基づく SmartDesign テストベンチを生成し、MSS FIC にトランザクションを発生させるカスタム BFM スクリプトと、波形追加・シミュレーション実行用のカスタム DO ファイルをインポート |
| `FIR_DEMO` | Discovery Kit に**書き込み済みの FIR フィルタデザイン**を生成（AN5165、[an5165-fir-demo.md](an5165-fir-demo.md)） |
| `MIV_RV32_CFG1` | Mi-V ソフト CPU リファレンスデザイン（構成は Mi-V GitHub organisation の PolarFire-Eval-Kit 説明を参照） |
| `MIV_RV32_CFG2` | 同上（構成 2） |
| `MIV_RV32_CFG3` | 同上（構成 3） |

- 例: `I2C_LOOPBACK` と `BFM_SIMULATION` は同時指定**不可**。
- 引数は標準ベースデザインを生成した上で、その構成に変更を加える。

### 3.2 フロー・変更引数（上記や互いに**併用可**）

| Argument | 内容 |
|---|---|
| `HSS_UPDATE` | このリリースに対応する **HSS リリース hex** をダウンロードし、Libero に **eNVM クライアント**として追加。**wget が必要**（Linux は通常導入済み。Windows は wget を導入して system path に追加） |
| `SYNTHESIZE` | 生成後に合成まで実行 |
| `PLACEROUTE` | 生成後に合成と配置配線まで実行 |
| `VERIFY_TIMING` | 生成後に合成・配置配線・タイミング検証まで実行 |
| `GENERATE_PROGRAMMING_DATA` | デバイス書込み用ビットストリーム生成に必要なファイルを生成 |
| `PROGRAM` | 全フロー実行後、接続中のデバイスに書込み。**"Run PROGRAM Action" 到達時にデバイス接続必須、接続は 1 台のみ** |
| `EXPORT_FPE` | 全フロー実行後、FlashPro Express ファイルを**ローカルディレクトリ**にエクスポート |
| `EXPORT_FPE:PATH` | 全フロー実行後、FlashPro Express ファイルを指定パスにエクスポート（例 `EXPORT_FPE:/home/user/jobs/`） |
| `AXI4_STREAM_DEMO` | CoreAXI4DMAController 上で AXI4 ストリーム I/F を構成するデモ。インクリメントデータの AXI4 Stream トランザクションを生成する `AXI4_STREAM_DATA_GENERATOR` モジュールを作成し DMA に接続（性能ベンチマーク用）。**Mi-V RV32 ソフト CPU デザインとは併用不可**。ガイド: `https://mi-v-ecosystem.github.io/redirects/demo-guides_mpfs-axi4-stream-demo` |
| `DESIGN_VERSION:VERSION` | ビットストリームに含める FPGA design version（例 `DESIGN_VERSION:12345`、**16 bit 値のみ**） |
| `SILICON_SIGNATURE:SIGNATURE` | ビットストリームに含める silicon signature。既定 **`D15C0417`**。指定すると既定値を上書き |
| `SMARTHLS:PATH` | SmartHLS で C++ 記述したハードウェアモジュールを作成・自動統合。引数は SmartHLS プロジェクトディレクトリのパス。同梱例: `SMARTHLS:./script_support/additional_configurations/smarthls/invert_and_threshold` |

- 併用例: `I2C_LOOPBACK HSS_UPDATE PROGRAM`
- フロー引数の包含関係: `SYNTHESIZE` ⊂ `PLACEROUTE` ⊂ `VERIFY_TIMING`、`PROGRAM` / `EXPORT_FPE` は全フロー

### 3.3 目的別の引数選び (原文外: README の記述を用途別に再整理)

| やりたいこと | 引数例（GUI 表記） |
|---|---|
| Linux 起動できる基板を Libero から一発で作る | `HSS_UPDATE PROGRAM` |
| 配布用 .job を作る | `HSS_UPDATE EXPORT_FPE:<path>` |
| FIR デモ（工場出荷デザイン）に戻す | `FIR_DEMO PROGRAM` |
| I2C ループバックのベアメタル例を試す | `I2C_LOOPBACK HSS_UPDATE PROGRAM` |
| MSS の BFM シミュレーション | `BFM_SIMULATION` |

---

## 4. 出力物と保存場所

| 出力物 | 場所 / 内容 | 出典の節 |
|---|---|---|
| Libero プロジェクト | スクリプト実行で生成（ディレクトリ名は README に記載なし） | Using the ... Tcl script |
| ログ（CLI） | `logfile:` で指定（例 `MPFS_DISCOVERY_KIT_REFERENCE_DESIGN.log`） | Generating designs from the command line |
| MSS 構成（保存済み） | `script_support` フォルダ（MSS Configurator で開ける） | MSS Configuration |
| 引数付き生成時の MSS 構成 | `script_support/additional_configurations/[configuration name]` | MSS Configuration |
| MSS コンポーネント生成時の XML | `script_support/components/[target]` | Boot mode 1 |
| 全構成の XML | `XML` フォルダ | XML |
| MSS コンポーネント | `.cxz`（Libero にインポート） | MSS Configuration |
| MSS 構成 XML | `.xml`（PolarFire SoC Configuration Generator → MPFS HAL 用ヘッダ） | MSS Configuration / XML |
| FlashPro Express ファイル | `EXPORT_FPE`: ローカルディレクトリ / `EXPORT_FPE:PATH`: 指定パス | 引数表 |
| 事前生成済み .job | GitHub の **Releases** | Description / Embedded SW UG |
| BFM 用ファイル | Simulation フォルダの `.vec`、カスタム `wave.do` | BFM Simulation |

### FlashPro Express ジョブの書込み

- Releases の .job、または `EXPORT_FPE` で出力した .job を FlashPro Express で書き込む。
- 操作手順は AN5165 と同じ（New → job 選択 → 保存先 → OK → Programmer 確認 → RUN → `RUN PASSED`）: [an5165-fir-demo.md §6.3](an5165-fir-demo.md)

---

## 5. デザイン内容

ブロック図は README の `./diagrams/block_diagram.svg`（未取得）。

### 5.1 MSS 周辺機器

| MSS Peripheral | Route to Device I/O | Board peripheral | Board Interface | Notes |
|---|---|---|---|---|
| SD/SDIO | MSS I/Os BANK 4 | uSD card socket | N/A | |
| GEM0 | SGMII I/Os BANK 5 | VSC8221 PHY Port0 | J3 (RJ45 connector) | Management through fabric I/Os |
| SPI0 | Fabric | N/A | J5 (mikroBUS socket) | |
| SPI1 | MSS I/Os BANK 2 B | N/A | 10 (RPI header) | 原文 "10"（J10 の意と思われる (原文外)） |
| MMUART0 | Fabric | USB-UART | J4 (USB-C port) | |
| MMUART1 | MSS I/Os BANK 2 | USB-UART | J4 (USB-C port) | HSS コンソール（Embedded SW UG） |
| MMUART4 | MSS I/Os BANK 2 | USB-UART | J4 (USB-C port) | U-Boot / Linux コンソール（Embedded SW UG） |
| I2C0 | Fabric | N/A | J6 (mikroBUS socket) | |
| GPIO1 | MSS I/Os BANK 2 | §5.5 | N/A | |
| GPIO2 | Fabric | §5.6 | N/A | |

### 5.2 MSS DDR4

- 基板上 **1GB DDR4** が MSS DDR コントローラに接続。
- ソフト CPU / ファブリックイニシエータから使う場合、**HSS またはベアメタルアプリで DDR をトレーニングし、MSS の FIC ターゲットを有効化**してからアクセスする。

### 5.3 ファブリックメモリマップ（FIC 使用状況）

| Initiator | Bus | Target | Board peripheral | Board interface | Address range |
|---|---|---|---|---|---|
| MSS: FIC0 | FIC0_INITIATOR | MSS_LSRAM | N/A | N/A | `0x6000_0000 -> 0x6000_1000` |
| MSS: FIC0 | FIC0_INITIATOR | DMA_CONTROLLER | N/A | N/A | `0x6001_0000 -> 0x6001_FFFF` |
| MSS: FIC3 | FIC_3_ADDRESS_GENERATION | PWM | N/A | J6 (mikroBUS socket) | `0x4000_0000 -> 0x4000_00FF` |
| MSS: FIC3 | FIC_3_ADDRESS_GENERATION | CoreGPIO | LEDs | N/A | `0x4000_0100 -> 0x4000_01FF` |
| MSS: FIC3 | FIC_3_ADDRESS_GENERATION | CoreI2C | N/A | 10 (RPI header) | `0x4000_0200 -> 0x4000_02FF` |
| MSS: FIC3 | FIC_3_ADDRESS_GENERATION | CoreUART | N/A | J6 (mikroBUS socket) | `0x4000_0300 -> 0x4000_03FF` |
| MSS: FIC3 | FIC_3_ADDRESS_GENERATION | SPI_FOR_7_SEG | N/A | J48 (7 seg display adapter) | `0x4000_0400 -> 0x4000_04FF` |
| MSS: FIC3 | FIC_3_ADDRESS_GENERATION | MIV_IHC | N/A | N/A | `0x5000_0000 -> 0x500F_FFFF` |
| DMA_CONTROLLER | DMA_INITIATOR | MSS: FIC0 | N/A | N/A | `0x0000_0000 -> 0xFFFF_FFFF` |

FIC ごとの整理:

| FIC | 方向 | 接続先 |
|---|---|---|
| FIC0 | MSS → ファブリック（イニシエータ） | MSS_LSRAM、DMA_CONTROLLER（レジスタ） |
| FIC0 | ファブリック → MSS（ターゲット） | DMA_CONTROLLER が DMA_INITIATOR として全 32-bit 空間にアクセス |
| FIC3 | MSS → ファブリック（`FIC_3_ADDRESS_GENERATION`） | PWM, CoreGPIO, CoreI2C, CoreUART, SPI_FOR_7_SEG, MIV_IHC |

FIC3 が低速周辺（APB）用、FIC0 が AXI 用という区分は PolarFire SoC の一般的な構成 (原文外。polarfire-soc-mss の [interconnect.md](../../polarfire-soc-mss/references/interconnect.md) で確認)。

### 5.4 ファブリック IP 一覧（メモリマップ・引数説明から抽出）

| IP / インスタンス | 用途 | 基板 I/F |
|---|---|---|
| MSS_LSRAM | ファブリック LSRAM | — |
| DMA_CONTROLLER（CoreAXI4DMAController、`AXI4_STREAM_DEMO` の説明より） | DMA | — |
| PWM | PWM 出力 | J6 (mikroBUS) |
| CoreGPIO | LED1〜LED7（GPIO2 と OR） | LEDs |
| CoreI2C | I2C | RPi ヘッダ |
| CoreUART（CoreUARTapb） | UART | J6 (mikroBUS) |
| SPI_FOR_7_SEG | 7 セグ表示アダプタ用 SPI | J48 |
| MIV_IHC | Inter-Hart Communication（割込み E51 / U54_1〜4） | — |

### 5.5 割込みマップ

| Source | IRQ |
|---|---|
| DMA_CONTROLLER | MSS_INT_F3M[2] |
| mBUS_INT | MSS_INT_F2M[3] |
| CoreI2C_INT[0] | MSS_INT_F2M[4] |
| DIP SW1 | MSS_INT_F2M[6] |
| DIP SW2 | MSS_INT_F2M[7] |
| DIP SW3 | MSS_INT_F2M[8] |
| DIP SW4 | MSS_INT_F2M[9] |
| SWITCH 1 | MSS_INT_F2M[10] |
| CoreUARTapb_RXRDY | MSS_INT_F2M[11] |
| CoreUARTapb_TXRDY | MSS_INT_F2M[12] |
| CoreUARTapb_PARITY_ERR | MSS_INT_F2M[13] |
| CoreUARTapb_OVERFLOW | MSS_INT_F2M[14] |
| CoreUARTapb_FRAMING_ERR | MSS_INT_F2M[15] |
| MIV_IHC:E51_IRQ | MSS_INT_F2M[63] |
| MIV_IHC:U54_1_IRQ | MSS_INT_F2M[62] |
| MIV_IHC:U54_2_IRQ | MSS_INT_F2M[61] |
| MIV_IHC:U54_3_IRQ | MSS_INT_F2M[60] |
| MIV_IHC:U54_4_IRQ | MSS_INT_F2M[59] |

### 5.6 GPIO1（MSS I/Os BANK 2）

| Bit | Direction | Source / sink |
|---|---|---|
| 9 | I/O | LED8 |
| 20 | I/O | RPi_GPIO7 |

### 5.7 GPIO2（Fabric 経由）

| Bit | Dir | Source / sink | | Bit | Dir | Source / sink |
|---|---|---|---|---|---|---|
| 0 | I/O | RPi_GPIO4 | | 15 | I/O | RPi_GPIO20 |
| 1 | I/O | RPi_GPIO17 | | 16 | I/O | RPi_GPIO21 |
| 2 | I/O | RPi_GPIO18 | | 17 | Out | LED1（CoreGPIO と OR） |
| 3 | I/O | RPi_GPIO27 | | 18 | Out | LED2（CoreGPIO と OR） |
| 4 | I/O | RPi_GPIO22 | | 19 | Out | LED3（CoreGPIO と OR） |
| 5 | I/O | RPi_GPIO23 | | 20 | Out | LED4（CoreGPIO と OR） |
| 6 | I/O | RPi_GPIO24 | | 21 | Out | LED5（CoreGPIO と OR） |
| 7 | I/O | RPi_GPIO25 | | 22 | Out | LED6（CoreGPIO と OR） |
| 8 | I/O | RPi_GPIO5 | | 23 | Out | LED7（CoreGPIO と OR） |
| 9 | I/O | RPi_GPIO6 | | 24 | In | GND |
| 10 | I/O | RPi_GPIO12 | | 25 | In | DIP5 |
| 11 | I/O | RPi_GPIO13 | | 26 | In | DIP6 |
| 12 | I/O | RPi_GPIO19 | | 27 | In | DIP7 |
| 13 | I/O | RPi_GPIO16 | | 28 | In | DIP8 |
| 14 | I/O | RPi_GPIO26 | | | | |

### 5.8 CoreGPIO

| Bit | Direction | Source / sink | Notes |
|---|---|---|---|
| 0〜6 | Out | LED1〜LED7（bit n → LED n+1） | GPIO2 と OR |

LED の制御元まとめ: LED1〜LED7 = GPIO2[17:23] OR CoreGPIO[0:6]、LED8 = GPIO1[9]。DIP1〜4 は割込み（F2M[6:9]）、DIP5〜8 は GPIO2[25:28]。

---

## 6. FPGA の書き込み（Libero）

- スクリプト完了後、必要ならデザインを変更し、Design Flow の任意のステージをダブルクリック。
- 前段が未完了のステージを選ぶと必要な工程を全部実行する（例: **Run Program Action** をダブルクリック → Synthesize, Place and Route 等を実行して書き込み）。
- ジャンパ設定・eMMC / SD 用 Linux イメージは MPFS Discovery Kit User Guide（Embedded SW UG、[embedded-software.md](embedded-software.md)）を参照。

## 7. BFM シミュレーション

`BFM_SIMULATION` で生成した場合:

- [ ] 1. Libero の **Stimulus Hierarchy** を開く
- [ ] 2. **Test_bench** を右クリック
- [ ] 3. **Simulate Pre-Synth Design** → **Open Interactively**
- BFM コマンドは Libero の **Files** タブ → Simulation フォルダの **`.vec`** ファイルを編集

## 8. ブートモードと eNVM

SoftConsole のブートモードプログラマ設定（die `MPFS095T` / package `FCSG325`）、boot mode 0 / 1 の手順は [an5282-design-flow.md §10](an5282-design-flow.md) にチェックリスト形式でまとめた。要点:

| Boot mode | 動作 | ツール |
|---|---|---|
| 0 | 全 MSS コア WFI、デバッガ接続待ち | SoftConsole v6.5 以降 |
| 1 | 全 MSS コアが eNVM から起動（HSS またはベアメタルアプリを eNVM クライアントに） | SoftConsole v6.4 以上 / Libero SoC v12.6 以上 |
