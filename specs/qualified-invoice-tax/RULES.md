# QualifiedInvoiceTax G1 Rule Corpus

## Scope

このRule Corpusは、適格請求書に記載する消費税額等を、呼出し側が既に分類した課税取引について計算するための最小仕様である。仕入税額控除の申告、税務判断、適格性判定、登録番号確認、会計処理は対象外とする。

基本の流れは次のとおりである。

~~~
Business Meaning
  → caller-supplied Tax Category
  → explicit Rule Set
  → applicable rate
  → rate-group aggregation
  → one rounding operation per invoice/rate group
~~~

Tax Category は率そのものではない。たとえば、現在の例示Rule Setが特定のCategoryに10%または8%を対応させても、Categoryと率の同一性を時代を越えて固定しない。

## 共通計算契約

- 入力金額は非負の日本円整数とする。
- Invoiceの各明細は、Rule Setにより適用率が解決された後、率グループごとに合計する。
- TaxExclusive の率グループ税額は、集計した税抜金額 × 適用率を丸める。
- TaxInclusive の率グループ税額は、集計した税込金額 × 適用率 ÷（1＋適用率）を丸める。10%は10/110、8%は8/108である。
- 端数処理単位は1円。切捨て、切上げ、四捨五入（Corpusでは非負値のHalfUp）は入力で明示する。
- Invoiceの税額合計は、率グループごとの丸め済み税額の合計である。税込入力のInvoice totalは入力された税込金額の合計とする。
- 同じInvoice、Rule Set、Input、RoundingPolicyからは常に同じ結果を返す。
- 法的な軽減税率該当性は呼出し側の責務であり、LibraryはBusiness MeaningまたはTax Categoryを推測・上書きしない。

## Rule一覧

| Rule ID | 要約 | Source IDs |
| --- | --- | --- |
| QIT-001 | 計算前に適用率ごとに集計する | S-001, S-002, S-003, S-005, S-006 |
| QIT-002 | 税抜・現行例示の一般課税Category | S-001, S-002, S-005, S-007 |
| QIT-003 | 税抜・現行例示の軽減対象Category | S-001, S-002, S-005, S-007 |
| QIT-004 | 税込・現行例示の一般課税Category | S-001, S-002, S-005, S-007 |
| QIT-005 | 税込・現行例示の軽減対象Category | S-001, S-002, S-005, S-007 |
| QIT-006 | Invoice×率グループで丸めを1回だけ行う | S-002, S-003, S-004, S-005 |
| QIT-007 | 丸め方を明示する | S-002, S-004 |
| QIT-008 | 明細単位丸め税額の合計をInvoice税額にしない | S-003, S-004, S-005 |
| QIT-009 | Rule Setを明示し、同じ入力を再現可能にする | S-001, S-002, S-006 |
| QIT-010 | 軽減税率の法的分類をLibraryが自動判断しない | S-001, S-003, S-005, S-006 |

## Rule details

### QIT-001 — 税額計算前に税率別で集計する

- Business meaning: 一つの適格請求書に含まれる取引を、適用率の異なるグループとして明確に扱う。
- Normative/Interpretive Source IDs: S-001, S-002, S-003, S-005, S-006
- Input: Invoiceの明細、各明細の caller-supplied Tax Category/Business Meaning、明示されたRule Set、Input basis。
- Expected behavior: CategoryからRule Setで率を解決し、同じ率の明細を先に集計してから税額を計算する。8%と10%の総額を先に混ぜない。
- Interpretation: 法57条の4第1項第4号・第5号、令70条の10、基本通達1-8-15の「税率の異なるごとに区分して合計」を計算契約へ落とし込む。
- Known ambiguity: 同じ率に複数のBusiness Meaning/Categoryが存在する将来の内訳表示はG1では定義しない。現在のG1 Rule Setは1率につき1 Categoryを例示する。

### QIT-002 — 税抜・一般課税Category計算

