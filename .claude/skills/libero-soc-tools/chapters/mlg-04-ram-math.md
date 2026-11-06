# Macro Library User Guide: RAM1K20・RAM64x12・MACC_PA・MACC_PA_BC_ROM（PDF p.57-78）

## Core Idea
ファブリックのハードブロック 3 種。**RAM1K20**（LSRAM、20,480bit、真のデュアルポート/2 ポート/ECC）、**RAM64x12**（µSRAM、768bit、1W1R）、**MACC_PA**（18x18 符号付き乗算＋48bit 加算/累算、プリアダ、ドット積、SIMD、カスケード）。通常は Libero の IP 生成で使うが、直接インスタンスする場合は**静的入力を 0/1 に固定**し、未使用ビットの結線規則に従う。

## Frameworks Introduced
- **RAM1K20 のモード判定**: 少なくとも一方のポート幅が 20 を超えると 2 ポートモード（A=読み出し、B=書き込み。A_CLK=読みクロック、B_CLK=書きクロック）。ECC は 2 ポートで両ポート幅 > 20 のときのみ（512x33）
- **RAM64x12 の読み出し 4 モード**: 読みアドレス 同期/非同期 × 読みデータパイプライン 有/無（sync-sync, sync-async, async-sync, async-async）
- **MACC_PA で使わないレジスタの処理**: `*_BYPASS`=1、`*_EN`=1、`*_SRST_N`/`*_ARST_N`=1、AL_N=1（何もレジスタしないとき）

## Key Concepts
- **A_WMODE / B_WMODE**: 00=読み出しポート保持、01=Feed-through（書き込みデータが読み出しに出る）、10=Read-before-write。01/10 は 2 ポートモードでは無効
- **A_BLK_EN / B_BLK_EN[2:0]**: 111 で動作、いずれかが 0 なら無動作（読みデータは 0）
- **A_BYPASS（Low アクティブ）**: 読みデータパイプラインレジスタ選択
- **ECC_EN / ECC_BYPASS**: 0x=ECC 無効、10=ECC パイプライン（読みが 1 クロック増）、11=ECC 非パイプライン。SB_CORRECT（1bit 訂正）、DB_DETECT（複数 bit 検出・未訂正）
- **BUSY_FB / ACCESS_BUSY**: FCB からのアクセスをロック / FCB アクセス中表示
- **CDIN / CDOUT[47:0]**: MACC 間カスケード。バス全体を別の MACC_PA(_BC_ROM) の全体に接続
- **ARSHFT17**: オペランド E を 17bit 算術右シフト（SIMD 時は 0）
- **DOTP / SIMD**: ドット積（9bit×2 組）/ 9bit 独立乗算×2。同時に 1 にしない

## Reference Tables
**RAM1K20 構成**
| モード | 構成 |
|---|---|
| 非 ECC デュアルポート | 各ポート 1Kx20, 2Kx10, 4Kx5, 8Kx2, 16Kx1（25 通り） |
| 非 ECC 2 ポート | 各ポート 512x40, 1Kx20, 2Kx10, 4Kx5, 8Kx2, 16Kx1（36 通り） |
| ECC 2 ポート | 512x33 / 512x33 |

**A_WIDTH/B_WIDTH とアドレス（Table 7-2/7-4）**
| WIDTH | 構成 | 使用アドレス | 0 固定ビット |
|---|---|---|---|
| 000 | 16Kx1 | [13:0] | なし |
| 001 | 8Kx2 | [13:1] | [0] |
| 010 | 4Kx4, 4Kx5 | [13:2] | [1:0] |
| 011 | 2Kx8, 2Kx10 | [13:3] | [2:0] |
| 100 | 1Kx16, 1Kx20 | [13:4] | [3:0] |
| 101 | 512x32/512x40（2 ポート）, 512x33（ECC） | [13:5] | [4:0] |
アドレスは MSB 詰め、データは LSB 詰め（未使用ビットは 0 に固定）。2 ポートの書き込みデータは A_DIN が上位、B_DIN が下位。

