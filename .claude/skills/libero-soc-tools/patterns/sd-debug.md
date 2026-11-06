# パターン: SmartDesign・SmartDebug

(原文外) は資料外から補った部分。

## 自作 HDL を SmartDesign に組み込み、変更を上位へ反映する
**When to use**: ファブリックに LVDS 送受信などの自作 HDL ブロックを追加し、MSS を含む SmartDesign トップにつなぐとき
**How**: File > Import > HDL Source Files（または File > New > HDL）→ バス接続が要るなら Design Hierarchy で右クリック Create Core from HDL して BIF を定義 → キャンバスへドラッグ → 結線 / Promote to Top-Level / 未使用入力は Tie・未使用出力は Mark Unused → Generate Component。HDL のポートを後で変えたら、上位で右クリック Update Instance → Generate をトップまで繰り返す（sd-01, sd-02, sd-03）
**Trade-offs**: HDL+ にすると BIF 一発結線できるがコア定義の管理が増える。Non-recursive 生成（既定）では下位が未生成でもトップが成功扱いになるので、Recursive にするか下位から順に生成する

## Live Probe で内部信号 2 本をオシロ観測する
**When to use**: 再コンパイルせずに内部 DFF の実時間波形を見たいとき（J12 のような専用 Live Probe ピンがあるボード）
**How**: Live Probe I/O を予約しユーザ I/O（特に入力）に使わない設計で書き込む → SmartDebug Design → Debug FPGA Array > Live Probes → 信号を右クリック Assign to Channel（A/B）→ プローブピンにオシロ → 終了は Unassign Channels。Tcl なら `set_live_probe -probeA {...} -probeB {...}` を 1 回で（dbg-02, dbg-06）
**Trade-offs**: 同時 2 本まで。対象は DFF と RAM/MATH 出力のみで組合せ論理は不可。Active Probe の読み書きで設定が崩れる。セキュリティ設定で無効化され得る

## FHB で条件停止して全 DFF を読む
**When to use**: タイミング依存の不具合で、特定条件の瞬間の内部状態を一括で見たいとき
**How**: Project Settings > Design flow > Enable FPGA Hardware Breakpoints Auto Instantiation → NDC に `auto_instantiate_fhb -inst_name {<clkint>}` をドメインごとに記述 → 再合成〜書き込み → トリガ信号を Live Probe Channel A に割当て → Edge・Delay（0-255）設定 → Arm Trigger → 停止後 Active Probe で読む / Step / Capture Waveform（.vcd, 最大 8192 サイクル）（dbg-03）
**Trade-offs**: 再合成が必要。DUT クロックが 160 MHz を超えると停止位置は保証されない。止めたドメイン以外は走り続ける。CLKINT_PRESERVE や Suspend Mode ON の設計には入らない

## Probe Insertion で任意ネットを空き I/O に出す
**When to use**: 組合せ論理や 3 本以上の信号をロジアナ/オシロで同時に見たいとき
**How**: Libero から SmartDebug Design（Integrated）→ Debug FPGA Array > Probe Insertion → ネットを Add → Package Pin に未使用ピンを割当て → Run（incremental P&R ＋自動再書き込み）。終わったら Delete All → Run で元に戻す（dbg-03）
**Trade-offs**: 空き I/O と再 P&R 時間が要る。Standalone では使えない。Run せずに Delete しても設計から消えない

## LVDS 受信のビットアラインを確認する
**When to use**: PF_IOD_GENERIC_RX でデータを受けたがビットずれ・エラーが疑われるとき
**How**: PF_IOD_GENERIC_RX を Dynamic / Fractional Dynamic で構成し、CORERXIODBITALIGN の debug ピン（RX_BIT_ALIGN_LEFT_WIN, RX_BIT_ALIGN_RGHT_WIN, BIT_ALGN_ERR, DEM_BIT_ALGN_TAPDLY, BIT_ALGN_DONE, BIT_ALGN_START）を接続かトップへ Promote → SmartDebug の Debug IOD → Get Training Data。Tcl は `debug_iod -iod_type {RX_DDRX_B_G_DYN} -inst_path {...}`（dbg-05, dbg-06）
**Trade-offs**: Static モードでは使えない。debug ピンを出さないと SmartDebug が検出できない。LVDS への適用は UG の明記ではなく解釈（原文外）
