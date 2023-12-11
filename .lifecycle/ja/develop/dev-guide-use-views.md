---
title: ビュー
summary: TiDB でビューの使用方法を学びます。

# ビュー

本書では、TiDB でビューを使用する方法について説明します。

## 概要

TiDB はビューをサポートしています。ビューは仮想テーブルとして機能し、そのスキーマはビューを作成する `SELECT` 文で定義されます。

- 使用される複雑なクエリに対してビューを作成できます。これにより、複雑なクエリを簡単で便利にします。
- 使用者に対して安全なフィールドとデータのみを公開するためにビューを作成できます。これにより、基になるテーブル内の機密フィールドとデータのセキュリティを保証できます。

## ビューの作成

TiDB では、`CREATE VIEW` 文を使用して複雑なクエリをビューとして定義できます。構文は次のとおりです。

```sql
CREATE VIEW ビュー名 AS クエリ;
```

既存のビューやテーブルと同じ名前のビューを作成することはできないことに注意してください。

たとえば、[複数テーブルの結合クエリ](/develop/dev-guide-join-tables.md) は、`JOIN` 文を介して `books` テーブルと `ratings` テーブルを結合して本と平均評価のリストを取得します。

後続のクエリの便宜のために、次のような文を使用してクエリをビューとして定義できます。

```sql
CREATE VIEW book_with_ratings AS
SELECT b.id AS book_id, ANY_VALUE(b.title) AS book_title, AVG(r.score) AS average_score
FROM books b
LEFT JOIN ratings r ON b.id = r.book_id
GROUP BY b.id;
```

## ビューのクエリ

ビューを作成したら、通常のテーブルと同様に `SELECT` 文を使用してビューをクエリできます。

```sql
SELECT * FROM book_with_ratings LIMIT 10;
```

TiDB がビューをクエリするときは、そのビューに関連付けられた `SELECT` 文をクエリします。

## ビューの更新

現在、TiDB においてビューは `ALTER VIEW ビュー名 AS クエリ;` をサポートしていませんが、以下の2つの方法でビューを「更新」できます。

-  `DROP VIEW ビュー名;` 文を使用して古いビューを削除し、次に `CREATE VIEW ビュー名 AS クエリ;` 文を使用して新しいビューを作成してビューを更新します。
-  `CREATE OR REPLACE VIEW ビュー名 AS クエリ;` 文を使用して、同じ名前の既存のビューを上書きします。

```sql
CREATE OR REPLACE VIEW book_with_ratings AS
SELECT b.id AS book_id, ANY_VALUE(b.title), ANY_VALUE(b.published_at) AS book_title, AVG(r.score) AS average_score
FROM books b
LEFT JOIN ratings r ON b.id = r.book_id
GROUP BY b.id;
```

## ビューに関連する情報の取得

### `SHOW CREATE TABLE|VIEW ビュー名` 文を使用

```sql
SHOW CREATE VIEW book_with_ratings\G
```

結果は次のとおりです。

```
*************************** 1. row ***************************
                View: book_with_ratings
         Create View: CREATE ALGORITHM=UNDEFINED DEFINER=`root`@`%` SQL SECURITY DEFINER VIEW `book_with_ratings` (`book_id`, `ANY_VALUE(b.title)`, `book_title`, `average_score`) AS SELECT `b`.`id` AS `book_id`,ANY_VALUE(`b`.`title`) AS `ANY_VALUE(b.title)`,ANY_VALUE(`b`.`published_at`) AS `book_title`,AVG(`r`.`score`) AS `average_score` FROM `bookshop`.`books` AS `b` LEFT JOIN `bookshop`.`ratings` AS `r` ON `b`.`id`=`r`.`book_id` GROUP BY `b`.`id`
character_set_client: utf8mb4
collation_connection: utf8mb4_general_ci
1 row in set (0.00 sec)
```

### `INFORMATION_SCHEMA.VIEWS` テーブルをクエリ

```sql
SELECT * FROM information_schema.views WHERE TABLE_NAME = 'book_with_ratings'\G
```

結果は次のとおりです。

```
*************************** 1. row ***************************
       TABLE_CATALOG: def
        TABLE_SCHEMA: bookshop
          TABLE_NAME: book_with_ratings
     VIEW_DEFINITION: SELECT `b`.`id` AS `book_id`,ANY_VALUE(`b`.`title`) AS `ANY_VALUE(b.title)`,ANY_VALUE(`b`.`published_at`) AS `book_title`,AVG(`r`.`score`) AS `average_score` FROM `bookshop`.`books` AS `b` LEFT JOIN `bookshop`.`ratings` AS `r` ON `b`.`id`=`r`.`book_id` GROUP BY `b`.`id`
        CHECK_OPTION: CASCADED
        IS_UPDATABLE: NO
             DEFINER: root@%
       SECURITY_TYPE: DEFINER
CHARACTER_SET_CLIENT: utf8mb4
COLLATION_CONNECTION: utf8mb4_general_ci
1 row in set (0.00 sec)
```

## ビューの削除

`DROP VIEW ビュー名;` 文を使用してビューを削除します。

```sql
DROP VIEW book_with_ratings;
```

## 制限

TiDB におけるビューの制限事項については、[ビューの制限事項](/views.md#limitations) を参照してください。

## 詳細情報

- [ビュー](/views.md)
- [CREATE VIEW 文](/sql-statements/sql-statement-create-view.md)
- [DROP VIEW 文](/sql-statements/sql-statement-drop-view.md)
- [ビューを使用した EXPLAIN 文](/explain-views.md)
- [TiFlink: TiKV と Flink を使用した強力な一貫性のあるマテリアライズドビュー](https://github.com/tiflink/tiflink)