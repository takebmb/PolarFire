# パターン: I/O・PDC・マクロ（LVDS）

(原文外) は資料外から補った部分。

## GPIO バンクで LVDS を受信する（PDC 制約）
**When to use**: Bank 1 などファブリック GPIO バンクに外部 LVDS 送信器をつなぐとき
**How**: HDL で INBUF_DIFF（PADP/PADN → Y）を置く → `set_iobank -bank_name <bank> -vcci 2.5`（LVDS33 なら 3.3）でバンク電圧を固定 → `set_io -port_name <P側ポート> -pin_name <P側ピン> -fixed true -io_std LVDS25 -ODT ON -ODT_VALUE 100 -VCM_RANGE MID`。外付け終端があれば `-ODT OFF`。入力が浮く可能性があれば `-ODT DYNAMIC` と `-RES_PULL`（フェイルセーフ）
**Trade-offs**: LVDS 入力の ODT 既定は ON なので外付け終端との二重終端に注意。VCM_RANGE はバンク内の真差動入力すべてで同じ値が必要（混在は edit_io で post-layout のみ）。差動ペアの N 側指定方法は原文に記載がなく、P 側指定 + IO_DIFF による N 側予約という読み方は推定（pdc-01〜03, mlg-02, mlg-05）

## GPIO バンクから LVDS を送信する
**When to use**: Bank 1 から外部へ LVDS データ/クロックを出すとき
**How**: OUTBUF_DIFF（D → PADP, PADN）を置き、`set_io ... -io_std LVDS25 -OUT_DRIVE <3|3.5|4|6> -SOURCE_TERM <OFF|100>`。既定は 6 mA、SOURCE_TERM OFF
**Trade-offs**: LVDS25/33/18G は Tribuff/Inout 非対応なので TRIBUFF_DIFF / BIBUF_DIFF は使えない（出力イネーブル制御や双方向 LVDS は不可）。差動出力に RES_PULL は無い。HSIO の LVDS18 は Output 不可なので HSIO バンクからは送信できない（pdc-02, pdc-03, mlg-02）

## I/O レジスタを IOD に取り込んでタイミングを詰める
**When to use**: 入力 setup / clock-to-out を改善したい単純な SDR 入出力
**How**: *.ndc に `set_ioff -port_name {DataA[*]} -IN_REG 1`（出力なら -OUT_REG、イネーブルなら -EN_REG）を書き、Compile/Synthesis 前に読ませる。post-layout では IOREG（IOINFF/IOUTFF/IOEFF）として確認できる
**Trade-offs**: 1 コマンドで有効化できるのは 1 オプションのみ、取り込める FF は 1 個で fanout 1。スカラーのみ（バスはワイルドカードか for ループ）。DDR やギアリングは本 3 資料の範囲外で、IOD 系 IP（原文外）を使う（pdc-04, mlg-05）

## IOD 汎用インターフェース（DDR 受信/送信・CDR）をレーン規則に沿って配置する
**When to use**: RX_DDR_L_A / TX_DDR_G_A や IOD CDR を使う LVDS バス
**How**: I/O Editor の IOD View でポートをドラッグし、緑表示（合法）の物理レーンに置く。同じ論理レーンの I/O は同一物理レーン、1 物理レーンに 1 論理レーン。CDR レーンの DQS_N スロットは空ける
**Trade-offs**: ピン選定がレーン構造に縛られる。違反は PRPF_002/003/004 で P&R 前に止まる。レーン数/バンクはダイ依存（ioed-02, pdc-04）

## 配置後に LVDS 受信条件を実機で詰める
**When to use**: 実機で受信マージンが足りない、同一バンクで VCM レンジを変えたい
**How**: edit_post_layout_design で `edit_io -port_name <port> -VCIM_RANGE LOW`（構文上の綴り）や `-ODT`, `-IN_DELAY` を変更。再配置不要
**Trade-offs**: ピン・IOSTD・fixed は変えられない。VCM 混在時は内蔵 100Ω 終端の精度が 2 レンジのうち悪い方の許容差になる。原文構文の `-VCIM_RANGE` 綴りは要確認（pdc-05, pdc-03）

## GPIO バンクの準備完了を待ってからインターフェースを動かす
**When to use**: 電源投入直後から LVDS を受ける回路のリセット設計
**How**: INIT マクロの GPIO_ACTIVE（バンクごとのキャリブレーション完了）や FABRIC_POR_N をリセット解除条件に入れる。自動キャリブは `set_iobank -auto_calib true`（既定）と `-auto_calib_ramp_time`（1–50 ms、既定 50）
**Trade-offs**: リセット解除が遅れる。キャリブを手動制御するなら BANKCTRL_GPIO の CALIB_START/LOAD/LOCK シーケンスが必要。GPIO_ACTIVE をリセット条件に使う設計は原文の「判定に使える」という記述からの応用（mlg-03, pdc-01）
