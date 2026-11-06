# doc/

フォルダは skill と 1 対 1 に対応する。新しい資料は、内容に合うフォルダに置き、対応する skill の資料一覧に追記する。

| フォルダ | 中身 | 対応する skill | 資料一覧 |
|---|---|---|---|
| `board/` | Discovery Kit 固有の資料（UG、QSG、回路図、.brd、リファレンスデザイン README、組込みソフト UG、AN5165、AN5282） | polarfire-soc-discovery-kit | `.claude/skills/polarfire-soc-discovery-kit/references/sources.md` |
| `device/` | PolarFire SoC デバイスの資料（MSS TRM）。Datasheet、User I/O UG などを取得したらここに置く | polarfire-soc-mss | `.claude/skills/polarfire-soc-mss/SKILL.md` |
| `libero/` | Libero SoC v2026.1 のツール UG（11 本取得済み） | libero-soc-tools | `libero/CATALOG.md`（未取得の 38 本の URL も載っている） |

- `board/` と `device/` の PDF にはパスワード保護（閲覧制限）付きのものがある。図は `pdftoppm` で画像にして確認する
- Libero の未取得資料は `https://ww1.microchip.com/downloads/aemDocuments/documents/FPGA/swdocs/<libero|synopsys>/<ファイル名>` から curl で取れる
