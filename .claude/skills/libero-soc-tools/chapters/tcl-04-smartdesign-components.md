# Libero Tcl: SmartDesign とコンポーネント（PDF p.46-48, 53, 69, 73, 88-89, 96, 101, 108, 127, 150-218, 261-262）

## Core Idea
SmartDesign の Tcl は「部品を作る → 置く → つなぐ → 生成する」の 4 段。部品は (1) カタログ IP を `create_and_configure_core` でコンポーネント化、(2) HDL モジュール、(3) 他の SmartDesign、(4) MSS（`.cxz` を `import_mss_component`）。ピン名は **`<インスタンス名>:<ピン名>`**、トップポートは名前だけ。最後に `generate_component` で HDL を生成しないと合成に入らない。

## Commands & Syntax

### コンポーネントを作る
**create_and_configure_core（p.47-48）**
```
create_and_configure_core -core_vlnv {Vendor:Library:Name:Version} \
  -component_name "component_name" -params "core_parameters"
```
| 引数 | 値・意味 |
|---|---|
| `-core_vlnv` | IP の VLNV（例 `Actel:SgCore:PF_CCC:1.0.115`） |
| `-component_name` | 作るコンポーネント名（既存名は不可） |
| `-params` | `"KEY:VALUE"` のリスト。DirectCore のパラメータ名は各コアのハンドブック参照 |

例:
```tcl
# 原文例。PLL_IN_FREQ_0 を Discovery Kit の 50 MHz 基準クロック想定に変更 (原文外)
create_and_configure_core -core_vlnv {Actel:SgCore:PF_CCC:1.0.115} \
  -component_name {PF_CCC_C0} \
  -params {"PLL_IN_FREQ_0:50" "GL0_0_IS_USED:true" "GL0_0_OUT_FREQ:150" \
           "GL0_1_IS_USED:true" "GL0_1_OUT_FREQ:50"}
```
- 既存コンポーネントの変更は `configure_core -component_name {PF_CCC_C0} -params {...}`（p.46）。
- VLNV のバージョンはインストール済みカタログに依存。`download_core -vlnv` / `download_latest_cores` で取得 (p.55-56)。

**import_mss_component（p.101, PolarFire SoC 専用）**
```
import_mss_component [-file {path to the *.cxz file}]
```
- PolarFire SoC MSS Configurator で作った `.cxz` を取り込む。リファレンスデザインは MSS Configurator を batch 実行して `.cxz` を作り、これで import する (原文外: README §2.5 より)。

例:
```tcl
import_mss_component -file {./src/file_name.cxz}   ;# 原文例。実際のパスはリファレンスデザインの script_support 配下を確認
```

**その他**: `import_component -file <x.cxf>`（p.96）、`create_hdl_core -file -module`（p.211、HDL+ コア化してバスIF を付ける）、`delete_component -component_name`（p.53）。

### SmartDesign を作る・保存する
```
create_smartdesign -sd_name "smartdesign_component_name"
open_smartdesign [-sd_name "smartdesign_component_name"]
save_smartdesign -sd_name "smartdesign_component_name"
generate_component -component_name "component_name" [-recursive 0|1 ]
```
- `open_smartdesign` は sd_* の前に open か create が必要。ただしスクリプト生成用途では不要と原文注記。
- `generate_component -recursive 1` で未生成の依存コンポーネントも生成。原文は個別生成を推奨。出力は `<prj>/component/work/<name>`。

### 置く（インスタンス化）
```
sd_instantiate_component -sd_name {sd} -component_name {component module name} [-instance_name {instance name}]
sd_instantiate_core      -sd_name {sd} -core_vlnv {vendor:library:name:verison} [-instance_name {name}]
sd_instantiate_hdl_module -sd_name {sd} -hdl_module_name {hdl module name} -hdl_file {hdl file} [-instance_name {name}]
sd_instantiate_hdl_core  -sd_name {sd} -hdl_core_name {hdl core module name} [-instance_name {name}]
sd_instantiate_macro     -sd_name {sd} -macro_name {macro module name} [-instance_name {name}]
```
| コマンド | 対象 | 注意 |
|---|---|---|
| `sd_instantiate_component` | SmartDesign・コア・MSS コンポーネント | 既定インスタンス名 `<名前>_<index>` |
| `sd_instantiate_core` | カタログ IP を直接（Direct Instantiation） | 構成は `sd_configure_core_instance` + `sd_save_core_instance_config` |
| `sd_instantiate_hdl_module` | import/link 済み HDL のモジュール | `-hdl_file` は import なら相対可、link ならフルパス |
| `sd_instantiate_macro` | プリミティブ（例 `MX2`, `MACC_PA`） | |

例:
```tcl
# 原文例を基に LVDS 受信モジュール想定へ変更 (原文外)
sd_instantiate_component -sd_name {top} -component_name {PF_CCC_C0}
sd_instantiate_hdl_module -sd_name {top} -hdl_module_name {lvds_rx} -hdl_file {hdl/lvds_rx.v} -instance_name {lvds_rx_0}
```