- Business meaning: Callerが軽減対象ではない課税資産の譲渡等として分類した税抜金額から、明示されたRule Setの適用率に基づく税額を求める。
- Normative/Interpretive Source IDs: S-001, S-002, S-005, S-007
- Input: TaxExclusive、caller-supplied general taxable-transfer Category、Rule Setの適用率、明示されたRoundingPolicy。
- Expected behavior: 集計税抜金額 × Rule Setの適用率を計算し、QIT-006/QIT-007に従って1回丸める。JP-QIT-2023-10-v1では例示率が0.10である。
- Interpretation: 令70条の10第1号の式を、率をCategoryから推測せず、Rule Setから解決する形にした。
- Known ambiguity: 将来の税率、経過措置、旧税率はこのRule Setに含めず、別Rule Setとして明示する。

### QIT-003 — 税抜・軽減対象Category計算

- Business meaning: Callerが軽減対象課税資産の譲渡等として分類した税抜金額から、明示されたRule Setの適用率に基づく税額を求める。
- Normative/Interpretive Source IDs: S-001, S-002, S-005, S-007
- Input: TaxExclusive、caller-supplied reduced taxable-transfer Category、Rule Setの適用率、明示されたRoundingPolicy。
- Expected behavior: 集計税抜金額 × Rule Setの適用率を計算し、QIT-006/QIT-007に従って1回丸める。JP-QIT-2023-10-v1では例示率が0.08である。
- Interpretation: 令70条の10第1号の括弧書きの式を使う。軽減対象とする分類自体は入力済みのものだけを受け取る。
- Known ambiguity: 飲食料品等が法的に軽減対象かどうか、外食等の境界は本Ruleの入力前提であり、G1では判断しない。

### QIT-004 — 税込・一般課税Category計算

- Business meaning: Callerが一般課税Categoryとして渡した税込金額に内包される税額を求める。
- Normative/Interpretive Source IDs: S-001, S-002, S-005, S-007
- Input: TaxInclusive、caller-supplied general taxable-transfer Category、Rule Setの適用率、明示されたRoundingPolicy。
- Expected behavior: 集計税込金額 × 適用率 ÷（1＋適用率）を計算し、JP-QIT-2023-10-v1の10%では税込金額 × 10/110として1回丸める。
- Interpretation: 令70条の10第2号の税込価額の方法と、NTA概要の税込10%記載例を用いる。
- Known ambiguity: 丸め後の税額と税込金額の差を、別の税抜金額として再配分する処理はG1のResultに含めない。

### QIT-005 — 税込・軽減対象Category計算

- Business meaning: Callerが軽減対象Categoryとして渡した税込金額に内包される税額を求める。
- Normative/Interpretive Source IDs: S-001, S-002, S-005, S-007
- Input: TaxInclusive、caller-supplied reduced taxable-transfer Category、Rule Setの適用率、明示されたRoundingPolicy。
- Expected behavior: 集計税込金額 × 適用率 ÷（1＋適用率）を計算し、JP-QIT-2023-10-v1の8%では税込金額 × 8/108として1回丸める。
- Interpretation: 令70条の10第2号の軽減対象の括弧書きと、NTA概要の税込8%記載例を用いる。
- Known ambiguity: 税込取引の値決め時に明細ごとに税額を表示することは、適格請求書の記載税額計算とは別の問題である。

### QIT-006 — Invoice×税率単位で1回だけ丸める

- Business meaning: 適格請求書に記載する税額等を、請求書全体の率グループ単位で一貫して決める。
- Normative/Interpretive Source IDs: S-002, S-003, S-004, S-005
- Input: 同一Invoice内に複数明細を持つInput、適用率ごとの集計、RoundingPolicy。
- Expected behavior: 各率グループにつき、集計後の未丸め税額へ丸めを1回だけ適用する。
- Interpretation: 令70条の10の1円未満処理、基本通達1-8-15、No.6371、NTA概要7ページの記載を一致させた。
- Known ambiguity: 請求書を何枚に分けるかは商取引上のInvoice境界であり、計算Libraryが推測しない。

