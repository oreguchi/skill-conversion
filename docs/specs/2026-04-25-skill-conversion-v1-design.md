# skill-conversion v1.0.0 設計仕様書

**作成日**: 2026-04-25
**起草**: oreguchi (auto-mode brainstorming)
**前段資料**: `C:\dev\01_ClaudeProject\skill-conversion実践2\docs\plans\2026-04-23-vb-winforms-skills-conversion\v2.1-retrospective-for-skill-conversion-v03.md`、`C:\dev\01_ClaudeProject\skill-conversion実践2\docs\2026-04-24-variant-comparison-review.md`
**target version**: skill-conversion v1.0.0（v0.2.0 からの major bump、stable リリース）

---

## 1. Overview & Motivation

### 1.1 動機

2026-04-24 のバリアント比較レビューで、`skill-conversion` v0.2.0 によって生成された vb-winforms-skills v2.0.0 が次のように評価された:

- 変換元忠実度 ◎ / 技術的正確性 ◎ / **ターゲット有用性 △**

技術的正確性は最高だったが、1.0.0 系にあった WinForms 特化機能（UI スレッドマーシャリング、`IProgress(Of T)`、フォーム注入）が**完全に脱落**。Phase 4 opt-in を選んでいたにもかかわらず、有用性の検出が機能しなかった。

### 1.2 真因の分析

skill-conversion の品質は **3 軸モデル**で表現できる:

| 軸 | 既存 v0.2 仕様での担保 | v2.0.0 での結果 |
|---|---|---|
| 変換元忠実度 | Phase 2 source-compare（fixpoint loop） | ◎ |
| 技術的正確性 | Phase 3 dotnet build | ◎（ただし出典性は弱い） |
| ターゲット有用性 | Phase 4 blind eval (opt-in) | **✗ scenario 設計依存で漏洩** |

Phase 4 opt-in は「ユーザーが有用性を重視している」シグナルだが、その期待値が **scenario 設計に伝わる仕組みがなかった**。scenario はソース内容から導出されるため、ソースに無い「ターゲット環境固有の期待機能」は最初から検出範囲外だった。

### 1.3 改善方針: 三位一体

v1.0.0 では以下 3 要素を統合する:

1. **Conversion Profile**: 期待値の宣言（高忠実度 / バランス / 高有用性）
2. **Catalog**: ターゲット環境特有の「期待機能」と「罠」のデータソース
3. **Phase 制御**: profile に応じた Phase 4 制御 / approved-additions 閾値 / Catalog 参照量の自動連動

これにより、ユーザーが Phase 0 §7 で profile を宣言した時点で、Phase 4 が「期待値に合致した有用性検出」を実行できる状態になる。

---

## 2. Conversion Profile（3 段階）

### 2.1 概念

**Conversion Profile** は、ユーザーが Phase 0 で宣言する「変換の期待値」。3 段階で「忠実度と有用性のトレードオフ位置」を表現する。

| Profile | 意味 |
|---|---|
| **高忠実度** (high-fidelity) | ソース忠実度最優先。ターゲット環境固有の付加価値より、ソース著者の意図保持を優先 |
| **バランス** (balanced) ⭐ デフォルト | 忠実度 + 有用性の両立。ターゲット環境の事実上必須機能までカバー |
| **高有用性** (high-utility) | ターゲット読者の有用性最優先。ターゲット環境特有の差別化機能まで取り込む |

### 2.2 Profile 制御連動表

profile 宣言が以下 4 軸を**機械的に決定**する:

| 軸 | 高忠実度 | バランス | 高有用性 |
|---|---|---|---|
| **Phase 4** | デフォルト on、opt-out 可（理由必須） | 強制 on | 強制 on |
| **approved-additions 閾値** | 厳格（カテゴリ A のみ） | 標準（A + B） | 寛容（A + B + C） |
| **Catalog 参照量** | 罠 + Tier 1 | 罠 + Tier 1 + Tier 2 | 罠 + Tier 1 + Tier 2 + Tier 3 |
| **Phase 2 fixpoint 収束** | 常に高 stakes（profile 不問） | 同左 | 同左 |

### 2.3 宣言の場所

v0.2 の Phase 0 §7（Phase 4 opt-in 判断）を **Profile 宣言** にリネーム・拡張する。

