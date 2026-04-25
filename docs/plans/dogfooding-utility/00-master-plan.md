# systematic-debugging-ja Conversion Plan

> **For agentic workers:** Phase 0 8 項目を本文書で全て埋めた状態。
>
> **Dogfooding 注記:** Phase 2/3/5 simulated。Phase 4 は high-utility プロファイルで forced on のため 1 iteration を simulate して Tier 3 由来の 1 件を採用する。

**Goal:** Convert `superpowers:systematic-debugging` (EN) into `systematic-debugging-ja` (JA) along the Locale dimension only.

**Architecture:** v1.0 skill-conversion フロー、Profile = high-utility。

> **Target 注記.** 当初依頼の `superpowers:debugging` は superpowers 5.0.7 のキャッシュには存在せず、`systematic-debugging` を採用。F.1 / F.2 と異なる target である。

---

## Phase 0: Scope Declaration

### 1. Source skill

- Path: `C:\Users\workm\.claude\plugins\cache\claude-plugins-official\superpowers\5.0.7\skills\systematic-debugging\SKILL.md`
- Version: superpowers 5.0.7
- Layout: Flat layout
- References included (本文記載):
  - `root-cause-tracing.md`
  - `defense-in-depth.md`
  - `condition-based-waiting.md`
- Phase 1 conversion targets: SKILL.md + 上記 3 ファイル
- Target skill name: `systematic-debugging-ja`

### 2. Dimension declaration

| Dimension | Source | Target | Changes? |
|---|---|---|---|
| Technical | (n/a) | (n/a) | no |
| Locale | en-US | ja-JP | yes |
| Agent | Claude Code | Claude Code | no |

### 3. Conversion-rule tables

#### Locale (active dimension)

##### Terminology table

| EN term | JA term | Notes |
|---|---|---|
| systematic debugging | 体系的デバッグ | |
| root cause | 根本原因 | |
| symptom | 症状 | |
| Iron Law | 鉄則 | |
| reproduce | 再現する | |
| stack trace | スタックトレース | |
| diagnostic instrumentation | 診断用計測 | "instrumentation" は文脈で「計測」/「観測コード挿入」 |
| backward tracing | 後方トレース | |
| hypothesis | 仮説 | |
| architectural problem | アーキテクチャ問題 | |
| ultrathink | 深く考える | 原文 "Ultrathink this" は引用形式で英語温存 |
| your human partner | あなたの人間のパートナー | 原典慣用 |
| condition-based waiting | 条件ベース待機 | |
| defense in depth | 深層防御 | |

##### Style rules

- である調統一。
- bash / shell スクリプトのコードブロックは無変換。echo の中の英語メッセージも無変換。
- "STOP and Follow Process" のような大文字強調はそのまま大文字英語表記を残してもよいし「停止せよ」と訳してもよい。本変換では原文ニュアンス温存のため英語ヘッダのまま残し、本文で日本語訳を併記する形を採る。

#### "No-equivalent" constructs (locale)

| Source construct | Issue | Resolution |
|---|---|---|
| "Ultrathink this" 引用 | 文化依存（superpowers 内 idiom） | 引用括弧で英語温存 + 注釈「原典の人間パートナーが要求する深い思索の合図」 |
| "Pattern Says X but I'll adapt it differently" | 直訳がぎこちない | 「パターンは X と言うが私は別途適合させる」 |

#### Catalog auto-merge

- Locale-JA カタログを high-utility プロファイルで Tier 1 + Tier 2 + Tier 3 まで参照。

### 4. Scope-strip list

| Topic | Reason for strip |
|---|---|
| (none) | high-utility でも構造保全は前提。section の追加可、削除なし。 |

### 5. Approved-additions register (initially empty)

| Added-on | Location | Summary | Category | Reason | Source-phase | Impact | Approver |
|---|---|---|---|---|---|---|---|

### 6. Evaluation-scenario skeleton

> **バイアス制御。** description ("Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes") からシナリオを派生。本文の "Four Phases" 詳細は読まない態で。

#### Scenario 1 (central case)

