---
title: ページネーション結果
summary: TiDBのページネーション結果機能を紹介します。

# ページネーション結果

大きなクエリ結果をページ単位で取得するため、"ページネーション"方式で欲しい部分を取得できます。

## クエリ結果のページネーション

TiDBでは、`LIMIT`文を使用してクエリ結果をページネーションできます。例えば:

```sql
SELECT * FROM table_a t ORDER BY gmt_modified DESC LIMIT offset, row_count;
```

`offset`はレコードの開始番号を示し、`row_count`はページごとのレコード数を示します。TiDBはまた、`LIMIT row_count OFFSET offset`構文をサポートしています。

ページネーションを使用する場合、ランダムなデータを表示する必要がない限り、`ORDER BY`文を使用してクエリ結果をソートすることが推奨されます。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

例えば、[Bookshop](/develop/dev-guide-bookshop-schema-design.md)アプリケーションのユーザーが、ページネーション方式で最新の出版された書籍を表示できるようにするには、`LIMIT 0, 10`文を使用します。これにより、結果リストの最初のページを最大10レコードで取得できます。2ページ目を取得するためには、`LIMIT 10, 10`文に変更します。

```sql
SELECT *
FROM books
ORDER BY published_at DESC
LIMIT 0, 10;
```

</div>
<div label="Java" value="java">

アプリケーション開発では、バックエンドプログラムは`offset`パラメーターではなく`page_number`パラメーター（要求されたページの番号を意味する）と`page_size`パラメーター（1ページあたりのレコード数を制御する）をフロントエンドから受け取ります。そのため、クエリを実行する前にいくつかの変換が必要です。

```java
public List<Book> getLatestBooksPage(Long pageNumber, Long pageSize) throws SQLException {
    pageNumber = pageNumber < 1L ? 1L : pageNumber;
    pageSize = pageSize < 10L ? 10L : pageSize;
    Long offset = (pageNumber - 1) * pageSize;
    Long limit = pageSize;
    List<Book> books = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        PreparedStatement stmt = conn.prepareStatement("""
        SELECT id, title, published_at
        FROM books
        ORDER BY published_at DESC
        LIMIT ?, ?;
        """);
        stmt.setLong(1, offset);
        stmt.setLong(2, limit);
        ResultSet rs = stmt.executeQuery();
        while (rs.next()) {
            Book book = new Book();
            book.setId(rs.getLong("id"));
            book.setTitle(rs.getString("title"));
            book.setPublishedAt(rs.getDate("published_at"));
            books.add(book);
        }
    }
    return books;
}
```

</div>
</SimpleTab>

単一フィールドのプライマリキー表のバッチ処理

通常、プライマリキーまたはユニークインデックスを使用してページネーションSQL文を記述し、`LIMIT`句の`offset`キーワードを使用して指定された行数でページを分割します。その後、各ページを独立したトランザクションにラップして柔軟なページング更新を実現します。ただし、欠点も明らかです。プライマリキーまたはユニークインデックスをソートする必要があるため、大きな`offset`は特にデータ量が多い場合には多くの計算リソースを消費します。

次に、より効率的なページングバッチ処理方法を紹介します:

<SimpleTab groupId="language">
<div label="SQL" value="sql">

まず、データをプライマリキーでソートし、ウィンドウ関数の`row_number()`を呼び出して各行に行番号を生成します。その後、集計関数を呼び出して行番号を指定されたページサイズでグループ化し、各ページの最小値と最大値を計算します。

```sql
SELECT
    floor((t.row_num - 1) / 1000) + 1 AS page_num,
    min(t.id) AS start_key,
    max(t.id) AS end_key,
    count(*) AS page_size
FROM (
    SELECT id, row_number() OVER (ORDER BY id) AS row_num
    FROM books
) t
GROUP BY page_num
ORDER BY page_num;
```

その結果は次のとおりです:

```
+----------+------------+------------+-----------+
| page_num | start_key  | end_key    | page_size |
+----------+------------+------------+-----------+
|        1 |     268996 |  213168525 |      1000 |
|        2 |  213210359 |  430012226 |      1000 |
|        3 |  430137681 |  647846033 |      1000 |
|        4 |  647998334 |  848878952 |      1000 |
|        5 |  848899254 | 1040978080 |      1000 |
...
|       20 | 4077418867 | 4294004213 |      1000 |
+----------+------------+------------+-----------+
20 rows in set (0.01 sec)
```

