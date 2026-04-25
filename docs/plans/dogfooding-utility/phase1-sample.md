# Phase 1 Sample — systematic-debugging-ja

> high-utility プロファイル下での Phase 1 翻訳サンプル。Tier 1 + Tier 2 + Tier 3 を全て参照範囲として運用される姿、9-cell で Tier 3 / Category C 由来の追加が user approval 経由で採用される様子を示す。

## サンプル A — SKILL.md 冒頭 + Iron Law

### 原文（EN）

```markdown
---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```
```

### 翻訳（JA）

```markdown
---
name: systematic-debugging-ja
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# 体系的デバッグ（Systematic Debugging）

## 概要

場当たり的な修正は時間を浪費し、新たなバグを生む。急場のパッチは根底の問題を覆い隠すだけである。

**中核原則：** 修正に着手する前に **必ず** 根本原因を見つける。症状への対処は失敗である。

**このプロセスの字義に反することは、デバッグの精神に反することである。**

## 鉄則

```
根本原因の調査なしに修正してはならない
```
```

### Tier 1 / Tier 2 / Tier 3 の適用ポイント

| Tier | ルール | 適用箇所 |
|---|---|---|
| Tier 1 | である調 | "場当たり的な修正は..." |
| Tier 1 | description 英語温存 | フロントマター無変換 |
| Tier 1 | 半角英数と JA 間にスペースなし | "JA)" / "JA は" 系 |
| Tier 2 | 専門用語と一般語の使い分け | "場当たり的" "急場のパッチ" 等の自然な訳語選択 |
| Tier 3 | (この冒頭部では未発火 — 後続の deep-think 合図セクションで発火) | — |

---

## サンプル B — Phase 4 由来の Tier 3 / Category C 追加（heavy）

### 追加対象セクション（target にのみ存在、原典にはない）

```markdown
### JA 圏 Claude Code 文脈の deep-think 合図

日本語環境では以下のような表現が、原典の "Ultrathink this" と同等のアーキテクチャ疑念合図である：

- 「もっと深く考えて」
- 「もう一段ロジックを追って」
- 「ultrathink して」
- 「踏み込んで考えてほしい」

これらが出たら Phase 1 に戻り、fundamentals を疑え。修正を重ねるな。
```

### 9-cell 判定の経路

- Phase 4 (forced on) で blind subagent が unclear-point として報告
- 提案を main agent が分類 → Category C (Differentiating, Catalog Tier 3 該当: 「Claude Code 環境固有の慣用表現を活用した utility 拡張」)
- Impact: heavy (新規節、複数文)
- 9-cell 表 high-utility 列 × C × heavy = **user approval required**
- ユーザに承認フォームを提示 → approve
- register に Phase 4 由来として記録
- Phase 2 にループ戻り → ゼロ変更で収束 (simulate)

---

## 9-cell の挙動の例示（high-utility 列）

| 仮想ケース | Category | Impact | high-utility 列の判定 | 結果 |
|---|---|---|---|---|
| 上記 deep-think 合図サブセクション (実発火) | C | heavy | user approval required | **承認後 register 追加** |
| Phase 1 で「git log を JA エンジニアが慣用的に読む流れ」一文追加 | B | minor | auto-approve | register 追加 |
| 仮想: subagent 並列起動を活用した hypothesis 並列検証コード例 | C | moderate | user approval required | (本 dogfooding では発火させず) |
| 仮想: 「JA 圏のコードレビュー文化での is-architecture-the-issue 議論作法」段落 | C | heavy | user approval required | (本 dogfooding では発火させず) |

→ high-utility 列は **C × heavy が user approval で採用可** という点が決定的に balanced と異なる。本 dogfooding の中核観察。

---

## Phase 1 中の auto-approve 発火 (Tier 2 由来)

### サンプル C — Common Rationalizations 表に B × minor の追加

#### 原文の 1 行

```markdown
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
```

#### 翻訳 + auto-approve 追加

```markdown
| 「複数修正を一度に当てれば時短」 | 何が効いたか切り分けられない。新規バグを生む。
                                  （日本語の git 環境では `git bisect` を併用すると分離が容易）|
```

括弧内が **B × minor の auto-approve 追加**。Locale-JA Tier 2 「JA 圏ツール表現の自然な接続」に該当。register に 1 行追加。

---

## サンプルから読み取れる 9-cell の挙動（high-utility 列）

| Profile | このサンプル中の Category 別 register 件数 |
|---|---|
| high-utility | A: 0 / B: 1 (minor, Phase 1) / C: 1 (heavy, Phase 4 経由) |

→ **C × heavy が register に入る** ことが high-utility プロファイルの設計目標。

これは high-utility プロファイルが意図どおりに **Tier 3 由来の utility 拡張を取り込む方向に偏っている** ことの証拠である。