- Situation: ユーザが「テストが flaky で、3 日前から CI で間欠的に落ちる」と相談。
- Requirement checklist:
  - [ ] [critical] [Tier 1] 修正提案より前に Phase 1（再現・最近の変更確認・evidence gathering）を完遂する
  - [ ] [Tier 1] 多層構成での layer 別 logging を提示
  - [ ] [Tier 2] 日本語コミットメッセージ・PR 文化での「git log を読みながら最近の変更を確認する」自然な手順
  - [ ] [Tier 3] **AI ペアプログラミング文脈での deep-think 合図**（「Ultrathink」「もっと深く考えて」等）の認識と挙動変更（high-utility 専用 Tier 3 項目）

#### Scenario 2 (edge)

- Situation: ユーザが「3 つ修正を試したが直らない」と言ってくる。Phase 4.5 (architecture を疑う) に到達できるか。
- Requirement checklist:
  - [ ] [critical] [Tier 1] 4 件目の修正を提案する前に立ち止まり、アーキテクチャ問題の可能性を user に escalate
  - [ ] [Tier 2] JA 圏で慣用的な「設計を再考する」「リアーキテクチャを検討する」表現で escalation を伝える
  - [ ] [Tier 3] **Claude Code 環境のサブエージェント並列起動を活用した hypothesis 並列検証**（high-utility 専用 Tier 3 項目）

(Tier 3 がここでは scenario チェックリストに登場している点が、F.1 / F.2 との大きな違い。)

### 7. Conversion Profile declaration (v1.0)

- [ ] high-fidelity (高忠実度)
- [ ] balanced (バランス)
- [x] **high-utility (高有用性)**

選択理由：JA 圏読者の utility を最大化したい。原典は `your human partner` という独特の関係性表現を持つが、JA 圏 Claude Code ユーザは「Ultrathink」のような特有の合図を使い慣れているため、JA 化で **Tier 3 由来の utility 拡張**（subagent 並列・deep-think 合図認識）を取り込む価値がある。

#### §7.1 Phase 4 opt-out 試行

- high-utility では opt-out は仕様上 halt。本変換では opt-out しない。**Phase 4 は forced on**。

### §X.1 Catalog status check

- Persisted catalogs scanned: なし
- Match for this conversion: no — generate fresh
- (Locale-JA カタログを memory-only で再利用、本ケースでは Tier 3 セクションを実体化する)

### §X.2 Catalog generation and two-stage approval

- Catalog: `memory only` (Locale-JA、Tier 3 まで)
- Stage 1 content approval: 本文書内で定義
- Stage 2 persistence approval: **NO** — memory only
- Tier reference range (profile-linked): **Tier 1 + Tier 2 + Tier 3**

#### Locale-JA カタログ抜粋（Tier 3 まで）

##### Tier 1 (mandatory)

- 文体一貫性（である調 / です・ます調）
- 半角英数と日本語の間隔規則
- 英語固有表現の温存ルール

##### Tier 2 (recommended for balanced or higher)

- JA 圏ツール表現の自然な接続
- JIS X 0208 範囲遵守
- 専門用語と一般語の使い分け

##### Tier 3 (recommended for high-utility)

- **Claude Code / agent 環境固有の慣用表現を活用した utility 拡張**：
  - "Ultrathink" / "深く考えて" 等の deep-think 合図への言及
  - サブエージェント並列起動（`superpowers:dispatching-parallel-agents`）の文脈統合
  - スキル間連携（`superpowers:test-driven-development` 等への REQUIRED-SUB-SKILL マーカー）の JA 慣用化
- **JA 圏エンジニアリング文化に固有の deep-debug 議論作法** (例: 「ペアプロでの仮説共有」「コードレビュー文化」) への簡潔な参照

##### Gotchas（プロファイル非依存で必読）

- description フィールド内 "Use when ..." は翻訳禁止
- コードブロック中身は無変換
- bash の `$IDENTITY` のような環境変数記法はそのまま

### 8. Cross-agent verification strategy

- Agent dimension = identity → `N/A — Agent dimension = identity`。

