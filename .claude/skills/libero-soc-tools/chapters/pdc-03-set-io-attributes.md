# PDC Commands Reference Guide: set_io の I/O 属性（ODT・VCM・RES_PULL ほか）（PDF p.11-15）

## Core Idea
`set_io` の属性引数は IOSTD ごとに許される値と既定値が違う。**GPIO の LVDS25/LVDS33/LVDS18G 入力は ODT 既定 ON・ODT_VALUE 100Ω 固定・VCM_RANGE 既定 MID・RES_PULL 既定 Up**。出力は OUT_DRIVE（既定 6mA）と SOURCE_TERM（OFF/100）を持つ。

## Frameworks Introduced
- **LVDS 受信属性の決め方**
  1. 終端: 外付け 100Ω が無ければ `-ODT ON -ODT_VALUE 100`（LVDS の ODT 値は 100 のみ）
  2. 同相電圧: 送信側に合わせて `-VCM_RANGE MID|LOW`。**同じバンクの真差動入力は全部同じ値**にする
  3. フェイルセーフ（入力が浮く可能性がある）: `-ODT DYNAMIC` と `-RES_PULL` で弱プル
- **LVDS 送信属性の決め方**: 駆動電流 `-OUT_DRIVE 3|3.5|4|6`、近端終端 `-SOURCE_TERM OFF|100`
- **VCM を同一バンクで混在させたい場合**: 配置配線後に Post Layout I/O Editing と `edit_io` で個別に設定（pdc-05）。このとき内蔵 100Ω 差動終端の精度は 2 レンジのうち悪い方の許容差になる

## Key Concepts
- **ODT（On-Die Termination）**: 終端抵抗をチップ内に持つ技術。入力方向
- **ODT DYNAMIC**: LVDS フェイルセーフモードで使う値
- **VCM_RANGE**: 差動入力の同相電圧レンジ（MID/LOW）
- **RES_PULL**: 弱プルアップ/ダウン/Hold。差動出力では非対応
- **CLAMP_DIODE**: 電源クランプダイオード。HSIO は常に ON
- **SOURCE_TERM**: 差動出力の近端終端
- **IN_DELAY / OUT_DELAY**: 静的遅延タップ。DDR/LPDDR/QDR メモリや I/O CDR など動的遅延 I/O では使えない
- **LOCK_DOWN**: 改ざん検知時に I/O をロックダウン（ON/OFF、既定 OFF）

## Commands & Syntax
LVDS25 受信（Bank 1、P 側ピンは基板の回路図から。値は原文の表から選択、組み合わせ例は原文外）:
```tcl
set_io -port_name LVDS_RX_P -pin_name <P_pin> -fixed true -io_std LVDS25 \
  -ODT ON -ODT_VALUE 100 -VCM_RANGE MID -RES_PULL Up -DIRECTION INPUT
```
LVDS25 送信:
```tcl
set_io -port_name LVDS_TX_P -pin_name <P_pin> -fixed true -io_std LVDS25 \
  -OUT_DRIVE 6 -SOURCE_TERM OFF -DIRECTION OUTPUT
```
LVDS フェイルセーフ（原文 p.15）: `set_io -RES_PULL <value>` と `set_io -ODT DYNAMIC`

## Reference Tables
**LVDS（GPIO: LVDS25 / LVDS33 / LVDS18G）の属性まとめ**
| 属性 | 方向 | 値 | 既定 |
|---|---|---|---|
| -ODT | In | OFF, ON, DYNAMIC | ON |
| -ODT_VALUE | In | 100 | 100 |
| -VCM_RANGE | In | MID, LOW | MID |
| -RES_PULL（入力） | In | Up, Down（差動に None/Hold の記載なし）※ | Up |
| -RES_PULL（出力） | Out | 非対応 | — |
| -CLAMP_DIODE | Inout | LVDS33: ON のみ / LVDS25: 表が矛盾（OFF/ON 既定 ON の行と ON のみの行の両方に記載）/ LVDS18G: OFF, ON 既定 OFF（入力は OFF のみ） | — |
| -OUT_DRIVE | Out | 3, 3.5, 4, 6 | 6 |
| -SOURCE_TERM | Out | OFF, 100 | OFF |
| -SCHMITT_TRIGGER / -SLEW | — | OFF のみ | OFF |
| -IN_DELAY | In | OFF, 0–127, 128, 130, 132 … 254 | OFF |
| -OUT_DELAY | Out | OFF, 0–127 | OFF |

※ 差動入力の RES_PULL 対象: PPDS25/33, HCSL33/25, BUSLVDSE25, LVDS33/25, LVDS18G, MINILVDS33/25, RSDS33/25, LVPECL33, SUBLVDS33/25, LCMDS33/25。I/O Editor 側の注記では P/N で異なる値を設定できず、表示上「Up」でも N 側は Down にプログラムされる（ioed-01）。

