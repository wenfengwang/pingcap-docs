---
title: 一時テーブル
summary: TiDBの一時テーブル機能を学んで、アプリケーションの中間データを保存する一時テーブルの使用方法を学んで、テーブル管理のオーバーヘッドを減らし、パフォーマンスを向上させます。

# 一時テーブル

一時テーブル機能はTiDB v5.3.0で導入されました。この機能は、アプリケーションの中間結果を一時的に保存する問題を解決し、頻繁にテーブルを作成および削除する手間を省くことができます。中間計算データを一時テーブルに保存できます。中間データが不要になると、TiDBは自動的に一時テーブルをクリーンアップおよびリサイクルします。これにより、ユーザーアプリケーションが複雑になるのを避け、テーブル管理のオーバーヘッドを減らし、パフォーマンスを向上させることができます。

このドキュメントでは、ユーザーシナリオと一時テーブルの種類について紹介し、使用例と一時テーブルのメモリ使用量を制限する方法、および他のTiDB機能との互換性の制限について説明します。

## ユーザーシナリオ

TiDB一時テーブルは、次のようなシナリオで使用できます：

- アプリケーションの中間一時データをキャッシュします。計算が完了した後、データは通常のテーブルにダンプされ、一時テーブルは自動的に解放されます。
- 短期間で同じデータに対して複数のDML操作を実行します。たとえば、eコマースのショッピングカートアプリケーションでは、商品の追加、変更、削除、支払いの完了、ショッピングカート情報の削除を行います。
- バッチで中間一時データを高速にインポートして、一時データのインポートのパフォーマンスを向上させます。
- バッチでデータを更新します。データを一時テーブルにバッチでインポートし、データの変更を完了した後にデータをファイルにエクスポートします。

## 一時テーブルの種類

TiDBの一時テーブルには、ローカル一時テーブルとグローバル一時テーブルの2種類があります。

- ローカル一時テーブルでは、テーブルの定義とデータは現在のセッションのみに可視です。このタイプはセッション内の中間データを一時的に保存するのに適しています。
- グローバル一時テーブルでは、テーブルの定義はTiDBクラスタ全体から見え、テーブルのデータは現在のトランザクションのみに見えます。このタイプはトランザクション内の中間データを一時的に保存するのに適しています。

## ローカル一時テーブル

TiDBのローカル一時テーブルのセマンティクスはMySQLの一時テーブルと一致しています。特性は次の通りです：

- ローカル一時テーブルのテーブル定義は永続的ではありません。ローカル一時テーブルは作成されたセッションのみに見え、他のセッションからはアクセスできません。
- 同じ名前のローカル一時テーブルを異なるセッションで作成でき、各セッションはそのセッションで作成されたローカル一時テーブルのみに読み書きします。
- ローカル一時テーブルのデータはセッション内のすべてのトランザクションに見えます。
- セッションが終了すると、そのセッションで作成されたローカル一時テーブルは自動的に削除されます。
- ローカル一時テーブルには通常のテーブルと同じ名前を持つことができます。この場合、DDLおよびDMLステートメントでは、ローカル一時テーブルが削除されるまで通常のテーブルは非表示になります。

ローカル一時テーブルを作成するには、`CREATE TEMPORARY TABLE`ステートメントを使用できます。ローカル一時テーブルを削除するには、`DROP TABLE`または`DROP TEMPORARY TABLE`ステートメントを使用できます。

MySQLと異なり、TiDBのローカル一時テーブルはすべて外部テーブルであり、SQLステートメントが実行されると内部一時テーブルは自動的に作成されません。

### ローカル一時テーブルの使用例

