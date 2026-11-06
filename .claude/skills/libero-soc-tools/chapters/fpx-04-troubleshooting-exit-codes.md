# FlashPro Express User Guide v2026.1: 終了コードとトラブル対処（PDF p.37-50）

## Core Idea
書き込みに失敗するとログに **Error Code（0x80xx）**と **Exit Code（負数を含む整数）**が出る。原因は大きく **電源・DEVRST_N・JTAG の信号品質**、**ファイルとデバイスの不一致**、**セキュリティ（パスキー・ロック）**の 3 系統。最初に試す対処は「電源の過渡を確かめる、**DEVRST_N を HIGH に固定する**、JTAG のノイズ・反射を測る」。

## Frameworks Introduced
- **失敗時の切り分け**
  1. **USB・プログラマ系**（Exit 4 / -7 / eFP6 HID）→ ケーブルを挿し直し、Refresh/Rescan、ケーブルの取り回しを見直す
  2. **IDCODE / DEVICEID 系**（0x8004, 0x8029, 0x8059/5A/5D）→ 正しい書き込みファイルとチェーン上のデバイスか。TRST が浮いていればプルアップ、GND 線は短く
  3. **電源・リセット系**（0x8003, 0x8032, 0x804F, 0x8040）→ 電源の過渡、DEVRST_N を HIGH、JTAG の電源ピンを監視
  4. **検証不一致**（0x8005-0x8009, 0x801B）→ 別の設計が入っているか、未書き込み、改ざん。同じファイルで再書き込み
  5. **セキュリティ系**（0x801D, 0x8010/11, 0x8021, 0x8013, 0x804F, 0x805B）→ DEVICE_INFO でロック状態を見て、パスキー・鍵の合うファイルを使う
- **PolarFire SoC の扱い**: PolarFire SoC 固有の表（Table 9-2）にあるのは 0x8020 の 1 件だけ。（原文外）ファブリック・sNVM・eNVM・セキュリティまわりの失敗は PolarFire の表（Table 9-3）で引く

## Key Concepts
- **Error Code**: デバイスまたはツールが返す 16 進コード（0x80xx）
- **Exit Code**: プロセスの終了値。原文の表で文字化けしている「�18」などは負数（-18）と読む（原文外: 文字化けからの推定）
- **DEVRST_N**: デバイスのリセットピン。書き込み前に HIGH に固定する
- **FAB_RESET_N**: GND に固定されていると 0x8030 になる
- **TRSTB / System Controller Suspend Mode**: 0x8001（DSN 読み出し失敗）の原因。TRSTB を High にするか SCSM を無効にする
- **UPK1 / FlashLock**: ユーザーパスキー 1。外部 digest チェックや書き込みアクションを許可するのに要る
- **Return for FA**: 工場データ破損などで、デバイスを交換して故障解析に返す

## Reference Tables
**全ファミリ共通（Table 9-1）**
| Programmer | Exit | メッセージ | 原因 → 対処 |
|---|---|---|---|
| eFP6 | -- | Failed to open eFP6 HID handle | ソフトとの同期ずれ → USB を抜き差し |
| FlashPro6 | -7 | Failed to perform cyusb_bulk_transfer | USB 接続不良（ランダム）→ 抜き差しして再スキャン |
| **FlashPro5 / 組込み FlashPro5** | **4** | General device I/O error | USB パケット消失 → ケーブルをしっかり挿し、他のケーブルを近くに置かない |

**PolarFire SoC（Table 9-2）**
| Error | Exit | メッセージ | 対処 |
|---|---|---|---|
| 0x8020 | -22 | Invalid Public Key for Boot Mode 3 eNVM client | 公開鍵を正しく入力し直す |

