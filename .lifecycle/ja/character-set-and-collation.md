---
title：文字セットと照合
summary：TiDBでサポートされる文字セットと照合の概要について学びます。
aliases：['/docs/dev/character-set-and-collation/','/docs/dev/reference/sql/characterset-and-collation/','/docs/dev/reference/sql/character-set/']

# 文字セットと照合

このドキュメントでは、TiDBでサポートされている文字セットと照合について紹介します。

## コンセプト

文字セットは、記号と符号化のセットです。TiDBのデフォルトの文字セットはutf8mb4であり、これはMySQL 8.0以降のデフォルトに一致します。

照合は、文字セット内の文字を比較するためのルールセットや文字の並べ替え順です。例えば、バイナリ照合では、`A`と`a`は等しくないことになります。

{{< copyable "sql" >}}

```sql
SET NAMES utf8mb4 COLLATE utf8mb4_bin;
SELECT 'A' = 'a';
SET NAMES utf8mb4 COLLATE utf8mb4_general_ci;
SELECT 'A' = 'a';
```

```sql
SELECT 'A' = 'a';
```

```sql
+-----------+
| 'A' = 'a' |
+-----------+
|         0 |
+-----------+
1行がセットされました (0.00秒)
```

```sql
SET NAMES utf8mb4 COLLATE utf8mb4_general_ci;
```

```sql
クエリが成功しました。0行が影響を受けました (0.00秒)
```

```sql
SELECT 'A' = 'a';
```

```sql
+-----------+
| 'A' = 'a' |
+-----------+
|         1 |
+-----------+
1行がセットされました (0.00秒)
```

TiDBのデフォルトはバイナリ照合を使用します。これは、MySQLのデフォルトである大文字と小文字を区別しない照合とは異なります。

## TiDBでサポートされている文字セットと照合

現在、TiDBは以下の文字セットをサポートしています。

{{< copyable "sql" >}}

```sql
SHOW CHARACTER SET;
```

```sql
+---------+-------------------------------------+-------------------+--------+
| Charset | Description                         | Default collation | Maxlen |
+---------+-------------------------------------+-------------------+--------+
| ascii   | US ASCII                            | ascii_bin         |      1 |
| binary  | binary                              | binary            |      1 |
| gbk     | Chinese Internal Code Specification | gbk_bin           |      2 |
| latin1  | Latin1                              | latin1_bin        |      1 |
| utf8    | UTF-8 Unicode                       | utf8_bin          |      3 |
| utf8mb4 | UTF-8 Unicode                       | utf8mb4_bin       |      4 |
+---------+-------------------------------------+-------------------+--------+
6行がセットされました (0.00秒)
```

TiDBは以下の照合をサポートしています。

```sql
SHOW COLLATION;
```

```sql
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
| utf8mb4_0900_ai_ci | utf8mb4 |  255 |         | Yes      |       1 |
| utf8mb4_0900_bin   | utf8mb4 |  309 |         | Yes      |       1 |
| utf8mb4_bin        | utf8mb4 |   46 | Yes     | Yes      |       1 |
| utf8mb4_general_ci | utf8mb4 |   45 |         | Yes      |       1 |
| utf8mb4_unicode_ci | utf8mb4 |  224 |         | Yes      |       1 |
+--------------------+---------+------+---------+----------+---------+
13行がセットされました (0.00秒)
```

