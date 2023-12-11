---
title: マルチテーブル結合クエリ
summary: このドキュメントでは、マルチテーブル結合クエリの使用方法について説明します。
---

# マルチテーブル結合クエリ

多くのシナリオでは、複数のテーブルからデータを取得するために1つのクエリを使用する必要があります。2つ以上のテーブルからデータを結合するには、`JOIN`ステートメントを使用できます。

## 結合タイプ

このセクションでは、結合タイプについて詳しく説明します。

### INNER JOIN

内部結合の結合結果は、結合条件に一致する行のみを返します。

![内部結合](/media/develop/inner-join.png)

例えば、最も多作な著者を知りたい場合は、著者テーブル（`authors`という名前）を書籍著者テーブル（`book_authors`という名前）と結合する必要があります。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

次のSQLステートメントでは、左側のテーブル`authors`と右側のテーブル`book_authors`を内部結合として結合条件`a.id = ba.author_id`で結合することを宣言するために、`JOIN`キーワードを使用します。結果セットには、結合条件を満たす行のみが含まれます。著者が本を1冊も書いていない場合、`authors`テーブルの彼のレコードは結合条件を満たさず、結果セットには表示されません。

```sql
SELECT ANY_VALUE(a.id) AS author_id, ANY_VALUE(a.name) AS author_name, COUNT(ba.book_id) AS books
FROM authors a
JOIN book_authors ba ON a.id = ba.author_id
GROUP BY ba.author_id
ORDER BY books DESC
LIMIT 10;
```

クエリの結果は次のとおりです。

```
+------------+----------------+-------+
| author_id  | author_name    | books |
+------------+----------------+-------+
|  431192671 | Emilie Cassin  |     7 |
|  865305676 | Nola Howell    |     7 |
|  572207928 | Lamar Koch     |     6 |
| 3894029860 | Elijah Howe    |     6 |
| 1150614082 | Cristal Stehr  |     6 |
| 4158341032 | Roslyn Rippin  |     6 |
| 2430691560 | Francisca Hahn |     6 |
| 3346415350 | Leta Weimann   |     6 |
| 1395124973 | Albin Cole     |     6 |
| 2768150724 | Caleb Wyman    |     6 |
+------------+----------------+-------+
10 rows in set (0.01 sec)
```

</div>
<div label="Java" value="java">

```java
public List<Author> getTop10AuthorsOrderByBooks() throws SQLException {
    List<Author> authors = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("""
        SELECT ANY_VALUE(a.id) AS author_id, ANY_VALUE(a.name) AS author_name, COUNT(ba.book_id) AS books
        FROM authors a
        JOIN book_authors ba ON a.id = ba.author_id
        GROUP BY ba.author_id
        ORDER BY books DESC
        LIMIT 10;
        """);
        while (rs.next()) {
            Author author = new Author();
            author.setId(rs.getLong("author_id"));
            author.setName(rs.getString("author_name"));
            author.setBooks(rs.getInt("books"));
            authors.add(author);
        }
    }
    return authors;
}
```

</div>
</SimpleTab>

### 左外部結合

左外部結合は、左側のテーブルのすべての行と、結合条件に一致する右側のテーブルの値を返します。右側のテーブルに一致する行がない場合は、`NULL`で埋められます。

![左外部結合](/media/develop/left-outer-join.png)

一部の場合では、複数のテーブルを使用してデータクエリを完了したいが、結合条件が満たされないためにデータセットが小さくなりすぎるのを避けたい場合があります。

例えば、Bookshopアプリのホームページでは、新しい本のリストと平均評価を表示したい場合があります。この場合、新しい本にはまだ誰も評価していない可能性があります。内部結合を使用すると、これらの評価されていない本の情報がフィルタリングされてしまい、期待される結果とは異なります。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

次のSQLステートメントでは、左側のテーブル`books`が左外部結合で右側のテーブル`ratings`に結合されることを宣言するために、`LEFT JOIN`キーワードを使用して、`books`テーブルのすべての行が返されるようにします。

