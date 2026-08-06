---
name: smart-commit
description: git の変更を精査・分割・逐次 commit し、push して PR を作成するまでの一連のワークフロー。秘匿情報の混入防止と人間可読な commit 粒度を保証する。merge はユーザー判断のため実行しない。
---

# Skill: Smart Commit

リポジトリ内の変更を安全かつ意味のある単位で commit し、リモートに push するワークフロー。

## 起動条件

- ユーザーが「commit して」「push して」「変更をまとめて」等の発言をした場合
- 複数ファイルの変更が溜まっており、整理して commit する必要がある場合

---

## Phase 1: 状況把握

以下のコマンドを並列実行し、変更の全体像を把握する。

| コマンド | 目的 |
|---------|------|
| `git status` | 変更・未追跡ファイルの一覧 |
| `git diff --stat` | 変更量の概要 |
| `git diff` | 変更内容の詳細 |
| `git log -n 5 --oneline` | 直近の commit メッセージスタイル確認 |

未追跡ファイルは `Read` で内容を確認する。

---

## Phase 2: 秘匿情報の精査

`git status` に表示される全ファイル（modified + untracked）に対して以下を照合し、commit 対象から除外すべきファイルを特定する。`.gitignore` で既に除外済みのファイルは `git status` に表示されないため対象外とする。

### 除外判定基準

| チェック対象 | 判定方法 | フォールバック |
|------------|---------|-------------|
| `.gitignore` に該当するか | パターンマッチ | ファイルが存在しない場合はスキップ |
| `.claudeignore` に該当するか | パターンマッチ | ファイルが存在しない場合はスキップ |
| ファイル内容に秘匿情報が含まれるか | AWS キー (`AKIA...`)、秘密鍵、トークン、パスワード等のパターン検索 | — |
| `settings.local.json` 等の個人設定か | ファイル名パターン | — |
| `.env` / `.pem` / `.key` 等の秘密ファイルか | 拡張子チェック | — |

### 判定結果の提示（三値判定）

判定は二値ではなく**三値**で行う。「秘匿情報がコミットすべきファイルの内部に埋め込まれている」ケース（例: ソースコードに直書きされた AWS キー）をファイル単位の除外で取りこぼさないため。

| 判定 | 意味 | 対応 |
|------|------|------|
| **commit OK** | 秘匿情報なし | そのまま commit 対象 |
| **除外** | ファイル自体が秘密 (`.env`/`.pem`/`settings.local.json` 等) | commit から外し、`.gitignore` 追加を提案 |
| **要修正** | コミットすべきファイル内部に秘匿情報が混入 | **キーが残る間は `git add` しない**。該当値を環境変数 / Secrets Manager 参照 / IAM ロールに置換 → Phase 2 を再実行しクリーンを確認 → その後 commit |

全ファイルの判定結果をテーブルで提示する。

```
| ファイル | 判定 | 理由 |
|---------|------|------|
| src/app.ts | commit OK | ソースコード、秘匿情報なし |
| .env.local | 除外 | 環境変数ファイル |
| lambda/handler.py | 要修正 | 正当なロジック変更だが AWS キー直書き。除去後に commit |
```

判定に迷うファイルがあれば AskUserQuestion で確認する。「要修正」がある場合は、除去方針を AskUserQuestion で提示し承認を得てから着手する。

### 除外ファイルの事後アクション

除外判定されたファイルが `.gitignore` に未登録の場合、恒久対策として `.gitignore` への追加を提案する。`.gitignore` 自体が存在しない場合は作成を提案する。提案は AskUserQuestion でユーザーに確認し、承認された場合は `.gitignore` の更新を最初の commit に含める。

---

## Phase 3: commit 分割計画

以下の原則に従い、commit を分割する。

### 分割原則

1. **意味的凝集性**: 1 commit = 1 つの論理的変更。関連するファイルをグルーピングする
2. **人間可読性**: commit メッセージを読むだけで変更内容が理解できる粒度
3. **依存順序**: 後続の commit が先行の commit に依存する場合、依存元を先に commit する
4. **設定 → ルール → ロジック → ドキュメント**: インフラ的変更から適用的変更の順に commit する

### 計画の提示

分割計画をテーブルで提示し、ユーザーの承認を得る。

```
| # | 対象ファイル | commit 意図 |
|---|------------|------------|
| 1 | .gitignore | 除外パターンの追加 |
| 2 | src/auth.ts, src/auth.test.ts | 認証ロジックのリファクタ |
```

---

## Phase 4: Issue 作成 + Feature ブランチ

commit の前にブランチを切る。

### 4-1. Issue 作成

Phase 3 の分割計画全体を1つの issue として作成する。

1. 変更内容から issue タイトル・本文を推定し AskUserQuestion で確認
2. `gh issue create --title "..." --body "..."`
3. 出力から issue 番号を取得

**スキップ条件**: ユーザーが既存 issue 番号を指定している場合（`#数字`）

### 4-2. Feature ブランチ作成

1. ブランチ名を生成: `feature/{issue番号}/{slug}`
   - slug: 英語 2〜4 単語のケバブケース、最大30文字
   - 日本語タイトルは内容を意訳（ローマ字禁止）
