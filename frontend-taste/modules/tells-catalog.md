---
module: tells-catalog
parent: frontend-taste
---

# Tells カタログ (観測ベース)

「AI が作った」とバレる視覚パターン (tells) の台帳。**出所を観測に限定し、観測なしの空想 tells を収載しない**。出所は 4 系統: ① frontend-design (公式 plugin) が名指しする AI default 3 look ② taste-skill 翻案のうち本環境で再現を観測したもの ③ 本環境の実生成物の観測 ④ trend 調査の「default 化」リスト (出典付き)。日本語圏固有 tells は初版は少数で正直に開始し、運用 (親 skill §6 の操作基準) で育てる。

検知は 2 形式に統一する。**主観の「なんとなく AI っぽい」を検知基準にしない**:

- **機械**: grep / 数え上げで判定できる条件 + 閾値 (判断ゼロ)
- **判断**: レンダリング screenshot に対して yes/no で答えられる判定質問。yes ならヒット

各項目の「宣言」列は「テンプレ宣言の有無」属性 — テンプレ設計単位の signature として**宣言済みの系列を default 化判定 (親 skill §6) から除外する**ための印。宣言済みの採用は Pre-Flight 手順 4 (subject 由来の理由 + 代替 1 案比較) を経ることが条件で、禁止クラスは置かない (親 skill §5 手順 4 と同義)。

## 1. 機械検査シード (Pre-Flight 手順 3 で必ず照合)