```sql
SELECT b.id AS book_id, ANY_VALUE(b.title) AS book_title, AVG(r.score) AS average_score
FROM books b
LEFT JOIN ratings r ON b.id = r.book_id
GROUP BY b.id
ORDER BY b.published_at DESC
LIMIT 10;
```

クエリの結果は次のとおりです。

```
+------------+---------------------------------+---------------+
| book_id    | book_title                      | average_score |
+------------+---------------------------------+---------------+
| 3438991610 | The Documentary of lion         |        2.7619 |
| 3897175886 | Torey Kuhn                      |        3.0000 |
| 1256171496 | Elmo Vandervort                 |        2.5500 |
| 1036915727 | The Story of Munchkin           |        2.0000 |
|  270254583 | Tate Kovacek                    |        2.5000 |
| 1280950719 | Carson Damore                   |        3.2105 |
| 1098041838 | The Documentary of grasshopper  |        2.8462 |
| 1476566306 | The Adventures of Vince Sanford |        2.3529 |
| 4036300890 | The Documentary of turtle       |        2.4545 |
| 1299849448 | Antwan Olson                    |        3.0000 |
+------------+---------------------------------+---------------+
10 rows in set (0.30 sec)
```

最新の出版された本はすでに多くの評価があります。上記の方法を確認するために、_The Documentary of lion_の評価をすべて削除してみましょう。

```sql
DELETE FROM ratings WHERE book_id = 3438991610;
```

再度クエリを実行します。_The Documentary of lion_の本は結果セットに表示されますが、右側のテーブル`ratings`の`score`から計算された`average_score`列は`NULL`で埋められます。

```
+------------+---------------------------------+---------------+
| book_id    | book_title                      | average_score |
+------------+---------------------------------+---------------+
| 3438991610 | The Documentary of lion         |          NULL |
| 3897175886 | Torey Kuhn                      |        3.0000 |
| 1256171496 | Elmo Vandervort                 |        2.5500 |
| 1036915727 | The Story of Munchkin           |        2.0000 |
|  270254583 | Tate Kovacek                    |        2.5000 |
| 1280950719 | Carson Damore                   |        3.2105 |
| 1098041838 | The Documentary of grasshopper  |        2.8462 |
| 1476566306 | The Adventures of Vince Sanford |        2.3529 |
| 4036300890 | The Documentary of turtle       |        2.4545 |
| 1299849448 | Antwan Olson                    |        3.0000 |
+------------+---------------------------------+---------------+
10 rows in set (0.30 sec)
```

`INNER JOIN`を使用した場合はどうなるでしょうか。試してみるのはあなた次第です。

</div>
<div label="Java" value="java">

```java
public List<Book> getLatestBooksWithAverageScore() throws SQLException {
    List<Book> books = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("""
        SELECT b.id AS book_id, ANY_VALUE(b.title) AS book_title, AVG(r.score) AS average_score
        FROM books b
        LEFT JOIN ratings r ON b.id = r.book_id
        GROUP BY b.id
        ORDER BY b.published_at DESC
        LIMIT 10;
        """);
        while (rs.next()) {
            Book book = new Book();
            book.setId(rs.getLong("book_id"));
            book.setTitle(rs.getString("book_title"));
            book.setAverageScore(rs.getFloat("average_score"));
            books.add(book);
        }
    }
    return books;
}
```

</div>
</SimpleTab>

### RIGHT OUTER JOIN

右外部結合は、右側のテーブルのすべてのレコードと、結合条件に一致する左側のテーブルの値を返します。一致する値がない場合は、`NULL`で埋められます。

![右外部結合](/media/develop/right-outer-join.png)

