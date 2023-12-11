---
title: SELECT | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSELECTの使用法についての概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-select/','/docs/dev/reference/sql/statements/select/']
---

# SELECT

`SELECT` ステートメントはTiDBからデータを読み取るために使用されます。

## 概要

**SelectStmt:**

![SelectStmt](/media/sqlgram/SelectStmt.png)

**FromDual:**

![FromDual](/media/sqlgram/FromDual.png)

**SelectStmtOpts:**

![SelectStmtOpts](/media/sqlgram/SelectStmtOpts.png)

**SelectStmtFieldList:**

![SelectStmtFieldList](/media/sqlgram/SelectStmtFieldList.png)

**TableRefsClause:**

```ebnf+diagram
TableRefsClause ::=
    TableRef AsOfClause? ( ',' TableRef AsOfClause? )*

AsOfClause ::=
    'AS' 'OF' 'TIMESTAMP' Expression
```

**WhereClauseOptional:**

![WhereClauseOptional](/media/sqlgram/WhereClauseOptional.png)

**SelectStmtGroup:**

![SelectStmtGroup](/media/sqlgram/SelectStmtGroup.png)

**HavingClause:**

![HavingClause](/media/sqlgram/HavingClause.png)

**OrderByOptional:**

![OrderByOptional](/media/sqlgram/OrderByOptional.png)

**SelectStmtLimit:**

![SelectStmtLimit](/media/sqlgram/SelectStmtLimit.png)

**FirstOrNext:**

![FirstOrNext](/media/sqlgram/FirstOrNext.png)

**FetchFirstOpt:**

![FetchFirstOpt](/media/sqlgram/FetchFirstOpt.png)

**RowOrRows:**

![RowOrRows](/media/sqlgram/RowOrRows.png)

**SelectLockOpt:**

```ebnf+diagram
SelectLockOpt ::= 
    ( ( 'FOR' 'UPDATE' ( 'OF' TableList )? 'NOWAIT'? )
|   ( 'LOCK' 'IN' 'SHARE' 'MODE' ) )?

TableList ::=
    TableName ( ',' TableName )*
```

**WindowClauseOptional**

![WindowClauseOptional](/media/sqlgram/WindowClauseOptional.png)

**TableSampleOpt**

```ebnf+diagram
TableSampleOpt ::=
    'TABLESAMPLE' 'REGIONS()'
```

## 構文要素の説明