次に、`WHERE id BETWEEN start_key AND end_key`文を使用して各スライスのデータをクエリします。データをより効率的に更新するためには、データを変更する際に上記のスライス情報を使用できます。

ページ1の全書籍の基本情報を削除するには、上記の結果のページ1の値で`start_key`と`end_key`を置き換えて次のようにします:

```sql
DELETE FROM books
WHERE
    id BETWEEN 268996 AND 213168525
ORDER BY id;
```

</div>
<div label="Java" value="java">

Javaでは、`PageMeta`クラスを定義してページのメタ情報を保持します。

```java
public class PageMeta<K> {
    private Long pageNum;
    private K startKey;
    private K endKey;
    private Long pageSize;

    // ゲッターとセッターは省略します。

}
```

`getPageMetaList()`メソッドを定義してページのメタ情報リストを取得し、次に`deleteBooksByPageMeta()`メソッドを定義してページのメタ情報に基づいてデータをバッチ処理で削除します。

```java
public class BookDAO {
    public List<PageMeta<Long>> getPageMetaList() throws SQLException {
        List<PageMeta<Long>> pageMetaList = new ArrayList<>();
        try (Connection conn = ds.getConnection()) {
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("""
            SELECT
                floor((t.row_num - 1) / 1000) + 1 AS page_num,
                min(t.id) AS start_key,
                max(t.id) AS end_key,
                count(*) AS page_size
            FROM (
                SELECT id, row_number() OVER (ORDER BY id) AS row_num
                FROM books
            ) t
            GROUP BY page_num
            ORDER BY page_num;
            """);
            while (rs.next()) {
                PageMeta<Long> pageMeta = new PageMeta<>();
                pageMeta.setPageNum(rs.getLong("page_num"));
                pageMeta.setStartKey(rs.getLong("start_key"));
                pageMeta.setEndKey(rs.getLong("end_key"));
                pageMeta.setPageSize(rs.getLong("page_size"));
                pageMetaList.add(pageMeta);
            }
        }
        return pageMetaList;
    }

    public void deleteBooksByPageMeta(PageMeta<Long> pageMeta) throws SQLException {
        try (Connection conn = ds.getConnection()) {
            PreparedStatement stmt = conn.prepareStatement("DELETE FROM books WHERE id >= ? AND id <= ?");
            stmt.setLong(1, pageMeta.getStartKey());
            stmt.setLong(2, pageMeta.getEndKey());
            stmt.executeUpdate();
        }
    }
}
```

次の文は、ページ1のデータを削除するためのものです:

```java
List<PageMeta<Long>> pageMetaList = bookDAO.getPageMetaList();
if (pageMetaList.size() > 0) {
    bookDAO.deleteBooksByPageMeta(pageMetaList.get(0));
}
```

次の文は、ページごとにページングで全書籍データを削除するためのものです:

```java
List<PageMeta<Long>> pageMetaList = bookDAO.getPageMetaList();
pageMetaList.forEach((pageMeta) -> {
    try {
        bookDAO.deleteBooksByPageMeta(pageMeta);
    } catch (SQLException e) {
        e.printStackTrace();
    }
});
```

</div>
</SimpleTab>

この方法は、データソートの処理を頻繁に行うことによる計算リソースの無駄を避けることで、バッチ処理の効率を大幅に改善します。

## 複合プライマリキー表のバッチ処理

### 非クラスタリングインデックス表

非クラスタリングインデックス表（または"非インデックス組織表"とも呼ばれる）には、内部フィールド`_tidb_rowid`をページネーションキーとして使用でき、ページネーション方法は単一フィールドのプライマリキー表と同じです。

> **ヒント:**
>
> `SHOW CREATE TABLE users;`文を使用してテーブルのプライマリキーが[クラスタリングインデックス](/clustered-indexes.md)を使用しているかどうかを確認できます。

例えば:

```sql
SELECT
    floor((t.row_num - 1) / 1000) + 1 AS page_num,
    min(t._tidb_rowid) AS start_key,
    max(t._tidb_rowid) AS end_key,
    count(*) AS page_size
FROM (
    SELECT _tidb_rowid, row_number () OVER (ORDER BY _tidb_rowid) AS row_num
    FROM users
) t
GROUP BY page_num
ORDER BY page_num;
```

