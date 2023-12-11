---
title: RUNAWAY_WATCHES
summary: `RUNAWAY_WATCHES` INFORMATION_SCHEMA テーブルの使用方法について学びます。

# RUNAWAY_WATCHES

`RUNAWAY_WATCHES` テーブルは、予想よりも多くのリソースを消費する暴走クエリの監視リストを示します。詳細については、[暴走クエリ](/tidb-resource-control.md#manage-queries-that-consume-more-resources-than-expected-runaway-queries) を参照してください。

```sql
USE INFORMATION_SCHEMA;
DESC RUNAWAY_WATCHES;
```

```sql
+---------------------+--------------+------+------+---------+-------+
| Field               | Type         | Null | Key  | Default | Extra |
+---------------------+--------------+------+------+---------+-------+
| ID                  | bigint(64)   | NO   |      | NULL    |       |
| RESOURCE_GROUP_NAME | varchar(32)  | NO   |      | NULL    |       |
| START_TIME          | varchar(32)  | NO   |      | NULL    |       |
| END_TIME            | varchar(32)  | YES  |      | NULL    |       |
| WATCH               | varchar(12)  | NO   |      | NULL    |       |
| WATCH_TEXT          | text         | NO   |      | NULL    |       |
| SOURCE              | varchar(128) | NO   |      | NULL    |       |
| ACTION              | varchar(12)  | NO   |      | NULL    |       |
+---------------------+--------------+------+------+---------+-------+
8 行が返されました (0.00 秒)
```

> **警告:**
>
> この機能は実験的なものです。本番環境で使用しないことをお勧めします。この機能は事前の通知なしに変更または削除される可能性があります。バグを見つけた場合は、GitHub で [issue](https://github.com/pingcap/tidb/issues) を報告できます。

## 例

暴走クエリの監視リストをクエリします:

```sql
SELECT * FROM INFORMATION_SCHEMA.RUNAWAY_WATCHES;
```

出力は次のようになります:

```sql
*************************** 1 行目 ***************************
                 ID: 20003
RESOURCE_GROUP_NAME: rg2
         START_TIME: 2023-07-28 13:06:08
           END_TIME: UNLIMITED
              WATCH: Similar
         WATCH_TEXT: 5b7fd445c5756a16f910192ad449c02348656a5e9d2aa61615e6049afbc4a82e
             SOURCE: 127.0.0.1:4000
             ACTION: Kill
*************************** 2 行目 ***************************
                 ID: 16004
RESOURCE_GROUP_NAME: rg2
         START_TIME: 2023-07-28 01:45:30
           END_TIME: UNLIMITED
              WATCH: Similar
         WATCH_TEXT: 3d48fca401d8cbb31a9f29adc9c0f9d4be967ca80a34f59c15f73af94e000c84
             SOURCE: 127.0.0.1:4000
             ACTION: Kill
2 行が返されました (0.00 秒)
```

リソースグループ `rg1` にウォッチアイテムをリストに追加します:

```sql
QUERY WATCH ADD RESOURCE GROUP rg1 SQL TEXT EXACT TO 'select * from sbtest.sbtest1';
```

再度、暴走クエリの監視リストをクエリします:

```sql
SELECT * FROM INFORMATION_SCHEMA.RUNAWAY_WATCHES\G;
```

出力は次のようになります:

```sql
*************************** 1 行目 ***************************
                 ID: 20003
RESOURCE_GROUP_NAME: rg2
         START_TIME: 2023-07-28 13:06:08
           END_TIME: UNLIMITED
              WATCH: Similar
         WATCH_TEXT: 5b7fd445c5756a16f910192ad449c02348656a5e9d2aa61615e6049afbc4a82e
             SOURCE: 127.0.0.1:4000
             ACTION: Kill
*************************** 2 行目 ***************************
                 ID: 16004
RESOURCE_GROUP_NAME: rg2
         START_TIME: 2023-07-28 01:45:30
           END_TIME: UNLIMITED
              WATCH: Similar
         WATCH_TEXT: 3d48fca401d8cbb31a9f29adc9c0f9d4be967ca80a34f59c15f73af94e000c84
             SOURCE: 127.0.0.1:4000
             ACTION: Kill
*************************** 3 行目 ***************************
                 ID: 20004
RESOURCE_GROUP_NAME: rg1
         START_TIME: 2023-07-28 14:23:04
           END_TIME: UNLIMITED
              WATCH: Exact
         WATCH_TEXT: select * from sbtest.sbtest1
             SOURCE: manual
             ACTION: NoneAction
3 行が返されました (0.00 秒)
```

`RUNAWAY_WATCHES` テーブルの各列フィールドの意味は次のとおりです:

- `ID`: ウォッチアイテムの ID。
- `RESOURCE_GROUP_NAME`: リソースグループの名前。
- `START_TIME`: 開始時刻。
- `END_TIME`: 終了時刻。 `UNLIMITED` は、ウォッチアイテムの有効期間が無制限であることを示します。
- `WATCH`: クイック識別の一致タイプ。値は次のとおりです: 
    - `Plan`: プランダイジェストが一致しています。この場合、 `WATCH_TEXT` 列にプランダイジェストが表示されます。
    - `Similar`: SQL ダイジェストが一致しています。この場合、 `WATCH_TEXT` 列に SQL ダイジェストが表示されます。
    - `Exact`: SQL テキストが一致しています。この場合、 `WATCH_TEXT` 列に SQL テキストが表示されます。
- `SOURCE`: ウォッチアイテムのソース。 `QUERY_LIMIT` ルールによって識別される場合は、識別された TiDB IP アドレスが表示されます。手動で追加された場合は、 `manual` が表示されます。
- `ACTION`: 識別後の対応操作。