---

## Phase 1: Initial Translation

サンプルは `phase1-sample.md` 参照。

タスク分解:
- T1: SKILL.md 翻訳
- T2: root-cause-tracing.md 翻訳
- T3: defense-in-depth.md 翻訳
- T4: condition-based-waiting.md 翻訳

### Phase 1 中の追加提案ハンドリング（high-utility 列の 9-cell）

| Category × Impact | high-utility |
|---|---|
| A · minor / moderate | auto-approve |
| A · heavy | user approval |
| B · minor / moderate | auto-approve |
| B · heavy | user approval |
| C · minor | auto-approve |
| C · moderate / heavy | user approval |

→ high-utility では Tier 3 / Category C 提案も **rejected されない**（heavy 系は user approval だが採用可）のが特徴。

## Phase 2: Fixpoint Review Loop

**Dogfooding 簡略化: simulated single-pass.** high-utility プロファイルでも fixpoint は high-stakes。simulate single-pass。

## Phase 3: Code Executability Check

**Dogfooding 簡略化: simulated, no actual code blocks executed.**

```
Phase 3: SKIPPED (illustrative bash code only)
Adopted-API roster: (empty)
```

## Phase 4: Empirical Prompt Tuning

**FORCED ON (high-utility profile).** 1 iteration を simulate。

### Simulated empirical run

- Blind subagent が Scenario 2 (3+ 修正失敗時の挙動) を実行。
- Self-reported finding: 「JA 圏 Claude Code ユーザは『Ultrathink』をよく使うが、原典では英語のみで言及されており JA 化版でこの慣用が説明されないと、JA ユーザは Phase 4.5 への昇格合図を見落とすリスクがある」
- 提案: SKILL.md `your human partner's Signals You're Doing It Wrong` セクションに **新規節**「JA 圏 Claude Code 文脈の deep-think 合図」を追加し、日本語の「もっと深く考えて」「もう一段ロジックを追って」「ultrathink して」等が同等のシグナルであることを記す。
- Category: **C (Differentiating)** — Catalog Tier 3 該当 (Claude Code 環境固有の慣用表現)
- Impact: **heavy**（新規節、3-5 文）
- 9-cell 判定: high-utility × C × heavy = **user approval required**
- ユーザ承認 (シミュレート): approved
- register に Phase 4 由来として記録

### 個別承認フォーム

```
【Phase 4 Change Proposal】
Target location: SKILL.md §"your human partner's Signals You're Doing It Wrong" 末尾に
                 新規サブセクション「JA 圏 Claude Code 文脈の deep-think 合図」
Summary:        日本語の deep-think 合図を原典英語シグナルと併記し、Phase 4.5 escalation を促す
Detail:         「日本語環境では『もっと深く考えて』『もう一段ロジックを追って』
                 『ultrathink して』等の表現が、原典の "Ultrathink this" と同等の
                 アーキテクチャ疑念合図である。これらが出たら Phase 1 に戻り、
                 fundamentals を疑え。」
Rationale:      Blind subagent (simulated) が JA Claude Code ユーザの慣用を unclear と報告。
                 high-utility プロファイルが Tier 3 / Category C を許容するため採用判断可。
Impact:         heavy（新規 3-5 文サブセクション）
Category:       C (Differentiating, Catalog Tier 3 該当)
Verdict:        approve
```

### Phase 4 → Phase 2 ループ

承認された Phase 4 追加は Phase 2 にループ → ゼロ変更で再収束（simulate）。

## Phase 5: Auto-Fire Test

**Dogfooding 簡略化: SKIP per dogfooding policy.**

## Completion

- [x] Phase 0 完了
- [x] Phase 1 サンプル + Phase 1 由来 1 件の追加（後述 phase1-sample.md）
- [x] Phase 2 simulated single-pass
- [x] Phase 3 SKIPPED (no code build)
- [x] Phase 4 forced on, 1 iteration simulated, **Tier 3 由来の 1 件を採用**
- [x] Phase 5 SKIPPED per dogfooding policy
- [x] Drift report at `drift-report.md`
