---
title: OracleとTiDBの関数および構文の比較
summary: OracleとTiDBの関数および構文の比較を学びます。

# OracleとTiDBの関数および構文の比較

本書はOracleとTiDBの関数および構文の比較を記載しています。これにより、Oracleの関数に基づいて対応するTiDBの関数を見つけることができ、OracleとTiDBの間の構文の違いを理解することができます。

> **注意:**
>
> 本書に記載されている関数と構文は、Oracle 12.2.0.1.0とTiDB v5.4.0に基づいています。他のバージョンでは異なる場合があります。

## 関数の比較

以下の表は、いくつかのOracleとTiDBの関数の比較を示しています。

| 関数 | Oracleの構文 | TiDBの構文 | メモ |
|---|---|---|---|
| 特定の型に値をキャストする | <li>`TO_NUMBER(key)`</li><li>`TO_CHAR(key)`</li> | `CONVERT(key,dataType)` | TiDBは、`BINARY`、`CHAR`、`DATE`、`DATETIME`、`TIME`、`SIGNED INTEGER`、`UNSIGNED INTEGER`、`DECIMAL`のいずれかの型に値をキャストすることができます。 |
| 日付を文字列に変換する | <li>`TO_CHAR(SYSDATE,'yyyy-MM-dd hh24:mi:ss')`</li> <li>`TO_CHAR(SYSDATE,'yyyy-MM-dd')`</li> | <li>`DATE_FORMAT(NOW(),'%Y-%m-%d %H:%i:%s')`</li><li>`DATE_FORMAT(NOW(),'%Y-%m-%d')`</li> | TiDBのフォーマット文字列は大文字と小文字を区別します。 |
| 文字列を日付に変換する | <li>`TO_DATE('2021-05-28 17:31:37','yyyy-MM-dd hh24:mi:ss')`</li><li>`TO_DATE('2021-05-28','yyyy-MM-dd hh24:mi:ss')`</li> | <li>`STR_TO_DATE('2021-05-28 17:31:37','%Y-%m-%d %H:%i:%s')`</li><li>`STR_TO_DATE('2021-05-28','%Y-%m-%d%T')` </li> | TiDBのフォーマット文字列は大文字と小文字を区別します。 |
| 現在のシステム時刻を秒単位で取得する | `SYSDATE` | `NOW()` | |
| 現在のシステム時刻をマイクロ秒単位で取得する | `SYSTIMESTAMP` | `CURRENT_TIMESTAMP(6)` | |
| 2つの日付間の日数を取得する | `date1 - date2` | `DATEDIFF(date1, date2)` | |
| 2つの日付間の月数を取得する | `MONTHS_BETWEEN(ENDDATE,SYSDATE)` | `TIMESTAMPDIFF(MONTH,SYSDATE,ENDDATE)` | `MONTHS_BETWEEN()`の結果（Oracle）と`TIMESTAMPDIFF()`の結果（TiDB）は異なります。`TIMESTAMPDIFF()`は整数を返します。注意: 2つの関数のパラメータは入れ替わっています。 |
| 日付に`n`日を加算する | `DATEVAL + n` | `DATE_ADD(dateVal,INTERVAL n DAY)` | `n`は負の値であっても構いません。|
| 日付に`n`ヶ月を加算する | `ADD_MONTHS(dateVal,n)`| `DATE_ADD(dateVal,INTERVAL n MONTH)` | `n`は負の値であっても構いません。 |
| 日付の日を取得する | `TRUNC(SYSDATE)` | <li>`CAST(NOW() AS DATE)`</li><li>`DATE_FORMAT(NOW(),'%Y-%m-%d')`</li> | TiDBでは、`CAST`と`DATE_FORMAT`は同じ結果を返します。 |
| 日付の月を取得する | `TRUNC(SYSDATE,'mm')` | `DATE_ADD(CURDATE(),interval - day(CURDATE()) + 1 day)`  | |
| 値を切り捨てる | `TRUNC(2.136) = 2`<br/> `TRUNC(2.136,2) = 2.13` | `TRUNCATE(2.136,0) = 2`<br/> `TRUNCATE(2.136,2) = 2.13` | データの精度は保持されます。四捨五入せずに対応する小数点以下の桁を切り捨てます。 |
| シーケンス内の次の値を取得する | `sequence_name.NEXTVAL` | `NEXTVAL(sequence_name)` | |
| ランダムなシーケンス値を取得する | `SYS_GUID()` | `UUID()` | TiDBはUniversal Unique Identifier（UUID）を返します。 |
| 左結合または右結合 | `SELECT * FROM a, b WHERE a.id = b.id(+);`<br/>`SELECT * FROM a, b WHERE a.id(+) = b.id;` | `SELECT * FROM a LEFT JOIN b ON a.id = b.id;`<br/>`SELECT * FROM a RIGHT JOIN b ON a.id = b.id;` | 相関クエリでは、TiDBでは(+）を使用して左結合または右結合を行うことはできません。代わりに`LEFT JOIN`または`RIGHT JOIN`を使用します。 |
| `NVL()` | `NVL(key,val)` | `IFNULL(key,val)` | フィールドの値が`NULL`である場合は`val`を返し、それ以外の場合はフィールドの値を返します。 |
| `NVL2()` | `NVL2(key, val1, val2)` | `IF(key is NULL, val1, val2)` | フィールドの値が`NULL`でない場合は`val1`を返し、それ以外の場合は`val2`を返します。 |
| `DECODE()` | <li>`DECODE(key,val1,val2,val3)`</li><li>`DECODE(value,if1,val1,if2,val2,...,ifn,valn,val)`</li> | <li>`IF(key=val1,val2,val3)`</li><li>`CASE WHEN value=if1 THEN val1 WHEN value=if2 THEN val2,...,WHEN value=ifn THEN valn ELSE val END`</li> | <li>フィールドの値が`val1`である場合、`val2`を返します。それ以外の場合`val3`を返します。</li><li>フィールドの値が条件1(`if1`)を満たす場合は`val1`を、条件2(`if2`)を満たす場合は`val2`を、条件3(`if3`)を満たす場合は`val3`を返します。</li> |
| 文字列`a`と`b`を連結する | <code>'a' \|\| 'b'</code> | `CONCAT('a','b')` | |
| 文字列の長さを取得する | `LENGTH(str)` | `CHAR_LENGTH(str)` | |
| 指定された部分文字列を取得する | `SUBSTR('abcdefg',0,2) = 'ab'`<br/> `SUBSTR('abcdefg',1,2) = 'ab'` | `SUBSTRING('abcdefg',0,2) = ''`<br/>`SUBSTRING('abcdefg',1,2) = 'ab'` | <li>Oracleでは、開始位置0は開始位置1と同じ効果があります。</li><li>TiDBでは、開始位置0は空の文字列を返します。文字列の先頭から部分文字列を取得するためには、開始位置は1である必要があります。</li> |
| 部分文字列の位置を取得する | `INSTR('abcdefg','b',1,1)` | `INSTR('abcdefg','b')` | `'abcdefg'`の最初の文字から`'b'`の最初の出現箇所の位置を検索します。 |
| 部分文字列の位置を取得する | `INSTR('stst','s',1,2)` | `LENGTH(SUBSTRING_INDEX('stst','s',2)) + 1` | `'stst'`の最初の文字から`'s'`の2番目の出現箇所の位置を検索します。 |
| 部分文字列の位置を取得する | `INSTR('abcabc','b',2,1)` | `LOCATE('b','abcabc',2)` | `abcabc`の2番目の文字から`b`の最初の出現箇所の位置を検索します。 |
| カラムの値を連結する | `LISTAGG(CONCAT(E.dimensionid,'---',E.DIMENSIONNAME),'***') within GROUP(ORDER BY DIMENSIONNAME)` | `GROUP_CONCAT(CONCAT(E.dimensionid,'---',E.DIMENSIONNAME) ORDER BY DIMENSIONNAME SEPARATOR '***')` | 指定されたカラムの値を`***`区切りで1つの行に連結します。 |
| ASCIIコードを文字に変換する | `CHR(n)` | `CHAR(n)` | OracleのTab (`CHR(9)`), LF (`CHR(10)`), CR (`CHR(13)`)文字は、TiDBの`CHAR(9)`、`CHAR(10)`、`CHAR(13)`に対応します。 |

## 構文の比較

このセクションでは、OracleとTiDBの間のいくつかの構文の違いについて説明します。

### 文字列の構文

Oracleでは、文字列はシングルクォート（''）で囲む必要があります。例: `'a'`。

TiDBでは、文字列はシングルクォート（''）またはダブルクォート（""）で囲むことができます。例: `'a'`および`"a"`。

### `NULL`と空の文字列の違い
```sql
Oracleは、`NULL` と空の文字列 `''` を区別しません。つまり、`NULL` は `''` と等しいです。

TiDBでは、`NULL` と空の文字列 `''` を区別します。

### `INSERT` 文で同じテーブルに読み書き

Oracleでは、`INSERT` 文で同じテーブルに読み書きすることができます。例：

```sql
INSERT INTO table1 VALUES (feild1,(SELECT feild2 FROM table1 WHERE...))
```

TiDBでは、`INSERT` 文で同じテーブルに読み書きすることはできません。例：

```sql
INSERT INTO table1 VALUES (feild1,(SELECT T.fields2 FROM table1 T WHERE...))
```

### クエリから最初のn行を取得

Oracleでは、クエリから最初のn行を取得するには、`ROWNUM <= n` 句を使用できます。例： `ROWNUM <= 10`。

TiDBでは、クエリから最初のn行を取得するには、`LIMIT n` 句を使用できます。例： `LIMIT 10`。Hibernateクエリ言語（HQL）で`LIMIT`を使用するとエラーが発生します。HibernateのステートメントをSQLステートメントに変更する必要があります。

### `UPDATE` 文で複数のテーブルを更新

Oracleでは、複数のテーブルを更新する際、特定のフィールド更新の関係をリストアップする必要はありません。例：

```sql
UPDATE test1 SET(test1.name,test1.age) = (SELECT test2.name,test2.age FROM test2 WHERE test2.id=test1.id)
```

TiDBでは、複数のテーブルを更新する際、`SET` ですべての特定のフィールド更新関係をリストする必要があります。例：

```sql
UPDATE test1,test2 SET test1.name=test2.name,test1.age=test2.age WHERE test1.id=test2.id
```

### 導出テーブルのエイリアス

Oracleでは、複数のテーブルをクエリする際、導出テーブルにエイリアスを追加する必要はありません。例：

```sql
SELECT * FROM (SELECT * FROM test)
```

TiDBでは、複数のテーブルをクエリする際、各導出テーブルに独自のエイリアスを付ける必要があります。例：

```sql
SELECT * FROM (SELECT * FROM test) t
```

### 集合演算

Oracleでは、最初のクエリ結果に存在し、2番目のクエリ結果に存在しない行を取得するには、`MINUS` 集合演算を使用できます。例：

```sql
SELECT * FROM t1 MINUS SELECT * FROM t2
```

TiDBでは、`MINUS` 操作はサポートされていません。`EXCEPT` 集合演算を使用できます。例：

```sql
SELECT * FROM t1 EXCEPT SELECT * FROM t2
```

### コメント構文

Oracleでは、コメント構文は `--Comment` です。

TiDBでは、コメント構文は `-- Comment` です。TiDBでは、`--` の後に空白があることに注意してください。

### ページネーション

Oracleでは、`OFFSET m ROWS` を使用して `m` 行をスキップし、`FETCH NEXT n ROWS ONLY` を使用して `n` 行を取得できます。例：

```sql
SELECT * FROM tables OFFSET 0 ROWS FETCH NEXT 2000 ROWS ONLY
```

TiDBでは、`LIMIT n OFFSET m` を使用して `OFFSET m ROWS FETCH NEXT n ROWS ONLY` を置き換えることができます。例：

```sql
SELECT * FROM tables LIMIT 2000 OFFSET 0
```

### `NULL` 値の並べ替え順

Oracleでは、`NULL` 値は `ORDER BY` 句によって次のように並べ替えられます：

- `ORDER BY column ASC` 文では、`NULL` 値は最後に返されます。

- `ORDER BY column DESC` 文では、`NULL` 値は最初に返されます。

- `ORDER BY column [ASC|DESC] NULLS FIRST` 文では、`NULL` 値は非NULL値の前に返されます。非NULL値は`ASC|DESC` で指定された昇順または降順で返されます。

- `ORDER BY column [ASC|DESC] NULLS LAST` 文では、`NULL` 値は非NULL値の後に返されます。非NULL値は`ASC|DESC` で指定された昇順または降順で返されます。

TiDBでは、`NULL` 値は `ORDER BY` 句によって次のように並べ替えられます：

- `ORDER BY column ASC` 文では、`NULL` 値は最初に返されます。

- `ORDER BY column DESC` 文では、`NULL` 値は最後に返されます。

次の表は、Oracle と TiDB の同等の `ORDER BY` 文の例を示しています：

| Oracle での `ORDER BY` | TiDB での同等の文 |
| :------------------- | :----------------- |
| `SELECT * FROM t1 ORDER BY name NULLS FIRST;`      | `SELECT * FROM t1 ORDER BY name;`  |
| `SELECT * FROM t1 ORDER BY name DESC NULLS LAST;`  | `SELECT * FROM t1 ORDER BY name DESC;` |
| `SELECT * FROM t1 ORDER BY name DESC NULLS FIRST;` | `SELECT * FROM t1 ORDER BY ISNULL(name) DESC, name DESC;` |
| `SELECT * FROM t1 ORDER BY name ASC NULLS LAST;`   | `SELECT * FROM t1 ORDER BY ISNULL(name), name;` |
```