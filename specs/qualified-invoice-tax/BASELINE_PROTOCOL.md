# QualifiedInvoiceTax G1 Baseline Protocol

## 目的と固定時点

この文書は、QualifiedInvoiceTax の実装を Coding Agent が個別に再現できるかを測るための実験手順である。製品テストや製品実装ではない。

- Protocol ID: QIT-G1-2026-09-11-v1
- 固定日: 2026-09-11（JST）
- 固定後の変更: 測定開始後は、閾値、採点方法、失敗分類、Corpus の意味を変更しない
- 対象: specs/qualified-invoice-tax/golden/cases.json にある source-traceable Golden cases
- 実装場所: /srv/dev/scratch/amane-jp-business-g1-baseline/
- Repository へのBaseline実装・Harnessの追加: しない

## Agent実行

1. Codex CLI と Claude Code が、認証済みで通常利用できる場合は両方を使う。片方を使えない場合は BLOCKED / unavailable と記録し、結果を作らない。
2. 各Agentは独立したFresh contextで、一度だけ同一の保存済みPromptを受け取る。
3. Agentには、公開要件、入出力契約、ビルド方法だけを渡す。cases.json、期待値、SOURCES.md、RULES.mdその他の内部成果物は渡さない。
4. Agentの生成後、修正依頼や失敗ケースの提示を行わず、最初の生成物をそのまま採点する。
5. first attempt は「生成 → build確認 → hidden corpus採点」までとする。
6. Agentの利用資格情報、設定、実装contextを他Agentへ共有しない。

Promptは prompt/baseline-prompt.md に保存し、その SHA-256 を結果へ記録する。同じバイト列を両Agentへ渡す。

## 公開入出力契約

Baseline Promptで次の契約を固定する。実装言語は .NET 10/C# とする。

- 実行形: dotnet run --project baseline.csproj -- <input-json-path>
- 出力: stdout に結果JSONを1件だけ出す
- 入力は、明示された ruleSet.rateByTaxCategory で taxCategory から適用率を解決する
- basis は TaxExclusive または TaxInclusive
- 税抜は basisAmount × rate、税込は basisAmount × rate / (1 + rate)
- roundingPolicy.mode は RoundDown、RoundUp、HalfUp のいずれかで、1円単位
- 明細を解決済み率ごとに集計してから、率グループごとに1回だけ丸める
- 同じ入力、同じRule Set、同じ丸め方からは同じ結果を返す

非負の円単位入力だけを採点対象とする。HalfUp はこのCorpusでは非負値の通常の四捨五入（0.5以上を切上げ）として扱う。

## 採点

### Behavioral score

- Corpus の全ケース数を TOTAL とする（Protocol固定時点では24ケースを予定し、最終確定値を結果に記録する）。
- 各ケースは、入力JSONに対する終了コード、JSONとしての出力、invoiceTotalYen、taxTotalYen、率別内訳の集合（率、Category、集計額、税額）が期待値と一致したときだけPASS。
- JSONのプロパティ順と内訳配列の順は意味を持たせない。内訳に重複、欠落、余分な率グループがあればFAIL。
- build FAILまたは実行不能の場合は、Behavioral scoreを 0 / TOTAL とする。
- RATE = PASSED / TOTAL × 100。小数第2位まで記録する。

### Strong / Weak

Strong は次の全条件を満たすfirst attemptとする。

~~~
build PASS
AND
behavioral cases >= 95%
AND
material rule violation = 0
~~~

次のいずれかに該当すればWeakとする。

~~~
behavioral cases < 95%
OR
material rule violation >= 1
OR
build FAIL
OR
実行不能
~~~

閾値は結果を見た後に変更しない。24ケースの場合、Strongに必要な最小値は23/24（95.83%）である。Corpusの最終件数が変わった場合も、閾値は95%のままとする。

## Material rule violation

割合に関係なく、次を1件以上確認したらMaterial violationとする。

1. 明細ごとに丸めた税額を合計し、それを適格請求書の率グループ税額として返す。
2. 8%と10%のグループを率解決前または率別集計前に混在させる。
3. 税込金額に単純に税率を掛けるなど、gross × rate / (1 + rate) と異なる式を権威的税額に使う。
4. 契約上必須の丸め方を暗黙の既定値にする、または指定された丸め方を無視する。
5. DateTime.Now 等の環境時刻からRule Set、適用率、現行制度を選ぶ。
6. 入力されたBusiness Meaning/Tax Categoryを無視し、Libraryが軽減税率対象かどうかを法的に判定する。
7. 同一Rule Set・同一Input・同一丸め方の結果が変わる。

判断時は、Harnessの再実行結果と生成コードを照合し、根拠を BASELINE_RESULTS.md に短く記録する。単なる命名や未使用サンプルではなく、権威的な計算経路を判定する。

## 再現性・採点順

- Hidden harnessはCorpusをAgent作業ディレクトリの外に置く。
- 各ケースを1回採点し、少なくとも同一入力を2回実行して決定性を確認する。
- build確認の後に採点する。採点後のAgentへの修正投入、再生成、再採点は禁止する。
- ケース追加・期待値変更・Rule ID変更は測定後に行わない。

## 記録項目

Agentごとに、Tool/Version、Fresh context、Prompt SHA-256、build、PASS/TOTAL、率、Material violations、first-attempt判定、実装サイズ（参考）、失敗理由を記録する。Cleanに実行できなかったAgentは数値を記録しない。

## 事後判断

各Agentの結果をこの固定基準でStrong/Weakに分類し、次のいずれかを1つ選ぶ。

~~~
PROCEED_TO_IMPLEMENTATION_DESIGN
CORPUS_FIRST_REASSESSMENT
STOP_IMPLEMENTATION
BLOCKED
~~~

両AgentがStrongなら、実装を直ちに進めず、Corpus単独の価値と per-project generation との差を再評価する。Material violationがあれば共通実装のVerified Reuse Advantageは強まるが、HumanのGOを自動決定しない。
