# トピック: GPIO バンクで LVDS を送受信する（資料横断）

PDC・Macro Library・Timing Constraints Editor・SmartDebug の要点を、LVDS の観点で 1 枚にまとめたもの。詳細は各節の末尾にある章ファイルを見る。Discovery Kit では Bank 1（GPIO バンク、VDDI1_5 = 2.5V/3.3V）が対象。

## I/O 規格と方向
- GPIO で使える LVDS は **LVDS25（VCCI 2.5V）/ LVDS33（3.3V）/ LVDS18G（1.8V）**。HSIO の LVDS18 は**入力専用**
- 方向は Input / Output のみ。**Tribuff・Inout は不可**（TRIBUFF_DIFF / BIBUF_DIFF は使えない）
- バンク電圧は先に固定する: `set_iobank -bank_name bank1 -vcci 2.5 -fixed true`。固定しないと、最初に置いた I/O の規格でバンク電圧が決まる
→ pdc-01, pdc-02

## HDL のマクロ
- 受信 `INBUF_DIFF`（PADP, PADN → Y）、送信 `OUTBUF_DIFF`（D → PADP, PADN）、差動クロック入力 `CLKBUF_DIFF` / `GCLKBUF_DIFF`
- I/O 規格はマクロでは決めず、PDC の `-io_std` で決める
- IOD の DDR やギアリングのマクロは Macro Library に**載っていない**。IP（PF_IOD_GENERIC 系）を使う（原文外）
→ mlg-02, mlg-05

## PDC の属性
- 受信: `-ODT` 既定 **ON**、`-ODT_VALUE` は **100 のみ**、`-VCM_RANGE MID|LOW`（既定 MID。**同じバンクの真差動入力では全部同じ値**。混在させるなら配置後に `edit_io`）、`-RES_PULL Up|Down`（既定 Up）
- 入力が浮く可能性があるとき（フェイルセーフ）は `-ODT DYNAMIC` と `-RES_PULL`
- 送信: `-OUT_DRIVE 3|3.5|4|6`（既定 6 mA）、`-SOURCE_TERM OFF|100`（既定 OFF）。差動出力に RES_PULL は無い
- 外付けの 100Ω 終端があるなら `-ODT OFF` にして二重終端を避ける
- P/N の指定方法は資料に明記が無い。P 側のピンだけ `-pin_name` に書き、N はツールが確保すると読める（原文外。I/O Editor の Pin View で確認）

```tcl
# 構文は原文。値の組み合わせは原文外
set_io -port_name LVDS_RX_P -pin_name <P_pin> -fixed true -io_std LVDS25 \
  -ODT ON -ODT_VALUE 100 -VCM_RANGE MID -RES_PULL Up -DIRECTION INPUT
set_io -port_name LVDS_TX_P -pin_name <P_pin> -fixed true -io_std LVDS25 \
  -OUT_DRIVE 6 -SOURCE_TERM OFF -DIRECTION OUTPUT
```
- 規格名の引数は `-io_std`（`-iostd` は誤り。原文の例にも誤記がある）
- I/O レジスタを IOD に入れるには NDC の `set_ioff`。IOD View のレーン規則は ioed-02
- GPIO バンクの自動キャリブレーションの完了は、`INIT` マクロの `GPIO_ACTIVE` でバンクごとに分かる
→ pdc-03, pdc-04, pdc-05, ioed-02, mlg-03

## タイミング制約（ソース同期）
- 両 UG に LVDS やソース同期 DDR 専用の例は無い。以下は文書化されたオプションの組み合わせ（原文外）
- 受信: 転送クロックの入力ポートに `create_clock`（または virtual clock）を置き、`set_input_delay -max/-min` を立上り基準で書く。立下り側は `-clock_fall -add_delay` で追加する
- **`-add_delay` を忘れると前の制約が上書きされる**。`-min` を省くと max と同じ値になる
- 送信: `create_generated_clock -divide_by 1 -source <PLL 出力ピン> [get_ports clk_out]`（原文 p.112 の例）
- PLL の `-phase` は、波形のシフトではなくクロック網の遅延として扱われる
→ tce-04, tce-03

## 実機での確認
- ビットの位置合わせは SmartDebug の **Debug IOD**（dbg-05）
- 内部信号を外に出すなら Live Probe（2 本、J12）か Probe Insertion（dbg-02, dbg-03）

## 資料に無いもの
最大データレートと DC/AC 特性は PolarFire SoC Datasheet にある。IOD の構成は User I/O User Guide。どちらも未取得。
