# QualifiedInvoiceTax G1 API Sketch

これは実装ではなく、G1で固定した最小API案である。.NET project、source code、interface hierarchy、DI abstractionは作成しない。

## 値

~~~
InputBasis = TaxExclusive | TaxInclusive
RoundingMode = RoundDown | RoundUp | HalfUp

TaxCategory = opaque caller-supplied string
BusinessMeaning = required caller-supplied string

RateRule {
  TaxCategory
  ApplicableRate       // Rule Set内の正確な率。例: "0.10" または "0.08"
}

RuleSet {
  Id
  EffectiveFrom
  RateByTaxCategory: list of RateRule
}

TaxLine {
  LineId
  TaxCategory
  BusinessMeaning
  AmountYen           // InputBasisに応じた税抜または税込の非負円整数
}

InvoiceTaxRequest {
  InvoiceId
  Basis: InputBasis
  RuleSet
  RoundingPolicy { Mode: RoundingMode, Unit: "JPY" }
  Lines: list of TaxLine
}
~~~

TaxCategoryは10%や8%を意味するenum値として定義しない。BusinessMeaningとCategoryは呼出し側が既に判断したものを受け取り、適用率はRule Setのmappingからだけ解決する。

## 操作

~~~
InvoiceTaxResult Calculate(InvoiceTaxRequest request)
~~~

Calculateは副作用のない単一操作とする。内部の最小手順は次のとおり。

1. InvoiceId、Basis、Rule Set ID、EffectiveFrom、明示されたRoundingPolicy、各LineのCategory/BusinessMeaningを検証する。
2. 各LineのTaxCategoryをrequest.RuleSetからApplicableRateへ解決する。対応がなければエラーにする。
3. ApplicableRateごとにAmountYenを合計する。8%と10%など異なる率の額を先に合算しない。
4. TaxExclusiveなら basisAmount × rate、TaxInclusiveなら basisAmount × rate ÷（1＋rate）を計算する。
5. 各率グループの未丸め税額へ、request.RoundingPolicy.Modeを1回だけ適用する。
6. 結果の税額合計を率グループ税額の合計として返し、同じ率グループの明細単位丸め税額は合算しない。
7. 出力内訳は率、Category、集計したBasisAmountYen、TaxYenを含め、率とCategoryの安定した順序で返す。

## Result

~~~
InvoiceTaxResult {
  InvoiceId
  InvoiceTotalYen       // TaxExclusive: basis + tax、TaxInclusive: 入力税込額の合計
  TaxTotalYen
  Breakdown: list of RateBreakdown
  RuleSetId
}

RateBreakdown {
  TaxCategory
  ApplicableRate
  BasisAmountYen
  TaxYen
}
~~~

税込入力では、InvoiceTotalYenは入力税込額の合計である。丸め済みTaxYenから逆算した税抜額をInvoiceTotalYenへ再計算しない。

## 制約

- Rule Setは必須で、IDと適用前提を結果から追跡できる。
- RoundingPolicyは必須。未指定時に切捨て等を選ぶ隠れたdefaultを置かない。
- ambient current time、DateTime.Now、実行環境の現行制度判定を使わない。
- Libraryは商品・役務の法的な軽減税率該当性を判断しない。
- 同じRule Set、Input、RoundingPolicyの結果は決定的である。
- 入力のrate mappingを無視した固定のCategory-to-rate定数を設けない。
- 小さな一つのCalculatorと値の型だけで足りる。汎用Rule Engine、DSL、複雑なinterface階層、DI containerは作らない。

## G1で扱わないAPI

申告税額、仕入税額控除、返品・値引きの符号処理、外貨、小数円、登録番号照合、商品分類、PDF/XML/UI/DBは含めない。
