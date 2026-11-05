# Chapter 8: Revision History / Microchip FPGA Support（UG p.24-26）

## Core Idea
文書は DS50003630E（Rev E、2025/07）。古い版には無い表があるので、ピン表を引くときは版を確認する。

## Reference Tables
| Rev | 日付 | 主な変更 |
|---|---|---|
| E | 07/2025 | Table 7-8（Push-button ピン）を追加 |
| D | 12/2024 | 50/125 MHz 発振器の型番を Fig 1-1、Table 1-2、Fig 3-2/3-3 に追加 |
| C | 11/2024 | Appendix B（User Hardware Pinout）を追加 |
| B | 04/2024 | Fig 1/2 更新、「2-Position Terminal Block」→「5V DC Jack」に置換、Table 3-1 追加、125 MHz 発振器の節を追加、Fig 3-1 に FPGA ピン名、Fig 4-1/4-2 更新 |
| A | 12/2023 | 初版 |

## Key Concepts
- **技術サポート**: www.microchip.com/support。ケース作成時に FPGA デバイス品番を書き、デザインファイルを添付する
- **Customer Service**（価格・注文など）: 北米 800.262.1060 / その他 650.318.4460 / Fax 650.318.8044
- **ISBN**: 979-8-3371-1624-2

## Anti-patterns
- **Rev A/B の資料でピン表を探す**: Appendix B は Rev C 以降、Push-button の表は Rev E で追加
- **「端子台給電」と書いた古い資料を信じる**: Rev B で 5V DC Jack に置き換わった

## Key Takeaways
1. 最新は Rev E（DS50003630E）
2. サポートに問い合わせる前に、オンラインの資料を確認する
3. ケースにはデバイス品番 MPFS095T-1FCSG325E とデザインファイルを添える

## Connects To
- **Ch6**: Appendix B（Rev C/E で追加された内容）
