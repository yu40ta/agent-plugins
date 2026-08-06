---
name: verifier-pattern
description: 安価な (または同格の) executor が成果物を作り、上位モデル (verifier) が完了判定・品質判定の時点でのみ介入して検査する「verifier delegation」パターンの opt-in 判断支援 skill。成立条件ゲート・verifier モデル選定・検証プロセスへの接続を判定する。手動起動のみで自動発火はしない。
---

# Skill: Verifier Pattern

事前計画で fan-out するパターン・実行中の随時相談パターンと並ぶ 3 つ目のパターン — **安価な executor が成果物を作り、上位モデルは完了判定・品質判定の時点でのみ介入する**パターン (verifier delegation) の opt-in 判断支援。判断が「作業の後端 (検証時点)」に集中するタスク形状に適合する。

**既存資産との棲み分け (本 skill はプロセスを再実装しない)**:

| 対象 | 棲み分け |
|---|---|
| 独立・敵対的レビューのプロセスそのもの | レンズ分割・独立 dispatch・裁定・反復といった検証プロセス自体は自環境の該当 skill (例: `../adversarial-review-loop/SKILL.md`) が SSOT。本 skill はここへの導線のみ持つ |
| 実行中の随時相談パターン | 判断が実行中に分散するタスク形状に対応。verifier のモデルアクセス制約 (fable 等) の詳細もそちら (`../advisor-pattern/SKILL.md`) が正本 |
| レビューの要否・段数の判断 | 検証プロセス = 上記の独立レビュー skill / 誰に検証させるか (tier 経済性) = 本 skill、の役割分担とする |

---

## 起動条件

- 手動起動。「executor + verifier 構成にしたい」「完了時だけ上位モデルにチェックさせたい」等、明示的にこのパターンを要求されたとき
- 自動発火はしない

---

## なぜ opt-in か

一次資料 (Fable コスト効率ハーネスの実測記事、2026-07-12 参照) は verifier 構成を advisor・orchestrator と並ぶ 3 パターン目として挙げるが、**verifier 単独の第一者ベンチマークは記事中に無い**。advisor には Parameter Golf、orchestrator には BrowseComp の実測が添えられているのと対照的に、3 パターン中で定量的実証が最も弱い。第三者の伝聞 (Mitchell Hashimoto の X 投稿、2026-07-03、記事内で引用) は Fable (xhigh) を planner/judge、GPT 5.5 (xhigh) を coder とする構成で「planning+judge のコストがフル round trip 比で大幅に低い」というものだが、これは統制されたベンチマークではなく個人の体験談であり、verifier 単独ではなく orchestrator+verifier の複合構成の効果測定である。実証が弱いからこそ Phase 4 の計測プロトコルを必須とし、opt-in に留める。

---

## Phase 1: 成立条件ゲート

着手する**前**に、以下 4 条件をすべて満たすか判定する。**1 つでも欠けたら不採用** (直接実行を選ぶ)。

| # | 条件 | 欠けた場合の代替 |
|---|---|---|
| 1 | **判断が検証時点に集中している** (計画・実行中ではなく完了確認の 1 点に判断が集中する) | 実行中に判断が分散 → 随時相談パターン (`../advisor-pattern/SKILL.md`) / 事前計画に判断が集中 → 上位モデルが直接計画する構成の方が適する |
| 2 | **検証可能な合否基準がある** (測定可能な終了状態・証明方法の明示・守るべき制約、の 3 要素で書けること)。基準は dispatch **前**に固定し (事前登録)、検証後に基準を再解釈しない | 基準が主観・未定義 → verifier の判定が印象論になり、上位モデル分のコストが判定品質に転化しない |
| 3 | **生成コスト ≫ 検証コストの非対称がある** (verifier は成果物+文脈を読んで判定するだけで済む) | 検証に生成と同等の再作業が要る → 最初から上位モデルで直接実行した方が早い |
| 4 | **単独実行との総額比較が済んでいる** (boundary duplication は検証対象の文脈量に比例して増える) | 比較せず着手 → 規模依存で結果が逆転しうる (advisor delegation パターンの BrowseComp200/フル対比と同型のリスク) |

---

## Phase 2: verifier モデル選定

**既定 opus**。独立レビュー skill 側 (検証 subagent の tier は opus・fable 指定禁止) の運用と整合させる既定である。

fable を verifier に使いたい場合:

