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
