# skill-conversion v1.0.0 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** skill-conversion プラグインを v0.2.0 から v1.0.0（stable リリース）に major bump。Conversion Profile + Catalog System + Phase 制御の三位一体を導入し、ターゲット有用性の検出を構造化する。

**Architecture:** ドキュメントベースのスキルプラグイン。SKILL.md と references/*.md の改訂・新設で仕様を表現。コード実装はなし（subagent prompt の文書化のみ）。検証は構造的整合性 + dogfooding。

**Tech Stack:** Markdown / Claude Code skill format / Git / GitHub CLI。

**Source spec:** `C:\dev\01_ClaudeProject\skill-conversion_v1.0.0開発\docs\specs\2026-04-25-skill-conversion-v1-design.md`

---

## Prerequisites（実装開始前）

### Task 0: 開発リポジトリの初期化

**Files:**
- Create directory: `C:\dev\01_ClaudeProject\skill-conversion_v1.0.0開発\`（既存）
- Copy from: `C:\Users\workm\.claude\plugins\cache\skill-conversion-marketplace\skill-conversion\0.2.0\` (read-only として参照)

- [ ] **Step 0.1: 開発リポジトリに v0.2.0 ソースをコピー**

```powershell
$src = "C:\Users\workm\.claude\plugins\cache\skill-conversion-marketplace\skill-conversion\0.2.0"
$dst = "C:\dev\01_ClaudeProject\skill-conversion_v1.0.0開発"

# .claude-plugin と skills/ を v0.2.0 起点で配置
Copy-Item -Path "$src\.claude-plugin" -Destination $dst -Recurse -Force
Copy-Item -Path "$src\skills" -Destination $dst -Recurse -Force
Copy-Item -Path "$src\LICENSE" -Destination $dst -Force -ErrorAction SilentlyContinue
Copy-Item -Path "$src\README.md" -Destination $dst -Force -ErrorAction SilentlyContinue
```

- [ ] **Step 0.2: git 初期化（必要なら）**

```bash
cd "C:/dev/01_ClaudeProject/skill-conversion_v1.0.0開発"
test -d .git || git init
git config user.name "oreguchi"
git config user.email "228712732+oreguchi@users.noreply.github.com"
```

- [ ] **Step 0.3: 初期 commit（v0.2.0 起点を固定）**

```bash
git add -A
git commit -m "chore: v0.2.0 baseline (forked for v1.0.0 development)"
git tag baseline-v0.2.0
```

- [ ] **Step 0.4: ファイル構造確認**

Run: `ls -la skills/skill-conversion/ skills/skill-conversion/references/`
Expected: SKILL.md + references/*.md 7ファイル + templates/4ファイル が存在

---

## Phase I-A: コア仕様改訂

### Task A.1: SKILL.md §7 を Profile 宣言にリネーム・拡張

**Files:**
- Modify: `skills/skill-conversion/SKILL.md`

- [ ] **Step A.1.1: 既存 §7 部分を Read**

```bash
grep -n "### 7" skills/skill-conversion/SKILL.md
```

- [ ] **Step A.1.2: §7 を以下の内容に置換**

§7 セクションを新規仕様に置換:

```markdown
### 7. Conversion Profile 宣言

ユーザーが変換の期待値を 3 段階で宣言する。profile が以下 4 軸を機械的に決定する：

- Phase 4 opt-in 状態
- approved-additions 承認閾値
- Catalog 参照範囲
- (Phase 2 fixpoint 収束基準は profile 不問で常に高 stakes)

**選択肢（必須）:**

- [ ] 高忠実度（high-fidelity）
- [ ] バランス（balanced）  ← デフォルト
- [ ] 高有用性（high-utility）

**省略時はバランス自動採用**（warning ログ）。不明な値は halt して再宣言を求める。

**Phase 4 opt-out 運用:** 高忠実度 profile を選び、かつ Phase 4 を opt-out する場合は §7.1 に**理由必須**で記載。バランス・高有用性で opt-out を試みた場合は halt（再選択させる）。

詳細は `references/profile-system.md` を参照。
```

- [ ] **Step A.1.3: §6 の scenario 骨子セクションに Catalog 参照指示を追記**

§6 の末尾（`Call superpowers:writing-plans` の前）に以下を追加:

```markdown
**Catalog 参照（v1.0 NEW）:** scenario 骨子作成時、profile が指定する Tier 範囲の Catalog 期待機能を Requirement checklist に組み込むこと。各 checklist アイテムには `[Tier N]` タグを付け、catalog 参照点を明示する。詳細は `references/catalog-system.md` 参照。
```

- [ ] **Step A.1.4: 整合性確認**

Run: `grep -n "Conversion Profile" skills/skill-conversion/SKILL.md`
Expected: §7 の Profile 宣言節が見える

Run: `grep -n "Catalog 参照" skills/skill-conversion/SKILL.md`
Expected: §6 の Catalog 参照指示が見える

- [ ] **Step A.1.5: Commit**

```bash
git add skills/skill-conversion/SKILL.md
git commit -m "feat(skill): Phase 0 §7 を Conversion Profile 宣言に拡張"
```

### Task A.2: SKILL.md に新規 §X (Catalog 状態確認・生成・承認) 追加

**Files:**
- Modify: `skills/skill-conversion/SKILL.md`

- [ ] **Step A.2.1: §7 の直後（Phase 1 の前）に新規 §X.1 と §X.2 を挿入**

§7 と Phase 1 の間に以下を追加:

```markdown
### X.1 Catalog 状態確認

Phase 0 開始時に永続化済み catalog をスキャンし、ユーザーに提示する：

```
[Phase 0 §X.1: Catalog 状態]
現在永続化済みの catalog:
- vb-net-winforms.md (last updated 2026-04-25)
- python-go.md (last updated 2026-04-10)

今回の変換対象に該当する catalog: vb-net-winforms.md
再利用しますか？ [Y/n]
```

- 再利用なら: catalog を Phase 0 §6 / Phase 1 へ取り込み
- 再生成なら: §X.2 へ進む

### X.2 Catalog 生成と 2 段階承認

**1 段階目: 内容承認（仕組み 4）**

subagent が `references/catalogs/generation-subagent-prompt.md` の Tier 分類クライテリアに従って catalog ドラフトを生成。生成後にユーザーが Tier 分類・項目内容をレビュー・承認する。承認後は Phase 0 で利用（メモリ上）。

**2 段階目: 永続化承認（NEW）**

内容承認後、ユーザーに永続化判断を確認:

```
[Catalog 永続化確認]
今回生成した catalog を永続化しますか？

□ 永続化する → references/catalogs/<lang>-<framework>.md に保存 + INDEX.md 自動更新
□ 今回限り → メモリのみ、Phase 5 完了後に破棄
```

詳細は `references/catalog-system.md` 参照。
```

- [ ] **Step A.2.2: Phase overview 図に Catalog ステップを追記**

冒頭の Phase overview の Phase 0 説明を以下に変更:

```
Phase 0: Scope Declaration          Profile 宣言、Catalog 状態確認・生成・2段階承認、
                                     conversion-rule tables、scope-strip list、
                                     approved-additions register 初期化、
                                     evaluation-scenario 骨子（Tier 連動）
```

- [ ] **Step A.2.3: 整合性確認**

Run: `grep -n "X.1 Catalog 状態確認" skills/skill-conversion/SKILL.md`
Expected: 新規節が見える

Run: `grep -n "X.2 Catalog 生成と 2 段階承認" skills/skill-conversion/SKILL.md`
Expected: 新規節が見える

- [ ] **Step A.2.4: Commit**

```bash
git add skills/skill-conversion/SKILL.md
git commit -m "feat(skill): Phase 0 に Catalog 状態確認・生成・2 段階承認の §X を追加"
```

### Task A.3: references/approved-additions.md に 9 マス承認テーブルとカテゴリ A/B/C を追加

**Files:**
- Modify: `skills/skill-conversion/references/approved-additions.md`

- [ ] **Step A.3.1: 既存ファイルを Read**

```bash
cat skills/skill-conversion/references/approved-additions.md
```

- [ ] **Step A.3.2: 「## Schema」の前に「## Category（v1.0 NEW）」を挿入**

```markdown
## Category（v1.0 NEW）

承認判定の二軸の一つ。追加内容の**性質**を分類する。

| カテゴリ | 性質 | 判定基準 |
|---|---|---|
| **A. 忠実度維持** | ソース忠実度を維持するための補正 | ソース内容を保持できないコンパイラ/ランタイム制約への対応、構文等価性確保 |
| **B. 環境必須** | ターゲット環境で事実上必須な基本機能 | Catalog Tier 1+2 に該当。「これが抜けるとスキルが不完全」と一般認知される |
| **C. 差別化機能** | ターゲット環境特有の差別化機能 | Catalog Tier 3 に該当。Tier 2 ほど普及していない、または特定ユースケース向け |

カテゴリは subagent または main agent が判定し、register に記録する。
```

- [ ] **Step A.3.3: 「## Schema」の表に Category 列を追加**

| Column | Required | Description |
|---|---|---|
| Added-on | yes | Absolute date (YYYY-MM-DD) |
| Location | yes | Section / heading / line anchor in the target |
| Summary | yes | One-line gist of the addition |
| **Category** | **yes（v1.0 NEW）** | **A / B / C のいずれか** |
| Reason | yes | Why this addition exists |
| Source-phase | yes | `Phase 1` or `Phase 4` |
| Impact | yes | `minor` / `moderate` / `heavy` |
| Approver | yes | User identifier who approved it |

- [ ] **Step A.3.4: 「## 9 マス承認テーブル（v1.0 NEW）」セクションを末尾の前に新規追加**

`## Anti-patterns` セクションの直前に以下を挿入:

```markdown
## 9 マス承認テーブル（v1.0 NEW）

Category × Profile による自動判定マトリクス。Impact (minor/moderate/heavy) と組み合わせて 27 セル。

| カテゴリ × Impact | 高忠実度 | バランス | 高有用性 |
|---|---|---|---|
| A. 忠実度維持・minor | 自動承認 | 自動承認 | 自動承認 |
| A. 忠実度維持・moderate | 自動承認 | 自動承認 | 自動承認 |
| A. 忠実度維持・heavy | user 承認必須 | user 承認必須 | user 承認必須 |
| B. 環境必須・minor | user 承認必須 | 自動承認 | 自動承認 |
| B. 環境必須・moderate | **拒否（原則）** | user 承認必須 | 自動承認 |
| B. 環境必須・heavy | **拒否** | user 承認必須 | user 承認必須 |
| C. 差別化機能・minor | **拒否** | user 承認必須 | 自動承認 |
| C. 差別化機能・moderate | **拒否** | **拒否** | user 承認必須 |
| C. 差別化機能・heavy | **拒否** | **拒否** | user 承認必須 |

### 各セル状態の挙動

- **自動承認**: subagent または main agent が判定後、承認形式で register に追記。user 承認は不要だが事後 review 可能
- **user 承認必須**: 承認フォーム（subject / detail / rationale / impact / category / verdict）を user に提示。承認後に register 追記
- **拒否（原則）**: profile の意図に反するため**取り込まない**。register には登録せず、drift-report の「Profile 拒否提案」セクションに記載

### 拒否された case の扱い

profile が「拒否」と判定した追加は、Phase 4 blind eval などで「ターゲット読者にとって有用」と subagent が報告したとしても、**取り込まれない**。これは profile 宣言の「期待値」を尊重するため。

ただし、user が profile を変えたい場合は **Phase 0 §7 で profile を再宣言可能**（変換工程の最初からやり直し）。
```

- [ ] **Step A.3.5: 整合性確認**

Run: `grep -nc "拒否" skills/skill-conversion/references/approved-additions.md`
Expected: 7 件以上（9 マス表 + 拒否 case 説明）

Run: `grep -n "## Category" skills/skill-conversion/references/approved-additions.md`
Expected: 1 件

- [ ] **Step A.3.6: Commit**

```bash
git add skills/skill-conversion/references/approved-additions.md
git commit -m "feat(approved-additions): Category A/B/C と 9 マス承認テーブルを追加"
```

### Task A.4: references/verification-stages.md に Phase 3 B2 出典確認サブステップ追加

**Files:**
- Modify: `skills/skill-conversion/references/verification-stages.md`

- [ ] **Step A.4.1: 既存の Phase 3 セクションを Read**

```bash
grep -n "## Phase 3" skills/skill-conversion/references/verification-stages.md
```

- [ ] **Step A.4.2: Phase 3 末尾の「Both categories need an audit trail.」段落の直後に「### Sub-step: 出典確認（B2、v1.0 NEW）」を挿入**

```markdown
### Sub-step: 出典確認（v1.0 NEW）

Phase 3 のビルド PASS 後、採用された API（特に scaffolding fix で導入された API）について、**公式性レベル**を 4 段階で記録する。

#### 公式性レベル

| レベル | 定義 |
|---|---|
| **Official** | Microsoft Learn / 公式リファレンス / 言語仕様書に記載がある |
| **Standard** | 主要書籍・公式チュートリアル・主要 OSS のドキュメントで記述がある |
| **Peripheral** | GitHub README / コミュニティ wiki / ブログのみ |
| **Unknown** | 出典未確認 |

#### 「両パターン併記」発動条件

採用 API の公式性が **Peripheral または Unknown** の場合、SKILL.md に**強制的に両パターン併記**する：

- 標準パターン（より公式性の高い API）の記述
- 採用 API の記述 + 利用条件・制約注記

#### Phase 3 結果報告のフォーマット拡張

既存 PASS/FAIL 報告に加え、以下を追加：

```
## 採用 API 一覧

| API | 公式性 | Package | 出典 URL |
|---|---|---|---|
| AddOptionsWithValidateOnStart(Of T) | Peripheral | Microsoft.Extensions.Hosting (≥8.0) | <url-if-any> |
| AddSingleton(Of T) | Official | Microsoft.Extensions.DependencyInjection | <url> |

## 両パターン併記必要な API
- AddOptionsWithValidateOnStart → SKILL.md §AddEmailServices に標準チェーン併記必須
```

#### 出典確認の実施方法

- subagent が catalog 生成時と類似した手法で公式性を判定
- 不明な場合は Unknown として記録（憶測しない）
- ユーザーが出典 URL を補強する場合は drift-report に追記
```

- [ ] **Step A.4.3: 整合性確認**

Run: `grep -n "出典確認" skills/skill-conversion/references/verification-stages.md`
Expected: B2 セクションが見える

Run: `grep -n "公式性" skills/skill-conversion/references/verification-stages.md`
Expected: 公式性レベル定義が見える

- [ ] **Step A.4.4: Commit**

```bash
git add skills/skill-conversion/references/verification-stages.md
git commit -m "feat(verification-stages): Phase 3 に B2 出典確認サブステップを追加"
```

### Task A.5: templates/plan-doc-template.md を改訂

**Files:**
- Modify: `skills/skill-conversion/references/templates/plan-doc-template.md`

- [ ] **Step A.5.1: 既存テンプレートを Read**

```bash
cat skills/skill-conversion/references/templates/plan-doc-template.md
```

- [ ] **Step A.5.2: §6 と §7 のテンプレ部を以下のように改訂**

§6 (Evaluation-scenario skeleton) の Scenario 1 の Requirement checklist を以下に変更:

```markdown
- Requirement checklist:
  - [ ] [critical] [Tier 1] <ソース忠実度の延長として必須項目>
  - [ ] [Tier 1] <主要機能カバレッジ項目>
  - [ ] [Tier 2] <balanced 以上で組み込む項目>
  - [ ] [Tier 3] <high-utility のみ組み込む項目（任意）>
```

§7 を以下に置換:

```markdown
### 7. Conversion Profile 宣言

- [ ] 高忠実度（high-fidelity）
- [x] バランス（balanced）  ← デフォルト
- [ ] 高有用性（high-utility）

§7.1 高忠実度 + Phase 4 opt-out の場合の理由（必要時のみ記載）:
> （理由）
```

§ X.1 / §X.2 のセクションを §7 の直後に追加:

```markdown
### X.1 Catalog 状態（v1.0 NEW）

- 永続化済み catalog 一覧確認結果: <list / なし>
- 該当組み合わせの再利用可否: 再利用 / 新規生成

### X.2 Catalog 内容（v1.0 NEW）

（生成 / 再利用された catalog の path と内容要約をここに記載）

- catalog: `<path or "memory only">`
- 永続化承認: <yes / no>
- 期待機能 Tier 範囲（profile 連動）: Tier 1 / Tier 1+2 / Tier 1+2+3
```

- [ ] **Step A.5.3: 整合性確認**

Run: `grep -n "Conversion Profile 宣言" skills/skill-conversion/references/templates/plan-doc-template.md`
Expected: 1 件

Run: `grep -nc "\[Tier" skills/skill-conversion/references/templates/plan-doc-template.md`
Expected: 4 件以上

- [ ] **Step A.5.4: Commit**

```bash
git add skills/skill-conversion/references/templates/plan-doc-template.md
git commit -m "feat(template): plan-doc-template に Profile 宣言と Catalog セクションを追加"
```

### Task A.6: templates/drift-report-template.md を改訂

**Files:**
- Modify: `skills/skill-conversion/references/templates/drift-report-template.md`

- [ ] **Step A.6.1: 既存テンプレートを Read**

```bash
cat skills/skill-conversion/references/templates/drift-report-template.md
```

- [ ] **Step A.6.2: 冒頭メタデータに Profile 宣言を追加**

```markdown
**Conversion Profile:** 高忠実度 / バランス / 高有用性 のいずれか
**Phase 4 opt-in 状態:** in / out（高忠実度のみ out 可、理由は §7.1 参照）
```

- [ ] **Step A.6.3: 「## Frontmatter verification」の前に「## 使用 Catalog（v1.0 NEW）」を追加**

```markdown
## 使用 Catalog（v1.0 NEW）

| Catalog | path | 永続化 status | 最終更新日 | 参照 Tier 範囲 |
|---|---|---|---|---|
| <name> | references/catalogs/<file>.md | persistent / memory-only | YYYY-MM-DD | Tier 1 / Tier 1+2 / Tier 1+2+3 |
```

- [ ] **Step A.6.4: 「## Phase 4 rejected proposals」の前に「## Phase 3 採用 API 一覧（v1.0 NEW）」と「## Profile 拒否提案（v1.0 NEW）」を追加**

```markdown
## Phase 3 採用 API 一覧（v1.0 NEW）

| API | 公式性 | Package | 両パターン併記実施 | 出典 URL |
|---|---|---|---|---|
| <api> | Official / Standard / Peripheral / Unknown | <pkg> | yes / no / N/A | <url> |

## Profile 拒否提案（v1.0 NEW）

profile によって自動的に拒否された追加（取り込まれていない）。

| Date | Location | Proposal summary | Category | Impact | 拒否理由 |
|---|---|---|---|---|---|
| YYYY-MM-DD | <loc> | <summary> | A / B / C | minor/moderate/heavy | profile=<X> によりカテゴリ <Y> を拒否 |
```

- [ ] **Step A.6.5: 整合性確認**

Run: `grep -n "使用 Catalog" skills/skill-conversion/references/templates/drift-report-template.md`
Expected: 1 件

Run: `grep -n "Phase 3 採用 API" skills/skill-conversion/references/templates/drift-report-template.md`
Expected: 1 件

- [ ] **Step A.6.6: Commit**

```bash
git add skills/skill-conversion/references/templates/drift-report-template.md
git commit -m "feat(template): drift-report-template に Profile / Catalog / 採用 API / 拒否提案セクションを追加"
```

### Task A.7: templates/register-template.md に Category 列を追加

**Files:**
- Modify: `skills/skill-conversion/references/templates/register-template.md`

- [ ] **Step A.7.1: 既存テンプレートを Read**

```bash
cat skills/skill-conversion/references/templates/register-template.md
```

- [ ] **Step A.7.2: 表に Category 列を追加**

既存の register 表のヘッダ行に `Category` 列を追加:

```markdown
| Added-on | Location | Summary | Category | Reason | Source-phase | Impact | Approver |
|---|---|---|---|---|---|---|---|
```

「## Column definitions」に Category の説明を追加:

```markdown
- **Category:** A (忠実度維持) / B (環境必須) / C (差別化機能) — `references/approved-additions.md` § Category 参照
```

- [ ] **Step A.7.3: 整合性確認**

Run: `grep -n "Category" skills/skill-conversion/references/templates/register-template.md`
Expected: 2 件以上（表ヘッダ + Column definition）

- [ ] **Step A.7.4: Commit**

```bash
git add skills/skill-conversion/references/templates/register-template.md
git commit -m "feat(template): register-template に Category 列を追加"
```

---

## Phase I-B: Catalog インフラ整備

### Task B.1: references/catalogs/ ディレクトリと catalog-template.md を新設

**Files:**
- Create directory: `skills/skill-conversion/references/catalogs/`
- Create: `skills/skill-conversion/references/catalogs/catalog-template.md`

- [ ] **Step B.1.1: ディレクトリ作成**

```bash
mkdir -p "skills/skill-conversion/references/catalogs"
```

- [ ] **Step B.1.2: catalog-template.md を作成**

ファイル内容:

````markdown
# <Lang> <Framework> — Conversion Catalog

> このファイルは catalog-template.md から生成された skill-conversion v1.0+ の catalog です。
> `references/catalog-system.md` の Tier 分類クライテリアに従って記述されています。

**生成日:** YYYY-MM-DD
**Source spec:** <path-to-skill-conversion-spec>
**永続化承認:** YYYY-MM-DD by <approver>

---

## 期待機能（Expected Capabilities）

ターゲット読者がこのスキルから当然得られると期待する機能のリスト。
profile に応じた Tier 範囲が Phase 0 §6 scenario 骨子作成時に必読。

### Tier 1（必須 — どの profile でも欠落させない）

ソース忠実度の延長として認識される基本機能。ソースが扱う機能のターゲット環境等価物。

- <feature 1>
- <feature 2>

### Tier 2（balanced 以上で推奨）

ソースに対応がないが、ターゲット環境で**事実上必須**な基本機能。
公式ドキュメント・主要書籍・チュートリアルで「推奨される基本パターン」として広く記述されているもの。

- <feature 3>
- <feature 4>

### Tier 3（high-utility で推奨）

ターゲット環境特有の**差別化機能・高度な機能**。
Tier 2 ほど普及していない、または特定ユースケース向け。

- <feature 5>
- <feature 6>

---

## 罠（Gotchas）

ターゲット環境固有のコンパイラ/ランタイム制約。Phase 1〜2 の §3b no-equivalent 表に自動マージされる。

### 言語制約

- <restriction 1>: <error code or note> → <代替方針>

### 型制約

- <restriction 2>: <error code or note> → <代替方針>

### 構文制約

- <restriction 3>: <error code or note> → <代替方針>

### API 制約

- <restriction 4>: <error code or note> → <代替方針>
````

- [ ] **Step B.1.3: 整合性確認**

Run: `ls skills/skill-conversion/references/catalogs/`
Expected: catalog-template.md

Run: `grep -nc "Tier" skills/skill-conversion/references/catalogs/catalog-template.md`
Expected: 6 件以上（Tier 1/2/3 の見出しと参照）

- [ ] **Step B.1.4: Commit**

```bash
git add skills/skill-conversion/references/catalogs/
git commit -m "feat(catalogs): catalogs/ ディレクトリと catalog-template.md を新設"
```

### Task B.2: references/catalogs/INDEX.md を新設（初期空）

**Files:**
- Create: `skills/skill-conversion/references/catalogs/INDEX.md`

- [ ] **Step B.2.1: INDEX.md を作成**

ファイル内容:

````markdown
# Catalog INDEX

このファイルは skill-conversion v1.0+ の永続化済み catalog 一覧です。
新規 catalog が永続化されるたびに自動更新されます。

## 永続化済み catalog 一覧

| Catalog | 対象組み合わせ | 最終更新日 | 用途要約 |
|---|---|---|---|
| (なし) | — | — | — |

---

## メンテナンス

- 永続化承認時に上記表に行が追加される
- catalog 削除時は当該行を削除（手動）
- 大幅改訂時は最終更新日を更新

## Format

- **Catalog**: ファイル名（例: `vb-net-winforms.md`）
- **対象組み合わせ**: lang + framework（例: VB.NET + WinForms）
- **最終更新日**: YYYY-MM-DD
- **用途要約**: 1 行でカバー範囲を要約
````

- [ ] **Step B.2.2: 整合性確認**

Run: `cat skills/skill-conversion/references/catalogs/INDEX.md`
Expected: 「(なし)」が表に含まれる初期状態

- [ ] **Step B.2.3: Commit**

```bash
git add skills/skill-conversion/references/catalogs/INDEX.md
git commit -m "feat(catalogs): INDEX.md を新設（初期空）"
```

### Task B.3: references/catalogs/generation-subagent-prompt.md を新設

**Files:**
- Create: `skills/skill-conversion/references/catalogs/generation-subagent-prompt.md`

- [ ] **Step B.3.1: generation-subagent-prompt.md を作成**

ファイル内容（subagent への指示書として使われる文書）:

````markdown
# Catalog Generation Subagent Prompt

> この文書は skill-conversion v1.0+ で catalog 生成時に subagent に渡される指示書です。
> 仕組み 1（明確な分類クライテリア）に対応します。

## あなたのタスク

ターゲット locale + framework の組み合わせに対する **Catalog** を生成してください。
catalog は `references/catalogs/catalog-template.md` の構造に従い、2 セクション（期待機能 + 罠）で構成します。

## Tier 分類クライテリア（厳密）

### Tier 1（必須）

判定基準: **ソース章立てに対応する箇所がある**、または「ソースが扱う機能のターゲット環境等価物」。

例: ソース C# WinForms スキルが「データバインディング」を扱うなら、VB.NET WinForms catalog の Tier 1 に「BindingSource、INotifyPropertyChanged」が入る。

### Tier 2（balanced 以上で推奨）

判定基準: ソースに対応がないが、**以下の条件のうち 2 つ以上を満たす**:

1. ターゲット環境の**公式ドキュメント**で「推奨される基本パターン」として記述
2. **主要書籍**（O'Reilly、Microsoft Press、Apress 等）で記述
3. **公式チュートリアル**（Microsoft Learn、MDN 等）で記述
4. ターゲット環境の主要 OSS の README で標準パターンとして紹介

例: VB.NET WinForms 環境で「UI スレッドマーシャリング」「IProgress(Of T)」「DI フォーム解決」は Tier 2。

### Tier 3（high-utility で推奨）

判定基準: ターゲット環境特有の**差別化機能・高度な機能**。Tier 2 の条件を満たさないが、専門書や advanced ドキュメントで紹介される機能。

例: VB.NET WinForms 環境で「PerMonitorV2 高 DPI」「Application.DoEvents 代替の async timer」は Tier 3。

## 罠（Gotchas）の記述方針

ターゲット環境固有のコンパイラ/ランタイム制約を 4 区分（言語制約・型制約・構文制約・API 制約）で列挙。

各エントリは以下の形式:
- 制約の説明
- エラーコード（あれば）
- 代替方針（同じ意図を達成する別パターン）

## 禁止事項

- **憶測で Tier 2/3 を分類しない**: 公式ドキュメントや主要書籍に記載がない機能は Tier 3 にも入れず、catalog から除外する
- **数を稼ぐために細分化しない**: 1 つの機能を 5 個に分割するより、まとめて 1 エントリにする
- **ソース固有の機能を Tier 1+ に入れない**: ソースが特殊な拡張をしている場合、それは Tier 1 ではなく Tier 2/3 で判定

## Output

ユーザーに `references/catalogs/<lang>-<framework>.md` 形式でドラフトを提示してください。
ユーザーは Tier 分類・項目内容をレビューし、必要に応じて修正・承認します（仕組み 4: 内容承認ゲート）。
````

- [ ] **Step B.3.2: 整合性確認**

Run: `grep -nc "Tier" skills/skill-conversion/references/catalogs/generation-subagent-prompt.md`
Expected: 8 件以上

Run: `grep -n "禁止事項" skills/skill-conversion/references/catalogs/generation-subagent-prompt.md`
Expected: 1 件

- [ ] **Step B.3.3: Commit**

```bash
git add skills/skill-conversion/references/catalogs/generation-subagent-prompt.md
git commit -m "feat(catalogs): catalog 生成 subagent prompt 文書を新設"
```

### Task B.4: references/catalog-system.md を新設（生成・承認・連動仕様の集約）

**Files:**
- Create: `skills/skill-conversion/references/catalog-system.md`

- [ ] **Step B.4.1: catalog-system.md を作成**

ファイル内容:

````markdown
# Catalog System（v1.0 仕様）

> Catalog の生成・承認・永続化・Scenario との連動を集約した仕様書。
> SKILL.md §X.1 / §X.2 の実装根拠です。

## 構造

各 catalog ファイル (`references/catalogs/<lang>-<framework>.md`) は **2 セクション固定**:

- 期待機能（Expected Capabilities）— Tier 1/2/3 の 3 段階
- 罠（Gotchas）— 全 profile で必読

詳細は `references/catalogs/catalog-template.md` を参照。

## Tier 分類クライテリア

`references/catalogs/generation-subagent-prompt.md` を参照（catalog 生成時の subagent 指示書）。

## 生成・承認フロー（W1' = 仕組み 1 + 4 + 5）

### Phase 0 §X.1: Catalog 状態確認

1. main agent が `references/catalogs/INDEX.md` を Read
2. 該当組み合わせ（lang + framework）が永続化済みか判定
3. ユーザーに提示し、再利用 / 再生成を選択させる
4. 再利用 → 既存 catalog を Phase 0 §6 に取り込み（生成スキップ）
5. 再生成 → §X.2 へ

### Phase 0 §X.2: Catalog 生成と 2 段階承認

#### 1 段階目: 内容承認（仕組み 4）

1. subagent dispatch（model: sonnet 推奨）。`references/catalogs/generation-subagent-prompt.md` を渡す
2. subagent が catalog ドラフトを生成（ファイル形式は catalog-template.md に準拠）
3. user に提示。Tier 分類・項目内容をレビュー
4. 修正後または変更なしで承認 → Phase 0 §6 で利用（メモリ上）

#### 2 段階目: 永続化承認（NEW）

1. main agent が user に永続化判断を確認
2. 永続化 yes → `references/catalogs/<lang>-<framework>.md` に保存 + INDEX.md に行追加
3. 永続化 no → メモリのみ、Phase 5 完了後に破棄
4. drift-report 「使用 Catalog」セクションに status を記録

## profile 連動の参照範囲

| Profile | Catalog 参照量 |
|---|---|
| 高忠実度 | 罠 + 期待機能 Tier 1 |
| バランス | 罠 + 期待機能 Tier 1 + Tier 2 |
| 高有用性 | 罠 + 期待機能 Tier 1 + Tier 2 + Tier 3 |

「罠」セクションは profile 不問で全件参照。

## Catalog ↔ Scenario の連動

### Phase 0 §6 scenario 骨子作成時

1. main agent が catalog の「期待機能」を Read（profile に応じた Tier 範囲）
2. 各 scenario の Requirement checklist に対象 Tier 範囲のアイテムを盛り込む
3. 各 checklist アイテムに `[Tier N]` タグを付与（catalog 参照点を明示）

### Phase 4 blind eval scorer

1. blind subagent が scenario を実行し、回答を生成
2. scorer（main agent または独立 subagent）が以下を機械的に検証:
   - 既存の `[critical]` タグ checklist のカバー（v0.2 仕様）
   - 新規 `[Tier N]` タグ checklist のカバー（v1.0 NEW）
3. Tier タグでカバー漏れがあれば、追加 scenario を提案 or approved-additions に登録候補として上申

## Phase 1〜2 の罠自動マージ

### §3b no-equivalent 表との連動

Phase 0 で取り込まれた catalog の「罠」セクションは、Phase 1〜2 の §3b no-equivalent 表に**自動マージ**される。手動で個別管理しない。

具体的には、main agent が conversion-rule tables 構築時に catalog 罠を §3b エントリとして取り込む。重複した場合は catalog 側を優先する。

## INDEX.md 自動メンテ

永続化承認時、main agent は以下を自動実行:

1. `references/catalogs/INDEX.md` の表に新規行を追加
2. 「Catalog / 対象組み合わせ / 最終更新日 / 用途要約」を埋める
3. 用途要約は subagent 生成時の summary を流用 or main agent が 1 行で要約

ファイル削除時は手動メンテ（user が drift-report で記録）。
````

- [ ] **Step B.4.2: 整合性確認**

Run: `grep -nc "Tier" skills/skill-conversion/references/catalog-system.md`
Expected: 10 件以上

Run: `grep -n "INDEX.md" skills/skill-conversion/references/catalog-system.md`
Expected: 4 件以上

- [ ] **Step B.4.3: Commit**

```bash
git add skills/skill-conversion/references/catalog-system.md
git commit -m "feat(catalog-system): Catalog 生成・承認・連動仕様を集約"
```

---

## Phase I-C: ロジック仕様文書化

### Task C.1: references/profile-system.md を新設（Profile 仕様の集約）

**Files:**
- Create: `skills/skill-conversion/references/profile-system.md`

- [ ] **Step C.1.1: profile-system.md を作成**

ファイル内容:

````markdown
# Conversion Profile System（v1.0 仕様）

> ユーザーが Phase 0 §7 で宣言する「変換の期待値」と、それが Phase 制御に与える影響を集約した仕様書。

## 3 段階 Profile

| Profile | 意味 |
|---|---|
| **高忠実度** (high-fidelity) | ソース忠実度最優先。ターゲット環境固有の付加価値より、ソース著者の意図保持を優先 |
| **バランス** (balanced) ⭐ デフォルト | 忠実度 + 有用性の両立。ターゲット環境の事実上必須機能までカバー |
| **高有用性** (high-utility) | ターゲット読者の有用性最優先。ターゲット環境特有の差別化機能まで取り込む |

## 4 軸の自動連動

profile 宣言が以下を機械的に決定する。

| 軸 | 高忠実度 | バランス | 高有用性 |
|---|---|---|---|
| **Phase 4** | デフォルト on、opt-out 可（理由必須） | 強制 on | 強制 on |
| **approved-additions 閾値** | 厳格（A のみ） | 標準（A + B） | 寛容（A + B + C） |
| **Catalog 参照量** | 罠 + Tier 1 | 罠 + Tier 1 + Tier 2 | 罠 + Tier 1 + Tier 2 + Tier 3 |
| **Phase 2 fixpoint 収束基準** | 常に高 stakes（profile 不問） | 同左 | 同左 |

詳細は以下を参照:
- approved-additions 閾値: `references/approved-additions.md` § 9 マス承認テーブル
- Catalog 参照量: `references/catalog-system.md` § profile 連動の参照範囲
- Phase 2 fixpoint: `references/fixpoint-loop.md`

## Phase 4 opt-out の運用（高忠実度のみ）

### 高忠実度で opt-out する場合

1. Phase 0 §7 で「高忠実度」を選択
2. §7.1 に**理由を必須記載**
3. drift-report 冒頭に `Phase 4: SKIPPED by user choice (high-fidelity profile, reason: <理由>)` を明示

### バランス・高有用性で opt-out を試みた場合

1. **halt** する
2. user に再選択を求める:
   - profile を高忠実度に変更（opt-out 可になる）
   - Phase 4 を実行する（強制 on を受け入れる）
3. このロジックは main agent が Phase 0 §7 完了時に判定

## デフォルトと省略時の挙動

- **省略時:** バランス自動採用、warning ログを出力
- **不明な値:** halt して再宣言を求める（タイポ防止）

## profile 宣言の保存場所

Phase 0 plan-doc の §7 セクションに記載される。後続の各 Phase は plan-doc を参照して profile を読み取る。

## profile 変更（mid-conversion）

進行中の変換で profile を変更したい場合:

1. Phase 0 §7 で再宣言（plan-doc を編集）
2. 変換工程を最初から（Phase 0 §6 scenario 骨子の作り直しを含む）やり直す必要がある
3. 部分的な再評価は許容しない（profile が一貫しなくなる）
````

- [ ] **Step C.1.2: 整合性確認**

Run: `grep -nc "Profile" skills/skill-conversion/references/profile-system.md`
Expected: 10 件以上

Run: `grep -n "halt" skills/skill-conversion/references/profile-system.md`
Expected: 2 件以上（opt-out halt + 不明値 halt）

- [ ] **Step C.1.3: Commit**

```bash
git add skills/skill-conversion/references/profile-system.md
git commit -m "feat(profile-system): Conversion Profile 仕様と 4 軸連動を集約"
```

### Task C.2: references/fixpoint-loop.md を軽微更新

**Files:**
- Modify: `skills/skill-conversion/references/fixpoint-loop.md`

- [ ] **Step C.2.1: 既存ファイルを Read**

```bash
grep -n "high-stakes\|high stakes\|高 stakes" skills/skill-conversion/references/fixpoint-loop.md
```

- [ ] **Step C.2.2: 「2 consecutive zero-change passes」セクション付近に v1.0 注記を追加**

該当箇所の直後に以下を追加:

```markdown
### v1.0 における収束基準

v1.0 では **profile 不問で常に高 stakes**（2 回連続ゼロ変更）を採用する。これは Conversion Profile 設計上、品質ガードレールを下げるべきでないという判断に基づく。

profile による差は Phase 2 fixpoint ではなく、Phase 4 opt-in 制御・approved-additions 閾値・Catalog 参照量で表現される。

詳細は `references/profile-system.md` を参照。
```

- [ ] **Step C.2.3: 整合性確認**

Run: `grep -n "v1.0 における収束基準" skills/skill-conversion/references/fixpoint-loop.md`
Expected: 1 件

- [ ] **Step C.2.4: Commit**

```bash
git add skills/skill-conversion/references/fixpoint-loop.md
git commit -m "docs(fixpoint-loop): v1.0 で profile 不問・常に高 stakes である旨を明示"
```

### Task C.3: references/conversion-dimensions.md を軽微更新

**Files:**
- Modify: `skills/skill-conversion/references/conversion-dimensions.md`

- [ ] **Step C.3.1: 既存ファイルを Read（変更箇所の特定）**

```bash
cat skills/skill-conversion/references/conversion-dimensions.md
```

- [ ] **Step C.3.2: 末尾に v1.0 補足を追加**

ファイル末尾に以下を追加:

```markdown
---

## v1.0 補足: Profile との関係

本ファイルが扱う dimension（Technical / Locale / Agent）は変換の**形式的な記述**であり、品質期待値を表す Conversion Profile（高忠実度 / バランス / 高有用性）とは独立した概念です。

例: 「C# → VB.NET」（Technical dimension）の変換でも、profile を高忠実度 / バランス / 高有用性 のいずれにするかは別途宣言する。

詳細は `references/profile-system.md` を参照。
```

- [ ] **Step C.3.3: 整合性確認**

Run: `grep -n "v1.0 補足" skills/skill-conversion/references/conversion-dimensions.md`
Expected: 1 件

- [ ] **Step C.3.4: Commit**

```bash
git add skills/skill-conversion/references/conversion-dimensions.md
git commit -m "docs(conversion-dimensions): Profile との独立性を補足"
```

### Task C.4: references/cross-agent-verification-menu.md を軽微更新

**Files:**
- Modify: `skills/skill-conversion/references/cross-agent-verification-menu.md`

- [ ] **Step C.4.1: 既存ファイルを Read**

```bash
grep -n "Phase 4" skills/skill-conversion/references/cross-agent-verification-menu.md | head -5
```

- [ ] **Step C.4.2: Phase 4 戦略セクションに v1.0 補足を追加**

該当セクションの末尾に以下を追加:

```markdown
### v1.0 補足

agent dimension が non-identity の場合、profile 連動の Phase 4 強制 on（バランス / 高有用性）と組み合わせると、cross-agent verification の戦略選択が必須となる。`references/profile-system.md` および本ファイルを併せて参照すること。

高忠実度 + agent non-identity + Phase 4 opt-out の場合、cross-agent 検証戦略は §3 / §5 のみ選択（§4 は対象外）。
```

- [ ] **Step C.4.3: 整合性確認**

Run: `grep -n "v1.0 補足" skills/skill-conversion/references/cross-agent-verification-menu.md`
Expected: 1 件

- [ ] **Step C.4.4: Commit**

```bash
git add skills/skill-conversion/references/cross-agent-verification-menu.md
git commit -m "docs(cross-agent-verification-menu): Profile との関係を補足"
```

---

## Phase I-D: ドキュメント更新

### Task D.1: README.md を新規作成

**Files:**
- Create: `README.md`

- [ ] **Step D.1.1: README.md を作成**

ファイル内容:

````markdown
# skill-conversion

Convert existing Claude Code skills across three dimensions — programming language/framework, natural-language locale, and target agent runtime — via a disciplined fixpoint review loop, approved-additions register, three-layer verification, and **Conversion Profile + Catalog system (v1.0)**.

## What's New in v1.0.0

- **Conversion Profile**: 3 段階（高忠実度 / バランス / 高有用性）で変換の期待値を宣言
- **Catalog System**: ターゲット環境特有の「期待機能」と「罠」を Tier 1/2/3 + 罠の構造で蓄積
- **9 マス承認テーブル**: Category × Profile による approved-additions 自動判定
- **Phase 3 出典確認**: 採用 API の公式性を 4 段階で評価し「両パターン併記」を発動

詳細は `RELEASE_NOTES.md` および `docs/specs/2026-04-25-skill-conversion-v1-design.md` を参照。

## Prerequisites

- `superpowers` プラグイン
- `empirical-prompt-tuning` プラグイン

## Installation

```
/plugin install superpowers
/plugin install empirical-prompt-tuning
/plugin marketplace add <path-to>/skill-conversion
/plugin install skill-conversion@skill-conversion-marketplace
```

## Usage

skill-conversion は既存スキルを別の言語・ロケール・agent に変換するスキルです。詳細は `skills/skill-conversion/SKILL.md` を参照。

## License

MIT — see `LICENSE`.

## Author

[oreguchi](https://github.com/oreguchi)
````

- [ ] **Step D.1.2: 整合性確認**

Run: `cat README.md | head -10`
Expected: タイトルと What's New セクションが見える

- [ ] **Step D.1.3: Commit**

```bash
git add README.md
git commit -m "docs(README): v1.0.0 向け README を新規作成"
```

### Task D.2: SKILL.md 冒頭の概要を v1.0 に合わせて更新

**Files:**
- Modify: `skills/skill-conversion/SKILL.md`

- [ ] **Step D.2.1: 既存 description / overview を Read**

```bash
head -50 skills/skill-conversion/SKILL.md
```

- [ ] **Step D.2.2: frontmatter description を更新**

冒頭の description フィールドを以下に変更:

```yaml
---
name: skill-conversion
description: "Convert an existing Claude Code skill (or other agent's skill) into a new variant by transforming one or more dimensions: programming language/framework (e.g., C# → VB.NET, React → Vue), natural language locale (e.g., EN → JA), or target agent (e.g., Claude Code → Copilot CLI / Gemini CLI / Codex). Any combination is supported. v1.0 では Conversion Profile（高忠実度 / バランス / 高有用性）と Catalog System によりターゲット有用性の検出を構造化。Use when asked to port, translate, adapt, or re-target an existing skill. Requires superpowers and empirical-prompt-tuning plugins."
---
```

- [ ] **Step D.2.3: Phase overview の Phase 0 行を v1.0 仕様に更新**

冒頭の `## Phase overview` 内、Phase 0 の説明を以下に置換（Task A.2.2 で既に実施済みなら確認のみ）:

```
Phase 0: Scope Declaration          Profile 宣言、Catalog 状態確認・生成・2段階承認、
                                     conversion-rule tables、scope-strip list、
                                     approved-additions register 初期化、
                                     evaluation-scenario 骨子（Tier 連動）
```

- [ ] **Step D.2.4: 整合性確認**

Run: `grep -n "v1.0" skills/skill-conversion/SKILL.md`
Expected: description および Phase 0 説明で v1.0 言及あり

- [ ] **Step D.2.5: Commit**

```bash
git add skills/skill-conversion/SKILL.md
git commit -m "docs(skill): SKILL.md 冒頭を v1.0 仕様に合わせて更新"
```

### Task D.3: migration guide を新規作成

**Files:**
- Create: `skills/skill-conversion/references/migration-v02-to-v10.md`

- [ ] **Step D.3.1: migration-v02-to-v10.md を作成**

ファイル内容:

````markdown
# Migration Guide: skill-conversion v0.2 → v1.0

> v0.2.0 で進行中の変換、または v0.2.0 の plan-doc を v1.0.0 で継続したいケースのためのガイド。

## 推奨方針

**進行中の変換は v0.2.0 仕様で完結する**ことを強く推奨します。理由:

- v1.0.0 で導入された Profile / Catalog の仕組みは、Phase 0 から取り込まれることを前提に設計されている
- 途中切替は §7 の意味変化（Phase 4 opt-in 判断 → Profile 宣言）等で混乱の元になる
- 既存の plan-doc 形式と v1.0 形式が部分的に互換性を持たない

## 完了済みの変換について

v0.2.0 で完了した変換成果物（drift-report、approved-additions register 等）は v1.0.0 でも引き続き参照可能です。**破壊的影響はありません**。

ただし以下の点に注意:

- v0.2.0 の register には Category 列がない → v1.0 表示時は「未分類」扱い
- 9 マス承認テーブルは v0.2.0 register には適用されない（過去の判定はそのまま）

## v1.0.0 への切替（必要な場合）

進行中の変換を v1.0.0 形式に切り替える場合:

### 1. plan-doc の §7 を Profile 宣言に書き換え

旧形式:

```markdown
### 7. Phase 4 opt-in decision
- [x] opt-in
- [ ] opt-out
```

新形式:

```markdown
### 7. Conversion Profile 宣言
- [ ] 高忠実度
- [x] バランス  ← 既存 opt-in を尊重するならバランス推奨
- [ ] 高有用性
```

### 2. Catalog 状態確認の追加

新規セクション §X.1 / §X.2 を §7 直後に追加。catalog 生成・2 段階承認を実施するか、catalog なし（profile=高忠実度の opt-out 等）で進めるか判断。

### 3. approved-additions register に Category 列を追加

既存 register エントリ全件に Category（A/B/C）を後付け。`references/approved-additions.md` の Category 定義に従って分類。

### 4. 9 マス承認テーブルの遡及適用

すでに承認済みのエントリは尊重する（変更なし）。今後の新規追加から 9 マステーブルを適用。

### 5. drift-report テンプレートの差分を反映

`references/templates/drift-report-template.md` に追加された以下のセクションを既存 drift-report に追記:

- 冒頭 Profile 宣言
- 使用 Catalog
- Phase 3 採用 API 一覧
- Profile 拒否提案

## サポート

切替時に問題があれば skill-conversion の Issue として報告してください。
````

- [ ] **Step D.3.2: 整合性確認**

Run: `grep -n "## " skills/skill-conversion/references/migration-v02-to-v10.md | head -10`
Expected: 主要セクションが見える

- [ ] **Step D.3.3: Commit**

```bash
git add skills/skill-conversion/references/migration-v02-to-v10.md
git commit -m "docs(migration): v0.2 → v1.0 移行ガイドを新規作成"
```

### Task D.4: RELEASE_NOTES.md を新規作成

**Files:**
- Create: `RELEASE_NOTES.md`

- [ ] **Step D.4.1: RELEASE_NOTES.md を作成**

ファイル内容:

````markdown
# Release Notes

## 1.0.0 — 2026-04-XX

**Major release: stable リリース**。Conversion Profile + Catalog System + Phase 制御の三位一体を導入。

### 新機能

- **Conversion Profile（3 段階）**: 高忠実度 / バランス / 高有用性。Phase 0 §7 で宣言
- **Catalog System**: ターゲット環境特有の「期待機能」（Tier 1/2/3）と「罠」を 2 セクションで蓄積。生成・2 段階承認（内容承認 + 永続化承認）・Scenario 連動
- **9 マス承認テーブル**: approved-additions の Category × Profile による自動判定
- **Phase 3 出典確認**: 採用 API の公式性を 4 段階（Official / Standard / Peripheral / Unknown）で評価し、Peripheral / Unknown 採用時は両パターン併記を必須化
- **drift-report 拡張**: Profile 宣言・使用 Catalog・採用 API 一覧・Profile 拒否提案セクション追加

### 破壊的変更

- Phase 0 §7 が「Phase 4 opt-in 判断」から「Conversion Profile 宣言」に変更
- approved-additions register に Category 列が追加（必須）
- Phase 4 opt-in/out 制御が profile 連動（高忠実度のみ opt-out 可、理由必須）
- Catalog 概念が新規導入（生成・承認・永続化のフローが追加）
- Phase 3 結果報告に「採用 API 一覧」が追加

詳細は `references/migration-v02-to-v10.md` を参照。

### スコープアウト（v1.x 以降の検討）

- Phase 4 baseline_variant 機能
- Catalog 生成の自動 PR 投稿（コミュニティ運用）
- 多言語 INDEX.md（日英併記等）

---

## 0.2.0

(初版に近い機能セット。Phase 0〜5 の基本フロー、approved-additions register、cross-agent verification menu を含む)
````

- [ ] **Step D.4.2: 整合性確認**

Run: `grep -n "1.0.0" RELEASE_NOTES.md`
Expected: 1 件以上

- [ ] **Step D.4.3: Commit**

```bash
git add RELEASE_NOTES.md
git commit -m "docs(release-notes): v1.0.0 セクションを追加"
```

### Task D.5: plugin.json と marketplace.json を更新

**Files:**
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`（あれば）

- [ ] **Step D.5.1: 既存 plugin.json を Read**

```bash
cat .claude-plugin/plugin.json
```

- [ ] **Step D.5.2: version と description を更新**

```json
{
  "name": "skill-conversion",
  "version": "1.0.0",
  "description": "Convert existing Claude Code skills across three dimensions — programming language/framework, natural-language locale, and target agent runtime — via a disciplined fixpoint review loop, approved-additions register, three-layer verification, and Conversion Profile + Catalog system (v1.0). Requires superpowers and empirical-prompt-tuning plugins.",
  "author": {
    "name": "oreguchi",
    "email": "228712732+oreguchi@users.noreply.github.com"
  },
  "license": "MIT"
}
```

- [ ] **Step D.5.3: marketplace.json があれば description を更新（同様の文言）**

```bash
test -f .claude-plugin/marketplace.json && cat .claude-plugin/marketplace.json
```

存在すれば description フィールドを v1.0 文言に更新。

- [ ] **Step D.5.4: 整合性確認**

Run: `grep '"version"' .claude-plugin/plugin.json`
Expected: `"version": "1.0.0"`

- [ ] **Step D.5.5: Commit**

```bash
git add .claude-plugin/
git commit -m "chore(plugin): version 1.0.0 + description を v1.0 仕様に更新"
```

---

## Phase I-E: Self-Verification

### Task E.1: design 矛盾の self-review

**Files:**
- Read only: `docs/specs/2026-04-25-skill-conversion-v1-design.md`
- Read only: `skills/skill-conversion/SKILL.md` および `references/*.md`

- [ ] **Step E.1.1: design doc を再読**

```bash
cat docs/specs/2026-04-25-skill-conversion-v1-design.md
```

- [ ] **Step E.1.2: 各 section の内容と実装ファイルを突き合わせる**

以下のチェックポイント:

1. design §2 (Profile 制御連動表) と `references/profile-system.md` の値が一致しているか
2. design §3.5 (Catalog ↔ Scenario 連動) と `references/catalog-system.md` の値が一致しているか
3. design §4.2 (9 マス承認テーブル) と `references/approved-additions.md` の表が一致しているか
4. design §5 (Phase 0〜5 への影響) と各 reference ファイルの記述が一致しているか
5. design §6 (後方互換性) と `references/migration-v02-to-v10.md` の記述が一致しているか

- [ ] **Step E.1.3: 矛盾を発見したら修正**

矛盾があった場合のみ inline 修正。すべて整合していれば次の Step。

- [ ] **Step E.1.4: 検証結果を記録**

新規ファイル `docs/plans/i-e-self-verification-result.md` を作成し、チェック結果を 1 件 1 行で記録:

```markdown
# I-E Self-Verification Result

**Date:** 2026-04-XX

| Check | Status | Notes |
|---|---|---|
| design §2 ↔ profile-system.md | PASS / FAIL | <notes> |
| design §3 ↔ catalog-system.md | PASS / FAIL | <notes> |
| design §4 ↔ approved-additions.md | PASS / FAIL | <notes> |
| design §5 ↔ 各 reference | PASS / FAIL | <notes> |
| design §6 ↔ migration-v02-to-v10.md | PASS / FAIL | <notes> |
```

- [ ] **Step E.1.5: Commit**

```bash
git add docs/plans/i-e-self-verification-result.md
git commit -m "verify(I-E): design ↔ implementation の self-review 完了"
```

### Task E.2: テンプレ consistency チェック

**Files:**
- Read only: `skills/skill-conversion/references/templates/*.md`

- [ ] **Step E.2.1: 全 template を再読**

```bash
ls skills/skill-conversion/references/templates/
for f in skills/skill-conversion/references/templates/*.md; do
  echo "=== $f ==="
  head -30 "$f"
done
```

- [ ] **Step E.2.2: 4 つのテンプレが相互に矛盾しないか確認**

チェックポイント:

1. plan-doc-template の §7 と drift-report-template の Profile 宣言行のフィールド名・値が一致
2. register-template の Category 列と approved-additions.md の Category 定義が一致
3. drift-report-template の「Phase 3 採用 API」フィールドと verification-stages.md の B2 出力フォーマットが一致
4. plan-doc-template の §X.1 / §X.2 と SKILL.md の §X.1 / §X.2 の節構造が一致

- [ ] **Step E.2.3: 矛盾発見時は inline 修正**

不整合があれば該当 template ファイルを修正してテスト再実行。

- [ ] **Step E.2.4: Commit（修正があった場合のみ）**

```bash
git add skills/skill-conversion/references/templates/
git commit -m "fix(template): consistency check で発見した不整合を修正"
```

### Task E.3: v0.2 完了済みドキュメントの v1.0 再評価

**Files:**
- Read only: `C:\dev\01_ClaudeProject\skill-conversion実践2\docs\plans\2026-04-23-vb-winforms-skills-conversion\drift-report.md`
- Read only: `C:\dev\01_ClaudeProject\skill-conversion実践2\docs\plans\2026-04-23-vb-winforms-skills-conversion\approved-additions.md`

- [ ] **Step E.3.1: vb-winforms-skills v2.0/2.1 の drift-report と approved-additions を Read**

```bash
cat "C:/dev/01_ClaudeProject/skill-conversion実践2/docs/plans/2026-04-23-vb-winforms-skills-conversion/drift-report.md"
cat "C:/dev/01_ClaudeProject/skill-conversion実践2/docs/plans/2026-04-23-vb-winforms-skills-conversion/approved-additions.md"
```

- [ ] **Step E.3.2: v1.0 仕様で再評価する仮想テストを実施**

仮想テスト:
- 仮に「バランス profile」で v2.0 を再変換していたら、9 マス表でどう判定されるか
- approved-additions の各エントリを A/B/C カテゴリに後付け分類（design §4.5 の実例マッピングを参考）
- 結論として、v1.0 で再変換しても drift verdict が「MODERATE」付近に収まるか

- [ ] **Step E.3.3: 検証結果を記録**

`docs/plans/i-e-3-v02-retrocompat-check.md` を新規作成:

```markdown
# I-E.3: v0.2 完了済みドキュメントの v1.0 再評価結果

**対象:** vb-winforms-skills v2.0.0 + v2.1.0 (累計 26 件の approved-additions)
**仮想 profile:** バランス
**実施日:** 2026-04-XX

## カテゴリ後付け分類

| エントリ | 元 Impact | 推定 Category | バランス profile での判定 |
|---|---|---|---|
| Span 全排除 | heavy | A | 承認 ✅ |
| `Sub(_)` → `Sub(state)` | minor | A | 自動承認 ✅ |
| WinForms テンプレ追加 | moderate | B | 承認 ✅ |
| UI スレッドマーシャリング統合 | heavy | B | 承認 ✅ |
| 高 DPI / フォーム Func ファクトリー | moderate | C | **拒否** ❌ |
| Akka.NET VB 注記 | minor | C | 承認 ✅ |
| ...（残りエントリも後付け） |  |  |  |

## drift verdict 比較

- v0.2 での verdict: MODERATE
- v1.0 仮想再変換での verdict: MODERATE (バランス profile で C 拒否があるが、heavy×2 を A で吸収できているため大幅変動なし)

## 結論

後方互換性は妥当。v0.2 完了済みドキュメントは v1.0 仕様で再評価しても drift verdict が大幅に変わらない。
```

- [ ] **Step E.3.4: Commit**

```bash
git add docs/plans/i-e-3-v02-retrocompat-check.md
git commit -m "verify(I-E.3): v0.2 完了済みドキュメントの v1.0 後方互換性検証完了"
```

---

## Phase I-F: Dogfooding（必須）

### Task F.1: 高忠実度 profile での試験変換

**Files:**
- Create: `docs/plans/dogfooding-fidelity/` ディレクトリ配下の plan-doc / drift-report 等

- [ ] **Step F.1.1: 試験対象を選定**

候補（小規模で完結する skill）:
- `superpowers:writing-skills` （EN → JA 変換）
- 公開済みの小規模 skill 1 つ

選定後、`docs/plans/dogfooding-fidelity/00-master-plan.md` を skill-conversion v1.0 を使って Phase 0 から開始。

- [ ] **Step F.1.2: Phase 0 で profile=高忠実度 を宣言**

plan-doc の §7 で:

```markdown
### 7. Conversion Profile 宣言
- [x] 高忠実度（high-fidelity）
- [ ] バランス
- [ ] 高有用性
```

Phase 4 を opt-out するなら §7.1 に理由を記載。

- [ ] **Step F.1.3: Phase 0 §X.1 / §X.2 で Catalog 生成（or skip）**

profile=高忠実度なので Catalog 参照量は「罠 + Tier 1」のみ。Catalog 生成は実施するが、Tier 2/3 は記述不要（高忠実度では参照されない）。

- [ ] **Step F.1.4: Phase 1〜5 を実行**

skill-conversion v1.0 の SKILL.md を入力に、各 Phase を進める。

- [ ] **Step F.1.5: 完了結果を記録**

`docs/plans/dogfooding-fidelity/dogfooding-result.md` に以下を記録:

- 変換対象
- 完了 Phase
- approved-additions register エントリ数（カテゴリ別、Impact 別）
- drift verdict
- 9 マステーブルで「拒否」されたエントリ数
- 発見した skill-conversion の不具合・改善案

- [ ] **Step F.1.6: Commit**

```bash
git add docs/plans/dogfooding-fidelity/
git commit -m "verify(I-F.1): 高忠実度 profile での dogfooding 完了"
```

### Task F.2: バランス profile での試験変換

**Files:**
- Create: `docs/plans/dogfooding-balanced/` 配下

- [ ] **Step F.2.1: 試験対象を選定**

F.1 と異なる対象を選定（同じ対象だと前回の影響を受ける）。

- [ ] **Step F.2.2: Phase 0 で profile=バランス を宣言**

```markdown
### 7. Conversion Profile 宣言
- [ ] 高忠実度
- [x] バランス（balanced）
- [ ] 高有用性
```

- [ ] **Step F.2.3: Phase 0 §X.1 / §X.2 で Catalog 生成**

profile=バランスなので Catalog 参照量は「罠 + Tier 1 + Tier 2」。Tier 2 まで記述すること。

- [ ] **Step F.2.4: Phase 1〜5 を実行**

- [ ] **Step F.2.5: 完了結果を記録（F.1 と同じフォーマット）**

`docs/plans/dogfooding-balanced/dogfooding-result.md` に記録。

- [ ] **Step F.2.6: Commit**

```bash
git add docs/plans/dogfooding-balanced/
git commit -m "verify(I-F.2): バランス profile での dogfooding 完了"
```

### Task F.3: 高有用性 profile での試験変換

**Files:**
- Create: `docs/plans/dogfooding-utility/` 配下

- [ ] **Step F.3.1: 試験対象を選定**

F.1 / F.2 と異なる対象を選定。

- [ ] **Step F.3.2: Phase 0 で profile=高有用性 を宣言**

```markdown
### 7. Conversion Profile 宣言
- [ ] 高忠実度
- [ ] バランス
- [x] 高有用性（high-utility）
```

- [ ] **Step F.3.3: Phase 0 §X.1 / §X.2 で Catalog 生成（Tier 1+2+3 まで）**

profile=高有用性なので Catalog 参照量は「罠 + Tier 1 + Tier 2 + Tier 3」。差別化機能まで記述。

- [ ] **Step F.3.4: Phase 1〜5 を実行**

- [ ] **Step F.3.5: 完了結果を記録**

`docs/plans/dogfooding-utility/dogfooding-result.md` に記録。

- [ ] **Step F.3.6: Commit**

```bash
git add docs/plans/dogfooding-utility/
git commit -m "verify(I-F.3): 高有用性 profile での dogfooding 完了"
```

### Task F.4: Dogfooding 結果の集約と微調整

**Files:**
- Create: `docs/plans/i-f-dogfooding-summary.md`

- [ ] **Step F.4.1: 3 件の dogfooding 結果を集約**

`docs/plans/i-f-dogfooding-summary.md` を作成:

```markdown
# I-F: Dogfooding 集約結果

**実施期間:** 2026-04-XX 〜 2026-04-XX

## 試験変換 3 件

| # | profile | 対象 | 完了 Phase | approved-additions 件数 | drift verdict | 拒否件数 |
|---|---|---|---|---|---|---|
| F.1 | 高忠実度 | <target> | 0-5 完了 | A:_, B:_, C:_ | <verdict> | _ 件 |
| F.2 | バランス | <target> | 0-5 完了 | A:_, B:_, C:_ | <verdict> | _ 件 |
| F.3 | 高有用性 | <target> | 0-5 完了 | A:_, B:_, C:_ | <verdict> | _ 件 |

## profile 連動の挙動差の実証

- 9 マステーブルでの拒否: 高忠実度 > バランス > 高有用性 の順で多い（期待通り）
- Catalog 参照量の差: Tier 1 / 1+2 / 1+2+3 の差が drift report に明示されているか
- Phase 4 opt-in: 高忠実度のみ opt-out 可、それ以外は強制 on

## 発見した skill-conversion の不具合・改善案

- <issue 1>
- <issue 2>
```

- [ ] **Step F.4.2: 必要に応じて skill-conversion 仕様を微調整**

dogfooding で発見した不具合があれば inline 修正。修正したら関連ファイルを再 commit。

- [ ] **Step F.4.3: Commit**

```bash
git add docs/plans/i-f-dogfooding-summary.md
git commit -m "verify(I-F.4): dogfooding 集約と微調整完了"
```

---

## Final Release

### Task R.1: v1.0.0 タグ付け + GitHub Release

**Files:**
- Tag: `v1.0.0`

- [ ] **Step R.1.1: 全コミットの確認**

```bash
git log --oneline | head -30
```

- [ ] **Step R.1.2: tag 作成**

```bash
git tag v1.0.0
git push origin main
git push origin v1.0.0
```

- [ ] **Step R.1.3: GitHub Release 作成**

```bash
gh release create v1.0.0 --title "v1.0.0" --notes-file RELEASE_NOTES.md
```

- [ ] **Step R.1.4: 完了確認**

```bash
gh release list --repo oreguchi/skill-conversion
```

Expected: v1.0.0 が Latest として表示

---

## 完了基準（design doc §8 参照）

すべて満たしたとき v1.0.0 のリリース準備完了:

- [ ] I-A〜I-D の改訂ファイル全件 commit 済み
- [ ] I-E self-verification で矛盾なし、過去変換の v0.2 ドキュメントとの後方互換性確認
- [ ] I-F dogfooding で 3 つの profile それぞれで 1 件ずつ試験変換完走
- [ ] plugin.json `version`: `1.0.0`
- [ ] RELEASE_NOTES.md に v1.0.0 セクション追加
- [ ] migration guide 記載済み
- [ ] v1.0.0 タグ付け + GitHub Release 作成

リリース後の追跡指標:
- v1.0.0 経由の変換で「Profile 拒否で却下された追加」の件数（バランス profile 利用時の有用性検出効果の証拠）
- Catalog 永続化承認率（W1 採用後の永続化動向）
- v0.2 → v1.0 切替の発生有無（後方互換性の妥当性）
