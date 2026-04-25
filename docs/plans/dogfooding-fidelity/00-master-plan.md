# writing-skills-ja Conversion Plan

> **For agentic workers:** Use `superpowers:executing-plans` or `superpowers:subagent-driven-development`. This plan was scaffolded by `skill-conversion` v1.0 — the 8 Phase 0 declarations below are all filled in before proceeding to Phase 1.
>
> **Dogfooding note:** Phases 2/3/4/5 are simulated for v1.0 dogfooding. See `dogfooding-result.md` for the simulation policy.

**Goal:** Convert `superpowers:writing-skills` (EN) into `writing-skills-ja` (JA) along the Locale dimension only.

**Architecture:** Apply the skill-conversion v1.0 flow (Phase 0 → Phase 5) with Profile = high-fidelity, fixpoint review, and approved-additions register.

---

## Phase 0: Scope Declaration

### 1. Source skill

- Path: `C:\Users\workm\.claude\plugins\cache\claude-plugins-official\superpowers\5.0.7\skills\writing-skills\SKILL.md`
- Version: superpowers 5.0.7
- Layout: Flat layout (`SKILL.md` + companion `*.md` files in same directory)
- References included:
  - `anthropic-best-practices.md`
  - `testing-skills-with-subagents.md`
  - `persuasion-principles.md`
  - `graphviz-conventions.dot`
  - `render-graphs.js`
- Phase 1 conversion targets: SKILL.md + the 3 markdown companions. The `.dot` and `.js` are non-prose (kept verbatim, no Locale conversion needed).
- Target skill name: `writing-skills-ja`

### 2. Dimension declaration

| Dimension | Source | Target | Changes? |
|---|---|---|---|
| Technical | (n/a — skill is process documentation) | (n/a) | no |
| Locale | en-US | ja-JP | yes |
| Agent | Claude Code | Claude Code | no |

### 3. Conversion-rule tables

#### Locale (active dimension)

##### Terminology table (EN → JA, fixed)

| EN term | JA term | Notes |
|---|---|---|
| skill | スキル | カタカナ固定 |
| pressure scenario | 圧力シナリオ | TDD 用語 |
| baseline behavior | ベースライン挙動 | |
| rationalization | 合理化 | 心理学用語そのまま |
| loophole | 抜け道 | |
| Iron Law | 鉄則 | |
| RED-GREEN-REFACTOR | RED-GREEN-REFACTOR | 英語のまま保持（TDD 慣用） |
| flowchart | フローチャート | |
| cross-reference | クロスリファレンス | |
| frontmatter | フロントマター | |
| token efficiency | トークン効率 | |
| description field | description フィールド | コード識別子は英語のまま |
| your human partner | あなたの人間のパートナー | 直訳。原文の固有表現を保持 |
| anti-pattern | アンチパターン | |
| red flag | レッドフラッグ | |

##### Style rules

- 文体は **である調** で統一（敬体は使わない）。命令・規則を述べる文書のため。
- Markdown 構造（見出し階層、表、コードブロック）は原文と 1:1 対応。
- コードブロック内（YAML、bash、TypeScript 例）は **原文のまま** — コード自体は技術的事実なので翻訳しない。コメント部のみ JA 化可（ただし高忠実度プロファイルのため、原文のコメントが英語ならそのまま残す）。
- "Use when..." フレーズは description 内では英語のまま保持（Claude が description を検索する際のキーワード一貫性を破壊しないため）。
- 半角英数と日本語の間にスペースは挿入しない（一般的な日本語技術文書の慣習）。

#### "No-equivalent" constructs (locale)

| Source construct | Issue in target | Resolution |
|---|---|---|
| "Use when ..." 慣用 | description field 検索キーワード | 英語のまま温存 |
| `<Good>` / `<Bad>` HTML タグ | Markdown 標準ではない（superpowers 内慣習） | 原文と同じく `<Good>` / `<Bad>` のまま |
| Cialdini 引用 | 文化依存だが固有名詞 | カタカナ「チャルディーニ」併記不要、原文のまま |

#### Catalog auto-merge (Phase 1–2)

- 本変換は Locale のみ。Catalog の Gotchas は「Locale-JA」カタログから自動マージ（後述 §X.2 参照）。

