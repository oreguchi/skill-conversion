# Phase 1 Sample — writing-skills-ja

> 高忠実度プロファイル下での Phase 1 翻訳サンプル。SKILL.md 冒頭から Iron Law / Common Rationalizations セクションまでの代表箇所を抜粋翻訳して、Tier 1 参照のみで運用される姿を示す。

## サンプル A — SKILL.md 冒頭（Overview セクション）

### 原文（EN）

```markdown
---
name: writing-skills
description: Use when creating new skills, editing existing skills, or verifying skills work before deployment
---

# Writing Skills

## Overview

**Writing skills IS Test-Driven Development applied to process documentation.**

**Personal skills live in agent-specific directories (`~/.claude/skills` for Claude Code, `~/.agents/skills/` for Codex)**

You write test cases (pressure scenarios with subagents), watch them fail (baseline behavior), write the skill (documentation), watch tests pass (agents comply), and refactor (close loopholes).

**Core principle:** If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.
```

### 翻訳（JA）

```markdown
---
name: writing-skills-ja
description: Use when creating new skills, editing existing skills, or verifying skills work before deployment
---

# スキル作成（Writing Skills）

## 概要

**スキル作成とは、プロセス文書に適用したテスト駆動開発（TDD）そのものである。**

**個人スキルはエージェント固有ディレクトリに配置する（Claude Code は `~/.claude/skills`、Codex は `~/.agents/skills/`）。**

テストケース（サブエージェントに対する圧力シナリオ）を書き、失敗を観察し（ベースライン挙動）、スキル本文（文書）を書き、テスト合格を観察し（エージェントが従う）、リファクタする（抜け道を塞ぐ）。

**中核原則：** スキルなしでエージェントが失敗するさまを観察していなければ、そのスキルが正しいことを教えているかどうか分からない。
```

### Tier 1 適用ポイント

| Tier 1 ルール | 適用箇所 |
|---|---|
| 文体は **である調** で統一 | "テストケースを書き、失敗を観察し" |
| description フィールド内 "Use when ..." は英語のまま温存 | フロントマターを EN のまま保持 |
| `~/.claude/skills` 等のパスは無変換 | 識別子は技術的事実 |
| frontmatter `name` のみ `-ja` サフィックス付き | name は識別子だが target skill 名定義に従う |
| 半角英数と日本語の間にスペースなし | "TDD)そのものである" / "Codex は" |

### Tier 2 / Tier 3 が **登場しない** ことの確認

- 現代日本語技術文書での「ですます調 vs である調」の使い分けトレンド（Tier 2 候補） → **embed しない**
- JIS X 0208 範囲外の半角カナ等、ロケール拡張ルール（Tier 3 候補） → **embed しない**

このサンプルは Tier 1 のみで完結している。スコープに踏み込んでいない。

---

## サンプル B — Iron Law セクション

### 原文（EN）

```markdown
## The Iron Law (Same as TDD)

```
NO SKILL WITHOUT A FAILING TEST FIRST
```

This applies to NEW skills AND EDITS to existing skills.

Write skill before testing? Delete it. Start over.
Edit skill without testing? Same violation.

**No exceptions:**
- Not for "simple additions"
- Not for "just adding a section"
- Not for "documentation updates"
- Don't keep untested changes as "reference"
- Don't "adapt" while running tests
- Delete means delete
```

### 翻訳（JA）

```markdown
## 鉄則（TDD と同じ）

```
失敗するテストなしにスキルを書くな
```

これは **新規スキル** にも、**既存スキルの編集** にも等しく適用される。

テストより先にスキルを書いた？ 削除せよ。最初からやり直せ。
テストなしにスキルを編集した？ 同じ違反である。

**例外なし：**
- 「ちょっとした追加」も例外ではない
- 「節を一つ足すだけ」も例外ではない
- 「ドキュメント更新だけ」も例外ではない
- テストなしの変更を「参考用」として残すな
- テスト実行中に「適合させる」な
- 削除は削除である
```

### Tier 1 適用ポイント（再確認）

- "Delete means delete" → 「削除は削除である」と直訳調。原典の言葉遊び（同語反復による強調）を温存。
- 箇条書きの語感を「〜ない」「〜な」（命令禁止形）で統一 → である調と整合。
- 二重引用符 `"..."` は「...」に変換（日本語標準）。

### 高忠実度プロファイルの判定（9-cell 適用）

このサンプル翻訳中、追加提案は **発生しなかった**。仮に発生したと仮定したケース：

#### 仮想ケース 1: 「日本の組織文化での適用例を 1 段落追加してはどうか」

- Category: C (Differentiating)
- Impact: heavy（新規段落）
- 9-cell 判定（high-fidelity 列）: **rejected** （C × heavy / high-fidelity）
- 結果: Profile-rejected proposals に記録、register には入らない、target にも採用されない

#### 仮想ケース 2: 「『削除は削除である』に注釈で『git で空コミットも禁止』と補足」

- Category: A (Fidelity-preserving)（原文意図の保全のための明確化補完）
- Impact: minor
- 9-cell 判定: **auto-approve**
- 結果: register に追加可

実際にはこの仮想ケースはどちらも発生しなかった（Locale 変換のみのため）。

---

## サンプルから読み取れる 9-cell の挙動

| Profile | このサンプル中の Category 別 register 件数 |
|---|---|
| high-fidelity | A: 0 / B: 0 / C: 0（C は profile-rejected プールに 0 件） |

→ register 空のまま Phase 1 終了。drift-report は「Phase 1 approved additions: なし、Phase 4 SKIPPED」と記録される。

これは **高忠実度プロファイルが意図どおり機能している証拠**。
