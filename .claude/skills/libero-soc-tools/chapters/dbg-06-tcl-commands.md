# SmartDebug UG: SmartDebug Tcl Commands（PDF p.190-273）

## Core Idea
GUI 操作の大半は Tcl で再現できる。Standalone SmartDebug の Project > Execute Script で実行し、Export Script File で GUI 操作をスクリプト化できる（TVS Monitor の GUI 操作は記録されない）。`-deviceName` はチェーンにデバイスが 1 つなら省略可。以下は PolarFire / PolarFire SoC で使うものに絞る（SerDes 系 `serdes_*` `prbs_test` `loopback_test` `ddr_read/write`(MDDR/FDDR) `check/read_flash_memory` は SmartFusion 2 / IGLOO 2 / RTG4 向け）。

## Commands & Syntax / Code Examples
**プロジェクト・接続（p.193, 207-209, 220-221, 231-232, 242, 250-251）**
```tcl
new_project [-location {project location}] [-name {name}] \
    -import_ddc {path to the DDC file} [-auto_construct {"TRUE"|"FALSE"}] \
    [-set_programmer {programmer}]
open_project -project {./SDProject/SDPrj.dprj}
close_project
import_ddc_file -import_ddc {./src/top.ddc} -device_name {MPF250T_ES}
set IDs [get_programmer_info]          ;# 接続中の FlashPro ID 一覧
set_debug_programmer -name {10841}
set_debug_device -name "MPF250T_ES"
read_id_code                           ;# 読めれば JTAG は正常
read_device_status [-deviceName "device name"] [-file "filename"]
rescan_programmer
update_fp6_programmers
```

**Live Probe（p.201, 244, 252-253, 263-264）**
```tcl
set_live_probe [-deviceName "device_name"] [-probeA "probe point A"] [-probeB "probe point B"]
unset_live_probe [-deviceName "device name"] [-probeA "TRUE | FALSE"] [-probeB "TRUE | FALSE"]
save_live_probe_list [-deviceName "device name"] -file {./live_probe_list.txt}
load_live_probe_list [-deviceName "device name"] -file {./live_probe_list.txt}
```
- probe 名は完全一致のみ（パターン不可）。A/B 両方使うなら 1 回の呼び出しで両方指定
- 原文の例は `unset_live_probes -probeA 1 -probeB 1`（複数形）と見出し `unset_live_probe` が食い違う。見出し側が正と思われる（原文外）

**Active Probe とグループ（p.191, 195, 200, 210, 214, 229, 240, 244, 246, 262, 264-265）**
```tcl
select_active_probe [-name probe_name_pattern]* [-reset true|false]   ;# * ? 可
read_active_probe [-deviceName device_name] [-name probe_name] \
    [-group_name bus_name | group_name] [-value_type b|h] [-file file_path]
write_active_probe [-deviceName device_name] -name probe_name -value true|false \
    -group_name group_bus_name -group_value "hex-value" | "binary-value"
write_active_probe -name out[5]:out[5]:Q -value true
write_active_probe -group_name grp1 -group_value "8'hF0"
create_probe_group -name my_new_grp
add_to_probe_group -name {DFN1_0_Q:DFN1_0/U0:Q} -group {probe_group}
move_to_probe_group -name {out[5]:out[5]:Q} -group {my_grp2}
remove_from_probe_group -name new_group.DFN1_0_Q:DFN1_0/U0:Q
ungroup -name "my_group"
delete_active_probe -all | -name {probe name}
save_active_probe_list -file "./my_probes.txt"
load_active_probe_list -file "./my_probes.txt"
```
- グループ/バス内の信号名は `group.signal` の形で前置する。1 信号でも読むとバス/グループ全体が読まれる
- group_value は `"<size>'h<value>"` / `"<size>'b<value>"`

**Probe Insertion（p.190-191, 228, 241）** — Integrated 専用
```tcl
add_probe_insertion_point -net {sw} -driver {sw_buf/IN:Y} -pin {Unassigned} -port {Probe_Insert0}
remove_probe_insertion_point -net {count_c[0]} -driver {Counter_out[0]:Q}
program_probe_insertion      ;# incremental P&R + 再書き込み
```