```markdown
### 7. Conversion Profile 宣言

- [ ] 高忠実度（high-fidelity）
- [x] バランス（balanced）  ← デフォルト
- [ ] 高有用性（high-utility）

宣言した profile に応じて、以下が自動的に決定される:
- Phase 4 opt-in 状態
- approved-additions 承認閾値
- Catalog 参照範囲

高忠実度を選び Phase 4 を opt-out する場合は、理由を §7.1 に記載すること。
```

### 2.4 デフォルトと省略時の挙動

- 省略時: バランス自動採用（warning ログ）
- 不明な値が宣言された場合: halt して再宣言を求める

### 2.5 Phase 4 opt-out の運用

- **opt-out 可は高忠実度のみ**
- opt-out 時は Phase 0 §7.1 に **理由を必須記載**
- drift-report の冒頭に「Phase 4: SKIPPED by user choice (高忠実度 profile, reason: ...)」を明示

バランス・高有用性で opt-out を試みた場合は halt（profile を高忠実度に変更するか、Phase 4 を実行するかを user に再選択させる）。この halt → 再選択ロジックは I-C（Profile 連動 Phase 4）で実装する。

---

## 3. Catalog System

### 3.1 構造

各 catalog ファイル (`references/catalogs/<lang>-<framework>.md`) は **2 セクション固定**:

```markdown
# <Lang> <Framework> — Conversion Catalog

## 期待機能（Expected Capabilities）

### Tier 1（必須）— ソース忠実度の延長として認識される基本機能
### Tier 2（balanced 以上で推奨）— ターゲット環境で事実上必須
### Tier 3（high-utility で推奨）— ターゲット環境特有の差別化機能

## 罠（Gotchas）— 全 profile で必読

### 言語制約 / 型制約 / 構文制約 / API 制約
```

### 3.2 Tier 分類クライテリア（subagent への厳密指示）

| Tier | 判定基準 |
|---|---|
| **Tier 1** | ソース章立てに対応する箇所がある、または「ソースが扱う機能のターゲット環境等価物」 |
| **Tier 2** | ソースに対応がないが、**ターゲット環境の公式ドキュメント・主要書籍・チュートリアルで「推奨される基本パターン」として広く記述**されている。「これが抜けるとそのスキルが不完全」と一般認知される |
| **Tier 3** | ターゲット環境特有の**差別化機能・高度な機能**。Tier 2 ほど普及していない、または特定ユースケース向け |

主観的判断を減らすため、複数の権威ある情報源（公式ドキュメント、Microsoft Learn、MDN、主要書籍）に記述があるかを Tier 2 の判定材料とする。

### 3.3 生成と承認のフロー（W1' = 仕組み 1 + 4 + 5）

```
1. Phase 0 開始時に永続化済み catalog をスキャン
   └─ 該当組み合わせがあれば: 「再利用しますか?」を user に確認
       ├─ 再利用 → Phase 0 §X に取り込み（生成スキップ）
       └─ 再生成 → 次へ

2. subagent が Tier 分類クライテリア（仕組み 1）に従って catalog ドラフト生成

3. 内容承認ゲート（仕組み 4）
   └─ user が Tier 分類・項目内容をレビュー・承認
       └─ 承認: Phase 0 で利用（メモリ上のみ）

4. 永続化承認ゲート（NEW、2 段階目）
   └─ user に「この catalog を永続化しますか?」を確認
       ├─ 永続化: references/catalogs/<file>.md に保存 + INDEX.md 自動更新
       └─ 今回限り: メモリ上のみ、Phase 5 完了後に破棄
```

### 3.4 認識可能性の 3 つの仕組み

| 仕組み | 具体内容 |
|---|---|
| **(a) Phase 0 開始時の状態表示** | 「現在永続化済み catalog: vb-net-winforms.md (last updated YYYY-MM-DD), python-go.md (...)」を user に提示 |
| **(b) `references/catalogs/INDEX.md` 自動メンテ** | 永続化承認時に追記される。各 catalog の「対象組み合わせ / 最終更新日 / 用途要約」を 1 行ずつ記載 |
| **(c) drift-report への記載** | 完了時の drift-report §「使用 Catalog」セクションに、利用した catalog を path + 永続化 status + 最終更新日で記載 |

### 3.5 Catalog ↔ Scenario の連動

**Phase 0 §6（scenario 骨子作成）で**:
- main agent が catalog の「期待機能」を読み込み、profile に応じた Tier 範囲を確保
- 各 scenario の Requirement checklist に `[Tier N]` タグを付けて catalog 参照点を明示
- Phase 4 blind eval の scorer は `[Tier N]` タグ付きアイテムのカバレッジを機械的に検証

