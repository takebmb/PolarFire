# Macro Library User Guide: 命名規則・組合せ論理・順序論理マクロ（PDF p.1-31）

## Core Idea
PolarFire / PolarFire SoC のプリミティブ（マクロ）を HDL から直接インスタンスするためのリファレンス。順序マクロ名は機能・極性を規則的に表す。**組合せ・順序マクロは MX4 と XOR8 を除き 1 論理エレメント**を使う。実体は 4 入力 LUT（CFG1〜CFG4）と SLE（順序論理エレメント）。

## Frameworks Introduced
- **順序マクロ名の読み方**: `DF|DL` + `I|N` + `1|0` + [`E0|E1`] + [`P0|P1`] + [`C0|C1`]
  - DF=D フリップフロップ / DL=D ラッチ
  - I=出力反転（QN）/ N=非反転（Q）
  - 1=立ち上がりエッジ・High 透過 / 0=立ち下がり・Low 透過
  - E0/E1=Enable 負/正論理、P0/P1=非同期 Preset 負/正論理、C0/C1=非同期 Clear 負/正論理
  - 例: `DFN1E1C0` = 立ち上がり DFF、Q 非反転、High アクティブ Enable、Low アクティブ非同期 Clear
- **真理値表の記号**: 0, 1, X（入力は Don't care、出力は不定）, Z（ハイインピーダンス）

## Key Concepts
- **WARNING_MSGS_ON**: 警告表示を切るためのパラメータ/ジェネリック（既定 ON = VHDL 'True' / Verilog '1'）
- **CFG1/2/3/4**: 1〜4 入力 LUT（post-layout）。INIT 幅は CFG2=4bit, CFG3=8bit, CFG4=16bit。出力 Y = INIT[入力値]（CFG4 は DCBA をインデックスに INIT[0..15]）
- **ARI1**: 算術演算を表すマクロ（pre-layout）。20bit INIT。下位 16bit が Y（A=0 で F0、A=1 で F1）、INIT[17:16] が生成 G、INIT[19:18] が伝搬 P。S = Y xor FCI、FCO は P=0 なら G、P=1 なら FCI
- **BUFD / INVD**: Compile 最適化で削除されないバッファ/インバータ（BUFF/INV は削除されうる）
- **SLE**: 順序論理エレメント。D, CLK, EN（High）, ALn（非同期ロード、Low）, ADn（非同期ロード値）, SLn（同期ロード、Low）, SD（同期ロード値）, LAT（High でラッチモード）
- **SLE_DEBUG**: SmartDebug 用にレジスタを保存・識別・分類する
- **SLE_INIT**: SLE の派生（詳細は SLE 参照）

## Reference Tables
**組合せマクロ（p.7-24）**
| マクロ | 機能 | 入力 |
|---|---|---|
| AND2/3/4, NAND2/3/4, OR2/3/4, NOR2/3/4 | 論理積/和と反転 | A, B[, C, D] |
| XOR2/3/4 | 排他的論理和 | A, B[, C, D] |
| XOR8 | 8 入力 XOR（奇数個 High で 1）。**2 論理モジュール** | A–H |
| MX2 | 2:1 MUX（S=0→A, S=1→B） | A, B, S |
| MX4 | 4:1 MUX。**2 論理モジュール** | D0–D3, S0, S1 |
| BUFF / BUFD | バッファ / 削除されないバッファ | A |
| INV / INVD | インバータ / 削除されないインバータ | A |
| ARI1 | 算術（キャリーチェーン） | A, B, C, D, FCI → Y, S, FCO |
| CFG1–4 | 任意の 1〜4 入力論理（INIT） | A[, B, C, D] |

**順序マクロ（p.25-31）**
| マクロ | 機能 | ポート |
|---|---|---|
| DFN1 | 立ち上がり DFF | D, CLK → Q |
| DFN1C0 | + Low アクティブ非同期 Clear | D, CLK, CLR |
| DFN1E1 | + High アクティブ Enable | D, E, CLK |
| DFN1E1C0 | Enable + 非同期 Clear | CLR, D, E, CLK |
| DFN1E1P0 | Enable + Low アクティブ非同期 Preset | D, E, PRE, CLK |
| DLN1 | High 透過ラッチ | D, G |
| DLN1C0 / DLN1P0 | + Low アクティブ Clear / Preset | CLR or PRE, D, G |
| SLE | 汎用順序エレメント | D, CLK, EN, ALn, ADn, SLn, SD, LAT → Q |

**SLE の要点（Table 2-17/2-18）**: ADn, SD, LAT は設計時に 0/1 に固定する静的信号。ALn=0 で Q=!ADn。LAT=0 で立ち上がりエッジ時に EN=1 かつ SLn=0 なら Q=SD、SLn=1 なら Q=D。

## Anti-patterns
- **電源投入時の FF 状態に依存**: 原文は SmartFusion 2 / IGLOO 2 について「初期状態不定、リセットで初期化を推奨」と警告（PolarFire 向けの明記はない）
- **SLE の ADn/SD/LAT を動的に駆動**: 静的信号として 0/1 固定
- **最適化で消したくないバッファに BUFF**: BUFD/INVD を使う

## Key Takeaways
1. マクロ名で FF の極性・Enable・Preset/Clear が読める
2. ほぼ全マクロ 1 LE、MX4 と XOR8 は 2
3. LUT は CFG1〜4（INIT 指定）、算術は ARI1 のキャリーチェーン
4. SLE が順序要素の実体。静的ピン（ADn/SD/LAT）は固定
5. BUFD/INVD は Compile で消えない

## Connects To
- **mlg-02-io-clocking**: I/O バッファとクロックマクロ
- **mlg-05-post-layout-macros**: CFG0, CFG1A–D, CFG4A, CC_CONFIG など post-layout 版
- **pdc-04-location-ndc-floorplan**: set_preserve（インスタンスを Compile で結合させない）
