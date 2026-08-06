---
module: trend-log
parent: frontend-taste
last_audit: 2026-07-20
---

# Trend 台帳

視覚デザイントレンドの採否台帳。**原理 (親 skill §4〜5) が主・本台帳は従** — 更新が止まっても skill は成立する (safe degradation)。

- **90 日降格ルール**: frontmatter の `last_audit` から 90 日を超えた項目は「参考情報」に降格し、採否判定 (disqualify) に使わない。tells-catalog へ送付済みの項目は tells 側の観測が生きている限り有効
- 更新は手動 `/taste-audit` のみ (自動発火機構なし)。一次手段は WebSearch/WebFetch による一次情報 (公式 DS リリースノート・CSS 仕様・State of CSS・アワード総括)
- 出典の格付け: **[一次]** = 当該分野の公式発行元自身のドキュメント・ブログ / **[言説]** = 個人ブログ・第三者メディア・百科事典の観測・意見 (断定的数値としては採用しない)

## 1. 採用候補

| パターン | 実装可否 | 要点 | 出典 |
|---|---|---|---|
| CSS Scroll-Driven Animations | 素の CSS (`@supports` フォールバック前提) | JS 不要のスクロール連動 (パララックス・リビール)。Chrome/Edge 完全・Safari 17+ 部分・Firefox フラグ (2026-07-19 時点 約 85%) | [一次] [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations)・[WebKit](https://webkit.org/blog/17101/a-guide-to-scroll-driven-animations-with-just-css/) |
| View Transitions API | 素の JS + CSS | 同一ドキュメントは 2025-10 に Baseline Newly available 到達。JS アニメライブラリの置き換え事例あり | [一次] [web.dev](https://web.dev/blog/same-document-view-transitions-are-now-baseline-newly-available) |
| CSS Anchor Positioning + Popover API | 素の CSS + HTML | ツールチップ・ドロップダウンをライブラリなしで。Baseline 2026 到達 (Chrome 125+/Firefox 132+/Safari 18.2+・約 91% カバー)。ただし `@position-try` (フォールバック) は Firefox 147+ 要 | [一次] [Chrome for Developers](https://developer.chrome.com/blog/anchor-positioning-api)・[言説] [OddBird](https://www.oddbird.net/2025/10/13/anchor-position-area-update/) |
| Container Queries | 素の CSS | 認知 86% に対しサイズクエリ実利用 41%・スタイルクエリ 7% (浸透途上) | [一次] [State of CSS 2025](https://2025.stateofcss.com/en-US/features/) |
| `:has()` / Subgrid | 素の CSS | 使用率 80.4% で最愛用機能に定着。ラッパー div と JS 分岐を減らせる | [一次] [State of CSS 2025](https://2025.stateofcss.com/en-US/features/) |
| Variable Fonts | 素の CSS | 1 ファイルで太さ・幅可変。サイズ削減の実利。主流化 | [言説] [Creative Boom](https://www.creativeboom.com/insight/font-trends-2025/) |
| Bento Grid | 素の CSS Grid/Subgrid | サイズ差で内容の重要度を運ぶ — **均一カード族 (tells #5) の正当な代替** | [言説] [Medium](https://medium.com/design-bootcamp/web-design-trend-bento-box-95814d99ac62) |
| WebGL/Shader 没入表現 | ライブラリ必要 (Three.js 等)・コスト高 | 映像との境界溶解 (歪み・グリッチ・ポストプロセス)。実装予算と相談 | [一次] [Awwwards](https://www.awwwards.com/the-rise-of-shaders-filters-and-effects-in-web-projects.html) |
| グレイン/ノイズテクスチャ | 素の CSS/SVG (`feTurbulence`) | 「AI 生成の完璧すぎる質感」への対抗。質感カテゴリ (subject-materials) と相性良 | [言説] [CSS-Tricks](https://css-tricks.com/grainy-gradients/) |
| マキシマリズム (折衷) | 表現次第 | 均質ミニマリズムへの反動。要所だけ大胆に (全面適用は文脈選定要) | [言説] [mindbees](https://www.mindbees.com/blog/maximalism-design-trend-2025/) |
| 巨大・キネティックタイポグラフィ | 素の CSS で概ね可 (`clamp()` + scroll-driven 併用) | 文字主役の構成。厳制約下の positive method (親 skill §4) と直結 | [一次] [Figma](https://www.figma.com/resource-library/web-design-trends/) |
| ダークモード標準化 + グロー表現 | 素の CSS | 環境のダークモード規範と親和。グローは焦点 1 箇所に | [一次] [Figma](https://www.figma.com/resource-library/web-design-trends/) |
| Liquid Glass (Apple・iOS 26〜27) | Web 適用は可読性配慮とセット | 設計されたガラス表現。透明度のユーザー調整へ揺り戻し (iOS 27) — 素朴な glassmorphism (tells #8) とは別物 | [一次] [Apple Newsroom](https://www.apple.com/newsroom/2025/06/apple-introduces-a-delightful-and-elegant-new-software-design/) |
| Material 3 Expressive (Google) | M3 案件で | 研究裏付けの springy モーション物理。確立された操作パターンを壊すと有用性を損なう点に注意 | [一次] [Google Blog](https://blog.google/products-and-platforms/platforms/android/material-3-expressive-android-wearos-launch/) |

## 2. default 化 (tells-catalog へ送付済み)

| パターン | tells # | 要点 | 出典 |
|---|---|---|---|
| 紫〜青グラデーション (indigo 系) | #7 | Tailwind 既定色起点で「AI 生成 UI の代名詞」化したとの言説 | [言説] [prg.sh](https://prg.sh/ramblings/Why-Your-AI-Keeps-Building-the-Same-Purple-Gradient-Website)・[dev.to](https://dev.to/alanwest/why-every-ai-built-website-looks-the-same-blame-tailwinds-indigo-500-3h2p) |
| 素朴な glassmorphism 乱用 | #8 | コントラスト不足・描画負荷の実害指摘 | [言説] [eidosdesign](https://eidosdesign.substack.com/p/glassmorphism-at-apple-new-old-trend) |
| Inter 一辺倒 (訴求面) | #9 | 「9 割が Inter」の Inter 疲れ言説。業務面では合理的選択 | [言説] [Medium](https://medium.com/design-bootcamp/inter-how-designers-are-slowly-stripping-away-your-brands-soul-with-a-font-fcf58ee1deaf) |
| shadcn/ui 既定テーマ未カスタマイズ | #10 | 既定スタイル量産による「Sea of Sameness」加速 | [言説] [freedesignmd](https://freedesignmd.com/blog/shadcn-looks-generic)・[AXE-WEB](https://axe-web.com/insights/ai-website-design-sameness/) |
| AI slop 視覚定型 (均一 16px 角丸カード・片側太ボーダー) | #11 | 「バレるサイン」として反復列挙。抽象見出しはコピー管轄で文言専用の skill へ | [言説] [925studios](https://www.925studios.co/blog/ai-slop-web-design-guide) |
| cream + serif + terracotta | #1 | 紫グラデの代替として広がったが、テンプレライブラリ登録を確認 — default 化の初期段階。無警戒な採用は避ける | [言説] [DESIGN.md Library](https://designmd.app/library) |
| 3 列 feature grid (アイコン上・テキスト下) + 4 列フッター + nav (logo 左/リンク中央/CTA 右) の複合 | #12 | AI 生成 UI の「均質化」言説で hero+gradient と並び具体的に名指しされる構成要素。複数独立ソースが同一の複合パターンを列挙 | [言説] [managed-code.com](https://www.managed-code.com/blog-post/why-ai-websites-look-the-same)・[shuffle.dev](https://shuffle.dev/blog/2026/01/why-do-most-ai-generated-websites-look-the-same/) |

## 3. 見送り (退潮)

| パターン | 判定理由 | 出典 |
|---|---|---|
| ニューモーフィズム | 低コントラスト・操作要素の視認性低下という実用欠陥で「トレンドでなくテクニック」に格下げ | [言説] [Webflow Blog](https://webflow.com/blog/neumorphism) |
| Corporate Memphis (ぬるっとした人物イラスト) | 陳腐化・パロディ対象。「完璧でない」手描き表現への揺り戻し | [言説] [Creative Bloq](https://www.creativebloq.com/news/corporate-memphis-style-is-dead) |
| トレンド牽引型ブルータリズム | 関心下降局面との言説 (月次データは一次未達で断定しない)。洗練されたネオブルータリズム・機能的ブルータリズムは別枠で定常 | [言説] [superdesign.dev](https://superdesign.dev/styles/brutalism) |
| 均質化した純ミニマリズム | サンセリフ + 無彩色への収斂批判。要所でのアクセント使用は継続 | [言説] [mindbees](https://www.mindbees.com/blog/maximalism-design-trend-2025/) |

## 4. 参考 (日本市場メモ・採否ラベル対象外)

- 縦長・情報量重視の LP 慣習: 「十分に説明しないと不親切」の価値観。BtoB・高額サービスで強い ([言説] [live-commerce](https://www.live-commerce.com/ecommerce-blog/landing-page/))
- コーポレートサイトの書体使い分け基準: 伝統 = 明朝系 / 先進 = モダンゴシック / 堅実 = バランス型ゴシック ([言説] [thinkbal](https://thinkbal.co.jp/magazine/uxui/company-website-design/))
- コーポレートサイトのモーション・没入化: パララックス・スクロールテリング。Scroll-Driven Animations 普及と同方向 ([言説] [jpc-ltd](https://www.jpc-ltd.co.jp/web/magazine/corporate/corporatedesign-trend/))
- 縦書き Web・和文可変フォントの動向は一次情報に到達できず未記載 (空欄を埋めない)

## 更新履歴

- 2026-07-19: 初版。UI/UX デザイントレンド調査 (2026-07-19 実施・WebSearch/WebFetch 一次情報主義) から台帳形式に整形して移植
- 2026-07-20: 較正 1 回目。90 日降格対象なし (経過 1 日)。CSS Anchor Positioning の Firefox 対応を訂正 (Baseline 2026 到達を確認)。「3 列 feature grid + 4 列フッター + nav 型」の複合パターンを default 化候補として追加 → tells-catalog #12 へ送付