**Phase 1〜2 fixpoint loop で**:
- catalog の「罠」セクションが §3b no-equivalent 表に**自動マージ**される（手動で個別管理しない）

### 3.6 v1.0.0 リリース時の同梱

- skill-conversion 本体に **catalog seed は同梱しない**（汎用ツール性を保つ）
- catalog は使用時に動的に生成・蓄積される（W1）

---

## 4. approved-additions 承認テーブル

### 4.1 二軸の定義

#### 軸 1: カテゴリ（追加内容の**性質**）

| カテゴリ | 性質 | 判定基準 |
|---|---|---|
| **A. 忠実度維持** | ソース忠実度を維持するための補正 | ソース内容を保持できないコンパイラ/ランタイム制約への対応、構文等価性確保 |
| **B. 環境必須** | ターゲット環境で事実上必須な基本機能 | Catalog Tier 1+2 に該当。「これが抜けるとスキルが不完全」と一般認知 |
| **C. 差別化機能** | ターゲット環境特有の差別化機能 | Catalog Tier 3 に該当。Tier 2 ほど普及していない、または特定ユースケース向け |

#### 軸 2: Impact（追加内容の**規模**）

既存 v0.2 仕様を踏襲:

- **minor** — 既存文の明確化（例: 1 行コメント追加、注記の補強）
- **moderate** — 欠落文脈の補完（例: 段落・コード断片追加、用語注記の復活）
- **heavy** — 新セクション追加・構造変更（例: 章丸ごと追加、フォーマット改変）

### 4.2 9 マス（実質 27 セル）承認テーブル

| カテゴリ × Impact | 高忠実度 | バランス | 高有用性 |
|---|---|---|---|
| **A**. 忠実度維持・minor | 自動承認 | 自動承認 | 自動承認 |
| **A**. 忠実度維持・moderate | 自動承認 | 自動承認 | 自動承認 |
| **A**. 忠実度維持・heavy | user 承認必須 | user 承認必須 | user 承認必須 |
| **B**. 環境必須・minor | user 承認必須 | 自動承認 | 自動承認 |
| **B**. 環境必須・moderate | **拒否（原則）** | user 承認必須 | 自動承認 |
| **B**. 環境必須・heavy | **拒否** | user 承認必須 | user 承認必須 |
| **C**. 差別化機能・minor | **拒否** | user 承認必須 | 自動承認 |
| **C**. 差別化機能・moderate | **拒否** | **拒否** | user 承認必須 |
| **C**. 差別化機能・heavy | **拒否** | **拒否** | user 承認必須 |

### 4.3 各セル状態の意味

| 状態 | 挙動 |
|---|---|
| **自動承認** | subagent または main agent が判定後、承認形式で register に追記。user 承認は不要だが事後 review 可能 |
| **user 承認必須** | 承認フォーム（subject / detail / rationale / impact / verdict）を user に提示。承認後に register 追記 |
| **拒否（原則）** | profile の意図に反するため**取り込まない**。register には登録せず、drift-report の「却下提案」に記載（理由: profile による拒否） |

### 4.4 拒否された case の扱い

profile が「拒否」と判定した追加は、Phase 4 blind eval などで「ターゲット読者にとって有用」と subagent が報告したとしても、**取り込まれない**。これは profile 宣言の「期待値」を尊重するため。

ただし、user が profile を変えたい場合は **Phase 0 §7 で profile を再宣言可能**（変換工程の最初からやり直し）。

### 4.5 v2.0/2.1 実例マッピング

| 追加（v2.0/2.1 で実施） | カテゴリ | Impact | 高忠実度 | バランス | 高有用性 |
|---|---|---|---|---|---|
| Span 全排除（A4 第 2 次） | A | heavy | 承認 ✅ | 承認 ✅ | 承認 ✅ |
| `Sub(_)` → `Sub(state)` | A | minor | 自動 ✅ | 自動 ✅ | 自動 ✅ |
| WinForms テンプレ追加 | B | moderate | **拒否** ❌ | 承認 ✅ | 自動 ✅ |
| UI スレッドマーシャリング統合 | B | heavy | **拒否** ❌ | 承認 ✅ | 承認 ✅ |
| 高 DPI / フォーム Func ファクトリー | C | moderate | **拒否** ❌ | **拒否** ❌ | 承認 ✅ |
| Akka.NET VB 注記 | C | minor | **拒否** ❌ | 承認 ✅ | 自動 ✅ |