### 4. Scope-strip list

| Topic | Reason for strip |
|---|---|
| (none) | Locale 変換のみ。本文構造を温存することが高忠実度プロファイルの中核要件。何も削らない。 |

### 5. Approved-additions register (initially empty)

| Added-on | Location | Summary | Category | Reason | Source-phase | Impact | Approver |
|---|---|---|---|---|---|---|---|

(空。本変換は Phase 4 オプトアウト + 高忠実度のため、Phase 1 で fidelity-preserving な追加が出ない限り空のまま終わる想定。)

### 6. Evaluation-scenario skeleton

> **バイアス制御注記。** 以下のシナリオは原典の `description`（"Use when creating new skills, editing existing skills, or verifying skills work before deployment"）と、もし原典が `When to Use This Skill` セクションを持っていればそこから派生させる。原典本文の見出し・コード例・参考ファイルセクションからは派生させない。

#### Scenario 1 (central case)

- Situation: ユーザが新しいスキル（例: `condition-based-waiting`）を作成しようとしている。原典記載どおり、TDD サイクル（RED-GREEN-REFACTOR）を踏んだスキル開発手順を求めている。
- Requirement checklist:
  - [ ] [critical] [Tier 1] エージェント（実行者）が、スキル本文を書く前に「ベースライン圧力シナリオでまず失敗を観察する」ステップを必ず通る
  - [ ] [Tier 1] フロントマターの description フィールドが「Use when ...」で始まることを示す
  - [ ] [Tier 1] description 内で skill のワークフロー要約をしないという CSO ルールを示す

#### Scenario 2 (edge)

- Situation: ユーザが既存スキルに「ちょっとした節を追加したい」と言ってくる。Iron Law（テストなしの追加禁止）に対するプレッシャーシナリオ。
- Requirement checklist:
  - [ ] [critical] [Tier 1] 「節追加だけだから」というラショナライゼーションを明示的に拒絶し、追加分にもベースラインテストを要求する
  - [ ] [Tier 1] 削除＝削除（"Delete means delete"）の方針を保持する

(本プロファイルは high-fidelity のため Catalog Tier 範囲は Tier 1 のみ。Tier 2 / Tier 3 のチェックリスト項目は scenarios に **意図的に embed しない**。)

### 7. Conversion Profile declaration (v1.0)

- [x] **high-fidelity (高忠実度)**
- [ ] balanced (バランス)
- [ ] high-utility (高有用性)

選択理由：原典 `writing-skills` は superpowers の方法論的中核（TDD-for-skills を厳格に適用）であり、原作者の意図を一字一句保つことが価値の中心。utility 拡張（target-environment 差別化機能の追加）はこのスキルの性質に対し有害。

#### §7.1 Reason if high-fidelity + Phase 4 opt-out (declared)

> **Phase 4 オプトアウト宣言。**
>
> Reason: Locale 変換のみのため、JA 化が原典の伝達意図を歪めない限り blind subagent 評価は冗長。`writing-skills` は方法論的厳格さ（rationalization テーブル、Iron Law、Red Flags）を売りにしており、blind subagent が「曖昧」と感じた箇所を fix することは原作者意図の歪曲リスクと天秤に掛けると割に合わない。高忠実度プロファイルが許す唯一の opt-out 経路を行使する。

### §X.1 Catalog status check (v1.0)

- Persisted catalogs scanned: `references/catalogs/INDEX.md` を参照 → ja-locale カタログは未登録
- Match for this conversion: no — generate fresh
- (Dogfooding 簡略化: カタログ生成サブエージェントは実際には dispatch せず、本文書内で手書きの "Locale-JA" 仮想カタログを定義する)

### §X.2 Catalog generation and two-stage approval (v1.0)

- Catalog: `memory only`（永続化しない）
- Stage 1 content approval: 本 Phase 0 ドキュメント内で定義（下記）。dogfooding のため subagent dispatch は省略。
- Stage 2 persistence approval: **NO** — Locale-JA カタログは memory only。Phase 5 終了で破棄。
- Tier reference range (profile-linked): **Tier 1 のみ**（高忠実度プロファイル）

#### Locale-JA カタログ抜粋（Tier 1 のみ範囲）

##### Tier 1 (mandatory)

