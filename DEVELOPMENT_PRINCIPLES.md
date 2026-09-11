# Development Principles

Status: Draft
Version: 0.3
Last updated: 2026-09-11
Language of record: 日本語

## Purpose

日本の業務システムで繰り返し必要になる、日本固有の業務ルール・標準・計算・検証を、安心して再利用できる.NET OSSとして提供する。

これは「日本向け便利ライブラリ集」を作ることではない。明示できる根拠と再現可能な結果を持つ、独立した問題領域の実装を提供する。

## North Star

日本の.NET業務開発者が、「これは自分で書かず、amane.JpBusinessを使う」と自然に判断できることを目指す。

GitHub Star、Package数、コード量はNorth Starではない。

## 正しさの定義

このプロジェクトでいう「正しさ」とは、**明示されたScope・Rule Set・Normative Sourceに対して、実装が期待される挙動と一致していること**を意味する。

これは、あらゆる法的状況で正しいこと、個別案件の税務判断として正しいこと、公的機関による認証を意味しない。

## Reproducibility

次を原則とする。

```text
Same Package Version
+ Same Rule Set
+ Same Input
→ Same Result
```

ライブラリ内部で `DateTime.Now`、`DateTime.UtcNow`、`DateTime.Today` などを参照し、暗黙に「現在のRule」を選択しない。時点が必要な場合は入力として明示する。

これは、現時点から巨大なVersioned Rule Engineを作るという意味ではない。

## Normative Source model

RuleやPackageごとに、次の情報源を区別する。

- **Normative Source**: 実装が適合すべき正本。
- **Interpretive Source**: 正本の解釈を補助する資料。
- **Reference Source**: 背景や比較のための参照資料。

正本が利用できない場合は着手しない。ただし「法律が常に最上位」という固定階層にはしない。技術仕様では、Specification、Schema、Schematron、Code ListなどがNormative Sourceになり得る。

「公開されている」ことと、「OSS実装で利用・引用・再配布できる」ことは別である。利用条件、引用条件、再配布条件も確認する。

## Theme Selection Gates

テーマの着手前に、少なくとも次のGateを確認する。

- **G-A Normative Source Availability**: 利用可能な正本と、その利用条件があるか。
- **G-B Deterministic Expected Results**: 入力に対する期待結果を決定できるか。
- **G-C Existing Alternatives**: 既存OSSや個別実装で十分でないか。

実装を最初に始めない。G0で着手可否を判断し、根拠・Scope・期待結果・代替手段を確認する。

## Evaluation Axes

長期的な評価軸は **Verified Reuse Advantage** とする。

これは、Human / Coding Agentによる案件ごとの個別再実装と比較し、仕様確認・実装・検証・保守を共通化する価値が十分あるか、という意味である。

## Package Strategy

- 巨大な全部入りPackageを作らない。
- Problem単位でPackageを分ける。
- 各Packageは単独で存在理由を説明できるようにする。
- Package数をKPIにしない。
- 次のPackageを自動的に作らない。

複数Packageで実際に共通化の需要が発生するまで、巨大Coreや共通基盤を先回りして作らない。

## API / Design Principles

将来のAPIは、間違った使い方をしにくく、Fail Safe by Designであることを目指す。

- 曖昧な入力を黙って補完しない。
- 金額には `decimal` を使う。
- 日付だけの概念には `DateOnly` を使う。
- ambientな現在時刻を使わない。
- immutableであることを優先する。
- deterministicな結果を返す。
- runtime依存関係を最小にする。

例えば、Business meaningから `TaxCategory`、Rule Set、Applicable Rateへと意味を明示的にたどれる設計を検討する。`TaxCategory.Standard == 10%` のように、TaxCategoryそのものが永続的な固定値を持つと誤解させる設計は避ける。率はScope、Rule Set、時点などの条件に依存し得る。

## Test Taxonomy

テストの種類と役割を分ける。すべてをGolden Testとは呼ばない。

### Golden Test

Normative Source等からExpected Resultを固定できる代表ケース。可能な限り、Case ID、Source ID、Relevant Section、Effective Date、Input、Expected Resultを紐付ける。Golden TestとRuleとSourceの対応表はOSS側に公開する。

### Boundary Test

境界値、閾値、およびその直前・直後の挙動を確認する。

### Property-based Test

個別の期待値ではなく、一般的不変条件を確認する。

### Regression Test

IssueやBugで発見されたケースを再発防止のために固定する。

## Human / Coding Agent Responsibility

Humanは、Scope、Normative Source、仕様解釈、Domain Model、Public APIの意味論、Acceptance Criteria、制度変更判断、Gate判定、Release判断を担う。

Coding Agentは、Implementation、Unit Test、Boundary Test、Property-based Test、Regression Test、Golden Caseのコード化、Benchmark、Refactoring、Sample、Documentation draft、Independent reviewを担う。

原則として、Humanが「何が正しいか」を決め、Agentが「その正しさをコードとテストで実現する」。

## Development Gates

開発の流れは次の大きな段階で管理する。

```text
Before Implementation
→ Implementation
→ Verification
→ Publication
→ Feedback
```

既存の判定点は次のとおりとする。

- **G0: 着手判定** — Scope、Source、期待結果、既存代替、需要を確認する。
- **G1: 実装継続判定** — 実装中に前提、根拠、検証可能性、維持可能性を再確認する。
- **G2: Release Readiness** — 正本をFresh確認し、検証結果、Supported / Unsupported、変更の影響を確認する。
- **G3: 公開後継続判定** — Feedback、需要、根拠、保守負荷を確認し、継続・縮小・停止を判断する。

## Existing OSS First

既存OSSで十分な場合は再実装しない。Existing OSSの確認はG0、G2、大きな機能追加前にFreshに行う。

## Versioning / Maintenance

Versioningの詳細は現時点では未決定とする。ただし、API非互換ではなくても計算結果が変わる変更、Rule Set identity、過去Ruleの再現は将来の重要な論点である。

制度変更を、説明のない挙動変更として黙って入れない。変更の根拠、適用範囲、Rule Setの識別、必要に応じて過去Ruleの再現方法を明示する。

## Responsibility and Scope

本OSSが提供するのは、**明示されたNormative Sourceに基づく計算・検証**である。

本OSSは、税務・法務判断の代替、公的機関による認証、あらゆる個別ケースでの法的正しさを主張しない。利用者は各自の責任でScope、根拠、適用条件を確認する。

## Avoid Overengineering

現時点で、巨大Core、汎用Rule Engine、独自DSL、Cloud / Dashboard、未使用Plugin System、将来Connectorを先回りして作らない。必要性が実際に発生し、Scopeと根拠を説明できる場合に改めて検討する。

## Release / Continuity / Stop

- Human approvalなしにReleaseしない。
- Release前にNormative SourceをFresh確認する。
- Supported / Unsupportedを明示する。
- 保守を停止する場合は明示する。
- Repositoryは削除せずarchiveし、Forkを妨げない。

次のいずれかに該当する場合は、着手・継続・公開を停止または見直す。

- 既存OSSで十分である。
- 個別再実装で十分である。
- Normative Sourceを利用できない。
- Expected Resultを決定できない。
- 需要がない。
- 案件固有の要素ばかりである。
- Maintenanceが過大である。
- Scopeを維持できない。
- READMEで価値を説明できない。
- FeedbackがないままFeatureだけが増える。

Stopは失敗ではない。根拠と維持可能性を守るための判断である。

## Current Direction

固定Roadmapは置かない。現時点では `ConsumptionTax` を最初の仮説として挙げられるが、これはG0通過前の候補にすぎない。将来候補をRoadmapとして約束しない。
