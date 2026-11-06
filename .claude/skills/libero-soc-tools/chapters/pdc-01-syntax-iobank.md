# PDC Commands Reference Guide: 構文規則・命名規則・set_iobank / reserve（PDF p.1-8）

## Core Idea
PDC（Physical Design Constraints）は Place-and-Route 用の I/O 属性・配置・配線制約。I/O Editor や Chip Planner で GUI 変更すると PDC が更新され、手書きもできる。**コマンドと引数は大文字小文字を区別する**。I/O を置く前にバンク電圧を `set_iobank` で明示しておくのが推奨。

## Frameworks Introduced
- **バンク電圧の決まり方（set_iobank 未指定時）**
  1. バンクに I/O が無ければ設定なし
  2. 最初に置かれた I/O の IOSTD でバンク電圧が決まる（例: LVCMOS25 → 2.5V）
  3. ユーザーが I/O を置かなければツールが必要に応じてバンクを設定して配置
  → 意図しない電圧で固まるのを防ぐため `set_iobank` で先に固定する
- **名前の書き方（Ch2）**: ポート名はネットリストどおり。インスタンス名/ネット名は、階層区切りでない `/` と `\` の前に `\` を付ける。ワイルドカード使用時は `\` をさらに 1 個追加。

## Key Concepts
- **GPIO / HSIO**: PolarFire の 2 種類の I/O バンク。対応 IOSTD が異なる（LVDS25/LVDS33 は GPIO のみ）
- **vcci**: バンクの I/O 電源電圧。**vref**: 入力基準電圧（全バンクに専用 vref ピンあり。そのピンに何も割り当てない）
- **-fixed**: バンクの技術（vcci/vref）をロック
- **-update_iostd**: 既配置 I/O を残したまま互換 IOSTD に変えるか
- **-auto_calib**: 電源投入時のバンク自動キャリブレーション（既定 true）
- **reserve**: 指定パッケージピンを予約（他デザイン用に空ける）

## Commands & Syntax
```tcl
# 継続行は行末に \ 。{ } の内側に空白を入れない（{PORT1} は可、{ PORT1 } は不可）
# [ ] を名前に含む場合は {} で囲むか \[ \] でエスケープ
set_iobank -bank_name <bank_name>\
  [-vcci <vcci_voltage>]\
  [-vref <vref_voltage>]\
  [-fixed <value>]\
  [-update_iostd <value>]\
  [-auto_calib <value>]\
  [-auto_calib_ramp_time <value>]

reserve -pin_name "list of package pins"
```
例（原文）: `set_iobank -bank_name bank0 -vcci 3.3` / `reserve -pin_name "F2 B4 B3"`

Discovery Kit Bank 1（GPIO）で LVDS25 を使う場合の例（バンク名表記は原文例の `bank0` に倣った推定。原文外）:
```tcl
set_iobank -bank_name bank1 -vcci 2.5 -fixed true
```

## Reference Tables
**バンク種別ごとの IOSTD（Table 3-1, p.6）**
| 種別 | 主な差動系 IOSTD |
|---|---|
| HSIO | LVDS18, RSDS18, MINILVDS18, SUBLVDS18, PPDS18, SLVS18, HCSL18 ほか SSTL/HSTL/POD/LVCMOS12-18 |
| GPIO | **LVDS25, LVDS33, LVDS18G**, RSDS25/33, MINILVDS25/33, SUBLVDS25/33, PPDS25/33, LVPECL33, LVPECLE33, SLVS25/33, HCSL25/33, MIPI25, MIPIE25, MLVDSE25, BUSLVDSE25, LCMDS25/33 ほか LVTTL, LVCMOS12-33, PCI, SSTL25/18/15, HSTL15, HSUL18 |

**-vcci と互換規格（Table 3-2, p.7）**
| vcci | 互換規格（抜粋） |
|---|---|
| 3.3V | LVTTL, LVCMOS33, PCI, **LVDS33**, LVPECL33, LVPECLE33, SLVS33, HCSL33, RSDS33, MINILVDS33, SUBLVDS33 |
| 2.5V | LVCMOS25, SSTL25I/II, PPDS25, SLVS25, HCSL25, MLVDSE25, MINILVDS25, RSDS25, SUBLVDS25, **LVDS25**, BUSLVDSE25 |
| 1.8V | LVCMOS18, SSTL18I/II, HSUL18I/II, SLVS18, HCSL18, LVDS18, **LVDS18G**, RSDS18, MINILVDS18, SUBLVDS18, PPDS18 |
| 1.5V | LVCMOS15, SSTL15I/II, HSTL15I/II, SLVSE15 |
| 1.35V | HSTL135I/II, SSTL135I/II |
| 1.2V | LVCMOS12, HSUL12I, HSTL12I, POD12I, MIPI12 |

**-vref（Table 3-3）**: 1.25V=SSTL25I / 1.0V=SSTL18I, HSUL18I / 0.75V=POD12I, HSTL15I, SSTL15I, HSUL12I, HSTL12I / 0.67V=SSTL135I, HSTL135I（LVDS は vref 不要）

**その他の引数**
| 引数 | 値 |
|---|---|
| -fixed | True=ロック / False |
| -update_iostd | True=配置を保持しホスト（IOSTD）を互換品に変更 / False=ロック済み I/O があれば失敗、未ロックは unplace |
| -auto_calib | true/false（既定 true）。MPF300TS_ES, MPF300T_ES, MPF300XT 非対応 |
| -auto_calib_ramp_time | VDDI ランプ時間 1–50 ms（既定 50） |

例外: バンク設定と非互換なピン割り当ては解除される。

## Anti-patterns
- **set_iobank 無しで最初の I/O に電圧を決めさせる**: 後から LVDS25 を置こうとして 3.3V バンクと衝突する
- **`{ PORT1 }` のように波括弧内に空白**: 失敗する
- **大文字小文字の揺れ**: Verilog ネットリストは大小区別。`A10` を `a10` と書くと失敗（VHDL は `\A\` 形式以外は区別しない）
- **vref 専用ピンに I/O を置く**: 原文は「設定しない」と明記

## Key Takeaways
1. PDC は大小区別。名前のエスケープ規則（`\/`, `\[ \]`）に従う
2. LVDS25 は vcci 2.5V、LVDS33 は vcci 3.3V、LVDS18G は 1.8V の GPIO バンクで使う
3. HSIO の LVDS は LVDS18 のみ。GPIO 用の LVDS25/33 とは別物
4. `set_iobank` でバンク電圧を先に固定すると、配置時の電圧衝突を防げる
5. 自動キャリブレーションの ramp time 既定 50 ms は基板の VDDI 立ち上がりに合わせて調整可

## Connects To
- **pdc-02-set-io-iostd**: IOSTD ごとの単端/差動対応と方向制限
- **pdc-03-set-io-attributes**: ODT・VCM・RES_PULL などの属性値
- **ioed-03-floorplanner-export-mss**: Floorplanner の I/O Bank Settings / Auto Assign I/O Bank（GUI で同じことをする）
- 姉妹 skill `polarfire-soc-discovery-kit` ch02: Bank 1 は VDDI1_5（J45/J46 で 2.5V/3.3V）
