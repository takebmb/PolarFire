# PDC Commands Reference Guide: set_io の構文・IOSTD・方向制限（PDF p.8-11, 16）

## Core Idea
`set_io` はポートに I/O 規格を割り当て、パッケージピンに配置・ロックする中心コマンド。IOSTD ごとに「単端/差動の可否」と「使えない方向（Input/Output/Tribuff/Inout）」が決まっている。**GPIO の LVDS25/LVDS33/LVDS18G は Input と Output にのみ使え、Tribuff と Inout には使えない**。

## Frameworks Introduced
- **set_io を書く順序**
  1. バンク電圧を決める（`set_iobank`、pdc-01）
  2. `-port_name` に HDL 最上位ポート名、`-pin_name` にパッケージピン、`-io_std` に規格
  3. `-fixed true` でロック（既定 true）
  4. 必要な属性（ODT, VCM_RANGE, RES_PULL …）を追加（pdc-03）
- **IOSTD 変更時の挙動**: 同じバンクの他 I/O と電圧非互換の IOSTD を割り当てると、そのポートの配置は無効になり自動で unassign される

## Key Concepts
- **-port_name**: I/O マクロのポート名
- **-pin_name**: 配置先パッケージピン名
- **-io_std**: I/O 規格（Table 3-7）
- **-fixed**: 配置ロック。ロックしたポートはレイアウトで動かない（既定 true）
- **Single / Differential**: その IOSTD が単端・差動のどちらで使えるか
- **Tribuff / Inout**: トライステート出力・双方向。差動規格の多くは不可
- **SHIELD**: SHIELD33/25/18/15/135/12。単端のみ、Input/Inout 不可

## Commands & Syntax
```tcl
set_io -port_name <port_name> \
  [-pin_name <package_pin>] \
  [-fixed <true|false>] \
  [-io_std <io_std_values>] \
  [-OUT_LOAD <value>] \
  [-RES_PULL <value>] \
  [-LOCK_DOWN <value>] \
  [-CLAMP_DIODE <value>] \
  [-SCHMITT_TRIGGER <value>] \
  [-SLEW <value>] \
  [-VCM_RANGE <value>] \
  [-ODT <value>] \
  [-ODT_VALUE] \
  [-OUT_DRIVE <value>] \
  [-IMPEDANCE <value>] \
  [-SOURCE_TERM <value>] \
  [-IN_DELAY <value>] \
  [-OUT_DELAY <value>]
```
原文のサンプル（p.3）では `-DIRECTION INPUT` / `-DIRECTION OUTPUT` も付いている（構文一覧には無い）。
```tcl
set_io -port_name EXT_RST_N -pin_name AD4 -fixed true -io_std LVCMOS15 \
  -IN_DELAY 6 -LOCK_DOWN Yes -ODT ON -ODT_VALUE 240 -RES_PULL Down \
  -SCHMITT_TRIGGER ON -DIRECTION INPUT
```
バス要素は `[ ]` をエスケープ: `set_io -port_name IO_in\[2\] -io_std LVCMOS25 -fixed true`

**差動ペアの指定**: 原文には P/N の両ピンを書く方法の記述がない。post-layout マクロ IO_DIFF が「N 側の位置を予約するプレースホルダ」（mlg-05）なので、P 側ピンだけを `-pin_name` に指定し N 側はツールが確保する、と読める（原文外の推定。I/O Editor の Pin View の Function 列で P/N パッドを確認すること）。

