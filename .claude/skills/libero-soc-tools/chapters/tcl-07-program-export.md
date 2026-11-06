# Libero Tcl: Bitstream 生成・書込み・ジョブ出力（PDF p.61-68, 80-83, 220-242, 426-429, 438-462）

## Core Idea
メモリクライアント確定後、`GENERATEPROGRAMMINGFILE` で含めるコンポーネント（Fabric/sNVM・eNVM・Security）を決めて Bitstream を作る。出口は 3 つ: **Libero から直接書く `run_tool -name {PROGRAMDEVICE}`**、**FlashPro Express 用 `.job` の `export_prog_job`**、**生ビットストリームの `export_bitstream_file`**。リファレンスデザインの `PROGRAM` / `EXPORT_FPE` 引数はそれぞれ前 2 者に相当する (原文外)。

## Commands & Syntax

### GENERATEPROGRAMMINGFILE（PolarFire SoC, p.228-230）
```
configure_tool -name {GENERATEPROGRAMMINGFILE} \
  -params {program_fabric_snvm: true | false} -params {program_security: true | false} \
  -params {program_envm: true | false} -params {sanitize_envm: true | false} \
  -params {sanitize_snvm: true | false}
run_tool -name {GENERATEPROGRAMMINGFILE}
```
| KEY | 意味 |
|---|---|
| `program_fabric_snvm` | Fabric と sNVM を含める |
| `program_security` | カスタムセキュリティを含める（定義時のみ true） |
| `program_envm` | eNVM を含める（eNVM がある時のみ true） |
| `sanitize_envm` / `sanitize_snvm` | サニタイズ有効化 |
- PolarFire（非 SoC）は `program_fabric` / `program_snvm`。名前が違うので流用しない。

### CONFIGURE_PROG_OPTIONS（p.222-224）
```
configure_tool -name {CONFIGURE_PROG_OPTIONS} -params {design_version:<value>} \
  -params {back_level_version:<value>} -params {silicon_signature:<value>}
```
| KEY | 値 |
|---|---|
| `design_version` | 0〜65535。SPM の Back Level より大きく |
| `back_level_version` | 0〜65535 |
| `silicon_signature` | 最大 8 桁 16 進（JTAG USERCODE で読める） |
- リファレンスデザイン引数 `DESIGN_VERSION:` / `SILICON_SIGNATURE:`（既定 D15C0417）はこれを設定していると考えられる (原文外)。
- `system_controller_suspend_mode` は v2026.1 から `new_project`/`set_device` の `-adv_options` へ移動。

### PROGRAMDEVICE（p.237-239）
```
configure_tool -name {PROGRAMDEVICE} -params {prog_action: params_value}
run_tool -name {PROGRAMDEVICE}
```
| `prog_action` | 内容 |
|---|---|
| `PROGRAM` | FPGA Array・eNVM クライアント・セキュリティを書込み（DO_VERIFY は任意） |
| `VERIFY` / `ERASE` | 検証 / 消去 |
| `DEVICE_INFO` / `READ_IDCODE` | 情報表示 / IDCODE 読出し |
| `VERIFY_DIGEST` / `ENC_DATA_AUTHENTICATION` / `READ_DEVICE_CERTIFICATE` | ダイジェスト照合 / 暗号認証 / 証明書読出し |
- 任意/推奨プロシージャ: `configure_tool -name {CONFIGURE_ACTION_PROCEDURES} -params {prog_optional_procedures:PROGRAM|DO_VERIFY;} -params {skip_recommended_procedures:}`（p.220）。

例:
```tcl
configure_tool -name {PROGRAMDEVICE} -params {prog_action:PROGRAM}   ;# (原文外: 原文例は VERIFY_DIGEST/DEVICE_INFO)
if {[catch {run_tool -name {PROGRAMDEVICE}}]} { puts "PROGRAM FAILED" }
```

### プログラマ設定（p.239-242, 426-429）
```
configure_tool -name {PROGRAMMER_INFO} -params {name:value}
select_programmer -programmer_id {programmer id} [-host_name {host name}] [-host_port {host port}]
set_programming_interface [-interface {JTAG | SPI_SLAVE}]
```
- FlashPro6 用 KEY: `flashpro6_force_tck_freq`（ON/OFF）、`flashpro6_tck_freq`（1〜6, 10, 16, 30 MHz）、`flashpro6_force_sck_freq`、`flashpro6_sck_freq`（既定 20 MHz、最大 40）、`flashpro6_vpump`（既定 ON）。FlashPro3/4/5 は `flashpro<N>_clk_mode` / `_force_freq` / `_freq` / `_vpump`。
- Discovery Kit 搭載の組込みプログラマがどの種別として見えるかは別資料で確認 (原文外)。
- 複数プログラマ接続時は `select_programmer` で 1 台に絞る（リモートは `-host_name` / `-host_port`、既定ポート 80）。

