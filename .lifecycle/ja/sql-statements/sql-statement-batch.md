---
title: BATCH
summary: TiDBデータベースのBATCHの使用方法の概要。

# BATCH

`BATCH`構文は、TiDBでDMLステートメントを複数のステートメントに分割して実行します。これは、トランザクションの原子性と分離性を**保証しない**ことを意味します。したがって、これは「非トランザクション」ステートメントです。

現在、`INSERT`、`REPLACE`、`UPDATE`、`DELETE`が`BATCH`でサポートされています。

`BATCH`構文は、列に基づいてDMLステートメントを複数の範囲に分割して実行します。各範囲では、1つのSQLステートメントが実行されます。

使用方法や制限の詳細については、[非トランザクショナルDMLステートメント](/non-transactional-dml.md)を参照してください。

`BATCH`ステートメントで複数のテーブルを結合する場合、曖昧さを避けるために列の完全なパスを指定する必要があります。

```sql
BATCH ON test.t2.id LIMIT 1 INSERT INTO t SELECT t2.id, t2.v, t3.v FROM t2 JOIN t3 ON t2.k = t3.k;
```

上記のステートメントでは、`test.t2.id`を分割する列として指定しています。もし以下のように`id`を使用すると、エラーが報告されます。

```sql
BATCH ON id LIMIT 1 INSERT INTO t SELECT t2.id, t2.v, t3.v FROM t2 JOIN t3 ON t2.k = t3.k;

Non-transactional DML, shard column must be fully specified
```

## 概要

```ebnf+diagram
NonTransactionalDMLStmt ::=
    'BATCH' ( 'ON' ColumnName )? 'LIMIT' NUM DryRunOptions? ShardableStmt

DryRunOptions ::=
    'DRY' 'RUN' 'QUERY'?

ShardableStmt ::=
    DeleteFromStmt
|   UpdateStmt
|   InsertIntoStmt
|   ReplaceIntoStmt
```

## MySQL互換性

`BATCH`構文はTiDB固有であり、MySQLと互換性がありません。

## 関連項目

* [非トランザクショナルDMLステートメント](/non-transactional-dml.md)