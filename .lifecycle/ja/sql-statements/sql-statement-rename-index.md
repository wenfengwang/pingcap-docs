---
title: RENAME INDEX | TiDB SQLステートメントリファレンス
summary: TiDBデータベースでのRENAME INDEXの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-rename-index/','/docs/dev/reference/sql/statements/rename-index/']
---

# RENAME INDEX

ステートメント`ALTER TABLE .. RENAME INDEX`は、既存のインデックスを新しい名前にリネームします。この操作はTiDBでは即時であり、メタデータの変更のみが必要です。

## 概要

```ebnf+diagram
AlterTableStmt
         ::= 'ALTER' 'IGNORE'? 'TABLE' TableName RenameIndexSpec ( ',' RenameIndexSpec )*

RenameIndexSpec
         ::= 'RENAME' ( 'KEY' | 'INDEX' ) Identifier 'TO' Identifier
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL, INDEX col1 (c1));
クエリ OK, 0 行が選択されました (0.11 秒)

mysql> SHOW CREATE TABLE t1\G
*************************** 1. 行目 ***************************
       テーブル: t1
テーブル作成: CREATE TABLE `t1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `c1` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `col1` (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
1 行が選択されました (0.00 秒)

mysql> ALTER TABLE t1 RENAME INDEX col1 TO c1;
クエリ OK, 0 行が選択されました (0.09 秒)

mysql> SHOW CREATE TABLE t1\G
*************************** 1. 行目 ***************************
       テーブル: t1
テーブル作成: CREATE TABLE `t1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `c1` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `c1` (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
1 行が選択されました (0.00 秒)
```

## MySQL互換性

TiDBの`RENAME INDEX`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)
* [CREATE INDEX](/sql-statements/sql-statement-create-index.md)
* [DROP INDEX](/sql-statements/sql-statement-drop-index.md)
* [SHOW INDEXES](/sql-statements/sql-statement-show-indexes.md)
* [ALTER INDEX](/sql-statements/sql-statement-alter-index.md)