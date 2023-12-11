---
title: `AS OF TIMESTAMP` 句を使用して履歴データを読む
summary: `AS OF TIMESTAMP` ステートメント句を使用して履歴データを読む方法について学びます。
---

# `AS OF TIMESTAMP` 句を使用して履歴データを読む

このドキュメントでは、`AS OF TIMESTAMP` 句を使用して履歴データを読む[遅延読み込み](/stale-read.md)機能について、TiDBでの特定の使用例と履歴データの保存戦略について説明します。

TiDBでは、特別なクライアントやドライバを必要とせず、標準のSQLインターフェースを介して `AS OF TIMESTAMP` SQL句を使用して履歴データを読むことができます。データが更新または削除された後、このSQLインターフェースを使用して更新または削除前の履歴データを読むことができます。

> **注記:**
>
> 履歴データを読む際、TiDBは古いテーブル構造のデータを返します。現在のテーブル構造が異なっていても、古いテーブル構造のデータを返します。

## 構文

以下の3つの方法で `AS OF TIMESTAMP` 句を使用できます。

- [`SELECT ... FROM ... AS OF TIMESTAMP`](/sql-statements/sql-statement-select.md)
- [`START TRANSACTION READ ONLY AS OF TIMESTAMP`](/sql-statements/sql-statement-start-transaction.md)
- [`SET TRANSACTION READ ONLY AS OF TIMESTAMP`](/sql-statements/sql-statement-set-transaction.md)

特定の時間点を指定したい場合、`AS OF TIMESTAMP` 句で日時の値を設定するか、時刻関数を使用できます。日時の形式は "2016-10-08 16:45:26.999" のように、ミリ秒を最小の時間単位として指定しますが、通常は "2016-10-08 16:45:26" のように秒の時間単位で日時を指定することができます。`NOW(3)` 関数を使用してミリ秒までの現在時刻を取得することもできます。数秒前のデータを読み取りたい場合は、`NOW() - INTERVAL 10 SECOND` などの式を使用することが**推奨**されます。

時間範囲を指定したい場合は、`TIDB_BOUNDED_STALENESS()` 関数を句で使用できます。この関数を使用すると、TiDBは指定された時間範囲内で適切なタイムスタンプを選択します。"適切" とは、このタイムスタンプより前に開始されたがアクセスされたレプリカで確定されていないトランザクションがないことを意味し、つまり、TiDBはアクセスされたレプリカで読み取り操作を実行し、かつ読み取り操作がブロックされないことを示します。この関数を呼び出すには、`TIDB_BOUNDED_STALENESS(t1, t2)` を使用します。`t1` と `t2` は、日時値または時刻関数を使用して指定できる時間範囲の両端を表します。

以下に `AS OF TIMESTAMP` 句の例を示します。

- `AS OF TIMESTAMP '2016-10-08 16:45:26'`: TiDBに対して、2016年10月8日の16:45:26での最新データを読むよう指示します。
- `AS OF TIMESTAMP NOW() - INTERVAL 10 SECOND`: TiDBに対して、10秒前に保存された最新データを読むよう指示します。
- `AS OF TIMESTAMP TIDB_BOUNDED_STALENESS('2016-10-08 16:45:26', '2016-10-08 16:45:29')`: TiDBに対して、2016年10月8日の16:45:26から16:45:29の時間範囲内で、可能な限り新しいデータを読むよう指示します。
- `AS OF TIMESTAMP TIDB_BOUNDED_STALENESS(NOW() - INTERVAL 20 SECOND, NOW())`: TiDBに対して、20秒前から現在までの時間範囲で、可能な限り新しいデータを読むよう指示します。