### CROSS JOIN
ジョイン条件が定数の場合、2つのテーブル間の内部ジョインを[クロスジョイン](https://en.wikipedia.org/wiki/Join_(SQL)#Cross_join)と呼びます。 クロスジョインは左側のテーブルのすべてのレコードを右側のテーブルのすべてのレコードに結合します。 左側のテーブルのレコード数が`m`で、右側のテーブルのレコード数が`n`である場合、結果セットには`m \* n`のレコードが生成されます。

### LEFT SEMI JOIN

TiDBはSQL構文レベルでは`LEFT SEMI JOIN table_name`をサポートしていません。ただし、[サブクエリ関連の最適化](/subquery-optimization.md)は、再構成された同等のJOINクエリに対してデフォルトの結合方法として`semi join`を使用します。

## 暗黙のジョイン

SQL標準にジョインを明示的に宣言する`JOIN`ステートメントが追加される前に、SQLステートメントで`FROM t1, t2`句を使用して2つ以上のテーブルを結合し、`WHERE t1.id = t2.id`句を使用して結合の条件を指定することができました。これは暗黙のジョインと解釈できますが、これは内部ジョインを使用してテーブルを結合します。

## ジョイン関連のアルゴリズム

TiDBは次の一般的なテーブル結合アルゴリズムをサポートしています。

- [Index Join](/explain-joins.md#index-join)
- [Hash Join](/explain-joins.md#hash-join)
- [Merge Join](/explain-joins.md#merge-join)

オプティマイザは結合されたテーブルのデータボリュームなどの要因に基づいて適切な結合アルゴリズムを選択して実行します。Joinに対してクエリがどのアルゴリズムを使用しているかは、`EXPLAIN`ステートメントを使用して確認できます。

TiDBのオプティマイザが最適な結合アルゴリズムに従って実行しない場合は、[オプティマイザヒント](/optimizer-hints.md)を使用してTiDBにより良い結合アルゴリズムを使用するように強制することができます。

たとえば、前述の左結合クエリの例が、オプティマイザに選択されていないハッシュ結合アルゴリズムを使用して実行が速くなると仮定すると、`SELECT`キーワードの後ろにヒント`/*+ HASH_JOIN(b, r) */`を追加できます。テーブルにエイリアスがある場合は、ヒントでエイリアスを使用してください。

```sql
EXPLAIN SELECT /*+ HASH_JOIN(b, r) */ b.id AS book_id, ANY_VALUE(b.title) AS book_title, AVG(r.score) AS average_score
FROM books b
LEFT JOIN ratings r ON b.id = r.book_id
GROUP BY b.id
ORDER BY b.published_at DESC
LIMIT 10;
```

ジョインアルゴリズムに関連するヒント:

- [MERGE_JOIN(t1_name [, tl_name ...])](/optimizer-hints.md#merge_joint1_name--tl_name-)
- [INL_JOIN(t1_name [, tl_name ...])](/optimizer-hints.md#inl_joint1_name--tl_name-)
- [INL_HASH_JOIN(t1_name [, tl_name ...])](/optimizer-hints.md#inl_hash_join)
- [HASH_JOIN(t1_name [, tl_name ...])](/optimizer-hints.md#hash_joint1_name--tl_name-)

## ジョインの順序

実際の業務シナリオでは、複数のテーブルのジョインステートメントが非常に一般的です。ジョインの実行効率は、ジョインで使用する各テーブルの順序に関連しています。 TiDBはJoin Reorderアルゴリズムを使用して、複数のテーブルのジョインの順序を決定します。

オプティマイザによって選択されたジョインの順序が期待どおりに最適でない場合は、`STRAIGHT_JOIN`を使用して、`FROM`句で使用されるテーブルの順序でクエリを結合するようにTiDBに強制することができます。

```sql
EXPLAIN SELECT *
FROM authors a STRAIGHT_JOIN book_authors ba STRAIGHT_JOIN books b
WHERE b.id = ba.book_id AND ba.author_id = a.id;
```

このJoin Reorderアルゴリズムの実装の詳細と制限についての詳細については、[Join Reorder Algorithmの紹介](/join-reorder.md)を参照してください。

## 関連情報

- [ジョインを使用する説明ステートメント](/explain-joins.md)
- [Join Reorderの紹介](/join-reorder.md)