---
title: ADMIN CHECK [TABLE|INDEX] | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのADMINの使用法の概要。
category: リファレンス
---

# ADMIN CHECK [TABLE|INDEX]

`ADMIN CHECK [TABLE|INDEX]` ステートメントは、テーブルとインデックスのデータの整合性をチェックします。

以下はサポートされていません：

- [FOREIGN KEYの制約](/foreign-key.md)のチェック。
- [クラスタ化されたプライマリキー](/clustered-indexes.md)が使用されている場合の、プライマリキーインデックスのチェック。

`ADMIN CHECK [TABLE|INDEX]` が問題を見つけた場合は、インデックスを削除して再作成することで問題を解決できます。問題が解決されない場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)することができます。

## 原則

`ADMIN CHECK TABLE` ステートメントは、以下の手順でテーブルをチェックします：

1. 各インデックスについて、インデックス内のレコード数がテーブル内のレコード数と同じかどうかをチェックします。

2. 各インデックスについて、各行の値を繰り返し取得し、その値をテーブル内の値と比較します。

`ADMIN CHECK INDEX` ステートメントを使用すると、指定されたインデックスのみをチェックします。

## 構文

```ebnf+diagram
AdminStmt ::=
    'ADMIN' ( 'SHOW' ( 'DDL' ( 'JOBS' Int64Num? WhereClauseOptional | 'JOB' 'QUERIES' NumList )? | TableName 'NEXT_ROW_ID' | 'SLOW' AdminShowSlow ) | 'CHECK' ( 'TABLE' TableNameList | 'INDEX' TableName Identifier ( HandleRange ( ',' HandleRange )* )? ) | 'RECOVER' 'INDEX' TableName Identifier | 'CLEANUP' ( 'INDEX' TableName Identifier | 'TABLE' 'LOCK' TableNameList ) | 'CHECKSUM' 'TABLE' TableNameList | 'CANCEL' 'DDL' 'JOBS' NumList | 'RELOAD' ( 'EXPR_PUSHDOWN_BLACKLIST' | 'OPT_RULE_BLACKLIST' | 'BINDINGS' ) | 'PLUGINS' ( 'ENABLE' | 'DISABLE' ) PluginNameList | 'REPAIR' 'TABLE' TableName CreateTableStmt | ( 'FLUSH' | 'CAPTURE' | 'EVOLVE' ) 'BINDINGS' )

TableNameList ::=
    TableName ( ',' TableName )*
```

## 例

`tbl_name` テーブルの全データとそれに対応するインデックスの整合性を確認するには、`ADMIN CHECK TABLE` を使用します：

{{< copyable "sql" >}}

```sql
ADMIN CHECK TABLE tbl_name [, tbl_name] ...;
```

整合性チェックに合格した場合は空の結果が返されます。それ以外の場合は、データの不整合を示すエラーメッセージが返されます。

{{< copyable "sql" >}}

```sql
ADMIN CHECK INDEX tbl_name idx_name;
```

上記のステートメントは、`tbl_name` テーブル内の `idx_name` インデックスに対応する列データとインデックスデータの整合性を確認するために使用されます。整合性チェックに合格した場合は空の結果が返されます。それ以外の場合は、データの不整合を示すエラーメッセージが返されます。

{{< copyable "sql" >}}

```sql
ADMIN CHECK INDEX tbl_name idx_name (lower_val, upper_val) [, (lower_val, upper_val)] ...;
```

上記のステートメントは、`tbl_name` テーブル内の `idx_name` インデックスに対応する列データとインデックスデータの、指定されたデータ範囲をチェックするために使用されます。整合性チェックに合格した場合は空の結果が返されます。それ以外の場合は、データの不整合を示すエラーメッセージが返されます。

## MySQL互換性

このステートメントは、MySQL構文に対するTiDBの拡張です。

## 関連情報

* [`ADMIN REPAIR`](/sql-statements/sql-statement-admin.md#admin-repair-statement)