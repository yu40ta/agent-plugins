---
name: advisor-pattern
description: 安価な executor モデルがタスク中の判断ポイントごとに上位モデル (advisor) へ相談する「advisor delegation」パターンの opt-in 判断支援 skill。成立条件ゲート・advisor モデル選定・既知の制約 (組織アクセスゲート等) を判定する。手動起動のみで自動発火はしない。
---

# Skill: Advisor Pattern

上位モデルが下位 worker にタスクを事前に fan-out する一般的なオーケストレーションパターンとは逆方向 — **安価な executor が判断ポイントで上位モデルに相談する**パターン (advisor delegation) の opt-in 判断支援。

## 起動条件

- 手動起動。「executor + advisor 構成にしたい」「上位モデルに都度相談させたい」「Fable を advisor にしたい」等、明示的にこのパターンを要求されたとき
- 自動発火はしない

---

## なぜ opt-in か

一次資料 (Fable コスト効率ハーネスの実測記事、2026-07-12 参照) では効果がタスク形状で正反対に振れる。Parameter Golf (ML 実験タスク) では Sonnet executor + Fable advisor 構成が、Fable 単独の改善効果の約 90% を約 34% のトークンコストで達成した。一方 BrowseComp200 (小規模・1 問あたり約 0.37M トークン) では同種の委譲構成が単独実行より 60% 高コストで性能向上ゼロだった。**効果はタスク形状・規模に依存し、常時有効ではない**ため opt-in とする。

---

## Phase 1: 成立条件ゲート

着手する**前**に、以下 4 条件をすべて満たすか判定する。**1 つでも欠けたら不採用** (直接実行を選ぶ)。

| # | 条件 | 欠けた場合の代替 |
|---|---|---|
| 1 | **判断がタスク全体に分散している** (1 時点の計画・レビューだけでなく実行中に繰り返し判断が要る) | 判断が計画時点に集中 → 上位モデルが直接計画した方が適する。判断が完了確認の 1 点に集中 → 「検証時点だけ上位モデルが介入する」verifier delegation パターンの方が適する |
| 2 | **委譲コストを上回るトークン量がある** (advisor への文脈転送・応答読解は 1 相談あたり概ね固定費) | 相談 1 回の文脈量が小さい → BrowseComp200 (0.37M トークン/問) で委譲が+60%高コストだった実測どおり元が取れない |
| 3 | **executor 側で prompt cache を維持できる実行系** (同一 worker に呼び出しを集約できる) | 呼び出しごとに新規 worker を起動する設計 → cache 再構築コストが委譲の効果を相殺する |
| 4 | **単独実行との総額比較が済んでいる** (委譲前に単独実行のトークン量見積もりと比較する。$/token が安いことは総額が安いことを意味しない) | 比較せず着手 → 規模依存で結果が反転しうる。BrowseComp フル (31M トークン/問、上位モデルが fan-out する構成) はスコア 96% をコスト 46% で達成した一方、小規模の BrowseComp200 では同種構成が損失超過だった |

1 つでも欠けたらこのパターンを採用しない。

**高難度タスクへの適用制限**: 難易度が高い実装タスクを安価 executor + advisor 構成で受けるのは、上記ゲートの通過に加えて**同型タスクでの計測実績 (Phase 4) がある場合に限る**。未実測のまま高難度タスクを本構成で受けず、難易度が高いタスクは既定で上位モデルの直接実行に倒す。

---

## Phase 2: advisor モデル選定

**advisor モデルはパラメータ化する。固定しない** (下記の理由により fable 決め打ちは事故る)。

| advisor 候補 | 特性 |
|---|---|
| **opus (既定)** | 組織アクセス不要。advisor tool の一般的なモデルペアリングで動作する |
| **fable** | 品質は最高だが要件あり: Claude Code v2.1.170 以降 **かつ組織の Fable 5 アクセス**が必要 |

**公式ドキュメント確認済み事実** (`code.claude.com/docs/en/advisor`、2026-07-12 取得):
- 有効化経路は `/advisor` コマンド / `advisorModel` 設定 / `--advisor` フラグの 3 つ。alias (opus/sonnet/fable) またはフル ID を受理
- `/advisor`・`advisorModel` 経由では、ペアリング不成立や allowlist 除外のモデルでも選択自体は保存され、advisor が付かないだけ (黙って不使用)
- `--advisor` フラグは「main model が advisor 非対応」または「組織の `availableModels` allowlist から除外」の場合に明示的にエラー終了する
- advisor は main model と同等以上の capability が必要 (例: Fable 5 main の advisor は Fable のみ)

**ユーザーの一次体験** (2026-07-12・doc 未記載の情報として区別する): fable を advisor に設定はできたが、呼び出すタイミングでエラーになった。上記の組織アクセス要件と整合するが、具体的な発生条件・エラーメッセージは doc に明記されていない。doc 確認済み事実とは帰属を分けて扱う。

