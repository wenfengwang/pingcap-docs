---
title: シーケンスの削除
summary: TiDBデータベースでのDROP SEQUENCEの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-drop-sequence/','/docs/dev/reference/sql/statements/drop-sequence/']
---

# DROP SEQUENCE

`DROP SEQUENCE`文は、TiDBでシーケンスオブジェクトを削除します。

## 概要

```ebnf+diagram
DropSequenceStmt ::=
    'DROP' 'SEQUENCE' IfExists TableNameList

IfExists ::= ( 'IF' 'EXISTS' )?

TableNameList ::=
    TableName ( ',' TableName )*

TableName ::=
    Identifier ('.' Identifier)?
```

## 例

{{< copyable "sql" >}}

```sql
DROP SEQUENCE seq;
```

```
クエリは OK で、0行が変更されました（0.10秒）
```

{{< copyable "sql" >}}

```sql
DROP SEQUENCE seq, seq2;
```

```
クエリは OK で、0行が変更されました（0.03秒）
```

## MySQL互換性

この文はTiDBの拡張機能です。実装はMariaDBで利用可能なシーケンスにモデル化されています。

## 関連情報

* [CREATE SEQUENCE](/sql-statements/sql-statement-create-sequence.md)
* [SHOW CREATE SEQUENCE](/sql-statements/sql-statement-show-create-sequence.md)