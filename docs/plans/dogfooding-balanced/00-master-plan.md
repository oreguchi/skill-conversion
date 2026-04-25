# test-driven-development-ja Conversion Plan

> **For agentic workers:** Use `superpowers:executing-plans` or `superpowers:subagent-driven-development`. Phase 0 8 項目は本文書で全て埋まっている。
>
> **Dogfooding 注記:** Phase 2/3/5 は simulated。Phase 4 は balanced プロファイルで forced on のため 1 iteration を simulate して 1 件の追加を採用する。

**Goal:** Convert `superpowers:test-driven-development` (EN) into `test-driven-development-ja` (JA) along the Locale dimension only — pretending the user wants balanced coverage.

**Architecture:** v1.0 skill-conversion フロー、Profile = balanced。

---

## Phase 0: Scope Declaration

### 1. Source skill

- Path: `C:\Users\workm\.claude\plugins\cache\claude-plugins-official\superpowers\5.0.7\skills\test-driven-development\SKILL.md`
- Version: superpowers 5.0.7
- Layout: Flat layout
- References included:
  - `testing-anti-patterns.md` (本文 `@testing-anti-patterns.md` 参照)
- Phase 1 conversion targets: SKILL.md + `testing-anti-patterns.md`
- Target skill name: `test-driven-development-ja`

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
| Test-Driven Development / TDD | テスト駆動開発 / TDD | 略語 TDD は併用 |
| Iron Law | 鉄則 | |
| RED-GREEN-REFACTOR | RED-GREEN-REFACTOR | 慣用維持 |
| failing test | 失敗するテスト | |
| sunk cost fallacy | サンクコストの誤謬 | |
| rationalization | 合理化 | |
| pragmatic | 実用的 | "pragmatic" の自己呼称ラショナライゼーションは「実用主義」と訳す |
| dogmatic | 教条的 | |
| edge case | エッジケース | カタカナ慣用 |
| your human partner | あなたの人間のパートナー | 原典慣用そのまま |
| YAGNI | YAGNI | 略語そのまま |
| mock | モック | カタカナ |
| dependency injection | 依存性注入（DI） | |

##### Style rules

- である調統一。
- TypeScript / bash コードブロック内は無変換。コードコメントは原文が短ければ JA 化、長文コメントなら原文温存（本ケースでは無変換でよい）。
- "Use when ..." はフロントマター内英語温存。

#### "No-equivalent" constructs (locale)

| Source construct | Issue | Resolution |
|---|---|---|
| `<Good>` / `<Bad>` HTML タグ | superpowers 慣習 | そのまま |
| 「Production code → test exists ...」の罫線図 | レイアウト依存 | 罫線そのまま、テキストのみ JA 化 |

#### Catalog auto-merge

- Locale-JA カタログを balanced プロファイルで Tier 1 + Tier 2 まで参照。本変換では同じ memory-only Locale-JA カタログを使う（ただし Tier 2 まで visible）。

### 4. Scope-strip list

| Topic | Reason for strip |
|---|---|
| (none) | Locale 変換のみ。balanced プロファイルでも本文構造の保全が前提。 |

### 5. Approved-additions register (initially empty)

| Added-on | Location | Summary | Category | Reason | Source-phase | Impact | Approver |
|---|---|---|---|---|---|---|---|

### 6. Evaluation-scenario skeleton

> **バイアス制御。** description ("Use when implementing any feature or bugfix, before writing implementation code") からシナリオを派生させ、本文（フローチャート、Iron Law、Common Rationalizations 等）の存在を **見ずに** 設計したつもりで書く。

#### Scenario 1 (central case)

- Situation: ユーザが「ユーザの email を validate する関数を実装したい。空欄を弾きたい」と依頼。エージェントが本スキルを参照し、TDD で進めるべきと判断。
- Requirement checklist:
  - [ ] [critical] [Tier 1] エージェントが production code より前に failing test を書く
  - [ ] [Tier 1] テスト失敗を実際に観察してから実装に入る（"Verify RED" ステップ）
  - [ ] [Tier 1] 実装は最小コード（YAGNI 遵守）
  - [ ] [Tier 2] 実装言語が JA コミュニティで多用される（Python/JS）の場合、`pytest -k` / `npm test --` の実行コマンド例を JA 文脈で読みやすく示す（Locale-JA Tier 2: 「JA 圏で広く使われるツール慣用句の橋渡し」）

#### Scenario 2 (edge)

- Situation: ユーザが「もう手動で動作確認したから、テストは後で書く」と言ってくる。エージェントは "Tests after achieve same goals" の合理化を拒絶できるか。
- Requirement checklist:
  - [ ] [critical] [Tier 1] "Tests after" を明確に拒絶し、原文の "What does this do? vs What should this do?" 区別を JA でも保つ
  - [ ] [Tier 1] サンクコスト合理化（"Deleting X hours is wasteful"）への counter を提示
  - [ ] [Tier 2] 「リファクタ中に既存テストを壊した時の戻し方」（JA 文脈の git revert / stash 使い方）への簡潔な参照（Tier 2 範囲）

(本プロファイルは balanced のため Tier 1 + Tier 2 を embed。Tier 3 は意図的に embed しない。)

### 7. Conversion Profile declaration (v1.0)

- [ ] high-fidelity (高忠実度)
- [x] **balanced (バランス)**  ← default
- [ ] high-utility (高有用性)

