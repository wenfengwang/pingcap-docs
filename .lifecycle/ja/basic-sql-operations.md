---
title: TiDBでSQLを探索する
summary: TiDBデータベースの基本的なSQLステートメントについて学びます。
aliases: ['/docs/dev/basic-sql-operations/','/docs/dev/how-to/get-started/explore-sql/']
---

# TiDBでSQLを探索する

TiDBはMySQLと互換性があり、ほとんどの場合、MySQLステートメントを直接使用できます。サポートされていない機能については、[MySQLとの互換性](/mysql-compatibility.md#unsupported-features)を参照してください。

<CustomContent platform="tidb">

SQLを試すためにTiDB互換性のあるMySQLクエリをテストしたり、TiDBクラスタをデプロイしてSQLステートメントを実行することができます。[TiDB Playground](https://play.tidbcloud.com/?utm_source=docs&utm_medium=basic-sql-operations)を試してみてください。

</CustomContent>

このページでは、DDL、DML、CRUDなど、基本的なTiDBのSQLステートメントについて説明します。TiDBステートメントの完全なリストについては、[TiDB SQL構文ダイアグラム](https://pingcap.github.io/sqlgram/)を参照してください。

## カテゴリ

SQLは以下の4つの機能に応じて次のように分類されます。

- DDL（データ定義言語）: データベースオブジェクト（データベース、テーブル、ビュー、インデックスなど）を定義するために使用されます。

- DML（データ操作言語）: アプリケーション関連のレコードを操作するために使用されます。

- DQL（データ問い合わせ言語）: 条件に基づいてレコードをクエリするために使用されます。

- DCL（データ制御言語）: アクセス権限とセキュリティレベルを定義するために使用されます。

一般的なDDLの機能には、オブジェクト（テーブルやインデックスなど）の作成、変更、削除があります。対応するコマンドは `CREATE`, `ALTER`, `DROP` です。

## データベースの表示、作成、および削除

TiDBのデータベースはテーブルやインデックスなどのオブジェクトのコレクションと見なすことができます。

データベースの一覧を表示するには、`SHOW DATABASES` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
SHOW DATABASES;
```

`mysql` という名前のデータベースを使用するには、次のステートメントを使用します：

{{< copyable "sql" >}}

```sql
USE mysql;
```

あるデータベース内のすべてのテーブルを表示するには、`SHOW TABLES` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
SHOW TABLES FROM mysql;
```

データベースを作成するには、`CREATE DATABASE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
CREATE DATABASE db_name [options];
```

`samp_db` という名前のデータベースを作成するには、次のステートメントを使用します：

{{< copyable "sql" >}}

```sql
CREATE DATABASE IF NOT EXISTS samp_db;
```

データベースが存在する場合にエラーを回避するには、`IF NOT EXISTS` を追加します。

データベースを削除するには、`DROP DATABASE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
DROP DATABASE samp_db;
```

## テーブルの作成、表示、および削除

テーブルを作成するには、`CREATE TABLE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
CREATE TABLE table_name column_name data_type constraint;
```

たとえば、`person` という名前のテーブルを作成し、その中に番号、名前、誕生日などのフィールドを含めるには、次のステートメントを使用します：

{{< copyable "sql" >}}

```sql
CREATE TABLE person (
    id INT(11),
    name VARCHAR(255),
    birthday DATE
    );
```

テーブル（DDL）を作成するステートメントを表示するには、`SHOW CREATE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
SHOW CREATE table person;
```

テーブルを削除するには、`DROP TABLE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
DROP TABLE person;
```

## インデックスの作成、表示、および削除

インデックスは、インデックス化された列をクエリする際の検索を高速化するために使用されます。値が一意でない列のインデックスを作成するには、`CREATE INDEX` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
CREATE INDEX person_id ON person (id);
```

または、`ALTER TABLE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
ALTER TABLE person ADD INDEX person_id (id);
```

値が一意である列のユニークなインデックスを作成するには、`CREATE UNIQUE INDEX` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
CREATE UNIQUE INDEX person_unique_id ON person (id);
```

または、`ALTER TABLE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
ALTER TABLE person ADD UNIQUE person_unique_id (id);
```

テーブル内のすべてのインデックスを表示するには、`SHOW INDEX` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
SHOW INDEX FROM person;
```

インデックスを削除するには、`DROP INDEX` または `ALTER TABLE` ステートメントを使用します。`DROP INDEX` は `ALTER TABLE` でネストできます：

{{< copyable "sql" >}}

```sql
DROP INDEX person_id ON person;
```

{{< copyable "sql" >}}

```sql
ALTER TABLE person DROP INDEX person_unique_id;
```

> **注意:**
> 
> DDL操作はトランザクションではありません。DDL操作を実行する際に `COMMIT` ステートメントを実行する必要はありません。

## データの挿入、更新、および削除

一般的なDML機能には、テーブルレコードの追加、変更、削除があります。対応するコマンドは `INSERT`, `UPDATE`, `DELETE` です。

テーブルにデータを挿入するには、`INSERT` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
INSERT INTO person VALUES(1,'tom','20170912');
```

いくつかのフィールドのデータを含むレコードをテーブルに挿入するには、`INSERT` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
INSERT INTO person(id,name) VALUES('2','bob');
```

テーブル内のレコードの一部のフィールドを更新するには、`UPDATE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
UPDATE person SET birthday='20180808' WHERE id=2;
```

テーブル内のデータを削除するには、`DELETE` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
DELETE FROM person WHERE id=2;
```

> **注意:**
> 
> `UPDATE` ステートメントと `DELETE` ステートメントは、フィルターとして `WHERE` 句のない場合、テーブル全体に対して操作します。

## データのクエリ

DQLは、テーブルまたは複数のテーブルから必要なデータ行を取得するために使用されます。

テーブル内のデータを表示するには、`SELECT` ステートメントを使用します：

{{< copyable "sql" >}}

```sql
SELECT * FROM person;
```

特定の列をクエリするには、`SELECT` キーワードの後に列名を追加します：

{{< copyable "sql" >}}

```sql
SELECT name FROM person;
```

```sql
+------+
| name |
+------+
| tom  |
+------+
1 行が選択されました (0.00 sec)
```

条件に一致するすべてのレコードをフィルタリングし、結果を返すために `WHERE` 句を使用します：

{{< copyable "sql" >}}

```sql
SELECT * FROM person where id<5;
```

## ユーザーの作成、権限の付与、および削除

DCLは通常、ユーザーの作成や削除、ユーザー権限の管理に使用されます。

ユーザーを作成するには、`CREATE USER` ステートメントを使用します。次の例では、パスワードが `123456` の `tiuser` という名前のユーザーを作成します：

{{< copyable "sql" >}}

```sql
CREATE USER 'tiuser'@'localhost' IDENTIFIED BY '123456';
```

`tiuser` に `samp_db` データベース内のテーブルを取得する権限を付与するには：

{{< copyable "sql" >}}

```sql
GRANT SELECT ON samp_db.* TO 'tiuser'@'localhost';
```

`tiuser` の権限を確認するには：

{{< copyable "sql" >}}

```sql
SHOW GRANTS for tiuser@localhost;
```

`tiuser` を削除するには：

{{< copyable "sql" >}}

```sql
DROP USER 'tiuser'@'localhost';
```