### つなぐ
```
sd_create_scalar_port -sd_name {sd} -port_name {port name} -port_direction {IN|OUT|INOUT}
sd_create_bus_port    -sd_name {sd} -port_name {port name} -port_direction {IN|OUT|INOUT} -port_range {[left:right]}
sd_connect_pins       -sd_name {sd} -pin_names {port or pin or slice names}
sd_connect_pin_to_port -sd_name {sd} -pin_name {pin name} [-port_name {port name}]
sd_connect_instance_pins_to_ports -sd_name {sd} -instance_name {instance name}
sd_connect_pins_to_constant -sd_name {sd} -pin_names {port or pin names} -value {constant value}
sd_mark_pins_unused   -sd_name {sd} -pin_names {port or pin names}
sd_invert_pins        -sd_name {sd} -pin_names {port or pin names}
sd_create_pin_slices  -sd_name {sd} -pin_name {port or pin _name} -pin_slices {port or pin slices}
```
| コマンド | 役割 |
|---|---|
| `sd_connect_pins` | 列挙したポート/ピン/スライスを 1 本のネットで結ぶ。サイズ不一致・ドライバ 2 本はエラー |
| `sd_connect_pin_to_port` | GUI の「Promote to Top Level」。ポート名省略時はピン名、重複時は `_<index>` |
| `sd_connect_pins_to_constant` | `-value` は `GND` / `VCC` / 16 進数 |
| `sd_mark_pins_unused` | 未使用出力の DRC 警告を消す |
| `sd_create_pin_slices` | バスを `{[4:3] [2:0]}` のように分割 |
| `sd_create_scalar_net` / `sd_create_bus_net` + `sd_connect_net_to_pins` | ネット名を明示したいとき |
- 原文注記: `sd_connect_pins_to_constant` / `sd_invert_pins` / `sd_mark_pins_unused` は v2021.1 で複数ピン非対応、とある（古い注記）。1 ピンずつ書くのが安全 (原文外)。
- `sd_connect_pin_to_port` は「Export Component Description」には出力されない対話操作扱い。

例:
```tcl
# 原文例を基にピン名を変更 (原文外)
sd_create_scalar_port -sd_name {top} -port_name {REF_CLK_50MHz} -port_direction {IN}
sd_connect_pins -sd_name {top} -pin_names {REF_CLK_50MHz PF_CCC_C0_0:REF_CLK_0}
sd_connect_pins -sd_name {top} -pin_names {PF_CCC_C0_0:OUT0_FABCLK_0 lvds_rx_0:clk}
sd_mark_pins_unused -sd_name {top} -pin_names {PF_CCC_C0_0:PLL_LOCK_0}
sd_connect_pins_to_constant -sd_name {top} -pin_names {lvds_rx_0:bypass} -value {GND}
save_smartdesign -sd_name {top}
generate_component -component_name {top}
```

### 変更・保守
- `sd_update_instance -sd_name -instance_name`（p.209）: 下位のポートが変わったら更新。
- `sd_replace_component`（p.194）、`sd_rename_port` / `sd_rename_instance` / `sd_rename_net`、`sd_delete_instances` / `sd_delete_ports` / `sd_delete_nets`、`sd_disconnect_pins`。
- `export_component_to_tcl`（p.69）: コンポーネントを再生成する Tcl を出力。手書きの雛形に使う。
- `export_interrupt_map -file {x.json} -sd_name {top}`（p.73）: 割込み接続マップを JSON 出力（PolarFire SoC 向け）。
- 合成属性: `sd_set_port_synth_attr` 等（p.196-206）。

### MSS Tcl（Ch.7, p.261-262）
```
mss_configure_envm -component_name {component_name} -cfg_file {file_path}
mss_configure_instance -component_name {component_name} -instance_name {instance_name} -params{param:value list}
mss_enable_instance / mss_disable_instance / mss_save_instance_config -component_name {...} -instance_name {...}
```
- 原文の例は `test_sb_MSS`（SmartFusion 2 系の命名）で対応ファミリの記載なし。PolarFire SoC の MSS は MSS Configurator（`.cfg` → `.cxz`）で設定するのが前提 (原文外)。

## Anti-patterns
- **generate_component を忘れる**: SmartDesign の HDL が古いまま合成される
- **ピン名にインスタンス名を付けない**: `CLK` はトップポート、`inst:CLK` がインスタンスピン
- **HDL のポートを変えたのに sd_update_instance しない**: 接続が旧定義のまま
- **VLNV のバージョンを決め打ち**: カタログに無いとエラー。`download_core` するか実環境の版に合わせる

## Key Takeaways
1. 順序: コンポーネント作成 → `create_smartdesign` → `sd_instantiate_*` → `sd_connect_*` → `save_smartdesign` → `generate_component`
2. ファブリックに LVDS 回路を足すなら HDL を import → `sd_instantiate_hdl_module` → トップポート作成 → 接続
3. MSS は `.cxz` を `import_mss_component`、構成変更は MSS Configurator 側
4. GUI で作った回路は `export_component_to_tcl` で Tcl 化できる

## Connects To
- [tcl-03-files-constraints.md](tcl-03-files-constraints.md): HDL の import とトップ設定
- [tcl-05-design-flow.md](tcl-05-design-flow.md): 生成後の合成
- polarfire-soc-mss skill: MSS 周辺・FIC の中身