> **警告:**
>
> TiDBは誤ってlatin1をutf8のサブセットとして扱います。これにより、latin1とutf8のエンコーディングで異なる文字を保存すると予期しない動作が発生する可能性があります。utf8mb4文字セットを強く推奨します。詳細は[TiDB #18955](https://github.com/pingcap/tidb/issues/18955)を参照してください。
>
> もし予測述語に対して文字列の接頭辞を持つ`LIKE`が含まれており、対象のカラムが非バイナリ照合に設定されている場合（接尾辞が`_bin`で終わっていない場合）、オプティマイザは現在、この述語を範囲スキャンに変換できません。その代わりに、フルスキャンを実行します。その結果、このようなSQLクエリは予期しないリソース消費につながる可能性があります。

> **注意:**
>
> TiDBのデフォルト照合（接尾辞が`_bin`であるバイナリ照合）は、[MySQLのデフォルト照合](https://dev.mysql.com/doc/refman/8.0/en/charset-charsets.html)（通常は接尾辞が`_general_ci`または`_ai_ci`の汎用照合）とは異なります。これにより、明示的な文字セットを指定しても、暗黙のデフォルト照合を選択することにより互換性のない動作が引き起こされる可能性があります。

テーブルに挿入された絵文字文字などの4バイトの文字をデフォルト照合（バイナリ照合）の文字セット`utf8`に挿入すると、`utf8mb4`の文字セットと照合では成功しますが、`utf8`の文字セットと照合では失敗します。

これを確認するために、4バイトの絵文字文字をテーブルに挿入するデフォルトの動作を示します。`utf8`文字セットの`INSERT`ステートメントは失敗し、`utf8mb4`は成功します。

```sql
CREATE TABLE utf8_test (
     c char(1) NOT NULL
    ) CHARACTER SET utf8;
```

```sql
Query OK, 0行が影響を受けました (0.09秒)
```

```sql
CREATE TABLE utf8m4_test (
     c char(1) NOT NULL
    ) CHARACTER SET utf8mb4;
```

```sql
Query OK, 0行が影響を受けました (0.09秒)
```

```sql
INSERT INTO utf8_test VALUES ('😉');
```

```sql
ERROR 1366 (HY000): incorrect utf8 value f09f9889(😉) for column c
```

```sql
INSERT INTO utf8m4_test VALUES ('😉');
```

```sql
Query OK, 1行が影響を受けました (0.02秒)
```

```sql
SELECT char_length(c), length(c), c FROM utf8_test;
```

```sql
空のセット (0.01秒)
```

```sql
SELECT char_length(c), length(c), c FROM utf8m4_test;
```

```sql
+----------------+-----------+------+
```sql
      +----------------+-----------+------+
      | char_length(c) | length(c) | c    |
      +----------------+-----------+------+
      |              1 |         4 | 😉     |
      +----------------+-----------+------+
      1 row in set (0.00 sec)
      ```
      
      ## 異なるレイヤーでの文字セットと照合順序
      
      文字セットと照合順序は、異なるレイヤーで設定できます。
      
      ### データベースの文字セットと照合順序
      
      各データベースには、文字セットと照合順序があります。次のステートメントを使用して、データベースの文字セットと照合順序を指定できます:
      
      ```sql
      CREATE DATABASE db_name
          [[DEFAULT] CHARACTER SET charset_name]
          [[DEFAULT] COLLATE collation_name]
      
      ALTER DATABASE db_name
          [[DEFAULT] CHARACTER SET charset_name]
          [[DEFAULT] COLLATE collation_name]
      ```
      
      ここでは`DATABASE`を`SCHEMA`に置き換えることができます。
      
      異なるデータベースは異なる文字セットと照合順序を使用できます。現在のデータベースの文字セットと照合順序を確認するには、`character_set_database`および`collation_database`を使用します:
      
      {{< copyable "sql" >}}
      
      ```sql
      CREATE SCHEMA test1 CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
      ```
      
      ```sql
      Query OK, 0 rows affected (0.09 sec)
      ```
      
      {{< copyable "sql" >}}
      
      ```sql
      USE test1;
      ```
      
      ```sql
      Database changed
      ```
      
      {{< copyable "sql" >}}
      
      ```sql
      SELECT @@character_set_database, @@collation_database;
      ```
      
      ```sql
      +--------------------------|----------------------+
      | @@character_set_database | @@collation_database |
      +--------------------------|----------------------+
      | utf8mb4                  | utf8mb4_general_ci   |
      +--------------------------|----------------------+
      1 row in set (0.00 sec)
      ```
      
      {{< copyable "sql" >}}
      
      ```sql
      CREATE SCHEMA test2 CHARACTER SET latin1 COLLATE latin1_bin;
      ```
      
      ```sql
      Query OK, 0 rows affected (0.09 sec)
      ```
      
      {{< copyable "sql" >}}
      
      ```sql
      USE test2;
      ```
      
      ```sql
      Database changed
      ```
      
      {{< copyable "sql" >}}
      
      ```sql
      SELECT @@character_set_database, @@collation_database;
      ```
      
      ```sql
      +--------------------------|----------------------+
      | @@character_set_database | @@collation_database |
      +--------------------------|----------------------+
      | latin1                   | latin1_bin           |
      +--------------------------|----------------------+
      1 row in set (0.00 sec)
      ```
      
      また、`INFORMATION_SCHEMA`で2つの値を確認できます:
      
      {{< copyable "sql" >}}
      
      ```sql
      SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
      FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = 'db_name';
      ```
      
      ### テーブルの文字セットと照合順序
      
      テーブルの文字セットと照合順序を指定するには、次のステートメントを使用できます:
      
      ```sql
      CREATE TABLE tbl_name (column_list)
          [[DEFAULT] CHARACTER SET charset_name]
          [COLLATE collation_name]]
      
      ALTER TABLE tbl_name
          [[DEFAULT] CHARACTER SET charset_name]
          [COLLATE collation_name]
      ```
      
      例:
      
      {{< copyable "sql" >}}
      
      ```sql
      CREATE TABLE t1(a int) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
      ```
      
      ```sql
      Query OK, 0 rows affected (0.08 sec)
      ```
      
      テーブルの文字セットと照合順序が指定されていない場合、データベースの文字セットと照合順序がそれらのデフォルト値として使用されます。文字セットを`utf8mb4`として指定し、照合順序を指定しない場合、照合順序はシステム変数[`default_collation_for_utf8mb4`](/system-variables.md#default_collation_for_utf8mb4-new-in-v740)の値によって決定されます。
      
      ### カラムの文字セットと照合順序
      
      カラムの文字セットと照合順序を指定するには、次のステートメントを使用できます:
      
      ```sql
      col_name {CHAR | VARCHAR | TEXT} (col_length)
          [CHARACTER SET charset_name]
          [COLLATE collation_name]
      
      col_name {ENUM | SET} (val_list)
          [CHARACTER SET charset_name]
          [COLLATE collation_name]
      ```
      
      カラムの文字セットと照合順序が指定されていない場合、テーブルの文字セットと照合順序がそれらのデフォルト値として使用されます。文字セットを`utf8mb4`として指定し、照合順序を指定しない場合、照合順序はシステム変数[`default_collation_for_utf8mb4`](/system-variables.md#default_collation_for_utf8mb4-new-in-v740)の値によって決定されます。
      
      ### 文字列の文字セットと照合順序
      
      各文字列には文字セットと照合順序が対応しています。文字列を使用するとき、このオプションを利用できます:
      
      {{< copyable "sql" >}}
      
      ```sql
      [_charset_name]'string' [COLLATE collation_name]
      ```
      
      例:
      
      {{< copyable "sql" >}}
      
      ```sql
      SELECT 'string';
      SELECT _utf8mb4'string';
      SELECT _utf8mb4'string' COLLATE utf8mb4_general_ci;
      ```
      
      ルール:
      
      + ルール 1: `CHARACTER SET charset_name`および`COLLATE collation_name`が指定されている場合、`charset_name`文字セットと`collation_name`照合順序が直接使用されます。
      + ルール 2: `CHARACTER SET charset_name`が指定されているが`COLLATE collation_name`が指定されていない場合、`charset_name`文字セットと`charset_name`のデフォルト照合順序が使用されます。
      + ルール 3: `CHARACTER SET charset_name`および`COLLATE collation_name`が指定されていない場合、`character_set_connection`および`collation_connection`システム変数で与えられる文字セットと照合順序が使用されます。
      
      ### クライアント接続の文字セットと照合順序
      
      + サーバーの文字セットと照合順序は、`character_set_server`および`collation_server`システム変数の値です。
      + デフォルトデータベースの文字セットと照合順序は、`character_set_database`および`collation_database`システム変数の値です。
      
      `character_set_connection`および`collation_connection`を使用して、各接続のために文字セットと照合順序を指定できます。`character_set_client`変数は、クライアント文字セットを設定するためのものです。
      
      クライアントに関連する文字セットと照合順序を設定するには、次のステートメントを使用できます:
      
      + `SET NAMES 'charset_name' [COLLATE 'collation_name']`
      
        `SET NAMES`は、クライアントがサーバーに対してSQLステートメントを送信するために使用する文字セットを示します。`SET NAMES utf8mb4`は、クライアントからのすべてのリクエストがutf8mb4を使用し、またサーバーからの結果もutf8mb4を使用することを示します。
        
        `SET NAMES 'charset_name'`ステートメントは、次のステートメントの組み合わせに等しいです:
        
        ```sql
        SET character_set_client = charset_name;
        SET character_set_results = charset_name;
        SET character_set_connection = charset_name;
        ```
        
        `COLLATE`はオプションです。省略された場合、`charset_name`のデフォルト照合順序が`collation_connection`に設定されます。
        
      + `SET CHARACTER SET 'charset_name'`
      
        `SET NAMES`と同様に、`SET NAMES 'charset_name'`ステートメントは次のステートメントの組み合わせに等しいです:
        
        ```sql
        SET character_set_client = charset_name;
        SET character_set_results = charset_name;
        SET charset_connection = @@charset_database;
        SET collation_connection = @@collation_database;
        ```
      
      ## 文字セットおよび照合順序の選択優先順位
      
      文字列 > カラム > テーブル > データベース > サーバー
      
      ## 文字セットと照合順序の選択に関する一般的なルール

      + ルール1: `CHARACTER SET charset_name`および`COLLATE collation_name`が指定されている場合、`charset_name`文字セットと`collation_name`照合順序が直接使用されます。
      + ルール2: `CHARACTER SET charset_name`が指定されているが`COLLATE collation_name`が指定されていない場合、`charset_name`文字セットと`charset_name`のデフォルト照合順序が使用されます。
      + ルール3: `CHARACTER SET charset_name`および`COLLATE collation_name`が指定されていない場合、より高い最適化レベルの文字セットと照合順序が使用されます。

      ## 文字の有効性チェック

      指定した文字セットが`utf8`または`utf8mb4`の場合、TiDBは有効な`utf8`文字のみをサポートしています。無効な文字の場合、TiDBは`不正なutf8値`エラーを報告します。TiDBのこの文字の有効性チェックはMySQL 8.0と互換性がありますが、MySQL 5.7またはそれ以前のバージョンとは互換性がありません。

      このエラー報告を無効にするには、`set @@tidb_skip_utf8_check=1;`を使用して文字のチェックをスキップします。

      > **ノート:**
      >
      > 文字のチェックをスキップすると、TiDBはアプリケーションによって記述された不正なUTF-8文字を検出できず、`ANALYZE`の実行時にデコードエラーやその他の未知のエンコーディングの問題が発生する可能性があります。文字列の有効性が保証されない場合は、文字のチェックをスキップしないことをお勧めします。
      ```
## 照合サポートフレームワーク

<CustomContent platform="tidb">

照合の構文サポートおよび意味のサポートは、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap) 構成項目に影響を受けます。構文サポートと意味のサポートは異なります。前者は、TiDBが照合を解析して設定できることを示しています。後者は、TiDBが文字列を比較する際に照合を正しく使用できることを示しています。