選択理由：JA 圏 TDD ユーザは原典の厳格さを保ちつつ、JA 圏で慣れた最低限のツール表現（pytest / npm test 等）が混じってもよい、というバランス志向のニーズを想定。

#### §7.1 Phase 4 opt-out 試行

- balanced プロファイル下で opt-out を試みると **halt** する規約（profile-system.md §Phase 4 opt-out 参照）。本変換では opt-out しない。**Phase 4 は forced on**。

### §X.1 Catalog status check

- Persisted catalogs scanned: なし
- Match for this conversion: no — generate fresh
- (Dogfooding 簡略化: subagent dispatch せず Locale-JA を本文書内で再利用)

### §X.2 Catalog generation and two-stage approval

- Catalog: `memory only` (Locale-JA)
- Stage 1 content approval: 本文書内で定義、approve
- Stage 2 persistence approval: **NO** — memory only
- Tier reference range (profile-linked): **Tier 1 + Tier 2**

#### Locale-JA カタログ抜粋（Tier 2 範囲まで）

##### Tier 1 (mandatory)

- 文体一貫性（である調 / です・ます調）
- 半角英数と日本語の間隔規則
- 英語固有表現の温存ルール

##### Tier 2 (recommended for balanced or higher)

- JA 圏で慣用化したツール表現の自然な接続（"pytest を実行する" / "npm test を走らせる" 等の動詞の選び方）
- 半角カタカナ・全角英数の混入禁止（JIS X 0208 の標準範囲遵守）
- 専門用語と一般語の使い分け（例: "test" は文脈で「テスト」/「テストケース」の使い分け）

##### Tier 3 (high-utility 専用 — 本プロファイルでは未参照)

- (memory only のため省略)

##### Gotchas（プロファイル非依存で必読）

- description フィールド内 "Use when ..." は翻訳禁止
- コードブロック中身は無変換
- `<Good>` / `<Bad>` タグ温存

### 8. Cross-agent verification strategy

- Agent dimension = identity → `N/A — Agent dimension = identity`。

---

## Phase 1: Initial Translation

サンプルは `phase1-sample.md` 参照。

タスク分解:
- T1: SKILL.md 翻訳
- T2: testing-anti-patterns.md 翻訳

### Phase 1 中の追加提案ハンドリング（balanced 列の 9-cell）

| Category × Impact | balanced |
|---|---|
| A · minor / moderate | auto-approve |
| A · heavy | user approval |
| B · minor | auto-approve |
| B · moderate | user approval |
| B · heavy | user approval |
| C · minor | user approval |
| C · moderate / heavy | rejected |

実 dogfooding シミュレーションでは **Phase 1 で 1 件の追加** が出る想定（後述 phase1-sample.md 参照）。

## Phase 2: Fixpoint Review Loop

**Dogfooding 簡略化: simulated single-pass.** balanced プロファイルでも fixpoint は high-stakes 扱い（test-driven-development は方法論的中核スキル）。本来は 2 連続ゼロ変更が要件だが simulate single-pass。

## Phase 3: Code Executability Check

**Dogfooding 簡略化: simulated, no actual code blocks executed.**

```
Phase 3: SKIPPED (illustrative TypeScript / bash code only)
Adopted-API roster: (empty)
```

## Phase 4: Empirical Prompt Tuning

**FORCED ON (balanced profile).** 1 iteration を simulate。

### Simulated empirical run

- 実際には empirical-prompt-tuning を dispatch せず、結果を 1 件想定で記述。
- Blind subagent が Scenario 1 を実行し、self-reported finding として「JA 文書の TDD で `pytest` の出力解釈が文化的にやや暗黙」と報告したと仮定。
- 提案: SKILL.md `Verify RED` セクションに「日本語ロケール環境で pytest 出力を読む際の注意」（数行）を追加。
- Category: B (Environment-essential) — Catalog Tier 2 に該当
- Impact: moderate (新規説明数行)
- 9-cell 判定: balanced × B × moderate = **user approval required**
- ユーザ承認 (シミュレート): approved
- register に `Phase 4` 由来として記録

### Phase 4 → Phase 2 ループ

承認された Phase 4 追加は Phase 2 にループ → ゼロ変更で再収束（simulate）。

### 個別承認フォーム（記録）

```
【Phase 4 Change Proposal】
Target location: SKILL.md §Verify RED (Watch It Fail) 末尾
Summary:        日本語ロケール環境での pytest 出力解釈の補足を追加
Detail:         「日本語ロケールの端末では pytest の失敗メッセージの一部が
                 文字化けする場合がある。LANG=C.UTF-8 を確認するか、
                 pytest -v で詳細出力を強制すれば回避できる」
Rationale:      Blind subagent (simulated) が、JA 圏で TDD を実践する際の
                 ロケール固有の落とし穴を unclear-point として報告。
Impact:         moderate（新規 3 行段落の追加）
Category:       B (Environment-essential, Catalog Tier 2 該当)
Verdict:        approve
```

## Phase 5: Auto-Fire Test

**Dogfooding 簡略化: SKIP per dogfooding policy.**

## Completion

- [x] Phase 0 完了
- [x] Phase 1 サンプル + 1 件の Phase 1 追加（後述 phase1-sample.md）
- [x] Phase 2 simulated single-pass
- [x] Phase 3 SKIPPED (no code build)
- [x] Phase 4 forced on (1 iteration simulated, 1 approved addition)
- [x] Phase 5 SKIPPED per dogfooding policy
- [x] Drift report at `drift-report.md`
