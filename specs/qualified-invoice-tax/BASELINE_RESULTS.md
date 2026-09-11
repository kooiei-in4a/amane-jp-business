# QualifiedInvoiceTax G1 Baseline Results

## 固定条件

- Protocol ID: `QIT-G1-2026-09-11-v1`
- 測定日: 2026-09-11（JST）
- Fresh main: `cfc1f97310ee8bbdb5a36c2ae65e6661f247d53b`
- Issue: #4、Open
- Branch: `issue/4-qualified-invoice-tax-g1`
- Worktree: `/srv/dev/worktrees/amane-jp-business-issue-4`
- Protocol SHA-256: `fd367763273990884a57afcef3e8f8ae095c796f990b2d36d9cf173307027274`
- Blind Prompt SHA-256: `ddbb4e7d3305c4396303430362e5e3dfea0e95c704d37696abede436e7fe5a3b`
- Golden Corpus SHA-256: `e48dd79d744951f72d326132e7f24ec6f156c94b928f658946989b00e3256e2d`

Protocol、Prompt、Corpusを固定してから測定した。Promptには期待値、Golden case、SOURCES、RULES、API Sketchを含めていない。採点用CorpusとHarnessはAgent作業ディレクトリの外 `/srv/dev/scratch/amane-jp-business-g1-baseline/harness/` に置いた。

## Corpus

- Total: 24
- Official examples: 8
- Rule-derived arithmetic: 16
- Source traceability: 全caseに `sourceIds` と `ruleIds` を付与
- Official anchors: NTA概要PDF 7ページの税抜・税込8%/10%、率別集計、明細単位丸め不可の数値を収録
- Formal rules: 10（QIT-001〜QIT-010）
- Corpusで動的にカバーしたRule ID: 9。QIT-010は入力分類を自動判定しないというAPI境界であり、コード監査で確認した。
- Unresolved ambiguities: 5

`jq`によるJSON検証、率別の有理数計算検算、税額・請求総額の整合性検算はすべて成功した。境界値・派生算術だけのcaseを公式例として扱っていない。

## Measurement

各Agentは同一PromptをFresh contextで一度だけ受け取り、生成後のソース修正なしで測定した。buildは生成物に対して独立に確認し、hidden caseを各1回実行した後、同じ入力をもう1回実行した。

| Agent | Tool version | Build | Behavioral | Rate | Determinism | Material violations | Classification |
| --- | --- | --- | ---: | ---: | --- | --- |
| Codex CLI | `codex-cli 0.153.4` | PASS | 24/24 | 100.00% | PASS | 0 | Strong |
| Claude Code | `2.1.236` | PASS | 24/24 | 100.00% | PASS | 0 | Strong |

Claude Codeの実行中はAgent側のdotnet確認が権限確認で止まったため、Coordinatorが同じ生成物を変更せず独立buildした。CodexもClaudeも、その後の再生成・修正依頼・失敗caseの提示・再採点は行っていない。実装サイズは参考値として、CodexがC# 1ファイル/558行、ClaudeがC# 8ファイル/614行である。

### Material violation audit

固定済み7分類を、hidden Harnessの再実行結果と生成コードへ照合した。

1. 明細ごとの丸め税額合計を権威値にする経路: なし。両実装とも率グループを先に集計してから丸める。
2. 8%と10%を混在させてから集計する経路: なし。適用率ごとに別Groupを作る。
3. 税込で `gross × rate / (1 + rate)` 以外を使う経路: なし。
4. 丸め方の暗黙defaultまたは指定無視: なし。`roundingPolicy.mode`を必須入力として扱う。
5. 環境時刻からRule Set・率・制度を選ぶ経路: なし。
6. Business Meaning/Tax Categoryの法的な軽減判定: なし。入力Categoryを明示mappingで率へ解決する。
7. 同一入力の結果変動: なし。全24件の再実行結果が一致した。

## G1 decision

`CORPUS_FIRST_REASSESSMENT`

両AgentがStrongだったため、固定Protocolに従い実装設計へ直ちに進まない。24件のsource-traceable corpusは、狭い税額算術の契約・公式アンカー・丸め誤りを独立に検証できる。一方、公開Promptだけでも両Agentが100%に達したため、現時点で共通Package化によるVerified Reuse Advantageが実証されたとはいえない。次段階では、Corpus単独の保守価値とper-project generationとの差を、人間のGO判断の下で再評価する。

## Package name recheck

候補名は `Amane.JpBusiness.QualifiedInvoiceTax` とする。`QualifiedInvoiceTax`は今回の範囲（適格請求書の率別税額算術）を直接表し、申告・分類・登録番号照合まで含む名前ではない。G1ではPackage、Project、NuGet設定を作成していない。

## Scope audit

- Repository additions: `specs/qualified-invoice-tax/` 配下のみ
- Product source: not added
- Product tests/projects: not added
- `.csproj` / `.sln` / `.slnx` / NuGet / Core / Rule Engine / DSL: not added
- CI / release / other package: not added
- `DEVELOPMENT_PRINCIPLES.md`: unchanged
- Push / PR / merge: not performed

Coordinatorの採点用実装・HarnessはRepositoryへ追加していない。