- fable は原則指定禁止。ユーザーがセッション起動時に明示選択する場合のみ使われ、本 skill から fable を既定にすることはしない
- **常用化には運用ポリシー側の例外化 (ユーザーの governance 判断) が必要**。本 skill の権限では解禁できない
- fable のアクセス要件・Agent ツール経由の代替起動経路の実測は `../advisor-pattern/SKILL.md` Phase 2 を参照 (重複記載しない)

**組み込みの軽量 evaluator と混同しないこと**: 公式ドキュメント (`code.claude.com/docs/en/goal`、2026-07-12 取得) によれば、目標駆動の完了判定機能に付属する evaluator は「configured small fast model」(既定は Haiku 相当) であり、上位モデル verifier ではない。evaluator は tool を呼べず会話に表出した内容だけで判定する、session-scoped の prompt-based Stop hook のラッパーである。上位モデル verifier を実現する経路は次の 2 つ:

1. 独立レビュー skill (`../adversarial-review-loop/SKILL.md`) の独立 subagent dispatch (opus 指定)
2. カスタムの prompt-based Stop hook (evaluator モデルを自前で opus に固定する)

---

## Phase 3: 実行規範

- **検証指示は 3 要素で書く**: 測定可能な終了状態・証明方法の明示・守るべき制約。verifier への指示にもそのまま一般化できる
- **検証プロセスは独立レビュー skill に従う**: レンズ分割・独立 dispatch・裁定表・反復上限は再実装せず、そのまま呼ぶ (`../adversarial-review-loop/SKILL.md`)
- **verifier の判定も ground truth にしない**: 裁定は親が行い実データで裏取りする
- **立証責任を反転する**: verifier の既定判定は FAIL。PASS には事前登録した基準ごとに「基準 → 証拠の引用 → 二値判定」の判定表を義務付ける。証拠が足りない場合は「判定不能」を正規の出力とする (無理に白黒つけさせると合格に流れる)。n 点満点等の点数尺度は使わない (LLM の採点は中央に圧縮され甘さの温床になる)
- **verifier には tool を持たせ、自分で再実行させる**: テスト・コマンドの実行結果を verifier 自身が取得する。transcript 越しの判定は認めない (組み込みの軽量 evaluator が tool を呼べないのと対照的に、こちらは呼べる構成にする)
- **盲検化を徹底する**: verifier には成果物 + 事前登録基準のみ渡す。作成経緯・executor の自己報告・期待する結論は渡さない
- **Fable 不可時は opus で代用する**

---

## Phase 4: 計測プロトコル

着手前後で消費トークンと wall-clock を記録する計測プロトコル (`../advisor-pattern/SKILL.md` Phase 4 と同型) に準拠する。第三者の伝聞しか実証がない以上、自環境での実測なしに常用化を決めない。同型タスクの「executor 単独 vs executor+verifier」を比較し、採否 (常用・特定条件のみ・不採用) を更新する。

---

## 注意事項

- **Mitchell Hashimoto の X 投稿を期待値にしない**: 統制されたベンチマークではなく個人の体験談であり、しかも orchestrator+verifier の複合構成であり verifier 単独の効果測定でもない。自環境のコスト削減率として引用しない
- **組み込みの軽量 evaluator と上位モデル verifier を混同しない**: 既定は Haiku 相当。上位モデル検証が要る場合は Phase 2 の 2 経路 (独立 subagent dispatch・カスタム Stop hook) を使う
- **fable 解禁は本 skill の権限外**: 例外化はユーザーの governance 判断であり、本 skill が単独で常用化を決めない
- **スコープの再確認**: 本 skill は「誰に (どの tier に) 検証させるか・それが割に合うか」の判断のみを扱う。レンズ分割・独立 dispatch・裁定・反復といった検証プロセスそのものは独立レビュー skill が SSOT であり、本 skill から重複記載しない

---

## 参照

- `../adversarial-review-loop/SKILL.md` (検証プロセスそのものの SSOT。本 skill はここへの導線のみ)
- `../advisor-pattern/SKILL.md` (兄弟パターン = 判断が実行中に分散するタスク形状。fable アクセス要件・Agent 経由代替経路の詳細はこちらが正本)
- 公式ドキュメント: `code.claude.com/docs/en/goal` (目標駆動の完了判定機能の evaluator 仕様・条件記述の 3 要素、2026-07-12 取得)
- 一次資料: Fable コスト効率ハーネスに関する記事 (2026-07-12 参照。Mitchell Hashimoto の X 投稿の伝聞引用を含む)
