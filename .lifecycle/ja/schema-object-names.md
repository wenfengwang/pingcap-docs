---
title: スキーマオブジェクト名
summary: TiDBのSQLステートメントでのスキーマオブジェクト名について学びます。
aliases: ['/docs/dev/schema-object-names/','/docs/dev/reference/sql/language-structure/schema-object-names/']
---

# スキーマオブジェクト名

<!-- markdownlint-disable MD038 -->

このドキュメントでは、TiDBのSQLステートメントでのスキーマオブジェクト名について紹介します。

スキーマオブジェクト名は、データベース、テーブル、インデックス、カラム、エイリアスを含むすべてのスキーマオブジェクトに名前を付けるためにTiDBで使用されます。これらのオブジェクトはSQLステートメントで識別子を使用して引用符で囲むことができます。

識別子を囲むためにバッククォートを使用できます。例えば、`SELECT * FROM t` は `` SELECT * FROM `t` `` としても書くことができます。ただし、識別子に1つ以上の特殊文字が含まれる場合や予約語の場合、それを表すスキーマオブジェクトを引用するためにはバッククォートで囲む必要があります。

{{< copyable "sql" >}}

```sql
SELECT * FROM `table` WHERE `table`.id = 20;
```

SQLモードで`ANSI_QUOTES`を設定すると、TiDBは二重引用符で囲まれた文字列`"`を識別子として認識します。

{{< copyable "sql" >}}

```sql
CREATE TABLE "test" (a varchar(10));
```

```sql
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your TiDB version for the right syntax to use line 1 column 19 near ""test" (a varchar(10))" 
```

{{< copyable "sql" >}}

```sql
SET SESSION sql_mode='ANSI_QUOTES';
```

```sql
Query OK, 0 rows affected (0.000 sec)
```

{{< copyable "sql" >}}

```sql
CREATE TABLE "test" (a varchar(10));
```

```sql
Query OK, 0 rows affected (0.012 sec)
```

引用符で識別子内にバッククォート文字を使用する場合は、バッククォートを2回繰り返します。例えば、テーブル`a`b`を作成するには以下のようにします:

{{< copyable "sql" >}}

```sql
CREATE TABLE `a``b` (a int);
```

`SELECT`ステートメントでは、識別子または文字列を使用してエイリアスを指定できます:

{{< copyable "sql" >}}

```sql
SELECT 1 AS `identifier`, 2 AS 'string';
```

```sql
+------------+--------+
| identifier | string |
+------------+--------+
|          1 |      2 |
+------------+--------+
1 row in set (0.00 sec)
```

詳細については、[MySQL スキーマオブジェクト名](https://dev.mysql.com/doc/refman/8.0/en/identifiers.html) を参照してください。

## 識別子修飾子

オブジェクト名は修飾されていない場合と修飾されている場合があります。例えば、以下のステートメントでは修飾されていない名前のテーブルが作成されます:

{{< copyable "sql" >}}

```sql
CREATE TABLE t (i int);
```

`USE`ステートメントを使用するか接続パラメータを設定してデータベースを構成していない場合は、`ERROR 1046 (3D000): No database selected` エラーが表示されます。このときに、データベース修飾名を指定することができます:

{{< copyable "sql" >}}

```sql
CREATE TABLE test.t (i int);
```

ドット`.`の周囲に空白文字を含めることができます。`table_name.col_name` と `table_name . col_name` は同等です。

この識別子を引用するには、以下を使用します:

{{< copyable "sql" >}}

```sql
`table_name`.`col_name`
```

次のようにする代わりに:

```sql
`table_name.col_name`
```

詳細については、[MySQL 識別子の修飾子](https://dev.mysql.com/doc/refman/8.0/en/identifier-qualifiers.html) を参照してください。