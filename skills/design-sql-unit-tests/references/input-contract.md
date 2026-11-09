# 実行時に注入する情報

文章・貼り付けSQL・添付・アクセス可能なファイル参照をそのまま受け取り、次の情報に正規化せよ。以下のYAML提出を利用者に強制するな。

```yaml
target:
  database: SQL Server # 初期値。別製品なら方言を明示して適用可否を判断する。
  object_name: null
  object_kind: null # procedure / inline_table_function / multistatement_table_function / scalar_function
  scope: retrieval
request:
  mode: design # scaffold / design / review
  language: ja
  output_format: null # 未指定は会話内。xlsx等は指定された場合に作成。
  requested_sql: [] # 必要に応じて setup / invoke / assert / cleanup
sources: []
# 各sourceには以下を指定可能。本文の直接入力と参照のどちらでもよい。
# - id: SPEC-01
#   role: specification
#   location: 文書パス、添付名、URL、または「このメッセージ」
#   section: 見出し・行・ページ
#   revision: null
#   authority: null # 今回の採用版等。推測で埋めない。
#   content: null
# role: specification / implementation / schema / dependency /
#       project_rules / output_template / approved_example / review_feedback
execution_context:
  fixture_strategy: null # 実テーブル／隔離DB／代替依存など
  dependency_scope: null # 単体の境界
  available_identity: null # 実行ユーザー依存がある場合のみ
  clock_control: null # 現在時刻依存がある場合のみ
  initialization_and_cleanup: null
```

## 入力の役割と不足時の扱い

| 情報 | 確認する内容 | 不足時の対応 |
|---|---|---|
| 仕様 | 入出力、業務条件、0件・NULL・エラー・順序・重複の契約 | 必要な契約を質問し、依存する期待結果の作成は回答まで待つ |
| 実装 | 対象全体の定義、引数・型・既定値、依存先 | 完全な定義を質問する。仕様ベースだけで進める指定があれば範囲を明記する |
| DDL | 型・長さ・精度、NULL、PK・FK・UNIQUE・CHECK・既定値 | 必要な定義を質問し、該当データと境界値の作成は回答まで待つ |
| 案件ルール | 網羅基準、単体の境界、対象外基準 | 合否・範囲に影響する部分は質問する。指定なしと分かれば標準方針を明記する |
| ひな形・承認済み例 | 列構成、粒度、データと証跡の示し方 | 標準の出力契約を使う |
| 過去の指摘 | 繰り返す指摘とその適用範囲 | 共通観点だけで設計する |
| 依存定義・環境 | ビュー・関数・権限・時刻等の振る舞い | 関連する定義・境界を質問し、依存する設計は回答まで待つ |

## 後から渡すための依頼文

```text
このskillで次の取得処理の単体テスト仕様書を作ってください。
対象：［オブジェクト名・種別］
仕様：［貼り付け／添付／参照］
実装：［貼り付け／添付／参照］
テーブル定義・依存定義：［貼り付け／添付／参照］
案件ルール・ひな形・レビュー指摘：［あれば指定］
出力形式：［会話内の表／Excel等］
テストの境界・データ準備方法：［分かれば指定］
不足情報や矛盾があれば先に質問し、回答が必要な部分は回答後に設計してください。
```

空欄は未提供として扱え。NULL許容や制約なしという事実に読み替えるな。

## 聞き返す形式

受領済みの情報を一文で示し、未提供のうち今回の設計に必要なものを質問せよ。例えば仕様がなく実装・DDLがある場合は、次のように聞け。

> 実装SQLとテーブル定義は確認できました。期待結果を決めるため、取得対象・除外条件・返却内容の仕様を教えてください。正式な仕様書がなければ、文章での説明で構いません。

質問の末尾に未承認の仮定によるケース一覧を続けるな。資料が用意できない場合は、次の回答で許容する設計範囲を確認せよ。
