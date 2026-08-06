---
name: repo-structure-designer
description: Claude 活用を前提とした repo のデフォルト dir 構成を設計する、または既存 repo の構成を最適化するスキル。L0核+段階拡張 (L0→L3) モデルに基づき、どの repo でもブレない構成判断を提供する。新規 repo 初期構成と既存 repo 最適化の2モード。手動起動 (/repo-structure-designer)。
---

# Skill: Repo Structure Designer

## 起動条件

- 手動起動 (`/repo-structure-designer`): ユーザーが「新しい repo を作る」「repo の dir 構成を決めたい」「この repo の構成を見直して/最適化して」等と依頼した場合に手動で適用する。自動発火はしない。

---

## 概要

Claude Code を活用する前提で、repo のディレクトリ構成を**設計**または**最適化**する判断基準スキル。複数の実 repo を調査し、Claude 活用の成熟度を「足し上げる層 (L0→L3)」として定式化した。

特定の repo 種別 (IaC/app/docs) に縛られない**汎用1本**。核となる重心は **docs(SSOT) と .claude(自動化) の両輪**。

## 中核モデル: L0核 + 段階拡張

**L0 から始め、必要になった層だけ足す。最初から L3 を作らない (YAGNI)。**

| 層 | 加わる要素 | 適用条件 |
|---|---|---|
| **L0 核** (必須) | `README.md` / エージェント向け指示ファイル / `.gitignore` / **秘匿境界セット** (`.secretlintrc.json` / `.claudeignore`※) / `.claude/settings.json`(+`settings.local.json`) / `docs/` | 全 repo 例外なし |
| **L1 設計駆動** | `docs/requirements.md` (SSOT) / `docs/specs/` / `docs/plans/` | 実装を伴う / 仕様が育つ repo |
| **L2 自動化** | `.claude/skills/` / `.claude/agents/` / `.claude/commands/` | 反復タスク・専門ロールがある repo |
| **L3 運用** | ADR ディレクトリ (意思決定記録) / `.claude/hooks/` / `refs/` | 長期運用・意思決定追跡が要る repo |

### 各 dir の役割

- `README.md` — 人間向けの入口。技術スタック・構成図・起動手順。
- **エージェント向け指示ファイル** — AI エージェント向けの指示書 (下記「指示書ファイルの選択」)。
- **秘匿境界セット** — 秘匿物を扱う repo では物理的担保が L0 必須:
  - `.gitignore` — git 追跡からの除外 (全 repo)。
  - `.secretlintrc.json` — コミット前/後の秘匿情報検出ルール。
  - `.claudeignore`※ — Claude Code の Read をプラットフォームレベルで遮断する仕組み (対応状況はツールのバージョンにより異なるため導入前に確認する)。**秘匿ファイル・巨大ファイル・`.env`/鍵類を抱える repo では必須**。秘匿物を一切持たない小 repo では任意 (※印の条件付き)。
- `.claude/settings.json` (共有) / `settings.local.json` (gitignore) — 権限・プラグイン設定。
- `docs/` — ドキュメント一元化。`requirements.md`(SSOT) → `specs/`(設計) → `plans/`(実装計画)。
- `.claude/skills|agents|commands/` — 反復タスク・専門ロール・定型コマンドの自動化資産。
- ADR ディレクトリ — 意思決定の記録 (`${category}/${title}.md` 等の形式)。
- `.claude/hooks/` — PreToolUse/PostToolUse のガード。
- `refs/` — 参照資料・スクリプト仕様。

## 指示書ファイルの選択 (L0 内の分岐)

- **原則、Claude Code が起動時に自動ロードする既定の指示ファイル名を使う**: 個人 repo・Claude Code 専業 repo はこれ。
- **`AGENTS.md` を主にするケース**: 複数 AI ツール (Copilot/Cursor 等) での利用が想定される repo、またはすでに `AGENTS.md` がある repo。
  - ⚠️ Claude Code は標準では `AGENTS.md` を自動ロードしない。→ 既定の指示ファイルを `@AGENTS.md` 参照のみの**薄いスタブ**にして併置する。

