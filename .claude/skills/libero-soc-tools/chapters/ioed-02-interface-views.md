# I/O Editor User Guide: インターフェース専用ビュー（Memory / XCVR / IOD）（PDF p.18-36）

## Core Idea
PolarFire はメモリ・IOD・トランシーバのインターフェースを決まった位置に置かないと QoR/タイミングが出ないため、I/O Editor は専用ビューで DRC 付きのドラッグ＆ドロップ配置を提供する。commit・保存すると set_io を含む PDC（`<project_folder>/constraints/io/user.pdc`）が生成される。**IOD View は FPGA I/O を使う非メモリインターフェース（CDR、汎用 DDR 受信/送信）用**。

## Frameworks Introduced
- **Memory Interface の割り当て手順**
  1. Memory Type（DDR3 / DDR4 / LPDDR3 / QDRII+）をプルダウンで選ぶ
  2. Ports タブ（Design Tree）からインスタンスをドラッグし Edge_Anchor 上にホバー（ツールチップで合法/違法）
  3. 合法な Edge_Anchor にドロップ
  4. Log にエラーが無く、Ports タブに鍵アイコンが付いたことを確認
  - 解除: Port タブでインスタンスを右クリック > Unplace <memory_interface_name>
- **XCVR の割り当て順**: 1. Transceiver Lanes → 2. TX PLL → 3. REFCLK
- **IOD の割り当て**: Ports タブからポートを表へドラッグ。コメントに合法位置が出る（緑=合法、赤=違法。違法は拒否）

## Key Concepts
- **Edge_Anchor**: メモリ配置位置。NORTH_NE, NORTH_NW, SOUTH_SE, SOUTH_SW, WEST_NW, WEST_SW。メモリ種で候補が変わる（DDR4 は DDR2/DDR3 より少ない）
- **Max Memory Width / Max Data Rate**: Edge_Anchor ごとの固定上限（read-only）。DRC はデータ幅を検査（データレート検査は「将来のリリース」）
- **Direct / Cascaded 接続**: REFCLK と TX_PLL/レーンが同じ Quad なら Direct、違えば Cascaded。カスケードは**上から下方向のみ**。Direct は信号品質、Cascaded は外付け部品と電力の削減
- **TXPLL_SSC / TXPLL**: SSC 版は Quad の全 4 レーン、非 SSC 版は隣接 2 レーンに接続可
- **IOD レーンコントローラ**: レーンクロックとバンククロックを I/O FIFO で橋渡し。DLL の PVT 補償遅延コードで 90° シフト。高いギア比のクロックドメインを扱う
- **IOD CDR**: CCC PLL の 4 位相から 8 位相を作り、入力データに合う位相を選ぶ。分周した再生クロックを DIVCLK としてファブリックへ
- **バンク/レーン**: PolarFire は最大 8 バンク、1 バンク 6 レーン（大きいダイはさらに多い）。実数はダイ依存

## Reference Tables
**IOD View で扱えるインターフェース（p.35）**
| 種別 | 内容 |
|---|---|
| CDR | IOD CDR インターフェース |
| RX_DDR_L_A / TX_DDR_G_A | 汎用 IOD インターフェース（受信 DDR / 送信 DDR。名称の L/G/A の意味は原文に記載なし） |
将来のリリースで追加予定と記述。

**IOD View の DRC（p.36）**
| ルール |
|---|
| 同じ論理レーンの I/O はすべて同じ物理レーンに置く |
| 1 物理レーンに置ける論理レーンは 1 つ |
| 論理レーンに属さない I/O はどの物理レーンでも可 |
| RGMII の *_RXC ポートは物理レーンの CLKIN_S_* 側 |
| CDR を置いた物理レーンでは DQS_N スロットが予約され I/O を置けない |
詳細は PolarFire FPGA and PolarFire SoC FPGA User I/O User Guide を参照と記載。

**XCVR View の構成と DRC（p.25-34）**
| 項目 | 内容 |
|---|---|
| Schematic View | REFCLK → TransmitPLL → レーン |
| Graphical Placement View | PAD から TXPLL・レーンまでの物理接続。合法=緑線、違法=赤線/赤矢印 |
| Signal Integrity View | TX Emphasis Amplitude, TX Impedance, TX Common Mode Adjustment, RX/TX Polarity, RX Insertion Loss, RX CTLE, RX Termination, RX P/N Board Connection, RX LOS Detector |
| DRC 対象 | TXPLL→レーン、REFCLK→TXPLL、REFCLK→レーン（レーン割り当て自体は常に合法） |
| PCIe | レーンは既定位置固定で外せない |

## Anti-patterns
- **メモリ幅・データレートを確認せずに Edge_Anchor へドロップ**: 例として ddr3 が 64 ポート必要なのに SOUTH_SE は 58 ピンしかなく拒否される
- **REFCLK から上側（カスケードを遡る）TXPLL/レーンへ接続**: 違法
- **CDR レーンの DQS_N にユーザー I/O を置く**: 予約スロット（PDC の PRPF_003 と同じ）
- **1 物理レーンに 2 つの論理レーン**: 違反

## Key Takeaways
1. 専用ビューで commit すると `constraints/io/user.pdc` に set_io が書かれる
2. Memory は Edge_Anchor 単位、XCVR は Lanes → TX PLL → REFCLK の順に置く
3. IOD View は CDR と RX_DDR_L_A/TX_DDR_G_A の汎用 IOD に対応
4. 汎用 IOD は「論理レーン＝1 物理レーン」が原則。DDR 受信系 LVDS バスはレーン境界を意識してピンを選ぶ（適用は原文外の推論）
5. REFCLK のカスケードは上から下のみ

## Connects To
- **pdc-04-location-ndc-floorplan**: PRPF_xxx の配置ルール、set_location の Edge_Anchor/TxPLL 位置
- **pdc-05-post-layout-edit**: XCVR 属性の edit_io、LANECTRL の遅延
- **mlg-05-post-layout-macros**: HS_IO_CLK（バンクの高速 I/O クロック）、IOREG
