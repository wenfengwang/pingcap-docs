---
title: ビュー
summary: `VIEWS` INFORMATION_SCHEMA テーブルについて学ぶ。

# ビュー

`VIEWS` テーブルは、SQL ビューに関する情報を提供します。

```sql
USE INFORMATION_SCHEMA;
DESC VIEWS;
```

出力は以下のようになります:

```sql
+----------------------+--------------+------+------+---------+-------+
| Field                | Type         | Null | Key  | Default | Extra |
+----------------------+--------------+------+------+---------+-------+
| TABLE_CATALOG        | varchar(512) | NO   |      | NULL    |       |
| TABLE_SCHEMA         | varchar(64)  | NO   |      | NULL    |       |
| TABLE_NAME           | varchar(64)  | NO   |      | NULL    |       |
| VIEW_DEFINITION      | longtext     | NO   |      | NULL    |       |
| CHECK_OPTION         | varchar(8)   | NO   |      | NULL    |       |
| IS_UPDATABLE         | varchar(3)   | NO   |      | NULL    |       |
| DEFINER              | varchar(77)  | NO   |      | NULL    |       |
| SECURITY_TYPE        | varchar(7)   | NO   |      | NULL    |       |
| CHARACTER_SET_CLIENT | varchar(32)  | NO   |      | NULL    |       |
| COLLATION_CONNECTION | varchar(32)  | NO   |      | NULL    |       |
+----------------------+--------------+------+------+---------+-------+
10 行が返されました (0.00 秒)
```

ビューを作成し、`VIEWS` テーブルをクエリします:

```sql
CREATE VIEW test.v1 AS SELECT 1;
SELECT * FROM VIEWS\G
```

出力は以下のようになります:

```sql
*************************** 1. 行目 ***************************
       TABLE_CATALOG: def
        TABLE_SCHEMA: test
          TABLE_NAME: v1
     VIEW_DEFINITION: SELECT 1
        CHECK_OPTION: CASCADED
        IS_UPDATABLE: NO
             DEFINER: root@127.0.0.1
       SECURITY_TYPE: DEFINER
CHARACTER_SET_CLIENT: utf8mb4
COLLATION_CONNECTION: utf8mb4_0900_ai_ci
1 行が返されました (0.00 秒)
```

`VIEWS` テーブルのフィールドは以下のように説明されています:

* `TABLE_CATALOG`: ビューが属するカタログの名前。この値は常に `def` です。
* `TABLE_SCHEMA`: ビューが属するスキーマの名前。
* `TABLE_NAME`: ビューの名前。
* `VIEW_DEFINITION`: ビューの定義。ビューが作成される際の `SELECT` 文によって作られます。
* `CHECK_OPTION`: `CHECK_OPTION` の値です。値のオプションは `NONE`, `CASCADE`, `LOCAL` です。
* `IS_UPDATABLE`: ビューに `UPDATE`/`INSERT`/`DELETE` が適用可能かどうか。TiDB では、この値は常に `NO` です。
* `DEFINER`: ビューを作成したユーザーの名前で、`'user_name'@'host_name'` の形式です。
* `SECURITY_TYPE`: `SQL SECURITY` の値です。値のオプションは `DEFINER` と `INVOKER` です。
* `CHARACTER_SET_CLIENT`: ビューが作成された際の `character_set_client` セッション変数の値です。
* `COLLATION_CONNECTION`: ビューが作成された際の `collation_connection` セッション変数の値です。