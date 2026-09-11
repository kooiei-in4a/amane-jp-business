# Blind coding baseline task

あなたは、既存のプロジェクトに依存しない最小の Qualified Invoice Tax 計算CLIを作るCoding Agentです。これは使い捨ての測定用実装です。

現在の作業ディレクトリだけを使ってください。親ディレクトリ、別の作業ディレクトリ、対象Repository、隠しCorpus、期待値、内部仕様書を探したり読んだりしないでください。追加の質問はせず、要件から実装してください。

## 作成物

現在のディレクトリ直下に、.NET 10でbuildできる次のCLIを作成してください。

- baseline.csproj
- 必要なC# source files

CLIは次の形で実行できるようにしてください。

~~~text
dotnet run --project baseline.csproj -- <input-json-path>
~~~

入力JSONを読み、stdoutへ結果JSONを1件だけ出してください。ログや説明文をstdoutへ出さないでください。入力エラーは非ゼロ終了にしてください。

## 入力契約

入力は次の概念を持ちます。

~~~json
{
  "ruleSet": {
    "id": "explicit-rule-set-id",
    "effectiveFrom": "YYYY-MM-DD",
    "rateByTaxCategory": [
      {
        "taxCategory": "caller-supplied-category",
        "applicableRate": "0.10"
      }
    ]
  },
  "input": {
    "invoiceId": "invoice-id",
    "basis": "TaxExclusive",
    "lines": [
      {
        "lineId": "line-id",
        "taxCategory": "caller-supplied-category",
        "businessMeaning": "caller-supplied business meaning",
        "amountYen": 123
      }
    ]
  },
  "roundingPolicy": {
    "mode": "RoundDown",
    "unit": "JPY"
  }
}
~~~

- basisはTaxExclusiveまたはTaxInclusiveです。
- amountYenは非負の日本円整数です。
- rateByTaxCategoryのmappingを使って、各lineのtaxCategoryから適用率を解決してください。
- taxCategoryとbusinessMeaningは呼出し側が分類済みの値です。商品名やbusinessMeaningから軽減税率該当性を推測・変更しないでください。
- Rule Setのid、effectiveFrom、rate mapping、basis、roundingPolicyは必須です。暗黙の現行日付や隠れた設定を使わないでください。
- roundingPolicy.modeはRoundDown、RoundUp、HalfUpのいずれかで、unitはJPYです。modeを省略したときの暗黙のdefaultを作らないでください。

## 計算要件

1. lineをRule Setで適用率へ解決してください。
2. 同じ適用率のlineを先に集計してください。異なる率の金額を先に合算してはいけません。
3. TaxExclusiveでは、率グループの集計税抜金額に適用率を掛けて税額を求めてください。
4. TaxInclusiveでは、率グループの集計税込金額に適用率を掛け、1＋適用率で割って税額を求めてください。税込10%は10/110、税込8%は8/108に相当します。
5. 各率グループの未丸め税額へ、指定されたroundingPolicyを1回だけ適用してください。lineごとの税額を丸めて合算してはいけません。
6. RoundDownは非負値の1円未満を切捨て、RoundUpは切上げ、HalfUpは非負値の通常の四捨五入（0.5以上を切上げ）です。
7. 計算は浮動小数点誤差で結果が変わらないようにしてください。
8. 同じRule Set、同じ入力、同じ丸め方からは常に同じ結果を返してください。

## 出力契約

結果JSONは少なくとも次の値を含めてください。

~~~json
{
  "invoiceId": "same-invoice-id",
  "ruleSetId": "same-rule-set-id",
  "invoiceTotalYen": 0,
  "taxTotalYen": 0,
  "breakdown": [
    {
      "taxCategory": "caller-supplied-category",
      "applicableRate": "0.10",
      "basisAmountYen": 0,
      "taxYen": 0
    }
  ]
}
~~~

- TaxExclusiveのinvoiceTotalYenは、入力税抜金額の合計＋率グループ税額の合計です。
- TaxInclusiveのinvoiceTotalYenは、入力税込金額の合計です。
- breakdownは率グループごとに1件だけ返し、集計したbasisAmountYenと税額を返してください。
- breakdownの配列順は率、次にtaxCategoryの安定した昇順にしてください。
- 出力の数値はJSON number、率は入力の正確な文字列表現または同値の明確な文字列表現にしてください。

## 制約と完了条件

- 法的な商品分類、登録番号照合、申告税額、会計仕訳、UI、DB、汎用Rule Engine、DSLは作らないでください。
- 現在時刻、DateTime.Now、環境変数によるRule Set選択を使わないでください。
- 小さな単一CLIとして実装してください。複雑なinterface階層やDI containerは不要です。
- 実装後に dotnet build baseline.csproj を実行し、build可能な状態にしてください。
- 期待値を含むテストCorpusを作成しないでください。