| # | 検知条件 | 閾値 | 出所 |
|---|---|---|---|
| M1 | palette 外 hex の出現 (base 4〜6 hex + 明度/alpha 派生以外の独立 hue。色は CSS 変数経由必須) | 出現 1 以上で violation | 親 skill §5 の palette 操作的定義 |
| M2 | 均一カード族: `repeat(auto-fill, minmax(…))` / `repeat(auto-fit, minmax(…))` / 固定数の `repeat(N, 1fr)`・`repeat(N, minmax(…))` + 同一スタイルのカード反復 | 出現 1 以上で照合ヒット (→ #5) | 本環境実測 (auto-fill 形)。固定数形は #5 の同族定義による |
| M3 | gradient (`linear-gradient` / `radial-gradient` 等) の出現数 | 2 以上で照合ヒット (→ #4) | 本環境実測 |
| M4 | border-radius 値のユニーク数 | 1 (全要素同一角丸) で照合ヒット (→ #6) | taste-skill 翻案 (本環境観測は未) |

**複合条件 (旗艦・default look #1 の捕捉)**: 単独条件をすべて通過する実例を本環境で観測済みのため、次の同時成立で捕まえる — cream 帯域の背景 hex (暖色の生成り白。R・G が高く B がそれより低い) + 明朝/serif 見出し (`font-family` に serif・Mincho 系) + 中央寄せ hero (`text-align: center`) + gradient 出現 1 以上。**4 条件同時成立 → Pre-Flight 手順 4 の宣言強制を自動トリガー**。

## 2. AI default 3 look (出所 ①: frontend-design が名指し)

| # | 症状 | 検知 | なぜ tells か | 代替 | 宣言 |
|---|---|---|---|---|---|
| 1 | default look #1: cream 地 + 高コントラスト serif/明朝見出し + terracotta 系アクセント | 機械 (§1 の複合条件) | subject と無関係に頻出する既定の組み合わせ。本環境でも実測 (→ §3 末尾の実測記録)。trend 調査でもテンプレ化の兆候が確認されている (→ trend-log) | subject の実在物から palette を引く (subject-materials)。この look 自体が subject 由来なら宣言して採用可 | 宣言可 |
| 2 | default look #2: near-black 地 + acid-green / vermilion の単発アクセント | 判断「暗背景 + 蛍光系アクセント 1 色だけの構成か」 | 「それっぽい tech 感」の既定。subject を選ばない | アクセント hue を subject の実在物から取る。暗背景自体は環境のダークモード規範と両立可 | 宣言可 |
| 3 | default look #3: broadsheet 風 (hairline 罫線 + border-radius 0 + 新聞的密度カラム) | 判断「hairline 罫線・角丸ゼロ・多段組が subject と無関係に採用されていないか」 | エディトリアル調の既定。内容が編集物でないのに新聞の皮を着せる | 罫線・密度は情報構造に従わせる (structure is information) | 宣言可 |

## 3. 本環境の実測 (出所 ③: 飲食店 HP テンプレートの CSS で観測)

| # | 症状 | 検知 | なぜ tells か | 代替 | 宣言 |
|---|---|---|---|---|---|
| 4 | 中央寄せ hero + linear-gradient 背景 | 機械: hero 相当ブロックに `text-align: center` + `linear-gradient` (M3 と連動) | hero をどう開くか考えない時の既定形。frontend-design の言う「hero is a thesis」の放棄 | subject の最も特徴的な要素で開く (非対称配置・実在物の写真/図・タイポ主役等) | 宣言可 |
| 5 | 均一カード族: 同一サイズ・同一 radius・同一 shadow のカードの敷き詰め | 機械 (M2) | 内容の重要度差を消し、どの業種でも同じ画になる。固定数の `repeat(N, 1fr)` リテラルも auto-fill/auto-fit + minmax の自動敷き詰めも同族として M2 で捕まえる | 内容の重みに応じたサイズ差 (bento 的な仕切り)・リスト/表など構造に合う形式への変更 | 宣言可 |

- #1 の複合一致も同テンプレートで実測済み: 背景 #fdf9f3 (cream 帯域) + 明朝系見出し + terracotta #b5482f + 中央寄せ hero + gradient 出現 2。単独の機械条件では M2・M3 しか引っ掛からず、look #1 としての「無難さ」は複合条件で初めて捕まえられた — これが §1 の複合条件の存在理由。

## 4. 観測待ち (出所 ②: taste-skill 翻案・本環境での再現観測は未)

| # | 症状 | 検知 | なぜ tells か | 代替 | 宣言 |
|---|---|---|---|---|---|
| 6 | 均一 border-radius (全要素同一の角丸 1 値) | 機械 (M4) | 角丸に役割の差 (小要素/カード/ピル) が無いのは shape を設計していない兆候 | 用途別の radius スケールを組む | 宣言可 |

- 本節は taste-skill から翻案した項目のうち**本環境での再現観測がまだ無いもの**を置く。機械条件 (M4) のみ有効として運用し、実例は運用で収集する (再現を観測したら §3 へ昇格)
- 観測済みの非該当反例: radius 3 トークンを小要素 4px / カード 10px / ピル 999px と用途別に使い分けた例 (§3 のテンプレートと同一)

## 5. trend 調査の default 化 (出所 ④: 出典付き。全量と更新履歴は trend-log)

| # | 症状 | 検知 | なぜ tells か | 代替 | 宣言 |
|---|---|---|---|---|---|
| 7 | 紫〜青グラデーション (indigo 系) | 機械: `linear-gradient` 内に紫〜青帯域の hex (#6366f1 等 indigo/violet 系) | Tailwind の既定色に由来する「AI 生成 UI の代名詞」との言説が広く共有される ([prg.sh](https://prg.sh/ramblings/Why-Your-AI-Keeps-Building-the-Same-Purple-Gradient-Website)・[dev.to](https://dev.to/alanwest/why-every-ai-built-website-looks-the-same-blame-tailwinds-indigo-500-3h2p)) | subject 由来の hue へ差し替え | 宣言可 |
| 8 | 素朴な glassmorphism 乱用 (半透明 + blur カードの敷き詰め) | 機械: `backdrop-filter: blur` の出現 3 以上 / 判断「ガラスが情報の階層を運んでいるか」 | コントラスト不足・描画負荷の実害が指摘される。設計されたガラス表現 (Liquid Glass) とは別物 ([eidosdesign](https://eidosdesign.substack.com/p/glassmorphism-at-apple-new-old-trend)) | 使うなら少数の焦点要素に限定し可読性を実測 | 宣言可 |
| 9 | Inter 一辺倒 (訴求面での) | 機械: A 訴求面の display 用途に Inter 単独指定 | 「9 割が Inter」という Inter 疲れの言説 ([Medium](https://medium.com/design-bootcamp/inter-how-designers-are-slowly-stripping-away-your-brands-soul-with-a-font-fcf58ee1deaf))。B 業務面では合理的選択のため対象外 | display face を subject に合わせて選ぶ (制約下は明朝/ゴシック軸の較正・親 skill §4) | 宣言可 |
| 10 | shadcn/ui 既定テーマそのまま (未カスタマイズ) | 判断「色トークン・角丸・タイポのいずれも既定値のままか」 | 既定スタイルの量産が「Sea of Sameness」を加速させるとの指摘 ([freedesignmd](https://freedesignmd.com/blog/shadcn-looks-generic))。既定値の放置は設計の不在に近く、subject 由来の宣言が最も成立しにくい項目 | テーマ層で色トークン・角丸・タイポ・アクセントを上書きしてから使う | 宣言可 |
| 11 | 均一 16px 角丸カード + 片側太カラーボーダー | 機械: `border-left` (または片側) に太幅 + アクセント色のカード反復 | 「AI slop」の定番サインとして列挙される ([925studios](https://www.925studios.co/blog/ai-slop-web-design-guide))。「Build the future」型の抽象見出しも同記事の定番だがコピー管轄のため文言専用の skill へ送る | 強調は面・タイポ・余白で作る | 宣言可 |
| 12 | 3 列 feature grid (アイコン上・テキスト下) + 4 列フッター + nav (logo 左/リンク中央/CTA 右) の複合構成 | 判断「機能紹介が均等 3 列アイコングリッド・フッターが均等 4 列・nav が logo 左/リンク中央/CTA 右の教科書配置を、subject の情報構造を検討せず採用していないか」 | AI 生成 UI の均質化言説で hero+gradient と並び名指しされる構成要素 ([managed-code.com](https://www.managed-code.com/blog-post/why-ai-websites-look-the-same)・[shuffle.dev](https://shuffle.dev/blog/2026/01/why-do-most-ai-generated-websites-look-the-same/)) | 機能の数・重要度差に応じた列数・配置に変える (bento 的な仕切り含む) | 宣言可 |

## 6. 運用

- 項目追加は観測とセット (どこで・いつ・何を観測したか 1 行残す)。trend 由来は出典 URL 必須
- 機械条件は**観測された違反形に即して書く** (例: #5 は `repeat(3, 1fr)` 等の固定数リテラルと auto-fill/auto-fit + minmax を合わせて「族」として定義した。単一表記の決め打ちは変形をすり抜ける)
- 宣言は禁止の解除ではなく採用の条件: どの項目も Pre-Flight 手順 4 の宣言 (subject 由来の理由 + 代替 1 案比較) を書けば採用できる (親 skill §5 手順 4 のとおり禁止クラスは置かない)
- 削除・降格は `/taste-audit` の裁定で行う (trend 由来項目は trend-log の 90 日ルールに連動)
