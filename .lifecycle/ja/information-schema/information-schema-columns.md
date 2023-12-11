---
title: COLUMNS
summary: `COLUMNS` INFORMATION_SCHEMA テーブルを学ぶ。

# COLUMNS

`COLUMNS` テーブルはテーブル内のカラムに関する詳細情報を提供します。

```sql
USE INFORMATION_SCHEMA;
DESC COLUMNS;
```

出力は次のとおりです：

```sql
+--------------------------+---------------+------+------+---------+-------+
| Field                    | Type          | Null | Key  | Default | Extra |
+--------------------------+---------------+------+------+---------+-------+
| TABLE_CATALOG            | varchar(512)  | YES  |      | NULL    |       |
| TABLE_SCHEMA             | varchar(64)   | YES  |      | NULL    |       |
| TABLE_NAME               | varchar(64)   | YES  |      | NULL    |       |
| COLUMN_NAME              | varchar(64)   | YES  |      | NULL    |       |
| ORDINAL_POSITION         | bigint(64)    | YES  |      | NULL    |       |
| COLUMN_DEFAULT           | text          | YES  |      | NULL    |       |
| IS_NULLABLE              | varchar(3)    | YES  |      | NULL    |       |
| DATA_TYPE                | varchar(64)   | YES  |      | NULL    |       |
| CHARACTER_MAXIMUM_LENGTH | bigint(21)    | YES  |      | NULL    |       |
| CHARACTER_OCTET_LENGTH   | bigint(21)    | YES  |      | NULL    |       |
| NUMERIC_PRECISION        | bigint(21)    | YES  |      | NULL    |       |
| NUMERIC_SCALE            | bigint(21)    | YES  |      | NULL    |       |
| DATETIME_PRECISION       | bigint(21)    | YES  |      | NULL    |       |
| CHARACTER_SET_NAME       | varchar(32)   | YES  |      | NULL    |       |
| COLLATION_NAME           | varchar(32)   | YES  |      | NULL    |       |
| COLUMN_TYPE              | text          | YES  |      | NULL    |       |
| COLUMN_KEY               | varchar(3)    | YES  |      | NULL    |       |
| EXTRA                    | varchar(30)   | YES  |      | NULL    |       |
| PRIVILEGES               | varchar(80)   | YES  |      | NULL    |       |
| COLUMN_COMMENT           | varchar(1024) | YES  |      | NULL    |       |
| GENERATION_EXPRESSION    | text          | NO   |      | NULL    |       |
+--------------------------+---------------+------+------+---------+-------+
21 行が返されました (0.00 秒)
```

`test.t1` テーブルを作成し、`COLUMNS` テーブルから情報をクエリします：

```sql
CREATE TABLE test.t1 (a int);
SELECT * FROM COLUMNS WHERE table_schema='test' AND TABLE_NAME='t1'\G
```

Outputは次のとおりです：

```sql
*************************** 1. row ***************************
           TABLE_CATALOG: def
            TABLE_SCHEMA: test
              TABLE_NAME: t1
             COLUMN_NAME: a
        ORDINAL_POSITION: 1
          COLUMN_DEFAULT: NULL
             IS_NULLABLE: YES
               DATA_TYPE: int
CHARACTER_MAXIMUM_LENGTH: NULL
  CHARACTER_OCTET_LENGTH: NULL
       NUMERIC_PRECISION: 11
           NUMERIC_SCALE: 0
      DATETIME_PRECISION: NULL
      CHARACTER_SET_NAME: NULL
          COLLATION_NAME: NULL
             COLUMN_TYPE: int(11)
              COLUMN_KEY:
                   EXTRA:
              PRIVILEGES: select,insert,update,references
          COLUMN_COMMENT:
   GENERATION_EXPRESSION:
1 row が返されました (0.02 秒)
```

`COLUMNS` テーブルのカラムの説明は以下の通りです：

* `TABLE_CATALOG`: カラムが属するカタログの名前。値は常に `def` です。
* `TABLE_SCHEMA`: カラムを含むスキーマの名前。
* `TABLE_NAME`: カラムを持つテーブルの名前。
* `COLUMN_NAME`: カラムの名前。
* `ORDINAL_POSITION`: テーブル内のカラムの位置。
* `COLUMN_DEFAULT`: カラムのデフォルト値。明示的なデフォルト値が `NULL` の場合、またはカラム定義に `default` 句が含まれていない場合、この値は `NULL` です。
* `IS_NULLABLE`: カラムが NULL 値を格納できるかどうか。カラムが NULL 値を格納できる場合、この値は `YES` です。それ以外の場合は `NO` です。
* `DATA_TYPE`: カラム内のデータの型。
* `CHARACTER_MAXIMUM_LENGTH`: 文字列カラムの場合、最大の文字長。
* `CHARACTER_OCTET_LENGTH`: 文字列カラムの場合、最大のバイト長。
* `NUMERIC_PRECISION`: 数値型カラムの数値精度。
* `NUMERIC_SCALE`: 数値型カラムの数値スケール。
* `DATETIME_PRECISION`: 時刻型カラムの秒未満の精度。
* `CHARACTER_SET_NAME`: 文字列カラムの文字セットの名前。
* `COLLATION_NAME`: 文字列カラムの照合順序の名前。
* `COLUMN_TYPE`: カラムの型。
* `COLUMN_KEY`: このカラムがインデックス化されているかどうか。このフィールドには次の値が含まれる場合があります：
    * Empty: このカラムはインデックス化されていないか、このカラムはインデックス化されており、複合カラムの非ユニークインデックスで 2 番目のカラムです。
    * `PRI`: このカラムは主キーまたは複数の主キーの 1 つです。
    * `UNI`: このカラムは一意なインデックスの最初のカラムです。
    * `MUL`: このカラムは非一意インデックスの最初のカラムであり、特定の値が複数回出現することが許可される。
* `EXTRA`: 与えられたカラムの追加情報。
* `PRIVILEGES`: 現在のユーザーがこのカラムに対して持っている権限。現在は TiDB で固定されており、常に `select,insert,update,references` です。
* `COLUMN_COMMENT`: カラム定義に含まれるコメント。
* `GENERATION_EXPRESSION`: 生成されたカラムの場合、この値はカラムの値を計算するために使用される式を表示します。生成されたカラムでない場合、値は空です。

対応する `SHOW` ステートメントは以下のとおりです：

```sql
SHOW COLUMNS FROM t1 FROM test;
```

出力は次のとおりです：

```sql
+-------+---------+------+------+---------+-------+
| Field | Type    | Null | Key  | Default | Extra |
+-------+---------+------+------+---------+-------+
| a     | int(11) | YES  |      | NULL    |       |
+-------+---------+------+------+---------+-------+
1 row が返されました (0.00 秒)
```