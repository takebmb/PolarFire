# Macro Library User Guide: I/O バッファ・差動バッファ・UJTAG・クロッキングマクロ（PDF p.32-46）

## Core Idea
HDL の最上位ポートにつなぐ I/O マクロは単端（INBUF/OUTBUF/TRIBUFF/BIBUF）と差動（`*_DIFF`、ポート `PADP`/`PADN`）の 2 系統。**LVDS 受信は INBUF_DIFF、LVDS 送信は OUTBUF_DIFF**、差動クロック入力は CLKBUF_DIFF / GCLKBUF_DIFF。I/O 規格（LVDS25 等）はマクロではなく PDC の `set_io -io_std` で決める。

## Frameworks Introduced
- **差動 I/O マクロの選び方**
  | 用途 | マクロ |
  |---|---|
  | LVDS データ受信 | INBUF_DIFF |
  | LVDS データ送信 | OUTBUF_DIFF |
  | 差動クロックをグローバル網へ | CLKBUF_DIFF（常時）/ GCLKBUF_DIFF（EN でゲート） |
  | 差動トライステート/双方向 | TRIBUFF_DIFF / BIBUF_DIFF（ただし LVDS25/33/18G は Tribuff/Inout 非対応 → pdc-02） |
- **ファブリック内部信号をクロック網へ**: 全チップ → CLKINT（ゲート付き GCLKINT）、行グローバル（ローカルクロック）→ RCLKINT（ゲート付き RGCLKINT）、降格させない → CLKINT_PRESERVE

## Key Concepts
- **PAD / PADP / PADN**: パッケージピンに直結するポート。PADP=正、PADN=負
- **Y / D / E**: Y=ファブリックへの入力データ、D=出力データ、E=出力イネーブル（High で駆動）
- **global network**: 全チップのグローバルクロック網
- **row global buffer**: 行グローバルバッファ。局所クロックを作る
- **EN（GCLK 系）**: Low でクロック網を止めて省電力。真理値表の q は内部ラッチ値
- **UJTAG / UJTAG_SEC**: ユーザー JTAG 回路へのアクセス。TMS, TDI, TCK, TRSTB, TDO は最上位ポートに接続必須

## Reference Tables
**単端 I/O マクロ**
| マクロ | 入力 | 出力 | 動作 |
|---|---|---|---|
| INBUF | PAD | Y | Y=PAD（PAD=Z で Y=X） |
| OUTBUF | D | PAD | PAD=D |
| TRIBUFF | D, E | PAD | E=0 で Z、E=1 で D |
| BIBUF | D, E, PAD | PAD, Y | E=1: PAD=D, Y=D / E=0: Y=PAD |
| CLKBUF | PAD | Y | グローバル網への入力 |
| CLKBIBUF | D, E, PAD | PAD, Y | 双方向＋入力をグローバル網へ |
| GCLKBUF | PAD, EN | Y | EN でゲート |

**差動 I/O マクロ（PDF 画像で確認）**
| マクロ | 入力 | 出力 | 真理値表の要点 |
|---|---|---|---|
| **INBUF_DIFF** | PADP, PADN | Y | (P,N)=(1,0)→1、(0,1)→0、(0,0)/(1,1)/(Z,Z)→X |
| **OUTBUF_DIFF** | D | PADP, PADN | D=0→(0,1)、D=1→(1,0) |
| TRIBUFF_DIFF | D, E | PADP, PADN | E=0→(Z,Z)、E=1 で D に従う |
| BIBUF_DIFF | D, E, PADP, PADN | PADP, PADN, Y | E=1: 出力（D=0→(0,1), Y=0）/ E=0: 入力（(1,0)→Y=1, (0,1)→Y=0, 同値/Z→X） |
| CLKBUF_DIFF | PADP, PADN | Y | INBUF_DIFF と同じ表（原文の表題も INBUF_DIFF のまま） |
| GCLKBUF_DIFF | PADP, PADN, EN | Y | (1,0) で q を出力、EN でゲート |

