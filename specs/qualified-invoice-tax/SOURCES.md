# QualifiedInvoiceTax G1 Source Map

## 確認方針

- 最終確認日: 2026-09-11（JST）
- Normative は法令本文、Interpretive は国税庁の通達・質疑応答、Reference は国税庁の説明資料・税率案内・利用条件として区別した。
- この文書とGolden Corpusは、公式本文の大量転載ではなく、リンク、短い要約、計算結果だけを収録する。
- Corpusで使うRule Setは、2023-10-01以後の現行制度を対象にした JP-QIT-2023-10-v1 である。これは税率Categoryの永続的な同一性を意味しない。
- ブログ、まとめサイト、Stack Overflow、AI回答は正本・解釈根拠に使っていない。

## 一覧

| ID | Classification | Issuer | Document |
| --- | --- | --- | --- |
| S-001 | Normative | 日本国（掲載: e-Gov法令検索） | 消費税法 第57条の4 |
| S-002 | Normative | 日本国（掲載: e-Gov法令検索） | 消費税法施行令 第70条の10 |
| S-003 | Interpretive | 国税庁 | 消費税法基本通達 1-8-15 |
| S-004 | Interpretive | 国税庁 | No.6371 端数計算 |
| S-005 | Reference | 国税庁 | 適格請求書等保存方式の概要（令和8年5月） |
| S-006 | Interpretive | 国税庁 | 適格請求書等保存方式に関するQ&A（令和8年5月改訂） |
| S-007 | Reference | 国税庁 | No.6303 消費税および地方消費税の税率 |
| S-008 | Reference | 国税庁 | 利用規約・免責事項・著作権 |

## Source details

### S-001 — 消費税法 第57条の4