**RAM1K20 主要ポート（Table 7-1）**: A_/B_ ADDR[13:0], BLK_EN[2:0], CLK（立ち上がり）, DIN[19:0], DOUT[19:0], WEN[1:0]（バイト単位）, REN, WIDTH[2:0]（静的）, WMODE[1:0]（静的）, BYPASS（静的）, DOUT_EN, DOUT_SRST_N, DOUT_ARST_N / ECC_EN, ECC_BYPASS（静的）, SB_CORRECT, DB_DETECT, BUSY_FB（静的）, ACCESS_BUSY

**RAM64x12 主要ポート（Table 8-1）**: W_EN, W_CLK, W_ADDR[5:0], W_DATA[11:0] / BLK_EN, R_CLK, R_ADDR[5:0], R_ADDR_BYPASS（静的）, R_ADDR_EN, R_ADDR_SL_N, R_ADDR_SD（静的）, R_ADDR_AL_N, R_ADDR_AD_N（静的）/ R_DATA[11:0], R_DATA_BYPASS（静的）, R_DATA_EN, R_DATA_SL_N, R_DATA_SD, R_DATA_AL_N, R_DATA_AD_N / BUSY_FB, ACCESS_BUSY

**MACC_PA の演算（Table 9-3 抜粋）**
| SIMD | DOTP | 結果 P / CDOUT |
|---|---|---|
| 0 | 0 | CARRYIN + C[47:0] + E[47:0] ± { (B[17:0] ± D[17:0]) x A[17:0] }（外側の ± は SUB、内側は PASUB） |
| 0 | 1 | CARRYIN + C + E + { (B[8:0] ± D[8:0]) x A[17:9] ± (B[17:9] ± D[17:9]) x A[8:0] } x 2^9（原文表記 "x 29"。上付きの欠落と解釈） |
| 1 | 0 | P[17:0] = CARRYIN + B[8:0] x A[8:0]、P[47:18] = C[47:18] + E[47:18] ± (B[17:9] ± D[17:9]) x A[17:9] |

**オペランド E の選択（Table 9-2）**: CDIN_FDBK_SEL=00 → 0 / 01 → P（ARSHFT17=1 で 17bit 算術右シフト）/ 1x → CDIN（同）。CDIN_FDBK_SEL[0]=1 のとき P_BYPASS=0 必須。
**OVFL_CARRYOUT_SEL**: 0=オーバーフロー（SUM[49]^SUM[48] | SUM[48]^SUM[47]）、1=キャリーアウト（C[47]^E[47]^SUM[48]、ファブリックで加算器を延長する用途）

**MACC_PA_BC_ROM の追加**: USE_ROM（1 で A を 16x18 ROM の ROM_ADDR[3:0] 番地から）、B2[17:0] / BCOUT[17:0]（B のパイプライン出力とカスケード）。**P と B2 は同一ブロックで同時に使えない**（片方は未接続）。INIT パラメータ [287:0] に 16 語 × 18bit（先頭 18bit が word 0）

## Anti-patterns
- **同一アドレスへの同時読み書き**: 読み値は不定（衝突検出なし、書き込みは正しく行われる）
- **2 ポートモードで WMODE=01/10**: 無効
- **静的入力（WIDTH, WMODE, BYPASS, ECC_*, BUSY_FB 等）をロジックで駆動**: 0/1 固定
- **CDOUT の一部ビットだけ使う**: バス全体を別 MACC の CDIN へ、または全未接続
- **SIMD=1 で D[8:0], C[17:0], E[17:0] を非 0**: 0 必須

## Key Takeaways
1. RAM1K20 は幅 > 20 で 2 ポート化（A=読み、B=書き）。ECC は 512x33 のみ
2. アドレスは MSB 詰め・未使用 LSB は 0、データは LSB 詰め
3. RAM64x12 は 1W1R、読み出し同期/非同期を選べる
4. MACC_PA: 18x18 + 48bit 累算、プリアダ、ドット積、SIMD、CDIN/CDOUT カスケード
5. 不要レジスタは BYPASS=1・EN=1・リセット=1 で殺す

## Connects To
- **mlg-05-post-layout-macros**: RAM1K20_IP, RAM64x12_IP, MACC_IP
- **mlg-01-naming-logic-sequential**: 周辺ロジック