2. `base_branch` は `gh repo view --json defaultBranchRef -q '.defaultBranchRef.name'` で検出する（検出不可なら `main` を既定）。
3. ブランチ作成方式は、**複数セッション・複数エージェントが同一 working tree を共有しうるか**で分岐する:
   - **並行作業がありうるリポ (worktree 推奨)**: メイン working tree を直接触らず、リポ配下に worktree を切って作業する
     ```bash
     git -C {main_repo} worktree add -b feature/{issue番号}/{slug} .worktrees/{slug} origin/{base_branch}
     # 以降 git -C {main_repo}/.worktrees/{slug} で作業
     ```
   - **単独作業のリポ (通常のブランチ切替でよい)**:
     ```bash
     git checkout {base_branch} && git pull origin {base_branch}
     git checkout -b feature/{issue番号}/{slug}
     ```

---

## Phase 5: 逐次 commit

計画に従い、1 commit ずつ実行する。

### commit ルール

- **`git add -A` / `git add .` は使用禁止**。必ず個別ファイル名を指定する
- commit メッセージは既存の履歴スタイルに準拠する（履歴が 3 件未満の場合は本スキルのデフォルトスタイルを適用）
- メッセージ形式: 英語、lowercase start、imperative mood
- Co-Authored-By を付与する
- HEREDOC 形式で commit メッセージを渡す

### commit メッセージの構成

```
<type>: <short summary in imperative mood>

<optional body: what and why, not how>

Co-Authored-By: {ハーネスが注入する既定 trailer (現行セッションのモデル名) に従う}
```

- type: `add` | `update` | `fix` | `remove` | `refactor` | `restructure`
- scope は使用しない（`<type>(<scope>):` 形式は不可）
- type の選択基準: 新規ファイル追加 → `add`、既存ファイル変更 → `update`、バグ修正 → `fix`、削除 → `remove`、構造変更 → `refactor` | `restructure`

---

## Phase 6: Push + PR 作成

> **worktree はリポ配下に作る (cwd 内・エージェントが完結できる)**:
> worktree をリポ配下に置けば cwd 内なので、`git -C {main_repo}/.worktrees/{slug}` の commit/push/status/gh を直接実行できる (add〜push〜remove まで完結)。
> **リポ外に worktree を作らない**: サンドボックス環境によっては cwd 外への書き込みが拒否され、`git worktree add` が失敗したり `git -C <外部worktree>` の read/write が拒否されたりする。この場合 commit/push はユーザーの手動実行に頼ることになり、長いコマンドが折り返し・空白欠落で壊れることもある。配下 worktree に統一することでこの問題自体を回避する。
> - 複数の GitHub アカウントを使い分けている場合は、`GH_CONFIG_DIR` 環境変数で認証設定を明示的に切り替える
> - PR 本文など長文は**事前に Write でファイル作成** → `gh pr create --body-file` で渡す

### 6-1. Push

```bash
git push -u origin {branch}
```

### 6-2. PR 作成

1. PR タイトル: commit が1つなら commit メッセージベース、複数なら issue タイトルベース
2. PR 本文:

```markdown
## Summary

- {変更内容のバレット 1〜3行}

## Test plan

- [ ] {検証項目}

Closes #{issue番号}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

3. `gh pr create --base {base_branch} --title "..." --body "..." --assignee {assignee}`
4. レビュワー候補が定まっている場合は AskUserQuestion で選択肢を提示する（候補が無い・個人開発で不要なら省略してよい）
5. 選択されたレビュワーを `gh pr edit {pr番号} --add-reviewer {selected}` で追加する

### 6-3. 結果サマリの提示

```
| # | commit hash | 内容 |
|---|------------|------|
| 1 | abc1234 | .gitignore — 除外パターン追加 |
| 2 | def5678 | 認証ロジックのリファクタ |

📌 Issue: #{issue番号} {URL}
📎 PR: #{pr番号} {URL}
🌿 Branch: feature/{issue番号}/{slug}
```

**merge はユーザー判断**。スキルからは実行しない。

---

## Phase 7: 最終確認

以下を並列実行し、状態を確認する。

| コマンド | 確認内容 | フォールバック |
|---------|---------|-------------|
| `git status` | working tree clean であること | — |
| `git log --oneline origin/{branch}..HEAD` | ローカルに未 push の commit がないこと | — |
| `gh pr view --json url,state` | PR が Open 状態で正しく作成されていること | — |

---

## 注意事項

- Phase 2 の精査は省略しない。秘匿情報の混入は取り返しがつかない
- commit 分割計画はユーザーの承認を得てから実行する
- push 前に必ず最終確認を行う
- **merge はスキルから実行しない**。PR を作成するところまでがスキルの責務
- エラー発生時は原因を調査し、AskUserQuestion で対処方針を確認する
- 複数の GitHub アカウントを使い分けている場合は、gh コマンドに正しい認証設定 (`GH_CONFIG_DIR` 等) を明示的に付与する
- **並行セッションによる git 状態変化に注意**: 作業中に別の Claude セッションやスケジュールタスクが同一リポの git 状態 (ブランチ・commit・working tree) を変更することがある。各 Phase の前後で想定どおりのブランチ・HEAD にいるかを確認し、`checkout`/`reset` 等の破壊的操作の前には必ず `git status` / `git branch -vv` / `git log` で現状を精査する。意図しない状態を発見しても、まず原因を調査し、ブランチ削除や強制リセットなどで安易に「片付けない」こと (別セッションの作業を破壊しうる)。