|構文要素|説明|
|:--------------------- | :-------------------------------------------------- |
|`TableOptimizerHints`| これはTiDBのOptimizerの動作を制御するヒントです。詳細は[Optimizer Hints](/optimizer-hints.md)を参照してください。 |
|`ALL`, `DISTINCT`, `DISTINCTROW` | `ALL`, `DISTINCT`/`DISTINCTROW`修飾子は、重複する行を返すかどうかを指定します。ALL (デフォルト) は、すべての一致する行を返すことを指定します。|
|`HIGH_PRIORITY` | `HIGH_PRIORITY`は、現在のステートメントに対して他のステートメントよりも高い優先度を与えます。 |
|`SQL_CALC_FOUND_ROWS`| TiDBはこの機能をサポートしておらず、[`tidb_enable_noop_functions=1`](/system-variables.md#tidb_enable_noop_functions-new-in-v40)が設定されていない限りエラーを返します。 |
|`SQL_CACHE`, `SQL_NO_CACHE` | `SQL_CACHE`と`SQL_NO_CACHE`は、`BlockCache`のTiKV（RocksDB）にリクエストの結果をキャッシュするかどうかを制御するために使用されます。`count(*)`クエリなどの大量のデータに対する一度限りのクエリでは、`BlockCache`内のホットユーザーデータをフラッシュするのを避けるために、`SQL_NO_CACHE`を記入することをお勧めします。 |
|`STRAIGHT_JOIN`| `STRAIGHT_JOIN`は、`FROM`句で使用されるテーブルの順序で結合クエリを強制します。オプティマイザが良くない結合順序を選択した場合、この構文を使用してクエリの実行を高速化できます。 |
|`select_expr` | 各`select_expr`は、取得する列を示します。列名と式を含みます。`\*`はすべての列を表します。|
|`FROM table_references` | `FROM table_references`句は、行を取得するテーブル（例:`select * from t;`）、またはテーブル（例:`select * from t1 join t2;`）、または0個のテーブル（例:`select 1+1 from dual;`は`select 1+1;`と同等です）を示します。|
|`WHERE where_condition` | `WHERE`句は、指定された条件または条件が満たされる行を選択することを示します。結果には条件を満たすデータのみが含まれます。|
|`GROUP BY` | `GROUP BY`句は、結果セットをグループ化するために使用されます。|
|`HAVING where_condition` | `HAVING`句と`WHERE`句は、両方とも結果をフィルタリングするために使用されます。`HAVING`句は`GROUP BY`の結果をフィルタリングし、`WHERE`句は集計前に結果をフィルタリングします。 |
|`ORDER BY` | `ORDER BY`句は、`select_expr`リスト内の列、式、またはアイテムに基づいてデータを昇順または降順でソートするために使用されます。|
|`LIMIT` | `LIMIT`句は、行の数を制限するために使用できます。`LIMIT`は1つまたは2つの数値引数を取ります。1つの引数の場合は、引数が返す最大行数を指定します。デフォルトでは最初の行が返されます。2つの引数の場合は、最初の行のオフセットを指定し、2つ目の引数が返す最大行数を指定します。TiDBは、`LIMIT n`と同じ効果を持つ`FETCH FIRST/NEXT n ROW/ROWS ONLY`構文もサポートしており、この構文では`n`を省略することができ、その効果は`LIMIT 1`と同じです。 |
|`Window window_definition`| これはウィンドウ関数の構文で、通常は何らかの解析計算を行うために使用されます。詳細は[Window Function](/functions-and-operators/window-functions.md)を参照してください。 |
| `FOR UPDATE` | `SELECT FOR UPDATE`句は、他のトランザクションからの同時更新を検出するために結果セット内のすべてのデータをロックします。クエリ条件に一致するが結果セットに存在しないデータ（例:現在のトランザクション開始後に他のトランザクションによって書き込まれた行データ）はロックされません。TiDBが[楽観的なトランザクションモード](/optimistic-transaction.md)を使用する場合、トランザクションの競合はステートメント実行フェーズで検出されません。そのため、現在のトランザクションは、PostgreSQLなどの他のデータベースのように`UPDATE`、`DELETE`、または`SELECT FOR UPDATE`を実行する他のトランザクションをブロックしません。コミットフェーズでは、`SELECT FOR UPDATE`で読み込まれた行は2段階でコミットされるため、コンフリクト検出に参加できます。コミットに失敗して、すべての`SELECT FOR UPDATE`句を含むトランザクションに対して書き込みコンフリクトが発生する場合、コミットは失敗します。コンフリクトが検出されない場合、コミットは成功します。ロックされた行の新しいバージョンが生成され、後で未コミットトランザクションがコミットされる際に書き込みコンフリクトを検出できるようになります。TiDBが[悲観的なトランザクションモード](/pessimistic-transaction.md)を使用する場合、動作は基本的に他のデータベースと同じです。詳細は、詳細は[MySQL InnoDBとの違い](/pessimistic-transaction.md#difference-with-mysql-innodb)を参照してください。TiDBは`FOR UPDATE`の`NOWAIT`修飾子をサポートしています。詳細は[TiDB Pessimistic Transaction Mode](/pessimistic-transaction.md#behaviors)を参照してください。 |
|`LOCK IN SHARE MODE` | 互換性を保証するために、TiDBはこの3つの修飾子を解析しますが、無視します。 |
| `TABLESAMPLE` | テーブルから行のサンプルを取得するための構文です。 |

## 例

### SELECT

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO t1 (c1)  VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
5 rows in set (0.00 sec)
```

```sql
mysql> SELECT AVG(s_quantity), COUNT(s_quantity) FROM stock TABLESAMPLE REGIONS();
+-----------------+-------------------+
| AVG(s_quantity) | COUNT(s_quantity) |
+-----------------+-------------------+
|         59.5000 |                 4 |
+-----------------+-------------------+
1 row in set (0.00 sec)

mysql> SELECT AVG(s_quantity), COUNT(s_quantity) FROM stock;
+-----------------+-------------------+
| AVG(s_quantity) | COUNT(s_quantity) |
+-----------------+-------------------+
|         54.9729 |           1000000 |
+-----------------+-------------------+
1 row in set (0.52 sec)
```

上記の例は、`tiup bench tpcc prepare`を使用して生成されたデータを使用しています。最初のクエリは`TABLESAMPLE`の使用を示しています。

### SELECT ... INTO OUTFILE

`SELECT ... INTO OUTFILE` ステートメントは、クエリの結果をファイルに書き込むために使用されます。

> **注記:**
>
> - このステートメントは TiDB セルフホスト環境にのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/) では使用できません。
> - このステートメントは、Amazon S3 や GCS などの[外部ストレージ](https://docs.pingcap.com/tidb/stable/backup-and-restore-storages)にクエリ結果を書き込むことをサポートしていません。

このステートメントでは、次の節を使用して出力ファイルの形式を指定できます:

- `FIELDS TERMINATED BY`: ファイル内のフィールドの区切り文字を指定します。たとえば、コンマ区切りの値（CSV）を出力するには `','` として指定し、タブ区切りの値（TSV）を出力するには `'\t'` と指定できます。
- `FIELDS ENCLOSED BY`: ファイル内の各フィールドを囲む囲み文字を指定します。
- `LINES TERMINATED BY`: ファイル内の行終端記号を指定します。

次のような3つの列を持つ `t` というテーブルがあるとします:

```sql
mysql> CREATE TABLE t (a INT, b VARCHAR(10), c DECIMAL(10,2));
Query OK, 0 rows affected (0.02 sec)

mysql> INSERT INTO t VALUES (1, 'a', 1.1), (2, 'b', 2.2), (3, 'c', 3.3);
Query OK, 3 rows affected (0.01 sec)
```

以下の例では、`SELECT ... INTO OUTFILE` ステートメントを使用してクエリ結果をファイルに書き込む方法を示しています。

**例 1:**

```sql
mysql> SELECT * FROM t INTO OUTFILE '/tmp/tmp_file1';
Query OK, 3 rows affected (0.00 sec)
```

この例では、`/tmp/tmp_file1` に次のようにクエリの結果が見つかります:

```
1       a       1.10
2       b       2.20
3       c       3.30
```

**例 2:**

```sql
mysql> SELECT * FROM t INTO OUTFILE '/tmp/tmp_file2' FIELDS TERMINATED BY ',' ENCLOSED BY '"';
Query OK, 3 rows affected (0.00 sec)
```

この例では、`/tmp/tmp_file2` に次のようにクエリの結果が見つかります:

```
"1","a","1.10"
"2","b","2.20"
"3","c","3.30"
```

**例 3:**

```sql
mysql> SELECT * FROM t INTO OUTFILE '/tmp/tmp_file3'
    -> FIELDS TERMINATED BY ',' ENCLOSED BY '\'' LINES TERMINATED BY '<<<\n';
Query OK, 3 rows affected (0.00 sec)
```

この例では、`/tmp/tmp_file3` に次のようにクエリの結果が見つかります:

```
'1','a','1.10'<<<
'2','b','2.20'<<<
'3','c','3.30'<<<
```

## MySQL 互換性

- 構文 `SELECT ... INTO @variable` はサポートされていません。
- 構文 `SELECT ... INTO DUMPFILE` はサポートされていません。
- 構文 `SELECT .. GROUP BY expr` は MySQL 5.7 での動作と異なり、`GROUP BY expr ORDER BY expr` を暗黙的に含意しません。代わりに TiDB は MySQL 8.0 の挙動に一致し、デフォルトの順序を含意しません。
- 構文 `SELECT ... TABLESAMPLE ...` は、他のデータベースシステムと [ISO/IEC 9075-2](https://standards.iso.org/iso-iec/9075/-2/ed-6/en/) 標準に互換性を持たせるための TiDB の拡張機能ですが、現在のところ MySQL ではサポートされていません。

## 関連情報

* [INSERT](/sql-statements/sql-statement-insert.md)
* [DELETE](/sql-statements/sql-statement-delete.md)
* [UPDATE](/sql-statements/sql-statement-update.md)
* [REPLACE](/sql-statements/sql-statement-replace.md)