# SmartDebug UG: Transceiver・IOD・DDR・TVS ほか（PDF p.69-103, 105-108, 111-159, 159-169）

## Core Idea
メイン画面のボタンは設計に該当 IP があるときだけ出る（Debug Transceiver、Debug IOD、Debug Fabric/MSS DDR Memory、TVS Monitor など）。Discovery Kit の LVDS 受信で効くのは **Debug IOD**（PF_IOD_GENERIC_RX のビットアライン結果）、MSS DDR の立上げ確認で効くのは **Debug MSS DDR I/O Margin**。トランシーバ（PF_XCVR）は Discovery Kit では主用途外なので要点のみ。Discovery Kit の Ethernet は MSS SGMII（Bank 5）で PF_XCVR ではなく、本 UG に MSS SGMII 専用のデバッグ画面の記述はない（原文外の確認事項）。「LVDS 受信」への当てはめも原文外の解釈（UG は PF_IOD_GENERIC_RX とだけ書く）。

## Frameworks Introduced
- **Debug IOD（p.105-108）**: LVDS などの IOD 受信のトレーニング結果を見る
  - 条件: PF_IOD_GENERIC_RX が **Dynamic / Fractional Dynamic** モード。PolarFire / PolarFire SoC のみ
  - Choose IOD で選び **Get Training Data** → CORERXIODBITALIGN ごとの Delay Tap、Eye Width、Sample Edge を表示。成否は Log に出る
  - **CORERXIODBITALIGN のデバッグ出力ピンをつなぐかトップへ Promote しておくこと**。必要ピン（p.197）: `RX_BIT_ALIGN_LEFT_WIN`, `RX_BIT_ALIGN_RGHT_WIN`, `BIT_ALGN_ERR`, `DEM_BIT_ALGN_TAPDLY`, `BIT_ALGN_DONE`, `BIT_ALGN_START`
- **Debug MSS DDR I/O Margin（PolarFire SoC, p.140-153）**: Debug MSS DDR Memory... → Get Training Data
  - 段階: Clock alignment → Command/Address → Memory init → Write leveling → Read gate → DQ/DQS optimization → Write calibration
  - Overall Training Status、失敗段の推定理由と対処をツールチップと下部ボックスに表示
- **Debug Fabric DDR I/O Margin（p.111-140）**: ファブリック DDR3/DDR4/LPDDR3 用。Get Training Data は約 2 分。Export Training Data で `<name>_ddr_training_report.txt` と JSON
- **TVS Monitor（p.156-159）**: TVS IP がある設計のみ。VDD / VDD18 / VDD25 / 温度を 1-60 秒間隔で表示・`.csv` ログ（既定 1 秒、グラフは直近 60 点）
- **Debug Transceiver（要点, p.69-103）**
  - Configuration Report: Tx/Rx PMA Ready、TX PLL、RX PLL、RX CDR PLL（全部緑が健全, FAQ p.177）
  - SmartBERT: PRBS7/9/15/23/31（PRBS9 以上は PMA 幅 32/40 bit のみ）、EQ-NearEnd ループバック、エラー数と BER。PCIe レーンは不可。CoreSmartBERT IP レーンはエラー注入可
  - Loopback Modes: EQ-NEAREND（〜10.3125 Gbps）/ EQ-FAREND / CDRFAREND / No Loopback（全二重のみ）
  - Static Pattern Transmit: Fixed（1010…）/ Max Run Length（1100…）/ User
  - Eye Monitor: Normal / Infinite Persistent / Design Initiated。**3 Gbps 未満は不可**、Rx にトラフィック必須。`Plot_Eye*.txt` を出力
  - Register Access（`PF_XCVR.xml` 由来、`.csv`）、Signal Integrity の `.pdc` import/export、Optimize Receiver（MPF100/200/300/500T）、PCIe LTSSM、Record Actions
- **SmartFusion 2 / IGLOO 2 / RTG4 専用（p.159-169, 177-189）**: Debug SerDes（レジスタ R/W、Loopback、PRBS、PHY Reset）と NVM/FlashROM/Analog 系 FAQ。PolarFire SoC では対象外

## Key Concepts
- **PF_IOD_GENERIC_RX**: 汎用 IOD 受信 IP（LVDS 等）。Dynamic モードでタップ遅延をトレーニングする
- **CORERXIODBITALIGN**: 受信ビットアライン IP。トレーニング結果の出力元
- **Eye Width / Sample Edge**: IOD トレーニングで得た有効窓と採用サンプル位置
- **DDR PHY training**: 起動時に時間・電圧方向の最適点を探す工程
- **TVS（Temperature and Voltage Sensor）**: ダイ温度と電源電圧のセンサ IP

## Reference Tables
**Fabric DDR 失敗段と対処（Table 3-5, p.125-126 要約）**
| 段 | 主因 | 対処 |
|---|---|---|
| FPGA Initialization | コントローラがリセット保持 | SYS_RESET_N を確認 |
| HS_IO_CLK-to-SYS_CLK | PLL_REF_CLK 未駆動 | 参照クロックのトグル、PLL_LOCK / SYS_CLK |
| CK-to-CA / Write Leveling / Read DQ/DQS / Write Calibration | CA/CK や DQ/DQS の SI | メモリ側モードレジスタ（DDRx Configurator の Memory Initialization）と FPGA 側 I/O Attribute Editor でドライブ強度・ODT を調整 |
| Training Iterator | 10 回試行しても不合格 | 上記各段の対処 |

**SCB 読み出しの注意（p.2）**: APB DRI バスが SCB を読んでいる最中に SmartDebug が SCB を読むとデータが壊れることがある

## Anti-patterns
- **CORERXIODBITALIGN の debug ピンを未接続で放置**: SmartDebug が IOD を検出できず Debug IOD が使えない
- **PF_IOD_GENERIC_RX を Static モードで作ってタップを見たがる**: Debug IOD は Dynamic / Fractional Dynamic のみ
- **3 Gbps 未満のレーンで Eye Monitor**: 無効
- **Infinite Persistent を長時間回して容量不足**: 1 反復ごとにファイルが増える。再起動で番号が 1 に戻り上書きされる

## Key Takeaways
1. LVDS 受信は PF_IOD_GENERIC_RX を Dynamic にし、CORERXIODBITALIGN の debug ピンを出しておけば Debug IOD で窓とタップが見える
2. MSS DDR の不調は Debug MSS DDR I/O Margin で失敗段を特定する
3. 機能ボタンは対応 IP が設計にあるときだけ表示される
4. トランシーバ健全性は Configuration Report 全緑＋SmartBERT エラー 0
5. 温度・電圧は TVS IP を入れれば TVS Monitor で取れる

## Connects To
- **dbg-06**: `debug_iod` / `debug_mss_ddr` / `export_mss_ddr_training_data` / `tvs_monitor` / `smartbert_test` / `loopback_mode`
- **sd-02**: CORERXIODBITALIGN の debug ピンを Promote to Top-Level
