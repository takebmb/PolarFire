# Libero Tcl: メモリ初期化と NVM クライアント（PDF p.30, 413-437）

## Core Idea
書込みデータは **(1) FPGA Array Data 生成 → (2) メモリクライアント設定（eNVM / sNVM / SPI Flash / Fabric RAM）→ (3) `generate_design_initialization_data` → (4) Bitstream 生成 → (5) 書込み or ファイル出力**。本章は (2)(3)。eNVM に HSS を入れるのは `configure_envm`。クライアントの中身は `.cfg` ファイルに `set_*_client` コマンドとして書き、`configure_*` に渡す 2 段構え。

## Commands & Syntax

### 原文の書込みステップ全体（p.30）
```tcl
run_tool -name {GENERATEPROGRAMMINGDATA}
generate_design_initialization_data
run_tool -name {GENERATEPROGRAMMINGFILE}
export_bitstream_file -file_name {test} -format {STP DAT PPD} -trusted_facility_file 1 \
  -trusted_facility_file_components {FABRIC SNVM} ...
export_prog_job -job_file_name {test} -bitstream_file_type {TRUSTED_FACILITY} \
  -bitstream_file_components {FABRIC SNVM} -program_design 1 -program_spi_flash 0 \
  -design_bitstream_format {PPD} ...
close_project -save 1
```
- (4)(5) は [tcl-07-program-export.md](tcl-07-program-export.md)。

### configure_envm（p.416-417）
```
configure_envm -cfg_file {path_to_configuration_file.cfg}
```
- Design and Memory Initialization ツールの eNVM タブを .cfg で設定（PolarFire SoC 対応）。**後で必ず `generate_design_initialization_data`**。

### set_plain_text_client（p.432）— eNVM.cfg / sNVM.cfg に書く
```
set_plain_text_client -client_name {name} -number_of_bytes {number} \
  -content_type {MEMORY_FILE | STATIC_FILL} \
  -content_file_format {Microchip-Binary 8/16/32 bit} -content_file {path} \
  -start_page {number} -use_for_simulation {0} -reprogram {0 | 1} -use_as_rom {0 | 1}
```
| 引数 | 値・意味 |
|---|---|
| `-client_name` | 英字始まり、`_` と数字可 |
| `-number_of_bytes` | クライアントサイズ |
| `-content_type` | `MEMORY_FILE`（ファイル内容）/ `STATIC_FILL`（ゼロ埋め） |
| `-content_file_format` | 原文は「現時点で `Microchip-Binary 8/16/32 bit` のみ」 |
| `-start_page` | 配置ページ |
| `-use_for_simulation` | 原文は「0 のみ可」 |
| `-reprogram` | 1 = 最終デザインで書き込む |
| `-use_as_rom` | 1 = 読み出し専用 |

例:
```tcl
# HSS を eNVM クライアントとして追加するイメージ (原文外: ファイル名は仮。
#  リファレンスデザインの HSS_UPDATE はダウンロードした HSS hex から .cfg を作って configure_envm していると推測。
#  形式（Intel-Hex 指定や -mem_file_base_address の有無）は実スクリプトの .cfg を手本にすること)
configure_envm -cfg_file {./script_support/ENVM.cfg}
generate_design_initialization_data
```
- 原文 `set_cipher_text_auth_client` の例（p.430）には構文欄にない `-mem_file_base_address {0x00000000}` が現れる。
- sNVM は `configure_snvm -cfg_file`（p.417）。認証付きは `set_plain_text_auth_client` / `set_cipher_text_auth_client`（各ページ 236 バイト）+ `set_usk_client -start_page -key {24 桁 16 進} -use_for_simulation -reprogram`（認証付きクライアントがあれば必須）。
- SmartFusion 2 系の `run_tool -name {UPDATE_ENVM}` / `nvm_update_serialization_client` / `nvm_update_storage_client` は PolarFire SoC では使わない。

### generate_design_initialization_data（p.420-421）
```
generate_design_initialization_data
```
- メモリファイル作成・クライアント追加・.cfg 書き出し・検証。引数なし。エラーはログに出る。

