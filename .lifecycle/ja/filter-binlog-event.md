---
title: バイナリログイベントのフィルタリング
summary: データを移行する際のバイナリログイベントのフィルタリング方法を学びます。

# バイナリログイベントのフィルタリング

このドキュメントでは、DMを使用して連続的な増分データレプリケーションを実行する際のバイナリログイベントのフィルタリング方法について説明します。詳細なレプリケーション手順については、以下のシナリオ別のドキュメントを参照してください。

- [MySQLからTiDBへの小規模データ移行](/migrate-small-mysql-to-tidb.md)
- [MySQLからTiDBへの大規模データ移行](/migrate-large-mysql-to-tidb.md)
- [小規模MySQLシャードのTiDBへの移行およびマージ](/migrate-small-mysql-shards-to-tidb.md)
- [大規模MySQLシャードのTiDBへの移行およびマージ](/migrate-large-mysql-shards-to-tidb.md)

## 構成

バイナリログイベントフィルタを使用するには、以下に示すようにDMのタスク構成ファイルに`filter`を追加します。

```yaml
filters:
  rule-1:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    events: ["truncate table", "drop table"]
    sql-pattern: ["^DROP\\s+PROCEDURE", "^CREATE\\s+PROCEDURE"]
    action: Ignore
```

- `schema-pattern`/`table-pattern`: 一致するスキーマまたはテーブルをフィルタリング
- `events`: バイナリログイベントをフィルタリング。サポートされているイベントは以下の表にリストされています。

  | イベント           | カテゴリ | 説明                       |
  | --------------- | ---- | --------------------------|
  | all             |      | すべてのイベントを含む       |
  | all dml         |      | すべてのDMLイベントを含む    |
  | all ddl         |      | すべてのDDLイベントを含む    |
  | none            |      | イベントを含まない          |
  | none ddl        |      | すべてのDDLイベントを除外     |
  | none dml        |      | すべてのDMLイベントを除外     |
  | insert          | DML  | 挿入DMLイベント           |
  | update          | DML  | 更新DMLイベント           |
  | delete          | DML  | 削除DMLイベント           |
  | create database | DDL  | データベース作成イベント   |
  | drop database   | DDL  | データベース削除イベント   |
  | create table    | DDL  | テーブル作成イベント       |
  | create index    | DDL  | インデックス作成イベント   |
  | drop table      | DDL  | テーブル削除イベント       |
  | truncate table  | DDL  | テーブル切り捨てイベント   |
  | rename table    | DDL  | テーブル名変更イベント     |
  | drop index      | DDL  | インデックス削除イベント   |
  | alter table     | DDL  | テーブル変更イベント       |

- `sql-pattern`: 指定されたDDL SQL文をフィルタリング。一致するルールは正規表現を使用することができます。
- `action`: `Do` または `Ignore` 

    - `Do`: 許可リスト。バイナリログイベントは、次の2つの条件のいずれかを満たす場合に複製されます。

        - イベントがルール設定に一致する場合。
        - sql-patternが指定されており、イベントのSQL文がsql-patternのオプションのいずれかに一致する場合。

    - `Ignore`: ブロックリスト。バイナリログイベントは、次の2つの条件のいずれかを満たす場合にフィルタリングされます。

        - イベントがルール設定に一致する場合。
        - sql-patternが指定されており、イベントのSQL文がsql-patternのオプションのいずれかに一致する場合。

    `Do` と `Ignore` の両方が構成されている場合、`Ignore` が `Do` よりも高い優先度を持ちます。つまり、`Ignore` と `Do` の両方の条件を満たすイベントはフィルタリングされます。

## アプリケーションシナリオ

このセクションでは、バイナリログイベントフィルタのアプリケーションシナリオについて説明します。

### すべてのシャーディング削除操作をフィルタリング

すべての削除操作をフィルタリングするには、以下のように `filter-table-rule` と `filter-schema-rule` を構成します。

```
filters:
  filter-table-rule:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    events: ["truncate table", "drop table", "delete"]
    action: Ignore
  filter-schema-rule:
    schema-pattern: "test_*"
    events: ["drop database"]
    action: Ignore
```

### シャーディングスキーマおよびテーブルのDML操作のみを移行

DML文のみを複製するには、2つの `バイナリログイベントフィルタルール` を以下のように構成します。

```
filters:
  do-table-rule:
    schema-pattern: "test_*"
    table-pattern: "t_*"
    events: ["create table", "all dml"]
    action: Do
  do-schema-rule:
    schema-pattern: "test_*"
    events: ["create database"]
    action: Do
```

### TiDBでサポートされていないSQL文をフィルタリング

TiDBでサポートされていないSQL文をフィルタリングするには、以下のように `filter-procedure-rule` を構成します。

```
filters:
  filter-procedure-rule:
    schema-pattern: "*"
    sql-pattern: [".*\\s+DROP\\s+PROCEDURE", ".*\\s+CREATE\\s+PROCEDURE", "ALTER\\s+TABLE[\\s\\S]*ADD\\s+PARTITION", "ALTER\\s+TABLE[\\s\\S]*DROP\\s+PARTITION"]
    action: Ignore
```

> **警告:**
>
> 移行が必要なデータをフィルタリングしないようにするために、できるだけ厳密にグローバルなフィルタリングルールを構成してください。

## 関連項目

[SQL式を使用したバイナリログイベントのフィルタリング](/filter-dml-event.md)