---
title: Stale Read（古いデータの読み込み）
summary: 特定の条件下でクエリを加速するためにStale Readを使用する方法を学びます。

# Stale Read（古いデータの読み込み）

Stale ReadはTiDBがTiDBに格納されたデータの過去のバージョンを読み取るための仕組みです。この仕組みを使用することで、特定の時点または指定された時間範囲内の対応する過去のデータを読み取ることができ、そのためストレージノード間のデータレプリケーションに起因する遅延を節約できます。Stale Readを使用すると、TiDBはデータ読み取りのためにレプリカをランダムに選択するため、すべてのレプリカがデータ読み取りに使用できます。

実践では、TiDBでStale Readを有効にするかどうかを慎重に検討してください。[使用シナリオ](/stale-read.md#usage-scenarios-of-stale-read)に基づいてStale Readを有効にしてください。アプリケーションが非リアルタイムデータの読み取りを許容できない場合は、Stale Readを有効にしないでください。

TiDBはStale Readの3つのレベルを提供しています：ステートメントレベル、トランザクションレベル、セッションレベル。

## 紹介

[Bookshop](/develop/dev-guide-bookshop-schema-design.md)アプリケーションでは、次のSQL文を使用して、最新の発行された書籍とその価格をクエリすることができます。

```sql
SELECT id, title, type, price FROM books ORDER BY published_at DESC LIMIT 5;
```

結果は次のとおりです：

```
+------------+------------------------------+-----------------------+--------+
| id         | title                        | type                  | price  |
+------------+------------------------------+-----------------------+--------+
| 3181093216 | The Story of Droolius Caesar | Novel                 | 100.00 |
| 1064253862 | Collin Rolfson               | Education & Reference |  92.85 |
| 1748583991 | The Documentary of cat       | Magazine              | 159.75 |
|  893930596 | Myrl Hills                   | Education & Reference | 356.85 |
| 3062833277 | Keven Wyman                  | Life                  | 477.91 |
+------------+------------------------------+-----------------------+--------+
5 rows in set (0.02 sec)
```

この時点（2022-04-20 15:20:00）のリストにおいて、*The Story of Droolius Caesar*の価格は100.0です。

同時に、販売者はこの本が非常に人気があり、価格を150.0に引き上げました。以下のSQL文を使用して：

```sql
UPDATE books SET price = 150 WHERE id = 3181093216;
```

結果は次のとおりです：

```
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

最新の書籍リストをクエリすることで、この本の価格が上がったことがわかります。

```
+------------+------------------------------+-----------------------+--------+
| id         | title                        | type                  | price  |
+------------+------------------------------+-----------------------+--------+
| 3181093216 | The Story of Droolius Caesar | Novel                 | 150.00 |
| 1064253862 | Collin Rolfson               | Education & Reference |  92.85 |
| 1748583991 | The Documentary of cat       | Magazine              | 159.75 |
|  893930596 | Myrl Hills                   | Education & Reference | 356.85 |
| 3062833277 | Keven Wyman                  | Life                  | 477.91 |
+------------+------------------------------+-----------------------+--------+
5 rows in set (0.01 sec)
```

最新のデータを使用する必要がない場合、強く整合性のある読み取り中に生じる遅延を避けるために、Stale Readでクエリすることができますが、これによって古いデータが返される可能性があります。

Bookshopアプリケーションにおいて、書籍一覧ページでのリアルタイムな価格は必要ないが、書籍の詳細および注文ページでのみ必要となる場合、アプリケーション全般のスループットを向上させるためにStale Readを使用できます。

## ステートメントレベル

<SimpleTab groupId="language">
<div label="SQL" value="sql">

特定の時点の前の書籍の価格をクエリするには、上記のクエリ文に `AS OF TIMESTAMP <datetime>` 句を追加します。

```sql
SELECT id, title, type, price FROM books AS OF TIMESTAMP '2022-04-20 15:20:00' ORDER BY published_at DESC LIMIT 5;
```

結果は次のとおりです：

```
+------------+------------------------------+-----------------------+--------+
| id         | title                        | type                  | price  |
+------------+------------------------------+-----------------------+--------+
| 3181093216 | The Story of Droolius Caesar | Novel                 | 100.00 |
| 1064253862 | Collin Rolfson               | Education & Reference |  92.85 |
| 1748583991 | The Documentary of cat       | Magazine              | 159.75 |
|  893930596 | Myrl Hills                   | Education & Reference | 356.85 |
| 3062833277 | Keven Wyman                  | Life                  | 477.91 |
+------------+------------------------------+-----------------------+--------+
5 rows in set (0.01 sec)
```

特定の時点だけでなく、次のように指定することもできます：

- `AS OF TIMESTAMP NOW() - INTERVAL 10 SECOND` は10秒前の最新データをクエリします。
- `AS OF TIMESTAMP TIDB_BOUNDED_STALENESS('2016-10-08 16:45:26', '2016-10-08 16:45:29')` は`2016-10-08 16:45:26` から `2016-10-08 16:45:29` の最新データをクエリします。
- `AS OF TIMESTAMP TIDB_BOUNDED_STALENESS(NOW() -INTERVAL 20 SECOND, NOW())` は20秒以内の最新データをクエリします。

指定されたタイムスタンプまたは間隔が現在時刻よりも早すぎたり、遅すぎたりすることはできません。また、`NOW()` はデフォルトで秒の精度を持ちます。より高い精度を実珅するには、ミリ秒の精度を使用するなどのパラメータを追加することができます。詳細については、[MySQL documentation](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_now)を参照してください。

期限切れのデータは、TiDBの[Garbage Collection](/garbage-collection-overview.md)によってリサイクルされ、データはクリアされるまで短い期間保持されます。この期間は[GC Life Time（デフォルト10分）](/system-variables.md#tidb_gc_life_time-new-in-v50)と呼ばれます。GCが開始すると、現在の時刻から時間が使用されて **GC Safe Point** が設定されます。GC Safe Pointよりも前のデータを読み取ろうとすると、TiDBは次のエラーを報告します：

```
ERROR 9006 (HY000): GC life time is shorter than transaction duration...
```

指定されたタイムスタンプが未来の時間の場合、TiDBは次のエラーを報告します：

```
ERROR 9006 (HY000): cannot set read timestamp to a future time.
```

</div>
<div label="Java" value="java">

```java
public class BookDAO {

    // 一部のコードは省略...

    public List<Book> getTop5LatestBooks() throws SQLException {
        List<Book> books = new ArrayList<>();
        try (Connection conn = ds.getConnection()) {
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("""
            SELECT id, title, type, price FROM books ORDER BY published_at DESC LIMIT 5;
            """);
            while (rs.next()) {
                Book book = new Book();
                book.setId(rs.getLong("id"));
                book.setTitle(rs.getString("title"));
                book.setType(rs.getString("type"));
                book.setPrice(rs.getDouble("price"));
                books.add(book);
            }
        }
        return books;
    }

    public void updateBookPriceByID(Long id, Double price) throws SQLException {
        try (Connection conn = ds.getConnection()) {
            PreparedStatement stmt = conn.prepareStatement("""
            UPDATE books SET price = ? WHERE id = ?;
            """);
            stmt.setDouble(1, price);
            stmt.setLong(2, id);
            int affects = stmt.executeUpdate();
            if (affects == 0) {
                throw new SQLException("Failed to update the book with id: " + id);
            }
        }
    }

    public List<Book> getTop5LatestBooksWithStaleRead(Integer seconds) throws SQLException {
        List<Book> books = new ArrayList<>();
        try (Connection conn = ds.getConnection()) {
            PreparedStatement stmt = conn.prepareStatement("""
            SELECT id, title, type, price FROM books AS OF TIMESTAMP NOW() - INTERVAL ? SECOND ORDER BY published_at DESC LIMIT 5;
            """);
            stmt.setInt(1, seconds);
            ResultSet rs = stmt.executeQuery();
            while (rs.next()) {
                Book book = new Book();
                book.setId(rs.getLong("id"));
                book.setTitle(rs.getString("title"));
                book.setType(rs.getString("type"));
                book.setPrice(rs.getDouble("price"));
                books.add(book);
            }
        } catch (SQLException e) {
            if ("HY000".equals(e.getSQLState()) && e.getErrorCode() == 1105) {
                System.out.println("WARN: cannot set read timestamp to a future time.");
            } else if ("HY000".equals(e.getSQLState()) && e.getErrorCode() == 9006) {
                System.out.println("WARN: GC life time is shorter than transaction duration.");
            } else {
                throw e;
            }
        }
        return books;
    }
}
```java
List<Book> top5LatestBooks = bookDAO.getTop5LatestBooks();

if (top5LatestBooks.size() > 0) {
    System.out.println("直近の書籍の価格（更新前）：" + top5LatestBooks.get(0).getPrice());

    Book book = top5LatestBooks.get(0);
    bookDAO.updateBookPriceByID(book.getId(), book.price + 10);

    top5LatestBooks = bookDAO.getTop5LatestBooks();
    System.out.println("直近の書籍の価格（更新後）：" + top5LatestBooks.get(0).getPrice());

    // ステールリードを使用します。
    top5LatestBooks = bookDAO.getTop5LatestBooksWithTxnStaleRead(5);
    System.out.println("直近の書籍の価格（ステールかもしれない）：" + top5LatestBooks.get(0).getPrice());

    // ステールリードデータを将来の時間で取得しようとします。
    bookDAO.getTop5LatestBooksWithStaleRead(-5);

    // 20分前のデータをステールリードしようとします。
    bookDAO.getTop5LatestBooksWithStaleRead(20 * 60);
}
```
```java
while (rs.next()) {
    Book book = new Book();
    book.setId(rs.getLong("id"));
    book.setTitle(rs.getString("title"));
    book.setType(rs.getString("type"));
    book.setPrice(rs.getDouble("price"));
    books.add(book);
}

// トランザクションをコミットします。
conn.commit();
} catch (SQLException e) {
if ("HY000".equals(e.getSQLState()) && e.getErrorCode() == 1105) {
    System.out.println("WARN: cannot set read timestamp to a future time.");
} else if ("HY000".equals(e.getSQLState()) && e.getErrorCode() == 9006) {
    System.out.println("WARN: GC life time is shorter than transaction duration.");
} else {
    throw e;
}
}
return books;
}
}
```

</div>
</SimpleTab>

## セッションレベル

履歴データの読み取りをサポートするために、TiDBはv5.4から新しいシステム変数`tidb_read_staleness`を導入しました。現在のセッションが読み取りを許可される履歴データの範囲を設定するために使用できます。そのデータ型は`int`で、スコープは`SESSION`です。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

セッションでStale Readを有効化する:

```sql
SET @@tidb_read_staleness="-5";
```

たとえば、値を`-5`に設定した場合、TiKVまたはTiFlashに対応する履歴データがあると、TiDBは5秒の時間範囲内で可能な限り新しいタイムスタンプを選択します。

セッションでStale Readを無効化する:

```sql
set @@tidb_read_staleness="";
```

</div>
<div label="Java" value="java">

```java
public static class StaleReadHelper{

public static void enableStaleReadOnSession(Connection conn, Integer seconds) throws SQLException {
    PreparedStatement stmt = conn.prepareStatement(
        "SET @@tidb_read_staleness= ?;"
    );
    stmt.setString(1, String.format("-%d", seconds));
    stmt.execute();
}

public static void disableStaleReadOnSession(Connection conn) throws SQLException {
    PreparedStatement stmt = conn.prepareStatement(
        "SET @@tidb_read_staleness=\"\";"
    );
    stmt.execute();
}

}
```

</div>
</SimpleTab>

## 詳細情報

- [Stale Readの使用シナリオ](/stale-read.md)
- [`AS OF TIMESTAMP`句を使用して履歴データを読み取る](/as-of-timestamp.md)
- `tidb_read_staleness`システム変数を使用して履歴データを読み取る](/tidb-read-staleness.md)