## 2つの動作モード

### モード A: 新規 repo の初期構成

1. **判断フロー**で必要な層を決める (下記)。
2. L0 から構成を提案。指示書ファイルを選択。
3. **worktree 強制の要否**を判断: 複数 Claude セッションで feature 作業が競合しうる repo なら、feature ブランチ作業を worktree に固定する PreToolUse hook の導入を提案する。
4. ユーザー承認後、必要なら scaffold (空 dir + 雛形ファイル) を作成。

### モード B: 既存 repo の構成最適化

1. 現状 dir を `ls`/`find` で把握し、**L0〜L3 にマッピング**する。
2. **不足/過剰を diff として提示** (例: 「L0 の `.secretlintrc.json` が無い」「小 repo に L3 の hooks が過剰」)。
3. **repo 種別を判定** (`git remote get-url origin`):
   - **個人リポ** (あなたの GitHub アカウント配下): 編集提案を直接実行してよい。
   - **チーム共有リポ** (所有者が自分でない): **提案のみ or PR 化**。各操作前に確認を取る。
   - worktree 強制リポ (専用の PreToolUse hook がある) は worktree で作業。
4. **破壊的な dir 移動は勝手に行わない**。既存の不統一は強制移行せず、新規作業から適用を基本とする。

## 判断フロー (この repo はどの層まで要るか)

```
まず必ず L0 を置く            → 核 (全 repo 共通)
実装を伴う / 仕様が育つ?       → Yes なら +L1 (requirements→specs→plans)
反復タスク・専門ロールがある?  → Yes なら +L2 (skills/agents/commands)
長期運用・意思決定の追跡が要る? → Yes なら +L3 (ADR/hooks/refs)
```

## 横断原則 (dir 配置を貫く規範)

1. **stock/flow 分離** — 恒久ナレッジ (ルール・テンプレ・ノウハウ) と時点情報 (ワークスペース・ログ・一時出力) を分ける。flow は gitignore 寄り。
2. **SSOT 明示** — `docs/requirements.md` を「正」とし、他は派生物。仕様変更はまず SSOT から。
3. **秘匿境界** — `.env` / `*.pem` / `credentials.json` 等は**絶対に git 管理外**。`settings.local.json` は gitignore。秘匿は「含めない」運用 + `.secretlintrc.json` で機械検査 + `.claudeignore` で Read 自体を遮断する多層防御。
   - **`.gitignore` は初期設計で時間をかける価値がある**: 一度追跡してしまったファイルの追跡解除コストは高い (履歴に残る・全クローンで再追従が要る)。リポ初期構成の段階で除外対象をしっかり設計しておく。
4. **docs 一元化** — ドキュメントは `docs/` に集約 (分散させない)。
5. **意思決定追跡** — 後から参照すべき決定は ADR 化する。

## アンチパターン

- ❌ 小 repo に L3 を過剰 scaffold (ADR・hooks を最初から作る)。→ 必要になってから足す。
- ❌ `tmp/` の散乱。→ 一時物は性質で振り分け、用途後に削除。
- ❌ 秘匿情報の混入 (.env をコミット、トークン直書き)。
- ❌ docs の分散 (`doc/` と `docs/` 併存、README にだけ仕様が埋まる)。
- ❌ 指示書ファイルの取り違え (複数ツール repo に Claude Code 専用の指示ファイルだけ置き、他ツールが読めない)。

## スコープ外 (YAGNI)

- repo 種別 (IaC/app/docs) ごとの専用テンプレは作らない (汎用1本の方針)。
- 本 skill は**判断基準の文書**。実 dir の自動生成 (scaffold) はモード A でユーザー承認後にのみ行う。

## 関連

- 変更着手前のブランチ/worktree 用意、意思決定記録の作成、成果物の整理といった周辺タスクを別途 skill 化している場合は、それらと組み合わせて使ってよい。
