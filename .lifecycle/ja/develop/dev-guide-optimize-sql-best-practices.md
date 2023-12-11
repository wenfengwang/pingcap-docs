---
title: パフォーマンスチューニングのベストプラクティス
summary: TiDBのパフォーマンスチューニングのベストプラクティスを紹介します。
---

# パフォーマンスチューニングのベストプラクティス

このドキュメントでは、TiDBデータベースの使用に関するいくつかのベストプラクティスを紹介します。

## DMLのベストプラクティス

このセクションでは、TiDBを使用する際のDMLに関連するベストプラクティスについて説明します。

### 複数行ステートメントの使用

テーブルの複数の行を変更する必要がある場合、複数行ステートメントを使用することをお勧めします:

```sql
INSERT INTO t VALUES (1, 'a'), (2, 'b'), (3, 'c');

DELETE FROM t WHERE id IN (1, 2, 3);
```

複数の単一行ステートメントを使用しないようにしてください:

```sql
INSERT INTO t VALUES (1, 'a');
INSERT INTO t VALUES (2, 'b');
INSERT INTO t VALUES (3, 'c');

DELETE FROM t WHERE id = 1;
DELETE FROM t WHERE id = 2;
DELETE FROM t WHERE id = 3;
```

### `PREPARE`の使用

同じSQLステートメントを複数回実行する必要がある場合、SQL構文を繰り返し解析するオーバーヘッドを避けるために、`PREPARE`ステートメントを使用することをお勧めします。

<SimpleTab>
<div label="Golang">

```go
func BatchInsert(db *sql.DB) error {
    stmt, err := db.Prepare("INSERT INTO t (id) VALUES (?), (?), (?), (?), (?)")
    if err != nil {
        return err
    }
    for i := 0; i < 1000; i += 5 {
        values := []interface{}{i, i + 1, i + 2, i + 3, i + 4}
        _, err = stmt.Exec(values...)
        if err != nil {
            return err
        }
    }
    return nil
}
```

</div>

<div label="Java">

```java
public void batchInsert(Connection connection) throws SQLException {
    PreparedStatement statement = connection.prepareStatement(
            "INSERT INTO `t` (`id`) VALUES (?), (?), (?), (?), (?)");
    for (int i = 0; i < 1000; i ++) {
        statement.setInt(i % 5 + 1, i);

        if (i % 5 == 4) {
            statement.executeUpdate();
        }
    }
}
```

</div>
</SimpleTab>

`PREPARE`ステートメントを繰り返し実行しないでください。それ以外の場合、実行効率が向上しません。

### 必要な列のみをクエリする

すべての列からデータを取得する必要がない場合は、`SELECT *`を使用してすべての列のデータを返さないでください。以下のクエリは非効率です:

```sql
SELECT * FROM books WHERE title = 'Marian Yost';
```

必要な列のみをクエリするようにしてください。例:

```sql
SELECT title, price FROM books WHERE title = 'Marian Yost';
```

### バルク削除の使用

大量のデータを削除する場合は、[バルク削除](/develop/dev-guide-delete-data.md#bulk-delete)を使用することをお勧めします。

### バルク更新の使用

大量のデータを更新する場合は、[バルク更新](/develop/dev-guide-update-data.md#bulk-update)を使用することをお勧めします。

### `DELETE`の代わりに`TRUNCATE`を使用して全体のテーブルデータを削除する

テーブルからすべてのデータを削除する必要がある場合は、`TRUNCATE`ステートメントを使用することをお勧めします:

```sql
TRUNCATE TABLE t;
```

全体のテーブルデータを削除するために`DELETE`を使用することはお勧めしません:

```sql
DELETE FROM t;
```

## DDLのベストプラクティス

このセクションでは、TiDBのDDLを使用する際のベストプラクティスについて説明します。

### 主キーのベストプラクティス

主キーを選択する際のルールについては、[主キーの選択時に従うべきルール](/develop/dev-guide-create-table.md#guidelines-to-follow-when-selecting-primary-key)を参照してください。

## インデックスのベストプラクティス

[インデックスのベストプラクティス](/develop/dev-guide-index-best-practice.md)を参照してください。

### インデックスの追加のベストプラクティス

TiDBはオンラインインデックス追加操作をサポートしています。[ADD INDEX](/sql-statements/sql-statement-add-index.md)または[CREATE INDEX](/sql-statements/sql-statement-create-index.md)ステートメントを使用してインデックスを追加できます。これによりテーブルのデータ読み取りおよび書き込みがブロックされません。また、次のシステム変数を変更して、インデックスの`re-organize`フェーズ中の並行性とバッチサイズを調整できます:

* [`tidb_ddl_reorg_worker_cnt`](/system-variables.md#tidb_ddl_reorg_worker_cnt)
* [`tidb_ddl_reorg_batch_size`](/system-variables.md#tidb_ddl_reorg_batch_size)

オンラインアプリケーションへの影響を減らすために、デフォルトのインデックス追加操作の速度は遅くなっています。対象のインデックス追加操作の対象列が読み取り負荷のみを含むか、オンラインワークロードと直接関係ない場合、上記の変数の値を適切に増やしてインデックス追加操作の速度を上げることができます:

```sql
SET @@global.tidb_ddl_reorg_worker_cnt = 16;
SET @@global.tidb_ddl_reorg_batch_size = 4096;
```

インデックス追加操作の対象列が頻繁に更新される場合（`UPDATE`、`INSERT`、`DELETE`を含む）、上記の変数を増やすと、より多くの書き込み競合が発生し、オンラインワークロードに影響を与えます。そのため、定数の繰り返しによる長いリトライにより、インデックス追加操作には時間がかかることがあります。この場合、上記の変数の値を減らして、オンラインアプリケーションとの書き込み競合を避けるようにすることをお勧めします:

```sql
SET @@global.tidb_ddl_reorg_worker_cnt = 4;
SET @@global.tidb_ddl_reorg_batch_size = 128;
```

## トランザクションの競合

<CustomContent platform="tidb">

トランザクションの競合を特定し解決する方法については、[ロック競合のトラブルシューティング](/troubleshoot-lock-conflicts.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

トランザクションの競合を特定し解決する方法については、[ロック競合のトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-lock-conflicts)を参照してください。

</CustomContent>

## TiDBを使用したJavaアプリケーションの開発のベストプラクティス

<CustomContent platform="tidb">

TiDBを使用したJavaアプリケーションの開発のベストプラクティスについては、[TiDBを使用したJavaアプリケーションの開発のベストプラクティス](/best-practices/java-app-best-practices.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDBを使用したJavaアプリケーションの開発のベストプラクティスについては、[TiDBを使用したJavaアプリケーションの開発のベストプラクティス](https://docs.pingcap.com/tidb/stable/java-app-best-practices)を参照してください。

</CustomContent>

### 関連項目

<CustomContent platform="tidb">

- [高並列書き込みのベストプラクティス](/best-practices/high-concurrency-best-practices.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

- [高並列書き込みのベストプラクティス](https://docs.pingcap.com/tidb/stable/high-concurrency-best-practices)

</CustomContent>