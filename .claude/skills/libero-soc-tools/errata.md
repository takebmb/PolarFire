# 原文の誤記・矛盾（各章ファイルにも注記済み）

| 資料 | 内容 | 章ファイル |
|---|---|---|
| PDC | 例の一つが `-iostd`（正しくは `-io_std`） | pdc-02 |
| PDC | `edit_io` の構文が `-VCIM_RANGE`（set_io は `-VCM_RANGE`。受理される綴りは未確認） | pdc-05 |
| PDC | `-io_std` の表（Table 3-7）に LVDS33 の行が無い | pdc-02 |
| PDC | CLAMP_DIODE の表で LVDS25 が「OFF/ON 既定 ON」と「ON のみ」の両方の行にある | pdc-03 |
| I/O Editor | Resistor Pull の既定値が Port View の節（Up）と Pin View の節（None）で食い違う | ioed-01 |
| Timing Constraints Editor | SDC の例に en dash（`–period`）が混じる。章ファイルでは `-` に直した | tce-03〜06 |
| SmartTime STA | Datasheet Report は PolarFire 非対応（p.92）なのに `report -type datasheet` の例が残っている | sta-03 |
| SmartTime STA | CDC の色対応表（Table 8-7）の表組みが崩れ、色が読み取れない | sta-04 |
| Tcl Reference | p.27 の例が廃止された `-pdc` を使っている | tcl-03 |
| Tcl Reference | `SCRIPT_ARGS` の区切り（空白か `+` か）と、`lindex $argv 0` が何を指すかの記述が揺れている | tcl-01 |
| Tcl Reference | family の表記が `PolarFire SoC`（空白あり）。GUI と AN5282 は `PolarFireSoC` | tcl-02 |
| SmartDebug | Tcl の例に綴りの誤り（`unset_live_probes`、`Select_active_probe`、ソフトハイフンの混入） | dbg-06 |
| License Guide | Linux 向けの `export LM_LICENSE_FILE port@hostname` に `=` が抜けている | inst-02 |
| Release Notes | v2026.1 の Welcome Page が MPFS250TC を「MPF200TC」と表示する | rn-01 |