- 文体一貫性（である調 / です・ます調のいずれかに統一）
- 半角英数と日本語の間隔規則の明示
- 英語固有表現（プログラミング識別子、API 名、frontmatter フィールド名）の英語温存ルール

##### Tier 2 / Tier 3

- (本プロファイルでは参照範囲外のため、このカタログから読まない。Tier 2 / 3 セクション内容は省略。)

##### Gotchas（プロファイル非依存で必読）

- 「Use when ...」のような description フィールド内英語慣用句は翻訳禁止（auto-fire の検索キーワード一貫性を破壊するため）。
- Markdown 内のコードブロック（言語識別子付き fence）は中身を翻訳しない。
- HTML 風タグ（`<Good>`, `<Bad>`）は superpowers 慣習なのでそのまま。

### 8. Cross-agent verification strategy

- Agent dimension = identity（Claude Code → Claude Code）。`N/A — Agent dimension = identity`。Phase 3 / 4 / 5 はデフォルト戦略。

---

## Phase 1: Initial Translation

抜粋サンプルは `phase1-sample.md` を参照（dogfooding のため SKILL.md 冒頭部 + Iron Law 節 + Common Rationalizations 節 のみを翻訳）。

タスク分解（フル変換時）:
- T1: SKILL.md 全体翻訳
- T2: anthropic-best-practices.md 翻訳
- T3: testing-skills-with-subagents.md 翻訳
- T4: persuasion-principles.md 翻訳
- T5: graphviz-conventions.dot コメントのみ JA 化（コード本体は無変換）
- T6: render-graphs.js コメントのみ JA 化（コード本体は無変換）

### Phase 1 中の追加提案ハンドリング

高忠実度プロファイルの 9-cell 表により：
- Category A (Fidelity-preserving) · minor/moderate → auto-approve
- Category A · heavy → user approval
- Category B (Essential) · minor → user approval / それ以上は **rejected**
- Category C (Differentiating) · 全 impact → **rejected**

dogfooding でのシミュレーション結果：Category B/C 提案は出ない（Locale 変換は本質的に B/C 拡張を呼ばない）。Category A も「文体統一のための助詞調整」程度で minor、register に追加するほどの drift にならず空のまま。

## Phase 2: Fixpoint Review Loop

**Dogfooding 簡略化: simulated single-pass.** 実際の fixpoint loop は dogfooding のスコープ外（contrived な example で収束させても意味がない）。

シミュレーション概要:
- 1 pass 実施想定
- Technical / Prose / Source-compare の 3 軸 → いずれもゼロ変更
- 高忠実度プロファイル + 高 stakes（superpowers の中核方法論スキル）→ 通常は 2 連続ゼロ変更が要件だが dogfooding では 1 pass で打ち切り

## Phase 3: Code Executability Check

**Dogfooding 簡略化: simulated, no actual code blocks executed.**

`writing-skills` 内のコードブロックは TypeScript / bash / YAML / dot の例示でありビルド対象ではない。原典のスキル設計上、コードはあくまで「説明用例」のため Phase 3 は SKIPPED 相当。

```
Phase 3: SKIPPED (code blocks are illustrative only, not buildable scaffolding)
Adopted-API roster: (empty)
```

## Phase 4: Empirical Prompt Tuning

**SKIPPED by user choice (high-fidelity profile, reason: §7.1 に記載)**

## Phase 5: Auto-Fire Test

**Dogfooding 簡略化: SKIP per dogfooding policy.**

通常運用では `phase5-writing-skills-ja.md` と `phase5-handoff.md` を生成しユーザに渡す必要があるが、本 dogfooding は v1.0 spec 検証目的のため Phase 5 ハンドオフは実施しない。本物の変換では必須。

## Completion (dogfooding scope)

- [x] Phase 0 完了（本ドキュメントが 8 項目を埋めた状態）
- [x] Phase 1 サンプル提示（`phase1-sample.md`）
- [x] Phase 2 simulated single-pass
- [x] Phase 3 SKIPPED (no code)
- [x] Phase 4 SKIPPED by user choice (high-fidelity opt-out)
- [x] Phase 5 SKIPPED per dogfooding policy
- [x] Drift report generated at `drift-report.md`