**クロッキング（Ch4）**
| マクロ | 入力 | 用途 |
|---|---|---|
| CLKINT | A | ファブリック信号 → グローバル網 |
| CLKINT_PRESERVE | A | 同上。実装中に降格されずグローバルに残る |
| GCLKINT | A, EN | ゲート付き CLKINT |
| RCLKINT | A | 行グローバルバッファ（ローカルクロック） |
| RGCLKINT | A, EN | ゲート付き RCLKINT |

**UJTAG ポート（Table 3-27）**
| ポート | 方向 | 内容 |
|---|---|---|
| UIREG[7:0] | Out | JTAG 命令レジスタ。16〜127 はユーザー定義命令に使える |
| URSTB | Out | Low アクティブ。TAP が Test-Logic-Reset のとき |
| UTDI / UTDO | Out / In | TAP の TDI / ユーザー TDO（IR がユーザー範囲のとき TDO へ） |
| UDRSH / UDRCAP / UDRUPD | Out | Shift_DR / Capture_DR / Update_DR で High |
| UDRCK | Out | TAP の TCK。CLKINT 等グローバルマクロに接続（しなければ Synthesis/Compile が追加） |
| TCK, TDI, TMS, TRSTB, TDO | — | 最上位ポートへ。TCK は内部プル無し（500Ω–1kΩ で GND か 3.3V、未使用なら GND）。TDI/TMS/TRSTB は内部弱プルアップ。TRSTB は 1kΩ で GND に落とすとリセット保持 |
UJTAG_SEC は追加で EN_SEC（High で TDI/TRSTB を上書き、未使用時 Low 固定）, TDI_SEC, TRSTB_SEC を持つ。

## Code Examples
LVDS 受信と送信の Verilog インスタンス例（ポート名は原文のマクロ定義どおり。周辺コードは原文外）:
```verilog
module lvds_top (
    input  wire RX_P, RX_N,      // Bank 1 GPIO, set_io -io_std LVDS25
    output wire TX_P, TX_N,
    input  wire tx_data,
    output wire rx_data
);
  INBUF_DIFF  u_rx (.PADP(RX_P), .PADN(RX_N), .Y(rx_data));
  OUTBUF_DIFF u_tx (.D(tx_data), .PADP(TX_P), .PADN(TX_N));
endmodule
```
DDR やギアリング（IOD）を使う場合はマクロ直結ではなく IOD 系 IP（Libero カタログの PF_IOD_GENERIC_RX/TX 等。原文外）を使う。本書は IOD の DDR マクロを収録していない（IOD レジスタは post-layout の IOREG のみ → mlg-05）。

## Anti-patterns
- **LVDS 送信に TRIBUFF_DIFF / BIBUF_DIFF**: LVDS25/33/18G は Tribuff/Inout 非対応（pdc-02）
- **差動クロックを INBUF_DIFF で受けてファブリックで配る**: グローバル網に載せるなら CLKBUF_DIFF / GCLKBUF_DIFF
- **UJTAG の UDRCK をグローバルに通さない**: ツールが勝手に挿入する
- **UJTAG_SEC の EN_SEC を未接続**: Low に固定

## Key Takeaways
1. LVDS 受信=INBUF_DIFF（PADP, PADN → Y）、送信=OUTBUF_DIFF（D → PADP, PADN）
2. 差動クロック入力は CLKBUF_DIFF / GCLKBUF_DIFF
3. IOSTD はマクロでなく PDC で指定
4. 内部生成クロックは CLKINT / RCLKINT 系でクロック網へ
5. 本書に IOD の DDR/ギアリングマクロは無い

## Connects To
- **pdc-02-set-io-iostd**: 方向制限（Tribuff/Inout）と差動対応
- **pdc-03-set-io-attributes**: LVDS の ODT・VCM・OUT_DRIVE
- **mlg-05-post-layout-macros**: IO_DIFF（N 側予約）、IOPADP_*/IOPADN_*、IOREG
- **ioed-02-interface-views**: IOD View（RX_DDR_L_A / TX_DDR_G_A）
