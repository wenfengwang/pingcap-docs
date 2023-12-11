---
title: SHOW CHARACTER SET | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSHOW CHARACTER SETの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-character-set/','/docs/dev/reference/sql/statements/show-character-set/']
---

# SHOW CHARACTER SET

このステートメントはTiDBで利用可能な文字セットの静的なリストを提供します。出力は現在の接続やユーザーの属性を反映しません。

## 概要

**ShowCharsetStmt:**

![ShowCharsetStmt](/media/sqlgram/ShowCharsetStmt.png)

**CharsetKw:**

![CharsetKw](/media/sqlgram/CharsetKw.png)

## 例

```sql
mysql> SHOW CHARACTER SET;
+---------+---------------+-------------------+--------+
| Charset | Description   | Default collation | Maxlen |
+---------+---------------+-------------------+--------+
| utf8    | UTF-8 Unicode | utf8_bin          |      3 |
| utf8mb4 | UTF-8 Unicode | utf8mb4_bin       |      4 |
| ascii   | US ASCII      | ascii_bin         |      1 |
| latin1  | Latin1        | latin1_bin        |      1 |
| binary  | binary        | binary            |      1 |
+---------+---------------+-------------------+--------+
5 rows in set (0.00 sec)
```

## MySQL互換性

TiDBでの`SHOW CHARACTER SET`ステートメントの使用はMySQLと完全に互換性があります。ただし、TiDBの文字セットのデフォルト照合順序はMySQLと異なる場合があります。詳細は[MySQLとの互換性](/mysql-compatibility.md)を参照してください。互換性の違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)してください。

## 関連情報

* [SHOW COLLATION](/sql-statements/sql-statement-show-collation.md)
* [文字セットと照合順序](/character-set-and-collation.md)