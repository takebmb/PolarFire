# Chapter 6 (Appendix A): Programming Using the Onboard FlashPro5（UG p.17-20）

## Core Idea
外部プログラマは不要。Libero で作った `.job`（FlashPro Express ジョブファイル）を FlashPro Express で開き、オンボード eFP5 経由で RUN する。

## Frameworks Introduced
- **FlashPro Express 書き込み手順**
  - 前提: ホスト PC に最新版の FlashPro Express
  - 準備:
    1. ホスト PC → J4 に USB-C を接続
    2. J47 で目的の電源元を選ぶ
    3. 電源 LED の点灯を確認
  - 書き込み:
    1. FlashPro Express を起動
    2. **New**、または **Project > New Job Project from FlashPro Express Job**
    3. Create New Job Project ダイアログ:
       - *Import FlashPro Express job file*: Browse → `.job` を選択
       - *FlashPro Express job project location*: Browse → 保存先を選択
    4. **OK** → FlashPro Express のメイン画面が開く
    5. **Programmer** 欄にプログラマ番号が出ていることを確認。出ていなければ接続を確認し **Refresh/Rescan Programmers**
    6. **RUN** → 成功すると **RUN PASSED**

## Key Concepts
- **FlashPro Express**: ジョブファイルで書き込む軽量ツール（Libero のプロジェクトは不要）
- **.job file**: Libero の Export FlashPro Express Job で作るプログラミングジョブ
- **eFP5**: オンボード FlashPro5（U16）。USB-JTAG
- **Programmer box**: 認識されたプログラマ番号が表示される欄
- **RUN PASSED**: 書き込み成功の表示

## Anti-patterns
- **プログラマ番号が出ないまま RUN を押す**: まず接続を確認し、Refresh/Rescan する
- **古い FlashPro Express を使う**: UG は最新版を前提にしている
- **Libero を入れずにドライバが入ったと思う**: プログラマのドライバは Libero SoC のインストーラに含まれる（Ch2）

## Key Takeaways
1. 必要なのは `.job` と FlashPro Express だけ
2. Programmer 欄が空なら Refresh/Rescan Programmers
3. RUN PASSED が出れば完了

## Connects To
- **Ch2**: Libero / ドライバのインストール、J47
- **Ch3**: JTAG（Bank 3）、eFP5 は SmartDebug / Identify / SoftConsole のデバッグにも使う
