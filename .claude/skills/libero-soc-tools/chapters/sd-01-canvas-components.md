# SmartDesign UG: Canvas とコンポーネントの配置・更新（PDF p.1-17）

## Core Idea
SmartDesign は IP コア・ユーザ HDL・他の SmartDesign をキャンバス上で結線し、DRC 済みで合成可能な HDL を生成するブロック図エディタ。生成物は FPGA 全体（トップ）にも、上位設計に再利用するサブシステムにもなる。部品の入れ方は「HDL モジュールをそのまま置く」「HDL+ コア化して BIF を持たせる」「Catalog の IP を Configure して置く」の 3 系統。

## Frameworks Introduced
- **SmartDesign 作成（p.7）**
  - File > New > SmartDesign、または Design Flow の Create SmartDesign
  - 名前を付けて OK → Design Hierarchy に出現。実体は `<project>\component\work\<name>\`
- **HDL ソースの取込み/新規作成（p.7-8）**
  - File > Import > HDL Source Files / HDL Source Folder → `<project>\hdl\` にコピーされる
  - File > New > HDL（既定 Verilog、標準テンプレートで初期化）
- **インスタンス化の 3 系統（p.9-13）**
  | 対象 | 置き方 | Configure 可否 |
  |---|---|---|
  | ユーザ HDL モジュール | Design Hierarchy からドラッグ、または右クリック Instantiate in `<active_SD>` | 不可（HDL エディタが開く） |
  | HDL+ コア | HDL を右クリック Create Core from HDL → ポート・パラメータ・**BIF** を定義 | パラメータ化 HDL コアとして可 |
  | IP コア | Catalog からドラッグ / Instantiate / Configure core → Create Component で名前 → Configurator | 可（IP コアのみ設定可能） |
  | SmartDesign / Design Block | Design Hierarchy からドラッグ | SmartDesign はキャンバスが開く |
- **HDL+ を使う判断**: 自作 HDL（例: LVDS 送受信ブロック）を AXI/APB などの BIF で一発結線したいときは Create Core from HDL で BIF を付ける。単純な信号結線だけなら素の HDL モジュールで足りる
- **下位の変更を上位へ反映する手順（p.15-16）**: 下位コンポーネントのポートが変わると上位のインスタンスが赤枠＋赤三角 ! になる
  1. 変更を含む一つ上の階層を開く
  2. インスタンスを右クリック → **Update Instance**
  3. その上位コンポーネントを Generate
  4. トップまで各階層で繰り返す（p.39）
- **コンポーネント差し替え（p.13-14）**: 右クリック Replace Component。ポート接続は維持。既定は同一コンポーネントの全インスタンスを置換。ポートリストが変わると一部接続が落ち、Log に出る
- **コアのバージョン更新（p.14-15）**: Update Component Version → Change to version。vault に無ければ自動ダウンロード。vault が read-only だと不可

## Key Concepts
- **Canvas**: 部品を列で配置し、列間をネットが縦に走る。入力ポートは左端、出力/inout は右端
- **Component / Instance**: Component は定義、Instance はキャンバス上の実体。同一 Component を複数置ける
- **HDL+ core**: ユーザ HDL から作るコア定義。BIF ピンを持てる
- **Catalog / vault**: IP カタログ。灰色斜体のコアは未ダウンロード（ダブルクリックか Download で取得）
- **Configurator**: IP コアの設定画面。再設定は右クリック Configure
- **Design Block**: Libero から publish された再利用ブロック
- **Component Types（p.59-60）**: Configured IP Core / Catalog 直置き IP / Block / SmartDesign / Parameterized HDL Core / HDL Module / Library Macro（例 RAM64x12）

## Reference Tables
**インスタンス右クリックメニュー（Table 3-1, p.16-17）**
| 操作 | 内容 |
|---|---|
| Configure | IP なら Configurator、SmartDesign なら canvas、HDL なら HDL エディタ（ダブルクリックと同じ） |
| Modify HDL | HDL コアのソースを開く |
| Replace Component / Update Component Version | 差し替え / バージョン変更 |
| Rename | HDL 命名規則違反は Log にエラー |
| Create / Flatten Hierarchical SmartDesign | 選択から子 SmartDesign を作る / 展開する |
| Remove Connections | 外せるピンを全切断（PAD 接続は外れない） |
| Highlight | 色付け。Unhighlight All で解除 |

**キャンバス操作（Table 1-1/1-2, p.4-6）**
- ドラッグ方向で Zoom: 左上=拡大、右上=縮小、左下=Fit、右下=範囲選択
- Ctrl+ドラッグ / ホイール押しドラッグで視点移動
- Reset Layout は配置・ハイライト・ピン順をすべて破棄して再配置。Auto-arrange は座標だけ変える
- Save to PDF でブロック図を出力

## Anti-patterns
- **下位 HDL のポートを変えたのに上位で Update Instance しない**: 赤三角のまま。DRC で Out-of-date Instance になる
- **Replace Component を全インスタンス置換の既定のまま実行**: 意図しないインスタンスまで差し替わる。1 個だけなら チェックを外す
- **Reset Layout を気軽に押す**: 手で整えた配置やピン順が消える。整列だけなら Auto-arrange

## Key Takeaways
1. SmartDesign の出力は DRC 済み・合成可能な HDL。トップにもサブブロックにもなる
2. Configure できるのは IP コアだけ。ユーザ HDL は HDL エディタで直す
3. 自作 HDL に BIF を持たせたいなら HDL+（Create Core from HDL）
4. 下位のポート変更は「Update Instance → Generate」をトップまで繰り返す
5. 灰色斜体の IP は vault 未取得。Download してから使う

## Connects To
- **sd-02**: ポート作成・BIF・結線
- **sd-03**: DRC、Generate、メモリマップ
- 姉妹 skill **polarfire-soc-discovery-kit**: リファレンスデザインの Tcl 生成（SmartDesign を含むプロジェクトが作られる）