</CustomContent>

v4.0以前、TiDBは[古い照合フレームワーク](#old-framework-for-collations)のみを提供していました。このフレームワークでは、TiDBは構文的にはほとんどのMySQLの照合を解析しますが、意味的にはすべての照合をバイナリ照合として取り扱っていました。

v4.0以降、TiDBは[新しい照合フレームワーク](#new-framework-for-collations)をサポートしています。このフレームワークでは、TiDBは異なる照合を意味的に解析し、文字列を比較する際に照合を厳密に遵守します。

### 古い照合フレームワーク

v4.0以前、TiDBでは、ほとんどのMySQLの照合を指定できましたが、これらの照合はデフォルトの照合に従って処理されます。つまり、バイト順が文字順を決定します。MySQLと異なり、TiDBは文字の末尾のスペースを処理しないため、次のような動作の違いが発生します。

{{< copyable "sql" >}}

```sql
CREATE TABLE t(a varchar(20) charset utf8mb4 collate utf8mb4_general_ci PRIMARY KEY);
```

```sql
Query OK, 0 rows affected
```

```sql
INSERT INTO t VALUES ('A');
```

```sql
Query OK, 1 row affected
```

```sql
INSERT INTO t VALUES ('a');
```

```sql
Query OK, 1 row affected
```

この場合、TiDBでは前述のステートメントが正常に実行されます。一方、MySQLでは、`utf8mb4_general_ci` が大文字と小文字を区別しないので、`Duplicate entry 'a'` エラーが報告されます。

```sql
INSERT INTO t1 VALUES ('a ');
```

```sql
Query OK, 1 row affected
```

この場合、TiDBでは前述のステートメントが正常に実行されます。しかし、MySQLでは、比較がスペースで埋められた後に行われるため、`Duplicate entry 'a '` エラーが返されます。

### 新しい照合フレームワーク

TiDB v4.0以降、完全な照合フレームワークが導入されました。

<CustomContent platform="tidb">

この新しいフレームワークでは、照合の意味的な解析がサポートされ、クラスタが初めて初期化される際に新しいフレームワークを有効にするかどうかを決定する `new_collations_enabled_on_first_bootstrap` 構成項目が導入されています。新しいフレームワークを有効にするには、`new_collations_enabled_on_first_bootstrap` を `true` に設定します。詳細については、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap) を参照してください。構成項目が有効になった後にクラスタを初期化する場合は、`mysql`.`tidb` テーブルの中の `new_collation_enabled` 変数で新しい照合が有効かどうかを確認できます。

{{< copyable "sql" >}}

```sql
SELECT VARIABLE_VALUE FROM mysql.tidb WHERE VARIABLE_NAME='new_collation_enabled';
```

```sql
+----------------+
| VARIABLE_VALUE |
+----------------+
| True           |
+----------------+
1 row in set (0.00 sec)
```

</CustomContent>

<CustomContent platform="tidb-cloud">

この新しいフレームワークでは、照合の意味的な解析がサポートされます。TiDBはクラスタが初めて初期化される際に新しいフレームワークをデフォルトで有効にします。

</CustomContent>

新しいフレームワークの下で、TiDBは `utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`utf8mb4_0900_bin`、`utf8mb4_0900_ai_ci`、`gbk_chinese_ci`、および `gbk_bin` 照合をサポートし、これらはMySQLと互換性があります。

`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`utf8mb4_0900_ai_ci`、および `gbk_chinese_ci` のいずれかを使用する場合、文字列比較は大文字と小文字を区別せず、アクセントを無視します。同時に、TiDBは照合の `PADDING` の動作を修正します。

{{< copyable "sql" >}}

```sql
CREATE TABLE t(a varchar(20) charset utf8mb4 collate utf8mb4_general_ci PRIMARY KEY);
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

```sql
INSERT INTO t VALUES ('A');
```

```sql
Query OK, 1 row affected (0.00 sec)
```

```sql
INSERT INTO t VALUES ('a');
```

```sql
ERROR 1062 (23000): Duplicate entry 'a' for key 't.PRIMARY' # TiDBはMySQLの大文字と小文字を区別しない照合と互換性があります。
```

```sql
INSERT INTO t VALUES ('a ');
```

```sql
ERROR 1062 (23000): Duplicate entry 'a ' for key 't.PRIMARY' # TiDBはMySQLとの互換性を持たせるため、`PADDING` の動作を修正します。
```

> **注意:**
>
> TiDBにおけるパディングの実装はMySQLと異なります。MySQLではスペースで埋めることでパディングが実装されていますが、TiDBでは末尾のスペースを切り取ることでパディングが実装されています。ほとんどのケースでは2つのアプローチは同じですが、例外は末尾の文字がスペース（0x20）よりも少ない文字を含む場合です。例えば、TiDBにおける `'a' < 'a\t'` の結果は `1` になりますが、MySQLにおける `'a' < 'a\t'` は `'a ' < 'a\t'` と同じであり、結果は `0` になります。

## 式での照合の強制値

式に複数の異なる照合の節が関与する場合、計算に使用される照合を推測する必要があります。ルールは次のとおりです。

+ 明示的な `COLLATE` 節の強制値は `0` です。
+ 2つの文字列の照合が互換性がない場合、異なる照合を持つ2つの文字列の連結の強制値は `1` になります。
+ カラム、`CAST()`、`CONVERT()`、または `BINARY()` の照合の強制値は `2` です。
+ システム定数（`USER ()` または `VERSION ()` によって返される文字列）の照合の強制値は `3` です。
+ 定数の強制値は `4` です。
+ 数値や中間変数の強制値は `5` です。
+ `NULL` や `NULL` から派生した式の強制値は `6` です。

照合を推測する場合、TiDBは強制値が低い式の照合を使用することを優先します。2つの節の強制値が同じ場合は、以下の優先順位に従って照合が決定されます。

binary > utf8mb4_bin > (utf8mb4_general_ci = utf8mb4_unicode_ci) > utf8_bin > (utf8_general_ci = utf8_unicode_ci) > latin1_bin > ascii_bin

次の状況では、TiDBは照合を推測することができず、エラーを報告します。

- 2つの節の照合が異なり、両方の節の強制値が `0` である場合。
- 2つの節の照合が互換性がなく、式の返される型が `String` の場合。

## `COLLATE` 節

TiDBは `COLLATE` 節を使用して式の照合を指定することをサポートしています。この式の強制値は `0` であり、最優先です。次の例を参照してください。

{{< copyable "sql" >}}

```sql
SELECT 'a' = _utf8mb4 'A' collate utf8mb4_general_ci;
```

```sql
+-----------------------------------------------+
| 'a' = _utf8mb4 'A' collate utf8mb4_general_ci |
+-----------------------------------------------+
|                                             1 |
+-----------------------------------------------+
1 row in set (0.00 sec)
```

詳細については、[接続の文字セットと照合](https://dev.mysql.com/doc/refman/8.0/en/charset-connection.html) を参照してください。