---
title: インポートのキャンセル
summary: TiDB での CANCEL IMPORT の使用方法の概要。

# キャンセル インポート

`CANCEL IMPORT` ステートメントは、TiDB で作成されたデータインポートジョブをキャンセルするために使用されます。

<!-- TiDB Cloud へのサポートノート：

この TiDB ステートメントは TiDB Cloud には適用されません。

-->

## 必要な権限

データインポートジョブをキャンセルするには、そのジョブの作成者であるか `SUPER` 権限を持っている必要があります。

## 概要

```ebnf+diagram
CancelImportJobsStmt ::=
    'CANCEL' 'IMPORT' 'JOB' JobID
```

## 例

ID が `1` のインポートジョブをキャンセルするには、次のステートメントを実行します。

```sql
CANCEL IMPORT JOB 1;
```

出力は次のようになります。

```
Query OK, 0 rows affected (0.01 sec)
```

## MySQL 互換性

このステートメントは MySQL 構文の TiDB 拡張です。

## 関連項目

* [`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)
* [`SHOW IMPORT JOB`](/sql-statements/sql-statement-show-import-job.md)