---
name: frontend-taste
description: HTML/LP・Web アプリ・フォーム/業務 UI・ダッシュボード等の視覚成果物から「AI 生成の無難さ (AI default)」を排除し、subject 固有の taste を出す規範。視覚成果物の設計・生成に着手するとき、およびユーザー提示前の Pre-Flight 検収として使う。手動 (/taste = 検収・/taste-audit = trend 棚卸し) で起動する。コピー (文言) は別の文言専用 skill の管轄で、本 skill は視覚のみを扱う。
---

# Frontend Taste — 視覚デザイン taste 規範

frontend 領域全般 (Web の HP/LP・Web アプリ・フォーム/業務 UI・ダッシュボード、モバイル UI への共通原理適用) の視覚成果物から AI default を排除し、subject 固有の taste を出す。

着想: [taste-skill](https://github.com/Leonxlnx/taste-skill) (MIT) と Anthropic 公式の frontend-design plugin。本 skill の各項目はこれらの verbatim 転記ではなく、実際の生成物の観測に基づく独自の書き下ろし。**共通の視覚規範 (palette lock・typography 役割・signature・motion 抑制・default 検知) を扱う専用のデザインシステム skill/plugin を別途運用している場合は、そちらを制作時の SSOT にして、本 skill の Pre-Flight (§5) を検収として組み合わせるとよい**。本 skill が持つ差分は 4 点: ① 適用軸と優先順位 (§1〜2) ② 検証手順 = Pre-Flight (§5) ③ 観測ベース tells カタログ (`modules/tells-catalog.md`) ④ trend 運用 (従・§6)。

## 起動条件

- **キーワード検知による誘導**: HTML/LP/ホームページ/UI/デザイン/ダッシュボード/フォーム画面等のキーワードを検知して本 skill への誘導と Pre-Flight の要約を注入する仕組みを持っている場合はそれに従う (スライド生成系の skill を別途運用している場合、スライド関連キーワード (スライド/Marp/pptx/デッキ/プレゼン) はそちらを優先させ、本 skill の対象に含めない)
- **手動**: `/taste` = 成果物の Pre-Flight 検収 / `/taste-audit` = trend-log の棚卸し (こちらは**自動発火機構を持たない・手動専用**)

## 1. taste モードと制約フラグ

**taste モード (単一軸・2 値)** — デザイン目的で選ぶ。着手前に必ず判定する:

| モード | 対象 | taste の出しどころ |
|---|---|---|
| A 訴求面 | LP・HP・プロモ・ポートフォリオ | subject 由来の palette・signature・レイアウト個性 |
| B 業務面 | フォーム・管理画面・ダッシュボード | 密度・情報構造の明瞭さ・エラー設計・a11y。装飾最小 |

**制約フラグ (taste モードと直交・複数選択)** — ハード制約はモードと分離して申告する:

- 配信制約: CDN 可否 / フォント調達 (システムのみ・セルフホスト可・web font 可) / JS 予算
- 実装形態: 静的 HTML / framework (React 等)
- プラットフォーム: web / モバイル。モバイルは共通原理 (default 検知・palette lock・宣言強制) のみ適用し、詳細はプラットフォーム DS (HIG/Material) を優先する。モバイル専用 tells は実案件の観測が溜まってから module 追加する (観測なしの詳細規範を先に書かない)

## 2. 優先順位と境界 (競合時にどちらが勝つか)

視覚系・データ可視化系の skill や規範を他にも運用している場合の、一般的な優先順位の考え方:

| 競合相手の性質 | 勝ち負け |
|---|---|
| 既存の確立したデザインシステム (Material Design 等) への準拠が求められる業務系案件 | そのデザインシステムのルールが勝つ / 訴求面のブランド表現は本 skill が勝つ |
| チャートの配色 (データ可視化専用の skill/ライブラリがある場合) | アクセシビリティ保証済みのパレット・バリデータが勝つ (a11y > subject palette) |
| 環境固有のダークモード規範・可読性優先規範 | 背景系トーンは環境規範が勝つ。アクセント・構成・signature は subject 由来 (本 skill) |
| スライド生成系の skill (対象外) | スライドの SSOT が勝つ。ただしスライド関連キーワードに当たらない縦 1 枚プレゼン HTML 等は本 skill の対象 |
| ライトモード・可読性優先の社内向けダッシュボード規範等 | 対象外 (別規範) |
| コピー (文言) 専用の skill ([ai-slop-jp](../ai-slop-jp/SKILL.md) 等) | コピーはそちらの管轄。本 skill の Pre-Flight は文言項目を持たない (相互参照) |
| 配色候補の比較ツール | 補助併用可 (配色候補の比較に使う) |

## 3. 規範 × モード適用マトリクス

◎ 必須 / ○ 効く / △ 文脈次第 / ✗ 課すと壊れる:

| 規範 | A 訴求面 | B 業務面 |
|---|---|---|
| signature (記憶に残る 1 要素) | ◎ (テンプレ量産時は template-level・§4) | ✗ (課すと過剰装飾。gate は密度・エラー網羅・a11y に差し替える) |
| subject 由来 palette lock | ◎ | ○ (brand color 1 hue + neutral 階調) |
| motion | ○ (orchestrated 1 演出まで) | △ (状態遷移のみ) |

Pre-Flight (§5) のチェック項目もモード別に分岐する (一律ゲートにしない)。

## 4. テンプレ量産の二層 taste

量産 (骨格使い回し + 差し込み) と anti-generic ゲートの矛盾は、taste を二層に分けて解く:

- **template-level taste**: 骨格そのものが tells に該当しない設計。signature は成果物単位でなく**テンプレ設計単位**で持たせる (例: 業種の world 由来のレイアウト構造)。使い回しても無難に落ちない骨格が量産の前提条件
- **deliverable-level taste**: 案件ごとの差し込み (subject materials) で個性を立てる。**色と語の差し替え (recolor) は signature とみなさない**

**厳制約 (CDN 不使用・システムフォント・JS 最小) 下の positive method** — 除外リストでなく「やれること」を名指しする:

- 非対称レイアウト・グリッドの崩し (CSS のみで可能)
- 余白のリズム設計・セクション間の密度コントラスト
- 級数 (サイズ) ジャンプ・ウェイト・字間による日本語タイポの較正 (書体は実質 明朝/ゴシックの 2 軸選択と正直に認める。セルフホストはサブセット化手順 (pyftsubset・unicode-range・サイズ上限) を書いた場合のみ許可)
- 写真ゼロ前提の figure 表現 (色面・罫線・タイポグラフィックな図) と、実写真が入った時に化ける差し込み口の設計
- 罫線・コントラスト・背景の面分割

**足し算層**: 設計前に subject の実在物 (色・形・質感・写真・語彙・数字・固有名詞) を収集する step を必須にする。収集の型は [`modules/subject-materials.md`](modules/subject-materials.md)。引き算 (tells 回避) だけでは「無臭で無個性」までしか行けない ([ai-slop-jp](../ai-slop-jp/SKILL.md) の「実在物が無いなら捏造せず、無いなりの設計に落とす」考え方と同構造)。

## 5. Pre-Flight (ユーザー提示前の最終成果物に適用)

手順固定。この順で必ず回す:

1. **レンダリング → screenshot 取得** (Playwright / chrome-devtools の viewport emulation を使う)。取得不能な環境ではコード検査のみに縮退し、その旨を明示する。

   **⚠️ 幅の測定は「要求した幅で測れているか」を先に検証する** — 要求値と実効値がずれる経路が 4 つ実測されており、いずれも「崩れていない」という偽陰性を生む:

   | # | 経路 | 症状 | 実測 |
   |---|---|---|---|
   | ① | Chrome headless CLI (`--screenshot`) | window 幅 500px 未満を 500px に切り上げる | 375 指定時 innerWidth=500・右端 125px 欠落 |
   | ② | chrome-devtools MCP の `emulate` / `resize_page` | 成功と報告して override が効かない (並列セッションとの競合時) | 375 指定時 innerWidth=449 のまま |
   | ③ | Playwright の viewport | 同一ブラウザを共有する別セッション・別 agent に横取りされる | 375 設定後に 1440 へ変わっていた |
   | ④ | ローカル HTTP サーバー (`python3 -m http.server`) | no-cache を付けないため、修正直後もブラウザが旧 HTML を返す | 修正済みなのに修正前の値 (溢れ 74px) を測っていた |

   **対策 (3 点を手順に含める)**:

   - **`window.innerWidth === 要求値` をアサートしてから測る**。①〜③ はこれで捕まる
   - **④ はアサートを通過する**ため、クエリ文字列の付与か `fetch(url, {cache: 'reload'})` で**キャッシュを回避**してから測る
   - 並列 agent がいる状況では、**幅を指定した iframe 内で測る**と viewport の横取り (③) を構造的に回避できる。あわせて**実装は agent に任せ測定は親が独占する**分担も有効
   - 幅を測るときは `document.body.scrollWidth === window.innerWidth` (横溢れ 0) も同時に確認する。②④ を踏んだ状態ではこの検査が実バグ 1 件 (プレースホルダのトークン長が `white-space: nowrap` の grid セルで min-content を押し広げる) を見逃していた
2. **画像に対して tells 照合** ([`modules/tells-catalog.md`](modules/tells-catalog.md) を Read)。CSS テキストだけで視覚を採点しない
3. **機械検査** (grep 可能な検知条件): tells-catalog の機械項目を検知条件 + 閾値どおりに照合する。シード 4 条件 = palette 外 hex の出現ゼロ (CSS 変数経由必須) / 均一カード族 (auto-fill・auto-fit + minmax、および固定数の repeat) / gradient 出現 2 以上 / border-radius 値のユニーク数 1。**旗艦 default look #1 は複合条件で捕まえる** (単独条件をすべて通過する実例を観測済みのため): cream 帯域の背景 hex + 明朝/serif 見出し + 中央寄せ hero + gradient の同時成立 → 手順 4 の宣言強制を自動トリガー
4. **default 一致時は宣言強制**: tells・default look に一致した場合、禁止ではなく「subject 由来の理由 + 代替 1 案との比較」を書いてから採用可とする (brief が default を正当化するケースを塞がない。自己申告の「default 非該当」チェックは置かない)
5. コピーは文言専用の skill へ ([ai-slop-jp](../ai-slop-jp/SKILL.md) 等。視覚 Pre-Flight に文言項目を持たない)

**palette の操作的定義**: base 4〜6 hex + 各 base からの機械的派生 (明度/alpha 操作) は可。**独立 hue の追加のみ禁止**。

## 6. trend 運用 (従・safe degradation)

- **原理 (§4〜5) が主・trend-log は従**。trend-log の更新が止まっても本 skill は成立する
- [`modules/trend-log.md`](modules/trend-log.md) frontmatter の `last_audit:` から **90 日超過の項目は「参考情報」に降格し、採否判定 (disqualify) に使わない**
- 実行は手動 `/taste-audit` のみ。一次手段は WebSearch/WebFetch による一次情報 (公式 DS リリースノート・CSS 仕様・State of CSS・アワード総括)。外部 LLM API は API キー保有時のみ・環境変数経由限定・平文直書き禁止とし、外部送信を制限する仕組みを別途運用していてそれと競合する場合は Web 調査に代替する
- **default 化判定の操作基準 (無自覚の収斂に限定)**: 直近の自己生成 3 本で同一パターンが 2 回以上出たら tells 送り。ただし対象は (a) テンプレ宣言の無い文脈での未登録パターンの反復、(b) 業種・テンプレを**跨いで**現れる骨格反復のみ。§4 で宣言済みの template-level signature の同一系列内再利用は除外し、その系列の検査は deliverable-level (差し込みの質・recolor 逃げの有無) に切り替える

## 注意事項

- modules 構成 (3 本): `modules/tells-catalog.md` (観測ベース tells・検知条件と閾値) / `modules/trend-log.md` (trend 台帳・90 日降格) / `modules/subject-materials.md` (足し算層の収集の型)。Pre-Flight 実行時は tells-catalog を必ず Read し、他は必要時のみ Read する
- tells は観測に限定する。**観測なしの空想 tells を追加しない**。出所は 4 系統のみ: frontend-design が名指しする AI default 3 look / taste-skill 翻案のうち本環境で再現を観測したもの / 本環境の実生成物の観測 / trend 調査の「default 化」リスト (出典付き)
- 「静かなページ + 1 signature」という本 skill 自身が生む新定型の監視は、未登録パターンの反復・テンプレ跨ぎの骨格反復に限定する (宣言済み系列内の意図的再利用は対象外)
- リリース後の較正は、プロンプト・skill の反復改善サイクルを別途運用している場合はその枠で行う (§6 の操作基準で tells を育てる)