→ profile を **「バランス」** で実行していたら、v2.0.0 で WinForms 統合・テンプレ追加が**最初から user 承認対象として浮上**し、A4 のような後付け統合が不要になる。これが v1.0.0 の核心的な改善効果。

---

## 5. Phase 0〜5 への影響

### 5.1 Phase 0（大幅変更）

| 既存 v0.2 § | v1.0.0 での変更 |
|---|---|
| §6 scenario 骨子 | Catalog の期待機能を参照（profile に応じた Tier 範囲を確保）。各 scenario の Requirement checklist に `[Tier N]` タグを付けて catalog 参照点を明示 |
| §7 Phase 4 opt-in 判断 | **Conversion Profile 宣言** にリネーム・拡張。Phase 4 opt-in 状態は profile が自動決定（高忠実度のみ opt-out 可、理由必須） |
| 新規 §X.1 | Catalog 状態確認（Phase 0 開始時に永続化済み一覧表示、該当組み合わせなら再利用確認） |
| 新規 §X.2 | Catalog 生成（必要時）+ 内容承認 + 永続化承認の 2 段階 |

### 5.2 Phase 1（軽微変更）

- §3b no-equivalent 表に **Catalog 罠が自動マージ**される（手動で個別管理しない）
- 翻訳 subagent の prompt に「**Catalog の期待機能 Tier 範囲をカバーすること**」を追加指示

### 5.3 Phase 2（変更なし）

- profile 不問で**常に高 stakes**（2 回連続ゼロ変更）
- 3 軸 fixpoint review はそのまま機能

### 5.4 Phase 3（B2 出典確認サブステップ追加）

- 既存 dotnet build PASS に加え、**採用 API の出典確認** を追加
- 公式性 4 段階分類（Official / Standard / Peripheral / Unknown）
- Peripheral / Unknown 採用時は **両パターン併記必須**
- Phase 3 結果報告に「採用 API 一覧」を含める

#### 公式性レベル定義

| レベル | 定義 |
|---|---|
| **Official** | Microsoft Learn / 公式リファレンスに記載がある |
| **Standard** | 主要書籍・公式チュートリアルで記述がある |
| **Peripheral** | GitHub README / コミュニティ wiki のみ |
| **Unknown** | 出典未確認 |

#### 「両パターン併記」発動条件

採用 API の公式性が Peripheral または Unknown の場合、SKILL.md に以下を強制:
- 標準パターン（より公式性高い API）の記述
- 採用 API の利用条件・制約注記

→ v2.1.0 で `AddOptionsWithValidateOnStart` に対して人手で行った両パターン併記が、**自動的にルール化**される。

### 5.5 Phase 4（中規模変更）

- **opt-in 制御を profile に連動**（高忠実度のみ opt-out 可）
- scenario 設計時に **Catalog Tier 期待機能を確保**（profile の Tier 範囲）
- blind eval の scorer が `[Tier N]` タグ付き checklist のカバレッジを**機械的に検証**
- approved-additions の判定が **9 マステーブルで自動化**（user 承認シーンが減少、profile 拒否は自動）

### 5.6 Phase 5（変更なし）

- auto-fire test は既存仕様のまま

### 5.7 Drift Report（軽微変更）

- 冒頭に **Profile 宣言** を明記
- 新規セクション「**使用 Catalog**」（path、永続化 status、最終更新日）
- 新規セクション「**Phase 3 採用 API 一覧**」（公式性レベル付き）
- profile 拒否で取り込めなかった追加は「却下提案」セクションに自動記載

---

## 6. 後方互換性 / 移行戦略

### 6.1 影響範囲

skill-conversion は**実行ツール**であり、**過去の変換成果物**（drift-report、approved-additions register 等）への破壊的影響はない。v0.2.0 で完了した変換は v1.0.0 でも引き続き参照可能。

| 区分 | 影響 |
|---|---|
| **完了済み変換** | 影響なし（過去の drift-report 等はそのまま有効） |
| **進行中変換**（v0.2.0 plan-doc） | v0.2.0 仕様で完結を推奨。途中切替は §7 の意味変化等で混乱の元 |
| **新規変換**（v1.0.0 開始） | v1.0.0 全面適用 |

### 6.2 主な破壊的変更

