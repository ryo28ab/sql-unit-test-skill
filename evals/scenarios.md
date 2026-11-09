# 動作確認用シナリオ

それぞれ独立した会話で実施する手動評価です。DBテストや自動評価プログラムではありません。

## 1. 実装だけがある

### 依頼

SQL Serverの取得ストアドのテスト仕様書を作ってください。実装は`SELECT OrderId FROM Orders WHERE UserId=@UserId AND Status='Done'`です。他の資料はまだありません。

### 確認

- 仕様と必要なDDL等を理由付きで質問する。
- 既に分かるDB製品を再質問しない。
- 回答前に、架空のDDL・状態値・期待結果を置いたケース一覧を作らない。
- 回答がない状態から勝手に暫定設計へ切り替えない。

## 2. 必要な情報が揃っている

### 依頼

SQL Serverのdbo.FindOrdersのテスト仕様書を会話内で作ってください。
仕様：@UserId intで指定したユーザーのDone注文のOrderIdだけを取得。CreatedAt降順。同日時内の順序は不問。対象なしと引数NULLは0件でエラーなし。
実装：CREATE PROCEDURE dbo.FindOrders @UserId int AS SELECT OrderId FROM Orders WHERE UserId=@UserId AND Status='Done' ORDER BY CreatedAt DESC。
DDL：CREATE TABLE Orders(OrderId int NOT NULL PRIMARY KEY, UserId int NOT NULL, Status varchar(10) NOT NULL CHECK(Status IN ('Done','Pending')), CreatedAt datetime2(0) NOT NULL)。
単体の境界はこのストアドとOrders実表。各ケースは隔離DBの空テーブルから開始。ほかの依存、案件独自ルール、ひな形はありません。DB実行とファイル作成は不要です。

### 確認

- 任意のひな形や回答済みのルールを要求して作業を止めない。
- 正常、他ユーザー、未完了、0件、複数件、NULL引数、順序等を根拠付きで設計する。
- 同日時内の並び順を固定しない。
- NULL禁止列にNULLを投入するケースを作らない。
- 件数だけでなく、具体的なOrderIdと重複個数を期待結果にする。

## 3. 仕様と実装が矛盾する

### 依頼

シナリオ2の資料のうち、仕様だけを「DoneとPendingの両方を取得」に変更して設計してください。実装は変更されていません。

### 確認

- 不一致を直接質問し、どちらが今回の採用仕様かを確認する。
- 実装を自動的に正解にしない。
- 回答に依存するケースの期待結果を、回答前に確定しない。

## 4. 更新処理の資料が揃っている

### 依頼

SQL Serverのdbo.CompleteOrdersのテスト仕様書を会話内で作ってください。
仕様：@UserId intで指定したユーザーのPending注文をすべてDoneにし、CompletedAtに実行時刻を設定する。他ユーザーの注文とDone済みの注文は変更しない。更新件数を戻り値で返す。対象なしは0を返しエラーなし。
実装：CREATE PROCEDURE dbo.CompleteOrders @UserId int AS BEGIN SET NOCOUNT ON; UPDATE Orders SET Status='Done', CompletedAt=SYSDATETIME() WHERE UserId=@UserId AND Status='Pending'; RETURN @@ROWCOUNT; END
DDL：CREATE TABLE Orders(OrderId int NOT NULL PRIMARY KEY, UserId int NOT NULL, Status varchar(10) NOT NULL CHECK(Status IN ('Done','Pending')), CompletedAt datetime2(7) NULL)。トリガーなし。
単体の境界はこのストアドとOrders実表。各ケースは隔離DBの空テーブルから開始。ほかの依存、案件独自ルール、ひな形はありません。DB実行とファイル作成は不要です。

### 確認

- 回答済みの情報を再質問せず、設計を開始する。
- 同じケース内に、更新される行（対象ユーザーのPending）と更新されない行（他ユーザーのPending、対象ユーザーのDone）を置く。
- 期待結果に実行後のOrdersの状態を、更新されない行の値を含めて具体的に示す。
- CompletedAtを固定値で決め打ちせず、実行前後の時刻の範囲など確認方法を示す。
- 戻り値（更新件数）を0／1／複数件で確認する。

## 5. 削除処理で副作用の定義が不足している

### 依頼

SQL Serverのdbo.DeleteCustomerのテスト仕様書を作ってください。仕様：@CustomerIdの顧客を削除する。実装：DELETE FROM Customers WHERE CustomerId=@CustomerId。CustomersのDDLは添付のとおりで、Ordersテーブルから外部キーで参照されています。OrdersのDDLとトリガーの有無はまだ渡していません。

### 確認

- 参照されている顧客を削除した場合の扱い（FK違反のエラー／カスケード削除）を決めるため、OrdersのDDLとトリガーの有無を質問する。
- 子テーブルの動作を推測して、期待結果を確定しない。
- 質問は理由付きで3問以内にまとめる。

## 6. 複数テーブル更新の途中失敗

### 依頼

ヘッダーと明細を1トランザクションで登録するストアドの資料（仕様・実装・DDL）を渡したうえで、テスト仕様書を作ってください。仕様には「明細の登録に失敗した場合はヘッダーも登録しない」とあり、実装はTRY／CATCHでROLLBACKしています。

### 確認

- 明細の登録で失敗させるケースを作り、ヘッダーも残らないことを実行後の状態で確認する。
- 失敗を起こす手段に、制約の無効化を使わない（処理側から制約違反を発生させる）。
- エラーの内容と、トランザクションが残っていないこと（@@TRANCOUNT）の確認方法を示す。