**本セッション実測の代替経路** (2026-07-12・1 回のみ): Agent ツールで `model: 'fable'` の subagent を直接起動する方法は、advisor tool の組織アクセスゲートを経由せずにエラーなく起動・応答した。ただし:
- 1 環境 1 回の実測であり、全環境・全アカウントでの再現は未確認。断定せず都度再確認する
- ビルトインの advisor tool (サーバーサイドで全 transcript を自動転送) と、Agent ツール経由の subagent 起動 (呼び出し側がプロンプトで文脈を構成する手動シミュレーション) は別機構。後者は前者の代替にはなるが同一実装ではない

---

## Phase 3: 実行規範

- **相談ポイントは事前 1 回に固定しない**: 一次資料の実験では advisor の初期 (事前) ランキングは実際に効いた改善と逆相関だった一方、実行中の複数チェックポイントでの相談 (steering・re-prioritization) が効いた。判断がタスク中に分散しているなら相談も分散させる
- **prompt cache は worker 単位で維持する**: 同一 worker (同一 subagent) に呼び出しを集約し、相談のたびに新規 subagent を起動しない。cache を毎回作り直すと安価な executor を使う効果が相殺される
- **委譲コストは固定費に近い**: boundary duplication (境界を越えるトークンは最低 2 回課金) と fan-out overlap (worker 間の重複調査) が 1 回の handoff あたりほぼ固定費でかかる。上位モデルはトークン効率自体が高いことがあり、$/token の安さと総額の安さは別物
- **advisor に合否判定をさせない**: advisor は tool を呼べず (何も実検証できない)、executor の語りを含む全 transcript を受け取る (anchoring が仕組み上不可避) — Phase 2 記載の doc 確認済み事実より、合否・品質認定は構造的に甘くなる。質問は「どこで失敗するか」「リスクを順位付けせよ」「選択肢 A/B/C のどれを捨てるか」の反証・順位付け・比較形式に限定し、「これで良いか」という承認を誘う形式では聞かない。合否判定は verifier delegation パターン (`../verifier-pattern/SKILL.md`) に分離する
- **手動 Agent 経路では同一 advisor agent を継続利用する**: 相談のたびに新規 agent を起動せず、同一 agent に追送 (SendMessage 等) して文脈を蓄積させる。都度相談では advisor に文脈蓄積がないという弱点への直接の対処であり、cache 効率も上がる
- **Fable 不可時は opus で代用する**

---

## Phase 4: 計測プロトコル

「効くはず」で常用に昇格させない。着手前後で消費トークン (input/output/cache) と wall-clock を記録し、同型タスクの「executor 単独 vs executor+advisor」を比較して採否 (常用・特定条件のみ・不採用) を判断する。

---

## 注意事項

- **fable を advisor に決め打ちしない**: 組織アクセスの有無で挙動が変わる (Phase 2 参照)。既定は opus とし、fable は組織アクセスが確認できた場合のみの選択肢とする
- **Agent ツール経由の回避策は 1 セッションの実測に基づく**: 別セッション・別アカウントで同じ挙動を保証しない。技術情報として発信・実装に使う際は都度再確認する
- **ビルトイン advisor tool と Agent subagent を混同しない**: 前者はサーバーサイドで全文脈を自動転送する。後者は呼び出し側がプロンプトで文脈を渡す。ドキュメント化・投稿発信時はどちらの機構の話かを明記する
- **固定チェックポイント方式には構造的弱点がある** (Fable subagent によるパターン批評、2026-07-12 実測): (1) 相談すべき判断ポイントを事前に正しく特定できる前提に立つため、チェックポイント間で生じる unknown unknowns に executor が「advisor に聞くべき」と気づけない、(2) 都度相談では advisor 側に文脈の蓄積がなく助言品質が落ちる。ビルトイン advisor tool は全 transcript 転送により (2) を緩和するが、Agent ツール手動経路 (Phase 2 の代替経路) では緩和されない (弱点 (2) の緩和策: Phase 3 の同一 agent 継続利用)
- **advisor を独立・敵対的レビューの担当に入れない**: advisor は相談を通じて成果物の作成に関与しており、レビュアーの独立性要件 (成果物の作成に関与していないこと) を満たさない。また advisor への相談実績があっても、完了報告前の独立検証は免除されない (相談はレビューの代替にならない)

---

## 参照

- `../verifier-pattern/SKILL.md` (兄弟パターン = 判断が検証時点に集中するタスク形状)
- 公式ドキュメント: `code.claude.com/docs/en/advisor` (advisorModel 仕様・組織アクセス要件)、`platform.claude.com/docs/en/agents-and-tools/tool-use/advisor-tool.md` (API レベルの advisor tool 仕様)
- 一次資料: Fable コスト効率ハーネスに関する記事 (2026-07-12 参照。Parameter Golf / BrowseComp200 / BrowseComp フルの実測を含む)
