# SmartDebug UG: メモリと MSS レジスタの読み書き（PDF p.33-45, 65-69, 103-105, 109-111, 153-156, 174-176）

## Core Idea
ファブリック RAM（LSRAM / uSRAM / TPSRAM）は Debug FPGA Array の Memory Blocks タブで、不揮発メモリ（sNVM / eNVM / µPROM）は専用ウィンドウで、PolarFire SoC の MSS レジスタは MSS Register Access で、すべて JTAG 経由で読み書きする。RAM アクセス中はユーザ回路側からその RAM にアクセスできない。

## Frameworks Introduced
- **ファブリック RAM の読み書き（p.33-44, FAQ p.174-176）**
  1. Debug FPGA Array > **Memory Blocks** タブ
  2. 左ペインからブロックを選ぶ（**L** = 論理ブロック、**P** = 物理ブロック。アイコンに文字がないものは選べない）
  3. Select / 右クリック Add / ドラッグで追加。1 度に 1 ブロック
  4. **Read Block** → 16 進マトリクス表示（行 16 ワード、View in 2D、Go to Address）
  5. 値を書き換えて **Write Block**（1 ロケーションずつ）
- **ECC 付き TPSRAM（p.38-44）**: SECDED。論理ビューで誤りを赤表示、物理ビューは Data / ECC Bits / Error Detected / Corrected Data（単一ビット誤りは訂正値を提示し右クリックでコピー可）
  - 書込みでエラー注入できる。3-4 bit 超の破壊は誤った訂正候補や未検出になり得る
  - **Scan** で ECC 付きブロックを一括検査（緑チェック / 赤丸）
- **MSS Register Access（PolarFire SoC, p.103-105）**
  - MSS コンポーネントが設計にあるときだけボタンが出る。**MSS の MPU が有効/設定済みであること**
  - Register Selection（フィルタ `*` 可）から Add / ドラッグ → Register Operations で Read / Write（16 進）
  - 常に見えるレジスタ群: AXISW, MPUCFG, ENVMCFG, IOSCBCFG, PFSOC_MSS_TOP_SCB_REGS, PFSOC_MSS_TOP_SYSREG ＋ MSS 構成で有効な周辺
  - フィールド書込みは read-modify-write。全体値とフィールド値が衝突したら全体値が優先
  - Export / Export All / Import は `.csv`（Name, Address, Field Name/Range/Value, Register Value, Comments）
- **sNVM（p.65-69）**: Client View（Libero で構成したクライアント単位）/ Page View（開始〜終了ページ）。保護ページは既定の USK で読む。別 USK に変えた場合は USK を手入力。ROM 扱いのページはランタイム変更不可
- **eNVM（PolarFire SoC, p.153-156）**: 128 KB、512 ページ × 256 B、Sector 0-3。Client View / Page View（0-511）。Export でテキスト出力
- **µPROM（p.109-111）**: User Design View（クライアント単位）/ Direct Address View（Start Address と 9-bit ワード数）

## Key Concepts
- **FCB（Fabric Control Bus）**: SmartDebug が RAM を読み書きする経路。操作中は一時的にユーザ側から RAM を奪う
- **論理ブロック / 物理ブロック**: 論理 = 設計上の RAM（深さ×幅を統合表示、Port Used で A/B 選択）、物理 = 個々の RAM マクロ
- **PolarFire の物理 RAM 形状**: uSRAM 64×12、LSRAM 16384×1 / 8192×2 / 4096×5 / 2048×10 / 1024×20、TPSRAM 512×40（ECC 時 512×33）
- **USK（User Secret Key）**: sNVM の保護ページ用鍵
- **Boot mode 0**: このモードでは eNVM をデバッグできない

## Reference Tables
**eNVM のセクタ（p.154）**
| Sector | Size | Offset | MSS Address |
|---|---|---|---|
| 2 | 8K | 0x00000000 | 0x20220000 |
| 0 | 56K | 0x00002000 | 0x20222000 |
| 1 | 56K | 0x00010000 | 0x20230000 |
| 3 | 8K | 0x0001E000 | 0x2023E000 |

**論理ビューが作れないケース（p.38, 45）**
- RTL 推論で Synopsys の推論ガイドラインに従っていない
- ガイドラインから外れる IP
- RAM 出力（A_DOUT/B_DOUT）を一部だけトップへ出し、残りの物理ブロックが最適化で消えた
- 対策: デバイス別の "Inferring ... RAM Blocks" アプリケーションノートに従う

## Anti-patterns
- **動作中の RAM を読んだ値を確定値として扱う**: 読み出し中はユーザアクセスが止まり、値が古い/不定なことがある
- **MSS の MPUCFG:PMPCFG_SCB_* を書き換える**: SmartDebug 自体の動作がおかしくなる
- **Boot mode 0 のまま eNVM を見ようとする**: アクティブなブートモードで MPU を有効にするアプリが走っている必要がある
- **MPU 未設定で MSS Register Access**: アクセスできない

## Key Takeaways
1. ファブリック RAM は Memory Blocks タブで Read Block → 編集 → Write Block
2. ECC 付き TPSRAM はエラー注入と Scan で健全性を確認できる
3. MSS レジスタは MSS Register Access で読み書きし、`.csv` で保存・再読込みできる
4. eNVM デバッグは Boot mode ≠ 0 と MPU 有効が前提
5. sNVM の保護ページは USK が必要

## Connects To
- **dbg-02**: Debug FPGA Array の他タブ
- **dbg-06**: `read_lsram` / `write_lsram` / `read_usram` / `write_usram` / `scan_ecc_memories` / `mss_*_register` / `read_envm_memory` / `read_snvm_memory` / `read_uprom_memory`
- 姉妹 skill **polarfire-soc-mss**: MSS レジスタ・eNVM アドレス・MPU