### configure_design_initialization_data（p.413-415）
```
configure_design_initialization_data [-second_stage_start_address {...}] \
  [-third_stage_uprom_start_address {...}] [-third_stage_spi_start_address {...}] \
  [-third_stage_snvm_start_address {...}] \
  [-third_stage_spi_type {SPIFLASH_NO_BINDING_PLAINTEXT | SPIFLASH_BINDING_DEFAULT | SPIFLASH_BINDING_UEK1 | SPIFLASH_BINDING_UEK2}] \
  [-third_stage_spi_clock_divider {1 | 2 | 4 | 6}] [-init_timeout {int between 1 and 128 seconds}] \
  [-auto_calib_timeout {ms}] [-broadcast_RAMs {0 | 1}] [-custom_cfg_file {...}] [-standalone_initialization {0 | 1}]
```
| 引数 | 値・意味 |
|---|---|
| `-second_stage_start_address` | sNVM のページ境界。第 1 段は 0xDC00（page 220）固定なので 0x0〜0xDB00 |
| `-third_stage_*_start_address` | uPROM / sNVM / SPI の第 3 段開始アドレス（32bit 16 進文字列） |
| `-third_stage_spi_clock_divider` | 1 = 80 MHz（既定）、2 = 40、4 = 20、6 = 13.33 MHz |
| `-init_timeout` | 1〜128 秒（既定 128） |
| `-auto_calib_timeout` | I/O キャリブレーション待ち（既定 3000 ms、MPF100T〜MPF500T のみ有効） |
| `-standalone_initialization` | v2026.1 で追加。GUI の Standalone Initialization |
- sNVM は 221 ページ × 256 バイト（ユーザ使用可 252 バイト）。

### Fabric RAM 初期化（p.416, 422, 434）
```
configure_ram -cfg_file {path_to_configuration_file.cfg}
```
- .cfg 内: `set_client -logical_instance_name {...} -storage_type {SNVM | UPROM | SPI} -content_type {NO_CONTENT | MEMORY_FILE} -memory_file_format {Intel-Hex | Motorola-S | Simple-Hex | Microsemi-Binary} -memory_file {path}`（変更時は `modified_client`）。

### SPI Flash（p.418-420, 427-428, 434-435）
```
configure_spiflash -cfg_file {path_to_configuration_file.cfg}
export_spiflash_image [-file_name {name}] [-export_dir {dir}] [-format {STP | BIN}] [-spiflash_device {...}]
```
- .cfg 内:
```
set_client -client_name {client name} \
  -client_type {FILE_SPI | FILE_SPI_GOLDEN | FILE_SPI_UPDATE | FILE_DATA_STORAGE_INTELHEX} \
  -content_type {MEMORY_FILE | STATIC_FILL} -content_file {} -start_address {} -client_size {} -program {0|1}
set_auto_update_mode {0|1}
set_manufacturer {MICRON | SPANSION | Macronix | Winbond }
```
- `-start_address` / `-client_size` は 10 進バイト。先頭はディレクトリ用に予約あり。`-program` は 1 のみ対応。
- `export_spiflash_image -format STP` のときだけ `-spiflash_device`（MT25QL01GB, MT25QU01GB, MT25QL02GCBB, MT25QU02GCBB, W25Q64JV, MT25QL128）が必須。
- 書込み: `configure_tool -name {PROGRAM_SPI_FLASH_IMAGE} -params {spi_flash_prog_action:PROGRAM_SPI_IMAGE}` → `run_tool -name {PROGRAM_SPI_FLASH_IMAGE}`（ERASE_SPI_FLASH / READ_SPI_IMAGE / VERIFY_SPI_IMAGE も可。PolarFire 系 + FlashPro6 は部分消去可）。

## Anti-patterns
- **configure_envm の後に generate_design_initialization_data を忘れる**: クライアントがビットストリームに入らない
- **SmartFusion 2 用の UPDATE_ENVM / nvm_update_* を使う**: PolarFire SoC は configure_envm + .cfg
- **sNVM 第 2 段アドレスを 0xDC00 以上に**: 第 1 段クライアント領域と衝突
- **SPI クライアントサイズをファイルより小さく**: `-client_size` はファイルサイズ以上

## Key Takeaways
1. NVM 設定は「.cfg に set_*_client → configure_* → generate_design_initialization_data」
2. HSS（eNVM）は `configure_envm -cfg_file`。手本はリファレンスデザインが生成する .cfg
3. 第 1 段初期化クライアントは sNVM 0xDC00 固定
4. SPI Flash は configure_spiflash → PROGRAM_SPI_FLASH_IMAGE、または export_prog_job `-program_spi_flash 1`

## Connects To
- [tcl-07-program-export.md](tcl-07-program-export.md): Bitstream 生成・書込み・.job 出力
- [tcl-09-index-analysis-prog.md](tcl-09-index-analysis-prog.md): Ch.10 の全コマンド
- polarfire-soc-discovery-kit skill の `references/reference-design.md`（HSS_UPDATE）
