# Phase 1 Sample — test-driven-development-ja

> Balanced プロファイル下での Phase 1 翻訳サンプル。Tier 1 + Tier 2 を参照範囲として運用される姿、9-cell 表で 1 件の追加が auto-approve される様子を示す。

## サンプル A — SKILL.md 冒頭 + Iron Law

### 原文（EN）

```markdown
---
name: test-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Test-Driven Development (TDD)

## Overview

Write the test first. Watch it fail. Write minimal code to pass.

**Core principle:** If you didn't watch the test fail, you don't know if it tests the right thing.

**Violating the letter of the rules is violating the spirit of the rules.**
```

### 翻訳（JA）

```markdown
---
name: test-driven-development-ja
description: Use when implementing any feature or bugfix, before writing implementation code
---

# テスト駆動開発（TDD）

## 概要

まずテストを書く。失敗するさまを観察する。合格させる最小のコードを書く。

**中核原則：** テストが失敗するさまを観察していなければ、そのテストが正しいことを検証しているかどうか分からない。

**ルールの字義に反することは、ルールの精神に反することである。**
```

### Tier 1 / Tier 2 の適用ポイント

| Tier | ルール | 適用箇所 |
|---|---|---|
| Tier 1 | である調 | "失敗するさまを観察する" |
| Tier 1 | description は EN 温存 | フロントマター無変換 |
| Tier 1 | 半角英数と JA の間にスペースなし | "TDD)" / "TDD は" |
| Tier 2 | JA 慣用化したツール表現 | （まだ tool 呼び出しが出ていないので未適用） |

---

## サンプル B — Common Rationalizations 表

### 原文（EN）

```markdown
| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests passing immediately prove nothing. |
| "Tests after achieve same goals" | Tests-after = "what does this do?" Tests-first = "what should this do?" |
```

### 翻訳（JA）

```markdown
| 言い訳 | 現実 |
|--------|---------|
| 「単純すぎてテスト不要」 | 単純なコードも壊れる。テストは 30 秒で書ける。 |
| 「あとでテストを書く」 | すぐ通るテストは何も証明しない。 |
| 「あとからテストを書いても同じ」 | 後付けテストは「これは何をするか？」を問う。先に書くテストは「これは何をすべきか？」を問う。 |
```

### Tier 2 適用ポイント — Phase 1 中の追加提案（auto-approve）

このサンプル翻訳中、以下の 1 件の追加が **balanced × B × minor = auto-approve** で register に登場する：

#### 追加 1: pytest / npm test の JA 圏慣用句（minor）

- Location: SKILL.md §Verify RED 直前
- Summary: 「`pytest` / `npm test` 等のテスト実行コマンドを JA 文脈で自然に呼び出す」一行注記
- Category: B (Environment-essential) — JA 圏 TDD ユーザは `pytest -v` 等の慣用に親しんでおり、Locale-JA Tier 2 に該当
- Impact: **minor**（既存文の clarification、新規節ではない）
- 9-cell 判定: balanced × B × minor = **auto-approve**
- 提案文: 「（本スキルは JS/TS 例を用いるが、Python であれば `pytest path/to/test_x.py`、Go であれば `go test ./...` のように、各言語の標準テストコマンドに置き換えて読む）」

→ register に Phase 1 由来として 1 行追加。drift-report で minor として集計。

---

## 9-cell の挙動の例示（balanced 列）

| 仮想ケース | Category | Impact | balanced 列の判定 | 結果 |
|---|---|---|---|---|
| 上記 pytest 注記 | B | minor | auto-approve | **register に追加** |
| Phase 4 の simulated 提案 (00-master-plan.md §Phase 4) | B | moderate | user approval required | ユーザ承認後 register 追加 |
| 仮想: 「日本企業の TDD 導入事例」段落 | C | heavy | rejected | profile-rejected 列に記録、target 不採用 |
| 仮想: 「Mocking ライブラリ Sinon の高度パターン」 | C | moderate | rejected | profile-rejected 列に記録 |
| 仮想: "Iron Law" 表現に「日本語版鉄則」併記 | A | minor | auto-approve | register に追加 |

balanced 列は **B/C × moderate/heavy で態度が変わる** のが特徴で、本 dogfooding ではこの帯がきちんと境界として機能することを観察できた。