**PolarFire（Table 9-3 抜粋）**
| Error | Exit | メッセージ | 主な原因 → 対処 |
|---|---|---|---|
| -- | 0 | Passed | -- |
| 0x8003 | 5 | Failed to enter programming mode | 電圧の不足・不安定、JTAG の信号品質、DEVRST_N が LOW → DEVRST_N を HIGH、電源確認 |
| 0x8004 | 6 | Failed to verify IDCODE | ファイルかデバイスの取り違え、信号品質 → 正しいファイル、TRST をプルアップ、GND 短縮 |
| 0x8005-0x8009 | 11 | Failed to verify FPGA Array / Fabric Configuration / Security / sNVM / eNVM | 別の設計か未書き込み、電源・信号品質 → 正しい設計か確認 |
| 0x8013 | -18 | External digest check via JTAG/SPI Slave is disabled | FlashLock/UPK1 付きのファイルが要る |
| 0x801B | -20 | CHECK FABRIC / sNVM / eNVM / UKDIGESTn digest verification: FAIL | 消去済み・破損・改ざん → 再書き込み |
| 0x801C | -20 | SYS digest verification: FAIL | 工場データ消失 → デバイス交換（FA） |
| 0x801D | -21 | Device security prevented operation | UPK1 の不一致 → DEVICE_INFO で確認、合うファイルを使う |
| 0x801F | -22 | Bitstream data is corrupted or noisy | ビットストリームを作り直す、電源確認 |
| 0x8021 | -23 | Invalid/Corrupted encryption key | 鍵の不一致、またはユーザー鍵が未書き込み |
| 0x8023 | -24 | Back level not satisfied | デザインバージョンを上げる |
| 0x8001 | -24 | Failed to read DSN | SCSM 中 → TRSTB を High か SCSM 無効、DEVRST_N を HIGH |
| 0x8027 | -26 | Insufficient device capabilities | 対象デバイスに合ったファイルを作る |
| 0x8029 | -27 | Incorrect DEVICEID | ファイルかデバイスの取り違え、信号品質 |
| 0x802B | -28 | Signal integrity issues on JTAG pins | JTAG のノイズ・反射を測る |
| 0x8030 | -31 | Invalid or inaccessible Device Certificate | FAB_RESET_N を HIGH に |
| 0x8032 | -32 | Instruction timed out | 電源、信号品質、DEVRST_N |
| 0x8010 / 0x8011 | -35 | Failed to unlock user pass key 1 / 2 | パスキーが合うファイルを使う |
| 0x804F | -38 | Bitstream programming action is disabled | SPM で書き込みが禁止 → FlashLock/UPK1 付きのファイル |
| 0x805B | -42 | security must be either programmed on a blank device or with the FPGA Fabric design | セキュリティだけのファイル → 空のデバイスに書くか、ファブリック込みで作り直す |
| 0x8040 | -32 | Brownout error detected during programming | 十分で安定した電源を使う |
| 0x8051 | -40 | Failed to zeroize the device | JTAG/SPI 経由のゼロ化が無効 → ユーザーセキュリティ設定で有効化 |
| 0x800A / 0x800C | 11 | Failed to verify Security | ユーザーロック・永久ロックの設定が変わった |

**SPI Flash（Table 9-5 抜粋）**
| Code | メッセージ | 対処 |
|---|---|---|
| 6 | Failed to recognize device density / verify SPI-Flash IDCode | 未接続・非対応・未給電・信号品質 |
| 202 | Erase Action is not applicable to this device | エラーではない（Avalanche AS301G208/AS302G208） |
| 203 / 204 | Failed to program / verify failed | 電源・信号品質の確認、デバイス交換 / 正しいデータで書き直す |
| 207 | Invalid SPI-Flash image detected | プログラマ・USB ポート・ケーブルを替える |
| 209 | Image byte size is greater than available memory space | 容量の大きい Flash にする |
| 210 | SPI-Flash is not blank | 先に消去する |

## Anti-patterns
- **Exit 4（FlashPro5）を基板の故障と決めつける**: まず USB ケーブルと周りのノイズを疑う。オンボード eFP5 もこのコードの対象
- **0x8003 / 0x8032 で書き込みを何度もやり直す**: 先に電源と DEVRST_N を見る
- **0x801B を見て改ざんだと早合点する**: 消去済みや別の設計でも出る。まず同じファイルで書き直す
- **SmartFusion 2 / IGLOO 2 の表で PolarFire SoC のコードを引く**: 同じ番号でも意味が違う（例 0x8013）

## Key Takeaways
1. eFP5 の典型的なエラーは Exit 4（USB パケット消失）。ケーブルと取り回しを直す
2. 書き込み系の失敗で最初に試すのは「DEVRST_N を HIGH、電源の過渡確認、JTAG の信号品質」
3. PolarFire SoC 固有のコードは 0x8020（Boot Mode 3 の公開鍵）だけ。それ以外は PolarFire の表で引く
4. セキュリティ系は DEVICE_INFO で状態を確かめてから、合う鍵のファイルを用意する
5. SPI Flash の 202 はエラーではない

## Connects To
- **fpx-02**: Ping / Scan and Check Chain / DEVICE_INFO / VERIFY
- **fpx-03**: バッチ実行とリモートのトラブル
- **polarfire-soc-discovery-kit**: キットの JTAG 配線（FT4232H チャネル A）と電源
