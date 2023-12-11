---
title: TiDBでのCRUD SQL
summary: TiDBのCURD SQLの簡単な紹介。
---

# TiDBでのCRUD SQL

このドキュメントはTiDBのCURD SQLの使い方について簡単に紹介します。

## 開始する前に

TiDBクラスタに接続していることを確認してください。接続していない場合は、[TiDBサーバーレスクラスタの構築方法](/develop/dev-guide-build-cluster-in-cloud.md#step-1-create-a-tidb-serverless-cluster)を参照してTiDBサーバーレスクラスタを作成してください。

## TiDBでSQLを探索する

> **注意:**
>
> このドキュメントは[TiDBでSQLを探索する](/basic-sql-operations.md)を参照して、その内容を簡略化しています。詳細については、[TiDBでSQLを探索する](/basic-sql-operations.md)を参照してください。

TiDBはMySQLと互換性があり、ほとんどの場合、直接MySQLステートメントを使用できます。サポートされていない機能については、[MySQLとの互換性](/mysql-compatibility.md#unsupported-features)を参照してください。

SQLを実験したり、MySQLクエリとTiDBの互換性をテストしたりするには、[TiDB Playground](https://play.tidbcloud.com/?utm_source=docs&utm_medium=basic-sql-operations)を試すことができます。また、まずTiDBクラスタをデプロイし、その後にSQLステートメントを実行することもできます。

このページでは、DDL、DML、およびCRUD操作などの基本的なTiDB SQLステートメントについて説明します。TiDBステートメントの完全なリストについては、[TiDB SQL構文ダイアグラム](https://pingcap.github.io/sqlgram/)を参照してください。

## カテゴリ

SQLは、その機能に応じて以下の4つに分類されます:

- **DDL（データ定義言語）**: データベース、テーブル、ビュー、およびインデックスなどのデータベースオブジェクトを定義するために使用されます。

- **DML（データ操作言語）**: アプリケーション関連のレコードを操作するために使用されます。

- **DQL（データクエリ言語）**: 条件に一致するレコードをクエリするために使用されます。

- **DCL（データ制御言語）**: アクセス権限やセキュリティレベルを定義するために使用されます。

以下では、主にDMLとDQLについて紹介します。DDLやDCLについて詳しくは、[TiDBでSQLを探索する](/basic-sql-operations.md)や[TiDB SQLの構文の詳細な説明](https://pingcap.github.io/sqlgram/)を参照してください。

## データ操作言語

一般的なDML機能には、テーブルレコードの追加、変更、削除があります。それに対応するコマンドは`INSERT`、`UPDATE`、および`DELETE`です。

テーブルにデータを挿入するには、`INSERT`ステートメントを使用してください:

```sql
INSERT INTO person VALUES(1,'tom','20170912');
```

いくつかのフィールドのデータを含むレコードをテーブルに挿入するには、`INSERT`ステートメントを使用してください:

```sql
INSERT INTO person(id,name) VALUES('2','bob');
```

テーブル内のレコードの一部のフィールドを更新するには、`UPDATE`ステートメントを使用してください:

```sql
UPDATE person SET birthday='20180808' WHERE id=2;
```

テーブル内のデータを削除するには、`DELETE`ステートメントを使用してください:

```sql
DELETE FROM person WHERE id=2;
```

> **注意:**
>
> フィルターとして`WHERE`句がない`UPDATE`および`DELETE`ステートメントは、テーブル全体に対して操作します。

## データクエリ言語

DQLは、テーブルまたは複数のテーブルから必要なデータ行を取得するために使用されます。

テーブル内のデータを表示するには、`SELECT`ステートメントを使用してください:

```sql
SELECT * FROM person;
```

特定の列をクエリするには、`SELECT`キーワードの後に列名を追加してください:

```sql
SELECT name FROM person;
```

その結果は以下のようになります:

```
+------+
| name |
+------+
| tom  |
+------+
1 rows in set (0.00 sec)
```

すべての条件に一致するレコードをフィルタリングし、その結果を返すには、`WHERE`句を使用してください:

```sql
SELECT * FROM person WHERE id < 5;
```