- Classification: Normative
- Issuer: 日本国。掲載者はデジタル庁の e-Gov 法令検索。
- Title: 消費税法（昭和63年法律第108号）
- URL / document identifier: [法令本文](https://laws.e-gov.go.jp/law/363AC0000000108)、[e-Gov法令API XML](https://laws.e-gov.go.jp/api/1/lawdata/363AC0000000108)、法令ID 363AC0000000108
- Version / publication date: 昭和63年12月30日公布。版番号の表示はないため、現行本文を2026-09-11にFresh確認した。
- Effective date: 適格請求書制度に関係する現行第57条の4は、NTA公式概要の制度開始日である2023-10-01以後の取引に適用するRule Setの根拠として扱う。
- Relevant section: 第57条の4第1項本文、第1項第3号から第5号。第4号は税抜価額または税込価額を税率の異なるごとに区分して合計した金額と適用税率、第5号はその区分金額ごとの消費税額等を定める。
- Citation/use condition: 法令上の記載事項と、令第70条の10へ委任していることのNormative anchorとしてリンクする。軽減対象かどうかは第1項第3号に関係するが、本Candidateは入力された分類を受け取り、法的判定をしない。
- Redistribution required?: No. 条文本文は収録せず、条番号と短い派生要約だけを記載する。

### S-002 — 消費税法施行令 第70条の10

- Classification: Normative
- Issuer: 日本国。掲載者はデジタル庁の e-Gov 法令検索。
- Title: 消費税法施行令（昭和63年政令第360号）第70条の10「適格請求書に記載すべき消費税額等の計算」
- URL / document identifier: [法令本文](https://laws.e-gov.go.jp/law/363CO0000000360)、[e-Gov法令API XML](https://laws.e-gov.go.jp/api/1/lawdata/363CO0000000360)、法令ID 363CO0000000360
- Version / publication date: 昭和63年12月30日公布。版番号の表示はないため、現行本文を2026-09-11にFresh確認した。
- Effective date: 適格請求書に係るRule Setでは2023-10-01以後の制度を対象とする。
- Relevant section: 第70条の10本文、第1号、第2号。税抜価額の率別合計に10%または8%を乗じる方法、税込価額の率別合計に10/110または8/108を乗じる方法のいずれかを示し、算出額の1円未満を処理する。
- Citation/use condition: 税抜・税込の計算式と1円未満端数処理のNormative anchorとして使う。具体的な切捨て・切上げ・四捨五入の選択はS-004と明示Rule Setに従う。
- Redistribution required?: No. 条文本文は収録しない。

### S-003 — 消費税法基本通達 1-8-15

- Classification: Interpretive
- Issuer: 国税庁
- Title: 消費税法基本通達 第8節 適格請求書発行事業者の義務、1-8-15「適格請求書に記載すべき消費税額等の計算に係る端数処理の単位」
- URL / document identifier: [第8節本文](https://www.nta.go.jp/law/tsutatsu/kihon/shohi/01/08.htm)
- Version / publication date: 消費税法基本通達は平成7年12月25日制定。現行Webページと1-8-15の制度対応注記を2026-09-11にFresh確認した。
- Effective date: 現行の適格請求書の端数処理運用を2023-10-01以後のRule Setで参照する。
- Relevant section: 1-8-15本文・注。税抜または税込の金額を税率の異なるごとに区分して合計した額を基礎にし、一の適格請求書につき税率ごとに1回処理すること、商品ごとの処理額の合計を記載額にできないことを示す。
- Citation/use condition: QIT-001、QIT-006、QIT-008のInterpretive anchor。1-8-4も、軽減対象である旨の表示が客観的に分かる必要を示すため、分類をLibraryが推測しない境界の参考にする。
- Redistribution required?: No. 通達本文を転載せず、条番号と派生ルールだけを記録する。

### S-004 — No.6371 端数計算

- Classification: Interpretive
- Issuer: 国税庁
- Title: No.6371 端数計算
- URL / document identifier: [NTA Tax Answer No.6371](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6371.htm)
- Version / publication date: ページ表示は「令和7年4月1日現在法令等」。現行ページを2026-09-11にFresh確認した。
- Effective date: ページ記載の現行法令等に基づく説明。Corpusは2023-10-01以後の適格請求書記載税額に限定する。
- Relevant section: 「適格請求書に記載すべき消費税額等の端数について」。一の適格請求書につき税率ごとに1回、切上げ・切捨て・四捨五入等は任意、商品ごとの税額を処理して合計することは不可と説明する。
- Citation/use condition: QIT-006、QIT-007、QIT-008とRoundDown/RoundUp/HalfUpのCorpus期待値の解釈根拠にする。
- Redistribution required?: No. 説明本文は転載しない。

### S-005 — 適格請求書等保存方式の概要

- Classification: Reference
- Issuer: 国税庁
- Title: 適格請求書等保存方式の概要 — インボイス制度の理解のために —
- URL / document identifier: [令和8年5月PDF](https://www.nta.go.jp/taxes/shiraberu/zeimokubetsu/shohi/keigenzeiritsu/pdf/0026004-099-02.pdf)
- Version / publication date: 令和8年5月。国税庁の現行掲載ページおよびPDF表紙を2026-09-11にFresh確認した。
- Effective date: PDFは制度開始日を令和5年10月1日として説明している。
- Relevant section: PDF 7ページ。税抜8%の27,060→2,164、税抜10%の28,158→2,815、明細単位処理の2,163/2,814、税込8%の29,223→2,164、税込10%の30,972→2,815を含む図解と説明。
- Citation/use condition: Golden Corpusの公式Anchor Caseと、率別集計・明細単位丸め不可の説明に使う。2026-09-11のFresh再確認で、PDF 7ページのアンカー数値が現行版でも同一であることを確認した。数値は必要最小限の派生データとして記録する。
- Redistribution required?: No. PDF本文や図版を再配布せず、ページ番号と計算結果だけを引用する。

### S-006 — 適格請求書等保存方式に関するQ&A

- Classification: Interpretive
- Issuer: 国税庁
- Title: 消費税の仕入税額控除制度における適格請求書等保存方式に関するQ&A
- URL / document identifier: [現行Q&A目次](https://www.nta.go.jp/taxes/shiraberu/zeimokubetsu/shohi/keigenzeiritsu/qa_01.htm)、[税額計算編PDF](https://www.nta.go.jp/taxes/shiraberu/zeimokubetsu/shohi/keigenzeiritsu/pdf/qa/01-16.pdf)
- Version / publication date: 平成30年6月開始のQ&A。現行目次は令和8年5月改訂と表示される。税額計算編を2026-09-11にFresh確認した。問118・119・126・128の個別表示には令和5年10月改訂の注記がある。
- Effective date: Q&Aはその掲載時点の現行制度説明として利用し、Corpusの計算対象はJP-QIT-2023-10-v1に固定する。
- Relevant section: Ⅴ「適格請求書等保存方式の下での税額計算」、問118、問119、問126、問128。税率別集計、税抜・税込の式、端数処理、請求書記載額を基礎にする場合を説明する。
- Citation/use condition: S-001/S-002の運用解釈を補い、Corpusの派生算術が新しい税法解釈を作っていないことを確認するために使う。
- Redistribution required?: No. Q&A本文を転載せず、問番号と短い要約だけを使う。

### S-007 — No.6303 消費税および地方消費税の税率

- Classification: Reference
- Issuer: 国税庁
- Title: No.6303 消費税および地方消費税の税率
- URL / document identifier: [NTA Tax Answer No.6303](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6303.htm)
- Version / publication date: ページ表示は「令和7年4月1日現在法令等」。現行ページを2026-09-11にFresh確認した。
- Effective date: 現行の標準税率10%・軽減税率8%の説明を、明示的な2023-10-01 Rule Setの率マッピングへ転記する。
- Relevant section: 税率表。合計10%（消費税7.8%＋地方消費税2.2%）と合計8%（6.24%＋1.76%）を示す。軽減対象の法律上の判定そのものは本Candidateの入力責務外とする。
- Citation/use condition: JP-QIT-2023-10-v1の例示率を確認する補助資料。Categoryから率を逆算する根拠にはしない。
- Redistribution required?: No。税率表の全文転載はしない。

### S-008 — 国税庁サイトの利用条件

- Classification: Reference
- Issuer: 国税庁
- Title: 利用規約・免責事項・著作権
- URL / document identifier: [国税庁利用規約・免責事項・著作権](https://www.nta.go.jp/chuijiko/copy.htm)
- Version / publication date: ページ上に版日表示なし。内容を2026-09-11にFresh確認した。
- Effective date: 現行Webサイトでの利用条件。
- Relevant section: 利用規約、公共データ利用規約に関する重要情報の「出典の記載」「編集・加工等を行ったことの記載」。原サイトへのリンク設定は自由とする案内も確認した。
- Citation/use condition: 本RepositoryはNTA資料を加工して作成した派生仕様であるため、各SourceのURLを残し、公式作成物と誤認されないよう派生物であることを明示する。ロゴ・図版は使わない。
- Redistribution required?: No。公式本文・図版を再配布しない。ただし利用する公式情報には出典を付し、編集・加工したことを記載する。

## Source counts

~~~yaml
COUNT: 8
NORMATIVE: 2
INTERPRETIVE: 3
REFERENCE: 3
~~~