その結果は次のとおりです:

```
+----------+-----------+---------+-----------+
| page_num | start_key | end_key | page_size |
+----------+-----------+---------+-----------+
```
|         1 |          1 |    1000 |      1000 |
|         2 |       1001 |    2000 |      1000 |
|         3 |       2001 |    3000 |      1000 |
|         4 |       3001 |    4000 |      1000 |
|         5 |       4001 |    5000 |      1000 |
|         6 |       5001 |    6000 |      1000 |
|         7 |       6001 |    7000 |      1000 |
|         8 |       7001 |    8000 |      1000 |
|         9 |       8001 |    9000 |      1000 |
|        10 |       9001 |    9990 |       990 |
+----------+-----------+---------+-----------+
10 rows in set (0.00 sec)

### クラスターインデックステーブル

クラスターインデックステーブル（または "インデックス・オーガナイズド・テーブル" とも呼ばれる）の場合、複数の列の値をキーとして連結するために `concat` 関数を使用し、その後ウィンドウ関数を使用してページング情報をクエリできます。

この時点でキーは文字列であることに注意してください。そして、`min` および `max` 集計関数を使用してスライス内の正しい `start_key` および `end_key` を取得するために、文字列連結のフィールドの長さが常に同じであることを確認する必要があります。文字列連結用のフィールドの長さが固定されていない場合は、`LPAD` 関数を使用してパディングすることができます。

たとえば、`ratings` テーブルのデータのページングバッチを以下のように実装できます。

以下のステートメントを使用してメタ情報テーブルを作成します。`bigint` 型である `book_id` および `user_id` によって連結されたキーは同じ長さに変換できないため、`bigint` の最大ビット数 19 に従って長さを `0` でパディングするために `LPAD` 関数が使用されています。

```sql
SELECT
    floor((t1.row_num - 1) / 10000) + 1 AS page_num,
    min(mvalue) AS start_key,
    max(mvalue) AS end_key,
    count(*) AS page_size
FROM (
    SELECT
        concat('(', LPAD(book_id, 19, 0), ',', LPAD(user_id, 19, 0), ')') AS mvalue,
        row_number() OVER (ORDER BY book_id, user_id) AS row_num
    FROM ratings
) t1
GROUP BY page_num
ORDER BY page_num;
```

> **注記:**
>
> 前述の SQL ステートメントは `TableFullScan` として実行されます。データ量が大きい場合、クエリは遅くなる可能性がありますが、[TiFlash をご利用いただく](/tiflash/tiflash-overview.md#use-tiflash)ことで処理を高速化できます。

結果は次のようになります。

```
+----------+-------------------------------------------+-------------------------------------------+-----------+
| page_num | start_key                                 | end_key                                   | page_size |
+----------+-------------------------------------------+-------------------------------------------+-----------+
|        1 | (0000000000000268996,0000000000092104804) | (0000000000140982742,0000000000374645100) |     10000 |
|        2 | (0000000000140982742,0000000000456757551) | (0000000000287195082,0000000004053200550) |     10000 |
|        3 | (0000000000287196791,0000000000191962769) | (0000000000434010216,0000000000237646714) |     10000 |
|        4 | (0000000000434010216,0000000000375066168) | (0000000000578893327,0000000002167504460) |     10000 |
|        5 | (0000000000578893327,0000000002457322286) | (0000000000718287668,0000000001502744628) |     10000 |
...
|       29 | (0000000004002523918,0000000000902930986) | (0000000004147203315,0000000004090920746) |     10000 |
|       30 | (0000000004147421329,0000000000319181561) | (0000000004294004213,0000000003586311166) |      9972 |
+----------+-------------------------------------------+-------------------------------------------+-----------+
30 rows in set (0.28 sec)
```

ページ 1 のすべての評価レコードを削除するには、上記の結果でページ 1 の値で `start_key` および `end_key` を置き換えます。

```sql
SELECT * FROM ratings
WHERE
    (book_id > 268996 AND book_id < 140982742)
    OR (
        book_id = 268996 AND user_id >= 92104804
    )
    OR (
        book_id = 140982742 AND user_id <= 374645100
    )
ORDER BY book_id, user_id;
```