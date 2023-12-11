---
title: SHOW COLLATION | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSHOW COLLATIONの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-collation/','/docs/dev/reference/sql/statements/show-collation/']
---

# SHOW COLLATION

このステートメントは、コンパチビリティを提供するためにMySQLクライアントライブラリと含まれている静的な照合の一覧を提供します。

> **注意:**
>
> 「新しい照合フレームワーク」が有効になっている場合、`SHOW COLLATION`の結果は変化します。新しい照合フレームワークの詳細については、[文字セットと照合](/character-set-and-collation.md#new-framework-for-collations)を参照してください。

## 概要

**ShowCollationStmt:**

![ShowCollationStmt](/media/sqlgram/ShowCollationStmt.png)

## 例

新しい照合フレームワークが無効になっている場合、バイナリ照合のみが表示されます。

```sql
mysql> SHOW COLLATION;
+-------------+---------+------+---------+----------+---------+
| Collation   | Charset | Id   | Default | Compiled | Sortlen |
+-------------+---------+------+---------+----------+---------+
| utf8mb4_bin | utf8mb4 |   46 | Yes     | Yes      |       1 |
| latin1_bin  | latin1  |   47 | Yes     | Yes      |       1 |
| binary      | binary  |   63 | Yes     | Yes      |       1 |
| ascii_bin   | ascii   |   65 | Yes     | Yes      |       1 |
| utf8_bin    | utf8    |   83 | Yes     | Yes      |       1 |
+-------------+---------+------+---------+----------+---------+
5 行 in set (0.02 秒)
```

新しい照合フレームワークが有効になっている場合、`utf8_general_ci` と `utf8mb4_general_ci` もサポートされます。

```sql
mysql> SHOW COLLATION;
+--------------------+---------+------+---------+----------+---------+
| Collation          | Charset | Id   | Default | Compiled | Sortlen |
+--------------------+---------+------+---------+----------+---------+
| ascii_bin          | ascii   |   65 | Yes     | Yes      |       1 |
| binary             | binary  |   63 | Yes     | Yes      |       1 |
| gbk_bin            | gbk     |   87 |         | Yes      |       1 |
| gbk_chinese_ci     | gbk     |   28 | Yes     | Yes      |       1 |
| latin1_bin         | latin1  |   47 | Yes     | Yes      |       1 |
| utf8_bin           | utf8    |   83 | Yes     | Yes      |       1 |
| utf8_general_ci    | utf8    |   33 |         | Yes      |       1 |
| utf8_unicode_ci    | utf8    |  192 |         | Yes      |       1 |
| utf8mb4_bin        | utf8mb4 |   46 | Yes     | Yes      |       1 |
| utf8mb4_general_ci | utf8mb4 |   45 |         | Yes      |       1 |
| utf8mb4_unicode_ci | utf8mb4 |  224 |         | Yes      |       1 |
+--------------------+---------+------+---------+----------+---------+
11 行 in set (0.001 秒)
```

## MySQL 互換性

TiDBにおける`SHOW COLLATION`ステートメントの使用はMySQLと完全に互換性があります。ただし、TiDBの文字セットはMySQLと比較して異なるデフォルトの照合を持っている場合があります。詳細については、[MySQLとの互換性](/mysql-compatibility.md)を参照してください。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW CHARACTER SET](/sql-statements/sql-statement-show-character-set.md)
* [Character Set and Collation](/character-set-and-collation.md)