> **注：**
>
> - TiDBの一時テーブルを使用する前に、[他のTiDB機能との互換性の制限](#compatibility-restrictions-with-other-tidb-features) と [MySQL一時テーブルとの互換性](#compatibility-with-mysql-temporary-tables) に注意してください。
> - TiDB v5.3.0より前のクラスタでローカル一時テーブルを作成した場合、これらのテーブルは実際には通常のテーブルであり、クラスタがTiDB v5.3.0またはそれ以降のバージョンにアップグレードされた後も通常のテーブルとして扱われます。

通常のテーブル `users` があるとします：

{{< copyable "sql" >}}

```sql
CREATE TABLE users (
    id BIGINT,
    name VARCHAR(100),
    PRIMARY KEY(id)
);
```

セッションAでは、ローカル一時テーブル `users` を作成すると、通常のテーブル `users` と競合しません。セッションAが`users`テーブルにアクセスすると、セッションAのローカル一時テーブル `users` にアクセスします。

{{< copyable "sql" >}}

```sql
CREATE TEMPORARY TABLE users (
    id BIGINT,
    name VARCHAR(100),
    city VARCHAR(50),
    PRIMARY KEY(id)
);
```

```
Query OK, 0 rows affected (0.01 sec)
```

`users` にデータを挿入すると、データはセッションAのローカル一時テーブル `users` に挿入されます。

{{< copyable "sql" >}}

```sql
INSERT INTO users(id, name, city) VALUES(1001, 'Davis', 'LosAngeles');
```

```
Query OK, 1 row affected (0.00 sec)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM users;
```

```
+------+-------+------------+
| id   | name  | city       |
+------+-------+------------+
| 1001 | Davis | LosAngeles |
+------+-------+------------+
1 row in set (0.00 sec)
```

セッションBでは、ローカル一時テーブル `users` を作成すると、通常のテーブル `users` またはセッションAのローカル一時テーブル `users` と競合しません。セッションBが`users`テーブルにアクセスすると、セッションBのローカル一時テーブル `users` にアクセスします。

{{< copyable "sql" >}}

```sql
CREATE TEMPORARY TABLE users (
    id BIGINT,
    name VARCHAR(100),
    city VARCHAR(50),
    PRIMARY KEY(id)
);
```

```
Query OK, 0 rows affected (0.01 sec)
```

`users` にデータを挿入すると、データはセッションBのローカル一時テーブル `users` に挿入されます。

{{< copyable "sql" >}}

```sql
INSERT INTO users(id, name, city) VALUES(1001, 'James', 'NewYork');
```

```
Query OK, 1 row affected (0.00 sec)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM users;
```

```
+------+-------+---------+
| id   | name  | city    |
+------+-------+---------+
| 1001 | James | NewYork |
+------+-------+---------+
1 row in set (0.00 sec)
```

### MySQL一時テーブルとの互換性

TiDBローカル一時テーブルの次の機能と制限はMySQL一時テーブルと同様です：

- ローカル一時テーブルを作成または削除すると、現在のトランザクションは自動的にコミットされません。
- ローカル一時テーブルが存在するスキーマを削除した後も、一時テーブルは削除されず、読み書き可能です。
- ローカル一時テーブルを作成するには `CREATE TEMPORARY TABLES` 権限が必要です。その後の操作には特別な権限が必要ありません。
- ローカル一時テーブルは外部キーやパーティションテーブルをサポートしません。
- ローカル一時テーブルに基づくビューを作成することはできません。
- `SHOW [FULL] TABLES` でローカル一時テーブルを表示しません。

以下の点でTiDBローカル一時テーブルはMySQL一時テーブルと互換性がありません：

- TiDBローカル一時テーブルは `ALTER TABLE` をサポートしません。
- TiDBローカル一時テーブルは `ENGINE` テーブルオプションを無視し、常に [メモリ使用量を制限](#limit-the-memory-usage-of-temporary-tables) します。
- `MEMORY` がストレージエンジンとして宣言された場合、TiDBローカル一時テーブルは `MEMORY` ストレージエンジンによって制限されません。
- `INNODB` または `MYISAM` がストレージエンジンとして宣言された場合、TiDBローカル一時テーブルは InnoDB の一時テーブルに特有のシステム変数を無視します。
- MySQLでは、同じ一時テーブルを同じSQLステートメントで複数回参照することは許可されません。TiDBローカル一時テーブルにはこの制限がありません。
- MySQLの一時テーブルを表示するシステムテーブル `information_schema.INNODB_TEMP_TABLE_INFO` はTiDBに存在しません。現在、TiDBにはローカル一時テーブルを表示するシステムテーブルはありません。
- TiDBには内部一時テーブルはありません。内部一時テーブルのためのMySQLシステム変数はTiDBには影響しません。

## グローバル一時テーブル

グローバル一時テーブルはTiDBの拡張機能です。特性は次の通りです：

- グローバル一時テーブルのテーブル定義は永続的で、すべてのセッションから見えます。
- グローバル一時テーブルのデータは現在のトランザクションのみに見えます。トランザクションが終了すると、データは自動的にクリアされます。
- グローバル一時テーブルは通常のテーブルと同じ名前を持つことはできません。

グローバル一時テーブルを作成するには、`CREATE GLOBAL TEMPORARY TABLE`ステートメントを使用し、`ON COMMIT DELETE ROWS`で終了します。グローバル一時テーブルを削除するには、`DROP TABLE`または`DROP GLOBAL TEMPORARY TABLE`ステートメントを使用できます。

### グローバル一時テーブルの使用例

> **注：**
>
> - TiDBの一時テーブルを使用する前に、[他のTiDB機能との互換性の制限](#compatibility-restrictions-with-other-tidb-features) に注意してください。
+ {R}
  - v5.3.0      。   。  。     。  
  `users`  A          `users`：
  
       sql

```mysql 
CREATE GLOBAL TEMPORARY TABLE users (
    id BIGINT,
    name VARCHAR(100),
    city VARCHAR(50),
    PRIMARY KEY(id)
) ON COMMIT DELETE ROWS;
```
```
     OK, 0    (0.01 )
```

   `users`    :

```mysql
BEGIN;
```
```
     OK, 0    (0.00 )
```

```mysql
INSERT INTO users(id, name, city) VALUES(1001, 'Davis', 'LosAngeles');
```
```
     OK, 1    (0.00 )
```

```mysql
SELECT * FROM users;
```
```
+------+-------+------------+
| id   | name  | city       |
+------+-------+------------+
| 1001 | Davis | LosAngeles |
+------+-------+------------+
1  (0.00 )
```

    :

```mysql
COMMIT;
```
```
     OK, 0    (0.00 )
```

```mysql
SELECT * FROM users;
```
```
Empty set (0.00 )
```

`users` A  `users`:

```mysql
SELECT * FROM users;
```
```
Empty set (0.00 )
```

>
>   SQL    ，  SQL            ，    ，     ，      。

限制临时表的内存使用
--------------
    table  ，  `ENGINE` ，    TiDB     ，      。  

    ，    [`tidb_tmp_table_max_size`](/system-variables.md#tidb_tmp_table_max_size-new-in-v530) 。    `tidb_tmp_table_max_size`   `64MB`  。

，  临时表的最大大小为  `256MB`  。

```mysql
SET GLOBAL tidb_tmp_table_max_size=268435456;
```

与其他 TiDB 功能的兼容性限制
----------------------
TiDB          **NOT**      TiDB    ：

- `AUTO_RANDOM`  
- `SHARD_ROW_ID_BITS` `PRE_SPLIT_REGIONS`  
-      table
- `SPLIT REGION`  
- `ADMIN CHECK TABLE` `ADMIN CHECKSUM TABLE`  
- `FLASHBACK TABLE` `RECOVER TABLE`  
-   `CREATE TABLE LIKE`
- Stale Read
-     
- SQL  
- TiFlash
-     table
- Placement Rules
-       `prepare plan cache` 。

TiDB    **NOT**    TiDB      feature：

-     `tidb_snapshot`  。

TiDB    工具支持
---------------
     ，     ，          。

     ，    ，    ，   ，      。

> **Note:**
>
> -  TiCDC     TiCDC v5.3.0  。
> - BR     BR v5.3.0  。            ， ，        。

## See also

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [CREATE TABLE LIKE](/sql-statements/sql-statement-create-table-like.md)
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)