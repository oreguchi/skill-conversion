# skill-conversion

**[English](#english)** | **[日本語](#日本語)**

---

## English

A Claude Code plugin that converts existing skills across any combination of three dimensions — **programming language / framework**, **natural-language locale**, and **target agent runtime** — with every deviation from the source individually approved and logged.

Naive "translate this skill" prompts silently add content, silently drop content, produce terminology drift, and leave no diff trail. This plugin replaces prompt-level care with a structural workflow that records every change and asks for approval before it happens.

### What's inside

| Component | Purpose |
|---|---|
| `SKILL.md` | The six-phase workflow (Phase 0 scope → Phase 5 auto-fire) and its user gates |
| `references/conversion-dimensions.md` | Definitions and combination rules for the three dimensions |
| `references/fixpoint-loop.md` | Three-axis review loop (technical / prose / source-compare) until a zero-change pass |
| `references/approved-additions.md` | Schema and lifecycle of the audit ledger |
| `references/verification-stages.md` | Phase 3–5 verification details |
| `references/cross-agent-menu.md` | Agent-specific conversion notes (Copilot CLI / Gemini CLI / Codex) |
| `references/drift-report.md` | Final drift summary template and verdict criteria |
| `references/templates/` | Starter scaffolds for the plan doc, register, and drift report |

### Three conversion dimensions

| Dimension | What changes | Example |
|---|---|---|
| **Technical** | Programming language or framework | C# → VB.NET, Python → Go, React → Vue |
| **Locale** | Natural language | English → Japanese, English → Chinese |
| **Agent** | Target runtime agent | Claude Code → Copilot CLI / Gemini CLI / Codex |

The three dimensions are independent. Any subset (including all three at once) is a valid conversion.

### Four core mechanisms

- **Conversion rule tables.** Before any translation, Phase 0 locks in a per-dimension mapping table (e.g. `def foo():` → `func foo()`) so ad-hoc per-occurrence decisions cannot drift the result.
- **Approved-additions register.** Anything that ends up in the target but was not in the source requires individual user approval, logged with reason and impact. Batch approval is an explicit anti-pattern.
- **Fixpoint review loop.** After translation, three review axes run in parallel (technical, prose, source-compare). Any change in any axis restarts all three. The loop continues until one full pass produces zero changes — two consecutive zero-change passes for high-stakes skills.
- **Drift report.** At completion, every approved addition and every rejected proposal feeds into a drift report with a final verdict (LOW / MODERATE / HIGH). The conversion is not complete until the user confirms the report.

### Six-phase workflow

```
Phase 0  Scope declaration       ← 8 items up front (dimensions, rule tables, eval scenarios, …)
  ↓
Phase 1  Initial translation     ← apply rule tables mechanically
  ↓
Phase 2  Fixpoint loop           ← 3 axes in parallel, until zero-change
  ↓
Phase 3  Code executability      ← does the converted code actually build / run?
  ↓
Phase 4  Empirical tuning        ← opt-in: blind-subagent evaluation
  ↓
Phase 5  Auto-fire test          ← fresh session verifies the description triggers
  ↓
Complete Drift report → user confirmation
```

### Self-validated (v0.1.0)

This plugin was shipped only after being run through its own Phase 4.

- 3 scenarios × 3 iterations = 9 blind-evaluator runs
  - S1: Python → Go (technical only)
  - S2: English → Japanese (locale only)
  - S3: English Claude Code → Japanese Copilot CLI + Go (all three dimensions)
- Precision 100%, retries 0 across every iteration
- Convergence at Iter3 (zero triggering signals)
- Final drift verdict: **LOW**

See [`docs/self-validation.md`](./docs/self-validation.md) for the full evidence summary.

### Prerequisites

Two other plugins must be installed first:

| Plugin | Purpose |
|---|---|
| `superpowers` | Planning, parallel-agent, and verification sub-skills this plugin calls |
| `empirical-prompt-tuning` | Blind-evaluator engine used in Phase 4 (opt-in) |

### Installing

```bash
/plugin install superpowers
/plugin install empirical-prompt-tuning
/plugin marketplace add oreguchi/skill-conversion
/plugin install skill-conversion@skill-conversion-marketplace
```

Verify in a fresh session by asking "what skills do you have?" — `skill-conversion:skill-conversion` should appear. Or invoke it directly with `/skill-conversion:skill-conversion`.

### Usage

Natural-language prompts. The plugin decides from the topic whether to fire.

- "Convert this C# skill to VB.NET"
- "Translate this skill to Japanese"
- "Port this Claude Code skill to Copilot CLI in Go"
- Combine freely: "Translate to Japanese and port to Gemini CLI"

On invocation, Phase 0 elicits the eight required declarations, a plan doc is written, and Phase 1–5 proceed with a user gate at every content addition.

### Design principles

- **Structural discipline over prompt-level care.** The workflow, not the instructions inside the prompt, is what prevents drift.
- **Individual approval, not batch.** Each addition is reviewed alone with its reason and impact. Batching makes the audit trail useless.
- **The register is load-bearing.** Phase 2's source-compare axis reads the register to distinguish intentional additions from defects, so the register must be maintained in real time.
- **Completion requires user sign-off.** No drift report acceptance, no completion — regardless of how clean the pass looks.

### When to use

- Producing localized / ported variants of a skill you maintain
- Teams who need an auditable answer to "why is this sentence here?"
- Any workflow that values quality and review cost over turnaround speed

### When not to use

- Authoring a new skill from scratch — use `superpowers:writing-skills` instead
- Single-line edits (typo fixes) — a direct edit is faster
- Non-skill Markdown (README, general docs) — this plugin is tuned for skill format specifically

### Compatibility

- Claude Code (primary target). Copilot CLI / Gemini CLI / Codex are supported as conversion *targets* (Agent dimension).
- Requires `superpowers` and `empirical-prompt-tuning` at install time.

### Feedback

Bugs, questions, suggestions: [open an issue](https://github.com/oreguchi/skill-conversion/issues).

### License

MIT. See [LICENSE](./LICENSE).

---

## 日本語

既存の Claude Code スキルを、**プログラミング言語 / フレームワーク**、**自然言語ロケール**、**ターゲットエージェント** の 3 次元（任意の組み合わせ）で変換するための Claude Code プラグインです。元スキルからのズレは、すべて一件ずつユーザー承認され、監査ログに残ります。

「このスキルを日本語に翻訳して」とアドホックに頼むと、元に無かった情報が静かに追加され、訳しにくい部分が静かに消え、用語にぶれが出て、差分も追えません。本プラグインは、プロンプトでの気配りに頼るのをやめて、**構造化されたワークフロー**で変更を残らず記録し、実行前にユーザー承認を取る形に置き換えます。

### 収録内容

| 構成要素 | 役割 |
|---|---|
| `SKILL.md` | 6 フェーズのワークフロー本体（Phase 0 スコープ宣言 → Phase 5 Auto-fire）とユーザーゲート |
| `references/conversion-dimensions.md` | 3 次元の定義と組み合わせ方 |
| `references/fixpoint-loop.md` | 3 軸レビューループ（技術／文章／元との差分）の不動点条件 |
| `references/approved-additions.md` | 監査台帳のスキーマとライフサイクル |
| `references/verification-stages.md` | Phase 3–5 の検証手順詳細 |
| `references/cross-agent-menu.md` | エージェント別の変換メモ（Copilot CLI / Gemini CLI / Codex） |
| `references/drift-report.md` | 最終差分レポートのテンプレートと判定基準 |
| `references/templates/` | plan doc / register / drift report の雛形 |

### 3 つの変換次元

| 次元 | 変えるもの | 具体例 |
|---|---|---|
| **Technical** | プログラミング言語・フレームワーク | C# → VB.NET、Python → Go、React → Vue |
| **Locale** | 自然言語 | 英語 → 日本語、英語 → 中国語 |
| **Agent** | 実行エージェント | Claude Code → Copilot CLI / Gemini CLI / Codex |

3 つの次元は独立しており、任意の部分集合（3 次元同時も可）が有効な変換として扱われます。

### 4 つの中核機構

- **変換規則表（conversion rule tables）。** 翻訳着手前の Phase 0 で、次元ごとに「元 → 先」の対応表を先に確定させます（例：`def foo():` → `func foo()`）。アドホックに都度判断することがなくなり、一貫性が担保されます。
- **追加台帳（approved-additions register）。** 元に無かった内容を先に足す場合は、必ず一件ずつユーザー承認を得て、理由と影響度とともに台帳に記録します。バッチ承認は明確にアンチパターンとして禁止しています。
- **不動点レビューループ（fixpoint review loop）。** 翻訳直後に 3 軸レビュー（技術／文章／元との差分）を並列実行します。どこか一つでも変更が入れば全 3 軸を再実行し、**1 パスで全軸ゼロ変更**になるまで繰り返します。重要度の高いスキルは **2 パス連続ゼロ変更** が収束条件です。
- **差分レポート（drift report）。** 完了時に、承認された追加・却下された提案を集計し、**LOW / MODERATE / HIGH** で総合判定します。このレポートをユーザーが承認するまで、変換は完了扱いになりません。

### 6 フェーズのワークフロー

```
Phase 0  スコープ宣言        ← 8 項目を先に確定（次元、規則表、評価シナリオ、…）
  ↓
Phase 1  初期翻訳             ← 規則表を機械的に適用
  ↓
Phase 2  Fixpoint ループ      ← 3 軸並列、ゼロ変更まで
  ↓
Phase 3  コード実行可能性     ← 変換後のコードは実際にビルド・実行できるか
  ↓
Phase 4  経験的チューニング   ← opt-in。ブラインドサブエージェントによる評価
  ↓
Phase 5  Auto-fire テスト     ← 新規セッションで description が発火するか
  ↓
完了    差分レポート → ユーザー最終承認
```

### 自己検証済み（v0.1.0）

本プラグインは、**自分自身を自分の Phase 4 にかけて**検証した結果として出荷されています。

- 3 シナリオ × 3 iteration = 計 9 回のブラインド評価
  - S1: Python → Go（技術次元のみ）
  - S2: 英語 → 日本語（ロケール次元のみ）
  - S3: 英語 Claude Code → 日本語 Copilot CLI + Go（3 次元同時）
- 全 iteration で精度 100%、retries 0
- Iter3 で triggering signal ゼロを達成して収束
- 最終 drift 判定：**LOW**

エビデンスの全貌は [`docs/self-validation.md`](./docs/self-validation.md) にまとめてあります。

### 必要な前提プラグイン

先に以下の 2 プラグインをインストールしてください:

| プラグイン | 役割 |
|---|---|
| `superpowers` | 本プラグインが呼び出す計画・並列エージェント・検証のサブスキル |
| `empirical-prompt-tuning` | Phase 4（opt-in）で使うブラインド評価エンジン |

### インストール

```bash
/plugin install superpowers
/plugin install empirical-prompt-tuning
/plugin marketplace add oreguchi/skill-conversion
/plugin install skill-conversion@skill-conversion-marketplace
```

新規セッションで「どんなスキルを持ってる？」と尋ねて `skill-conversion:skill-conversion` が表示されれば成功です。`/skill-conversion:skill-conversion` で直接呼び出すこともできます。

### 使い方

自然言語のプロンプトで呼び出します。話題からスキルが自律的に発火を判断します。

- 「この C# スキルを VB.NET に変換して」
- 「このスキルを日本語に翻訳して」
- 「この Claude Code スキルを Go で Copilot CLI 向けに移植して」
- 組み合わせも自由：「日本語訳して Gemini CLI 向けに移植して」

発火後は Phase 0 が必要な 8 項目をヒアリングし、plan doc を生成して Phase 1–5 へ進みます。内容を追加する場面には必ずユーザーゲートが挟まります。

### 設計方針

- **プロンプト作法ではなく、ワークフロー構造で防ぐ。** 指示文に「勝手に追加しないで」と書くのではなく、追加自体を構造的にブロックします。
- **個別承認、バッチ承認は禁止。** 追加は 1 件ずつ理由と影響度を添えて承認されます。バッチにすると audit trail が形骸化します。
- **台帳は動作の一部。** Phase 2 の差分チェックは台帳を参照して「正当な追加」と「混入」を区別します。そのため台帳はリアルタイムで更新される必要があります。
- **ユーザー承認を経るまで完了しない。** どれほどきれいに通ったとしても、drift report の承認を受けるまで変換は未完了扱いです。

### 向いている用途

- 自分のスキルをローカライズ版・別エージェント移植版として展開したいプラグイン作者
- 「なぜこの記述が入っているか」を audit できる変換プロセスを必要とするチーム
- 速度より品質を優先できる（レビューを複数回回す時間がある）ワークフロー

### 向かないケース

- ゼロからスキルを新規作成したい → `superpowers:writing-skills` を使ってください
- 1 箇所だけ直したい（誤字修正など）→ 直接編集したほうが速いです
- スキル以外の Markdown の翻訳（README、一般ドキュメント等）→ 本プラグインはスキル形式に特化しています

### 対応範囲

- 本体の動作ホストは Claude Code。Copilot CLI / Gemini CLI / Codex は変換の *ターゲット*（Agent 次元）として対応しています。
- インストール時点で `superpowers` と `empirical-prompt-tuning` が必要です。

### フィードバック

不具合、質問、改善提案などは [Issue](https://github.com/oreguchi/skill-conversion/issues) からお寄せください。

### ライセンス

MIT（[LICENSE](./LICENSE) 参照）