**-ODT（Table 3-15）**
| IOSTD | 値 | 既定 |
|---|---|---|
| LVCMOS12/15/18/25 | OFF, ON | OFF |
| HSUL18I/II | OFF, ON, DYNAMIC | OFF |
| SSTL15/18, HSUL12I, LVSTL11, POD12, SSTL135, HSTL15/12/135, **LVDS18G, LVDS33, LVDS25**, LVPECL33/E33/25, MINILVDS33/25, RSDS33/25, SUBLVDS33/25, LCMDS33/25 | OFF, ON, DYNAMIC | ON |

**-ODT_VALUE（Table 3-16、PDF 画像で確認）**
| IOSTD | 値 |
|---|---|
| LVCMOS12/15/18 | 120, 240（既定 120） |
| LVCMOS25 | 120 |
| HSUL12I | 60, 120, 240（既定 120） |
| SSTL15I/II | 20, 30, 40, 60, 120（既定 30） |
| SSTL135I/II | 20, 30, 40, 60, 120（既定 40） |
| SSTL18I/II | 50, 75, 150（既定 50） |
| POD12I/II | 34, 40, 48, 60, 80, 120, 240（既定 60） |
| **LVDS18G, LVDS33, LVDS25**, LVPECL33/25, MINILVDS33/25, RSDS33/25, SLVSE15, SUBLVDS33/25, LCMDS33/25 | **100** |
| HSTL15/12/135, HSUL18 | 50 |

**-OUT_DRIVE（Table 3-17、mA）**
| GPIO IOSTD | 値（既定） |
|---|---|
| LVCMOS12 | 2,4,6,8（8） |
| LVCMOS15 | 2,4,6,8,10（8） |
| LVCMOS18 | 2,4,6,8,10,12（8） |
| LVCMOS25 | 2,4,6,8,12,16（8） |
| LVCMOS33, LVTTL | 2,4,8,12,16,20（8） |
| **LVDS18G, LVDS25, LVDS33**, MINILVDS25/33, LCMDS33/25 | **3, 3.5, 4, 6（6）** |
| PPDS25/33, RSDS25/33 | 1.5, 2, 3（3） |
| SUBLVDS25/33 | 1, 1.5, 2（2） |
| BUSLVDSE25, MLVDSE25, LVPECLE33 | 16 |
| MIPIE25, SLVSE15 | 8 |
| PCI | 20 |
HSIO: LVCMOS12/15 = 2,4,6,8,10（8）、LVCMOS18 = 2..12（8）、SLVSE15 = 8

**-VCM_RANGE（Table 3-14）**: GPIO の HCSL33/25, LVDS18G, LVDS33, LVDS25, LVPECL33/E33, MINILVDS33/25, MIPI25/E25, MLVDSE25, PPDS33/25, RSDS33/25, SLVS33/25, SLVSE15, BUSLVDSE25, SUBLVDS33/25 は MID/LOW（既定 MID）。SSTL/HSTL/HSUL は MID のみ。LCMDS33/25 は LOW のみ。

**その他**: -OUT_LOAD 既定 5 pF。-SCHMITT_TRIGGER は GPIO の LVCMOS25/33, LVTTL, PCI と HSIO の LVCMOS18/15 のみ OFF/ON。-SLEW は LVCMOS25/33, LVTTL, PCI のみ OFF/ON。-IMPEDANCE は IOSTD 依存（出力）。

## Anti-patterns
- **外付け 100Ω 終端があるのに ODT 既定 ON のまま**: LVDS 入力の ODT 既定は ON。二重終端になる（外付け有無の判断は原文外）
- **同一バンクで VCM_RANGE MID と LOW を混在**: PDC では不可。edit_io で post-layout 設定する
- **差動出力に RES_PULL**: 非対応
- **入力 LVDS18G に CLAMP_DIODE ON**: OFF のみ

## Key Takeaways
1. LVDS 入力: ODT 既定 ON、ODT_VALUE は 100 のみ、VCM 既定 MID
2. LVDS 出力: OUT_DRIVE 3/3.5/4/6 mA（既定 6）、SOURCE_TERM OFF/100
3. フェイルセーフは ODT DYNAMIC + RES_PULL
4. VCM レンジはバンク内で統一
5. 動的遅延 I/O（DDR メモリ、CDR）では IN_DELAY/OUT_DELAY を PDC で書けない
6. CLAMP_DIODE の LVDS25 は原文表が自己矛盾している

## Connects To
- **pdc-02-set-io-iostd**: 方向制限
- **pdc-05-post-layout-edit**: edit_io で VCM 混在などを post-layout 調整
- **ioed-01-port-pin-view**: 同じ属性の GUI 列（On-Die Termination, Vcm Input Range, Source Termination）
- **mlg-03-special-macros**: BANKCTRL_GPIO（GPIO バンクのキャリブレーション制御）