> **注記:**
>
> `AS OF TIMESTAMP` 句の一般的な使用法は、特定のタイムスタンプを指定することに加えて、数秒前のデータの読み取りです。この方法を使用する場合、5秒より古い履歴データを読むことが推奨されます。
>
> 遅延読み込みを使用する場合、TiDBとPDノードにNTPサービスをデプロイする必要があります。これにより、TiDBが指定したタイムスタンプを最新のTSO割り当ての進行よりも進ませる（数秒前のタイムスタンプなど）ことや、GCセーフポイントのタイムスタンプよりも遅れる状況を避けることができます。指定したタイムスタンプがサービス範囲を超えてしまった場合、TiDBはエラーを返します。
>
> 遅延読み込みデータのレイテンシを低減し、適時なデータを取得するために、TiKVの `advance-ts-interval` 構成項目を変更することができます。詳細は[遅延読み込みのレイテンシを低減する](/stale-read.md#reduce-stale-read-latency)を参照してください。

## 使用例

このセクションでは、いくつかの例を使用して `AS OF TIMESTAMP` 句の使用方法を説明します。まず、データの復元を準備する方法を紹介し、その後、それぞれの方法で `AS OF TIMESTAMP` を `SELECT`、`START TRANSACTION READ ONLY AS OF TIMESTAMP`、`SET TRANSACTION READ ONLY AS OF TIMESTAMP` に使用する方法を示します。

### データサンプルの準備

データを復元するために、まずテーブルを作成し、いくつかのデータを挿入します。

```sql
create table t (c int);
```

```
クエリは成功しました。影響を受けた行: 0(実行時間: 0.01秒)
```

```sql
insert into t values (1), (2), (3);
```

```
クエリは成功しました。影響を受けた行: 3(実行時間: 0.00秒)
```

テーブル内のデータを表示します。

```sql
select * from t;
```

```
+------+
| c    |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)
```

現在の時刻を表示します。

```sql
select now();
```

```
+---------------------+
| now()               |
+---------------------+
| 2021-05-26 16:45:26 |
+---------------------+
1行のセット(実行時間: 0.00秒)
```

1行のデータを更新します。

```sql
update t set c=22 where c=2;
```

```
クエリは成功しました。影響を受けた行: 1(実行時間: 0.00秒)
```

行のデータが更新されたことを確認します。

```sql
select * from t;
```

```
+------+
| c    |
+------+
|    1 |
|   22 |
|    3 |
+------+
3 rows in set (0.00 sec)
```

### `SELECT` ステートメントを使用して履歴データを読む

[`SELECT ... FROM ... AS OF TIMESTAMP`](/sql-statements/sql-statement-select.md) ステートメントを使用して、過去の時間点からデータを読み取ることができます。

```sql
select * from t as of timestamp '2021-05-26 16:45:26';
```

```
+------+
| c    |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)
```

> **注記:**
>
> `SELECT` ステートメントで複数のテーブルを読み取る場合、TIMESTAMP EXPRESSION のフォーマットが一致していることを確認する必要があります。例えば、`select * from t as of timestamp NOW() - INTERVAL 2 SECOND, c as of timestamp NOW() - INTERVAL 2 SECOND;` のように適切なテーブルの `AS OF` 情報を `SELECT` ステートメントで指定する必要があります。そうしないと、`SELECT` ステートメントはデフォルトで最新データを読み取ります。

### `START TRANSACTION READ ONLY AS OF TIMESTAMP` ステートメントを使用して履歴データを読む

[`START TRANSACTION READ ONLY AS OF TIMESTAMP`](/sql-statements/sql-statement-start-transaction.md) ステートメントを使用して、過去の時間点を基準に読み取り専用トランザクションを開始することができます。トランザクションは指定された時間の履歴データを読み取ります。

```sql
start transaction read only as of timestamp '2021-05-26 16:45:26';
```

```
クエリは成功しました。影響を受けた行: 0(実行時間: 0.00秒)
```

```sql
select * from t;
```

```
+------+
| c    |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 rows in set (0.00 sec)
```

```sql
commit;
```

```
クエリは成功しました。影響を受けた行: 0(実行時間: 0.00秒)
```

トランザクションが確定された後、最新データを読み取ることができます。

```sql
select * from t;
```

```
+------+
| c    |
+------+
|    1 |
|   22 |
|    3 |
+------+
3 rows in set (0.00 sec)
```

> **注記:**
>
> `START TRANSACTION READ ONLY AS OF TIMESTAMP` ステートメントでトランザクションを開始する場合、それは読み取り専用トランザクションです。このトランザクションでは書き込み操作は拒否されます。

### `SET TRANSACTION READ ONLY AS OF TIMESTAMP` ステートメントを使用して履歴データを読む

[`SET TRANSACTION READ ONLY AS OF TIMESTAMP`](/sql-statements/sql-statement-set-transaction.md) ステートメントを使用して、指定された過去の時間点を基準に次のトランザクションを読み取り専用トランザクションとして設定することができます。トランザクションは指定された時間の履歴データを読み取ります。

```sql
set transaction read only as of timestamp '2021-05-26 16:45:26';
```

```
クエリは成功しました。影響を受けた行: 0(実行時間: 0.00秒)
```

```sql
begin;
```

```
クエリは成功しました。影響を受けた行: 0(実行時間: 0.00秒)
```

```sql
```sql
select * from t;
```

```
+------+
| c    |
+------+
|    1 |
|    2 |
|    3 |
+------+
3 行が選択されました (0.00 秒)
```

```sql
commit;
```

```
クエリが正常に完了しました。 0 行が影響を受けました (0.00 秒)
```

トランザクションがコミットされた後、最新のデータを読むことができます。

```sql
select * from t;
```

```
+------+
| c    |
+------+
|    1 |
|   22 |
|    3 |
+------+
3 行が選択されました (0.00 秒)
```

> **注意:**
>
> `SET TRANSACTION READ ONLY AS OF TIMESTAMP` ステートメントでトランザクションを開始した場合、読み取り専用トランザクションです。このトランザクションでは書き込み操作は拒否されます。