## Reference Tables
**-io_std と単端/差動（Table 3-7、PDF 画像で確認）**
| -io_std | Single | Differential |
|---|---|---|
| LVTTL, LVCMOS33/25/18/15/12, PCI | YES | NO |
| POD12I, POD12II, HSTL15I/II, HSTL135I/II, HSTL12I/II, SSTL18I, SSTL15I, SSTL135I/II, SSTL25I/II, HSUL18I/II, HSUL12I/II | YES | YES |
| SSTL18II, SSTL15II | YES | NO |
| **LVDS, LVDS25, LVDS18, LVDS18G**, BUSLVDS, BUSLVDSE, BUSLVDSE25, MLVDS, MLVDSE, MLVDSE25 | NO | YES |
| PPDS33/25/18, SLVS33/25/18, HCSL33/25/18, SLVSE15, MIPI25, MIPIE25, MIPIE33 | NO | YES |
| MINILVDS(33/25/18), RSDS(33/25/18), SUBLVDS33/25/18, LCMDS33/25/18 | NO | YES |
| LVPECL（入力のみ）, LVPECL33, LVPECLE33 | NO | YES |
| SHIELD33/25/18/15/135/12 | YES | NO |

注: Table 3-7 に `LVDS33` の行は無い（他の表では GPIO 対応として列挙）。原文の欠落とみられる。

**set_io の GPIO 対応規格（Table 3-6）**: LVTTL, LVCMOS33, PCI, LVCMOS12/15/18/25, SSTL25I/II, SSTL18I/II, HSUL18I/II, SSTL15I/II, HSTL15I/II, SLVS33, HCSL33, HCSL25, MIPI25, MIPIE25, LVPECL33, LVPECL25, LVPECLE33, LVDS18G, **LVDS25, LVDS33**, RSDS25/33, MINILVDS25/33, SUBLVDS25/33, PPDS25/33, SLVSE15, MLVDSE25, BUSLVDSE25, LCMDS33/25, SHIELD33/25/18/15/12。LCMDS18 は MPF300XT 非対応。

**使えない方向（Table 3-19, p.16）**
| 方向 | 不可の IOSTD |
|---|---|
| Input | SLVSE15, MLVDSE25, BUSLVDSE25, MIPIE33, LVPECLE33, SHIELD* |
| Output | SLVS33/25, HCSL33/25, LVPECL33/25, MIPI25, LVDS18, RSDS18, MINILVDS18, SUBLVDS18, PPDS18, SLVS18, HCSL18, LCMDS18 |
| Tribuff | 上の Output 不可に加え **LVDS18G, LVDS25, LVDS33**, RSDS25/33, MINILVDS25/33, SUBLVDS25/33, PPDS25/33, LCMDS25/33 |
| Inout | **LVDS18G, LVDS33, LVDS18, LVDS25**, RSDS*, MINILVDS*, SUBLVDS*, PPDS*, SLVS*, HCSL*, LVPECL33/25, MIPI25, MIPIE25, SHIELD*, LCMDS* |

読み方: GPIO の LVDS25/33 は **受信（Input）も送信（Output）も可**。HSIO の LVDS18 は受信専用（Output 不可）。

## Anti-patterns
- **LVDS25 で TRIBUFF_DIFF / BIBUF_DIFF を使う**: Tribuff/Inout は非対応。送信は OUTBUF_DIFF、受信は INBUF_DIFF
- **HSIO バンク（Discovery Kit では Bank 0）で LVDS 送信**: LVDS18 は Output 不可
- **異電圧の IOSTD を同じバンクに追加**: 既存ポートの配置が自動で外れる
- **`-iostd` と `-io_std` の混在**: 原文 p.4 の例は `-iostd`、構文は `-io_std`。構文どおり `-io_std` を使う

## Key Takeaways
1. `set_io -port_name … -pin_name … -io_std … -fixed true` が基本形
2. LVDS25/LVDS33/LVDS18G は差動専用、Input/Output 可、Tribuff/Inout 不可
3. HSIO の LVDS18 は入力専用
4. 方向は HDL で使うマクロ（INBUF_DIFF / OUTBUF_DIFF）で決まる
5. 差動ペアは P 側ピンを指定する運用が想定される（原文外。Pin View で確認）

## Connects To
- **pdc-01-syntax-iobank**: vcci と IOSTD の対応
- **pdc-03-set-io-attributes**: 各属性の値表
- **mlg-02-io-clocking**: INBUF_DIFF / OUTBUF_DIFF のポート
- **ioed-01-port-pin-view**: Pin View の Function 列で P/N パッドを確認