### export_bitstream_file（PolarFire SoC 構文, p.61-68 抜粋）
```
export_bitstream_file [-file_name "file_name"] [-export_dir "export_dir"] \
  [-format "PPD | DAT | HEX | STP | SPI" ] \
  [-trusted_facility_file "TRUE | FALSE"] \
  [-trusted_facility_file_components "FABRIC | SNVM | ENVM | FABRIC_SNVM" ] \
  [-master_file "TRUE | FALSE" ] [-master_file_components "SECURITY | FABRIC | SNVM | ENVM | FABRIC_SNVM" ] \
  [-encrypted_uek1_file ...] [-encrypted_uek2_file ...] \
  [-zeroization_likenew_action "TRUE | FALSE" ] [-zeroization_unrecoverable_action "TRUE | FALSE" ] \
  [-sanitize_snvm "TRUE | FALSE" ] [-sanitize_envm "TRUE | FALSE" ] \
  [-trusted_facility_keep_fabric_operational "TRUE | FALSE" ] [-uek1_high_water_mark {1234} ] ...
```
| 引数 | 値・意味 |
|---|---|
| `-file_name` | **設計名で始めること**。既定は設計名 |
| `-export_dir` | 既定 `<designer>/<design_name>/export` |
| `-format` | 空白区切りで複数。省略時 PPD と DAT。PolarFire 系は CHAIN_STP / SVF 不可 |
| `-trusted_facility_file` 他 | trusted_facility / master / uek1 / uek2 のどれか 1 つ以上を 1。trusted_facility=1 なら他は 0 |
| `-zeroization_unrecoverable_action` | 1 = 全消去し**再書込み不能** |
| `*_high_water_mark` | OWP 有効かつ SPI 出力時に必要 |

例:
```tcl
# 原文の既定セキュリティ例に eNVM を追加 (原文外)
export_bitstream_file -trusted_facility_file 1 -trusted_facility_file_components {FABRIC SNVM ENVM}
```

### export_prog_job（PolarFire 系構文, p.80-83）
```
export_prog_job -job_file_name {file} -export_dir {dir} \
  -bitstream_file_type {TRUSTED_FACILITY | MASTER | UEK1 | UEK2} \
  -bitstream_file_components {SECURITY | ENVM | FABRIC | SNVM | FABRIC_SNVM} \
  -zeroization_likenew_action {0 | 1} -zeroization_unrecoverable_action {0 | 1} \
  -program_design {0 | 1} -program_spi_flash {0 | 1} -include_plaintext_passkey {0 | 1} \
  -design_bitstream_format {PPD | STP} \
  -prog_optional_procedures {action1|procedure1|procedure2;action2|procedure1|procedure2|procedure3;} \
  -skip_recommended_procedures {action1 | procedure1 | procedure2 ; action2 | procedure1 | procedure2 | procedure3;}
```
- 既定: type `TRUSTED_FACILITY`、format `PPD`（FlashPro6 で高速）、`program_design 1`、`program_spi_flash 0`。`SECURITY` は MASTER 時のみ。SNVM は FABRIC と一緒に。プログラミングモード（JTAG/SPI-Slave）も job に入る。戻り値 0/1。

例:
```tcl
# 原文 Ch.2 例を基に eNVM（HSS）を含め出力先を指定 (原文外)
export_prog_job -job_file_name {MPFS_DISCO} -export_dir {./export} \
  -bitstream_file_type {TRUSTED_FACILITY} -bitstream_file_components {FABRIC SNVM ENVM} \
  -zeroization_likenew_action 0 -zeroization_unrecoverable_action 0 \
  -program_design 1 -program_spi_flash 0 -design_bitstream_format {PPD}
```

### FlashPro Express Tcl（Ch.11, p.438-462）
```
create_job_project -job_project_location {job project location} -job_file {path and name of job file} -overwrite value
set_programming_action [ -name {name} ] -action { action }
run_selected_actions [-name "name"] [-prog_spi_flash "TRUE | FALSE"] [-disable_prog_design "TRUE | FALSE"] \
  [-spi_flash_image "spi_flash_image"] [-spi_flash_action "spi_flash_action"]
```
- `.job` から FPExpress プロジェクトを作り、`set_programming_action -action {PROGRAM}` → `run_selected_actions`（`-name` 省略で全接続プログラマ）。`get_connected_programmers` / `refresh_prg_list` / `ping_prg` で接続確認。
- FPExpress 自体の起動・バッチ引数は本書に記載なし（FlashPro Express 資料の章を参照 (原文外)）。

## Anti-patterns
- **PolarFire 用 `program_fabric` を PolarFire SoC に渡す**: SoC は `program_fabric_snvm`
- **`-zeroization_unrecoverable_action 1`**: デバイス廃棄。既定 0 のまま
- **ジョブから ENVM を落とす**: HSS が書かれず Linux が起動しない (原文外: eNVM ブート前提の場合)
- **プログラマを複数挿したまま PROGRAMDEVICE**: `select_programmer` で特定（リファレンスデザインも「接続は 1 台のみ」）
- **`-file_name` を設計名以外で始める**: export_bitstream_file の制約違反

## Key Takeaways
1. PolarFire SoC の Bitstream 構成は `program_fabric_snvm` / `program_envm` / `program_security`
2. 直接書込みは `configure_tool -name {PROGRAMDEVICE} -params {prog_action:PROGRAM}` → `run_tool`
3. 配布は `export_prog_job`。eNVM を含めるなら `-bitstream_file_components {FABRIC SNVM ENVM}`
4. 版数・署名は `CONFIGURE_PROG_OPTIONS`

## Connects To
- [tcl-06-memory-init.md](tcl-06-memory-init.md): eNVM/sNVM/SPI クライアント（Bitstream の前段）
- [tcl-05-design-flow.md](tcl-05-design-flow.md): run_tool の全体像
- FlashPro Express の章（fpx-*）、polarfire-soc-discovery-kit skill の AN5165 手順