### QIT-007 — 丸め方法を明示する

- Business meaning: 同じ金額でも、呼出し側が選んだ端数処理方針を再現できるようにする。
- Normative/Interpretive Source IDs: S-002, S-004
- Input: RoundDown、RoundUp、またはHalfUpと、1円単位の明示指定。
- Expected behavior: 指定モードを必須とし、指定がない場合に暗黙の既定値を選ばない。Corpusでは非負値のRoundDown、RoundUp、HalfUpを使う。
- Interpretation: 令70条の10は1円未満を処理するとし、No.6371は切上げ、切捨て、四捨五入等を任意の方法として説明する。本APIでは任意性をhidden defaultにせず入力へ出す。
- Known ambiguity: 負数の返還・値引きに対する符号付き丸めや、HalfUpの国際的な名称差はG1の非負Inputから除外する。

### QIT-008 — 明細単位丸め税額合計をInvoice税額としない

- Business meaning: 明細表示用の参考税額と、適格請求書の率別記載税額を区別する。
- Normative/Interpretive Source IDs: S-003, S-004, S-005
- Input: 同じ率で、個別丸めの合計と集計後丸めの結果が異なる複数明細。
- Expected behavior: 権威的なInvoice税額は集計後丸めの結果とし、明細ごとの丸め済み税額の合計を返さない。
- Interpretation: 基本通達1-8-15の注、No.6371、NTA概要7ページの認められない例から導く。個別税額を参考表示すること自体は、このRuleが禁止する対象ではない。
- Known ambiguity: 明細価格を税込へ変換する値決め処理の端数は、適格請求書の記載税額と同じ意味ではないため、G1 APIでは扱わない。

### QIT-009 — Rule Setを明示し同一入力を再現可能にする

- Business meaning: どの時点・前提の率を適用したかを呼出し側と結果で追跡できるようにする。
- Normative/Interpretive Source IDs: S-001, S-002, S-006
- Input: Rule Set ID、effectiveFrom等のRule Set metadata、Category-to-rate mapping、明示されたInputとRoundingPolicy。
- Expected behavior: Rule Setがない、Categoryの対応がない、またはRoundingPolicyがない場合は計算を続行しない。環境時刻、ローカル設定、暗黙の現行制度を参照しない。
- Interpretation: 法令の計算方法は取引の税率区分を前提にしているため、API上の適用前提を固定した設計ガードである。決定性はソフトウェア契約として追加する。
- Known ambiguity: 過去・将来の法改正対応のRule Set内容と移行方式はG1後に別途決める。

### QIT-010 — 軽減税率の法的分類を自動判断しない

- Business meaning: Libraryは税額算術に集中し、商品・役務の法的分類責任を利用者から奪わない。
- Normative/Interpretive Source IDs: S-001, S-003, S-005, S-006
- Input: Callerが明示したBusiness MeaningとTax Category。商品名だけからの分類要求は受け付けない。
- Expected behavior: 入力されたCategoryをRule Setで率へ解決するだけで、飲食料品、外食、新聞等の該当性を推論・修正しない。
- Interpretation: 法57条の4の記載事項と基本通達1-8-4は、軽減対象である旨を明らかにする責務を示す。NTA資料の制度説明を自動分類器の仕様へ拡張しない。
- Known ambiguity: 外部の法令分類サービスと連携する将来設計は本Candidateの範囲外。

## 未解決事項

1. 現行以外の税率、経過措置、制度改正のRule Setは未定義である。
2. 同一適用率に複数Categoryがある場合の内訳単位は未定義である。G1では率別計算を優先する。
3. Invoice境界を何で決めるかは呼出し側の責務である。
4. 負数の返品・値引き、外貨、小数円、地方消費税を別欄に分ける結果は対象外である。
5. このCorpusでの税額は適格請求書に記載する消費税額等であり、課税期間の申告税額や仕入税額控除額ではない。