| 領域 | v0.2.0 | v1.0.0 |
|---|---|---|
| Phase 0 §7 | Phase 4 opt-in 判断 | **Conversion Profile 宣言**（拡張） |
| approved-additions 承認 | user 承認 1 件ずつ | **9 マステーブル + user 承認**（profile 連動） |
| Catalog | 概念なし | 必須（生成・2 段階承認・永続化） |
| Phase 4 | user 自由判断 | **profile 連動**（高忠実度のみ opt-out 可、理由必須） |
| Phase 3 | dotnet build PASS のみ | **+ 出典確認（B2）** |

### 6.3 移行ガイド方針

進行中の変換がある場合は **継続を推奨**:

| 案 | 内容 |
|---|---|
| **継続**（推奨） | v0.2.0 仕様のまま完結。次の新規変換から v1.0.0 を採用 |
| **切替**（非推奨） | 進行中の plan-doc を v1.0.0 形式に書き換え。手順は SKILL.md の migration guide セクションに記載 |

### 6.4 バージョン番号

- plugin.json: `0.2.0` → **`1.0.0`**（major bump = breaking change）
- v0.x 系は archive 扱い、v1.0.0 を stable リリースとする

---

## 7. 実装フェーズ計画

writing-plans に引き継ぐための実装単位の分解。各 Phase は独立性を保ちつつ、依存関係を明示。

### 7.1 実装 Phase

| # | Phase | 内容 | 依存 |
|---|---|---|---|
| **I-A** | コア仕様改訂 | SKILL.md（§7 → Profile 宣言、新規 §X catalog 関連）、references/approved-additions.md 改訂（9 マステーブル + カテゴリ A/B/C 定義）、references/verification-stages.md に B2 追加、templates/ 改訂 | なし（最初に実施） |
| **I-B** | Catalog インフラ | references/catalogs/ ディレクトリ新設、INDEX.md 自動メンテ仕様、catalog-template.md、catalog 生成 subagent prompt（仕組み 1: 分類クライテリア） | I-A 完了後 |
| **I-C** | ロジック仕様文書化 | 9 マス判定ロジック、Profile 連動 Phase 4、Catalog ↔ Scenario 連動、Phase 3 出典確認の 4 段階公式性 + 両パターン併記発動条件 | I-A、I-B 並行可 |
| **I-D** | ドキュメント更新 | README.md / SKILL.md 概要、migration guide、RELEASE_NOTES.md v1.0.0 セクション、plugin.json version bump | I-A〜I-C 完了後 |
| **I-E** | Self-Verification | design 矛盾の self-review、テンプレ consistency、v0.2 完了済みドキュメント（vb-winforms-skills 等）を v1.0 仕様で**再評価しても drift verdict が大幅に変わらないことを確認**（後方互換性の妥当性検証） | I-A〜I-D 完了後 |
| **I-F**（必須） | Dogfooding | skill-conversion v1.0.0 を使って小規模な試験変換を実施。結果に基づく微調整 | I-E 完了後 |

### 7.2 スコープアウト（v1.x 以降の検討）

- Phase 4 baseline_variant 機能（B4。Profile + Catalog で代替できるため不要と判断）
- Catalog 生成の自動 PR 投稿（コミュニティ運用）
- 多言語 INDEX.md（日英併記等）

### 7.3 推定規模

- I-A〜I-D で約 10〜15 ファイルの新規/改訂
- I-E は読み合わせ + 矛盾チェックで 1 セッション
- I-F は小規模試験変換 + 微調整で 1〜2 セッション
- 全体で 2〜3 セッション規模

---

## 8. 完了基準

以下すべてが満たされたとき v1.0.0 のリリース準備完了:

- [ ] I-A〜I-D の改訂ファイル全件 commit 済み
- [ ] I-E self-verification で矛盾なし、過去変換の v0.2 ドキュメントとの互換性確認
- [ ] I-F dogfooding で 1 件以上の変換が完走。理想は **3 つの profile（高忠実度 / バランス / 高有用性）それぞれで 1 件ずつ実施**（Profile 連動の挙動差を実証） |
- [ ] plugin.json `version`: `1.0.0`
- [ ] RELEASE_NOTES.md に v1.0.0 セクション追加
- [ ] migration guide 記載済み

リリース後の追跡指標:
- v1.0.0 経由の変換で「Profile 拒否で却下された追加」の件数（バランス profile 利用時の有用性検出効果の証拠）
- Catalog 永続化承認率（W1 採用後の永続化動向）
- v0.2 → v1.0 切替の発生有無（後方互換性の妥当性）