**FHB・計測（p.203, 205-208, 243）** — FHB 自動挿入が前提
```tcl
fhb_control -halt  -clock_domain {"FCCC_0/GL0_INST" "FCCC_0/GL1_INST"}
fhb_control -run   -clock_domain {all}
fhb_control -step {number of steps} -clock_domain {...}
fhb_control -reset -clock_domain {...}
fhb_control -arm_trigger -trigger_signal {q_0_c[14]:count_1_q[14]:Q} \
    -trigger_edge_select {rising} -delay 0 -clock_domain {"FCCC_0/GL0_INST"}
fhb_control -disarm_trigger -trigger_signal {...} -trigger_edge_select {rising} -delay 0 -clock_domain {...}
fhb_control -capture_waveform {10} -vcd_file {D:/wvf_location/waveform.vcd}
fhb_control -clock_domain_status -clock_domain {...}
set_live_probe -probeA {Q_c:DFN1_0:Q}
event_counter -run
event_counter -stop -after 5
run_frequency_monitor -signal {Q_c:DFN1_0:Q} -time {0.1}   ;# time は 0.1/1/5/8/10
get_user_clock_frequencies [-deviceName "M4"]
```

**メモリ・レジスタ（p.214-219, 230, 233-238, 245, 266-268）**
```tcl
read_lsram -name {.../INST_RAM1K20_IP} -fileName {output.txt}
read_lsram -logicalBlockName {Fabric_Logic_0/U2/F_0_F0_U1} -port {Port A}
write_lsram -name {PF_DPSRAM_C0_0/INST_RAM1K20_IP} -offset 3 -value 69905
write_lsram -logicalBlockName {PF_DPSRAM_C0_0/PF_DPSRAM_C0_0} -port {Port B} \
    -offset {1} -logicalValue {0xA} -tpsramValue 300
read_usram -logicalBlockName {Fabric_Logic_0/U3/F_0_F0_U1} -port {Port A}
write_usram -logicalBlockName {Fabric_Logic_0/U3/F_0_F0_U1} -port {Port A} -offset 1 -logicalValue {00FFF}
scan_ecc_memories [-fileName {./output.txt}]
mss_add_register -reg_name {ATHENA:CSRMAIN} -reset 0
mss_read_register [-reg_name {...}]          ;# 引数なしで追加済み全レジスタ
mss_write_register -reg_name {MMUART0_LO:IER} -value {0xFFFFFFFF}
mss_export_register -file_name {./mss_exp.csv} -all
mss_import_register -file_name {./MssRegisters_SmartDebug.csv}
read_envm_memory -startpage "0" -endpage "205"
read_snvm_memory -startpage "0" -endpage "2" -fileName "snvm.txt" -uskKey {0:0:0}
read_uprom_memory -startaddress {0xA} -words {10}
```

**IOD・DDR・TVS（p.196-197, 205, 261）**
```tcl
debug_iod -iod_type {RX_DDRX_B_G_DYN} -inst_path {PF_IOD_GENERIC_RX_C1_0}
        ;# iod_type: RX_DDRX_B_G_DYN / RX_DDRX_B_R_DYN / RX_DDRX_B_G_FDYN
debug_mss_ddr
export_mss_ddr_training_data -file D:/exportedData.txt
debug_ddr -ddr_type {DDR4} -data_width 32 -slot {NORTH_NE} -inst_path {PF_DDR4_C0_0} -frequency 800.00
export_ddr_training_data -file D:\exportedData.txt
tvs_monitor -start -interval 10 -file {./temp.csv}
tvs_monitor -stop
```

**トランシーバ（要点, p.202-204, 211-212, 221-226, 239, 253-261, 269-273）**: `smartbert_test -start|-stop|-reset_counter -lane {Q0_LANE0} -pattern {prbs7}`、`loopback_mode -lane {Q0_LANE0} -apply -type {EQ-NearEnd}`、`static_pattern_transmit`、`plot_eye -lane {...} -file {...}`、`eye_monitor_power`、`optimize_receiver` / `optimize_dfe`、`signal_integrity_export|import|write`、`xcvr_read_register` / `xcvr_write_register`、`transceiver_lane_reset`、`record_actions -start_recording` / `-stop_recording -file {...}`、`pcie_ltssm_status`

## Anti-patterns
- **`set_live_probe` を A と B で 2 回に分けて呼ぶ**: 片方の設定が他方に影響する
- **`select_active_probe` なしで `read_active_probe` を期待どおりに使う**: 読み対象は select で管理される
- **原文の例をそのまま貼る**: 例に誤記がある（`Select_active_probe` の大文字、`unset_live_probes`、`write_usram` 例中の `write_lsram`、`­` ソフトハイフン混入）。コピー時に `-` とコマンド名を確認する（原文外の注意）

## Key Takeaways
1. `read_id_code` が通れば JTAG は生きている
2. Live Probe は `set_live_probe -probeA ... -probeB ...` を 1 回で
3. Active Probe は select → read / write、グループは create / add / move / ungroup
4. FHB 系は `fhb_control`、計測は `event_counter` / `run_frequency_monitor`（どちらも Channel A 前提）
5. LVDS の IOD 確認は `debug_iod`、MSS DDR は `debug_mss_ddr`

## Connects To
- **dbg-01〜05**: 各 GUI 機能の説明
