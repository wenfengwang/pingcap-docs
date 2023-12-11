---
title: 単一のテーブルからクエリデータを取得
summary: このドキュメントではデータベース内の単一のテーブルからクエリデータを取得する方法について説明します。

# 単一のテーブルからデータをクエリ

このドキュメントでは、SQLとさまざまなプログラミング言語を使用して、データベース内の単一のテーブルからデータをクエリする方法について説明します。

## 開始する前に

以下の内容は、[Bookshop](/develop/dev-guide-bookshop-schema-design.md) アプリケーションを例に取り上げ、TiDB内の単一のテーブルからデータをクエリする方法を示しています。

データをクエリする前に、以下の手順を完了していることを確認してください：

<CustomContent platform="tidb">

1. TiDBクラスタを構築します（[TiDB Cloud](/develop/dev-guide-build-cluster-in-cloud.md)または[TiUP](/production-deployment-using-tiup.md)を使用することをお勧めします）。

</CustomContent>

<CustomContent platform="tidb-cloud">

1. [TiDB Cloud](/develop/dev-guide-build-cluster-in-cloud.md)を使用してTiDBクラスタを構築します。

</CustomContent>

2. [Bookshopアプリケーションのテーブルスキーマとサンプルデータをインポート](/develop/dev-guide-bookshop-schema-design.md#import-table-structures-and-data)します。

<CustomContent platform="tidb">

3. [TiDBに接続](/develop/dev-guide-connect-to-tidb.md)します。

</CustomContent>

<CustomContent platform="tidb-cloud">

3. [TiDBに接続](/tidb-cloud/connect-to-tidb-cluster.md)します。

</CustomContent>

## 単純なクエリを実行

Bookshopアプリケーションのデータベースでは、「authors」テーブルに著者の基本情報が格納されています。`SELECT ... FROM ...` ステートメントを使用してデータベースからデータをクエリできます。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

MySQLクライアントで次のSQLステートメントを実行します：

```sql
SELECT id, name FROM authors;
```

以下の出力が表示されます：

```
+------------+--------------------------+
| id         | name                     |
+------------+--------------------------+
|       6357 | Adelle Bosco             |
|     345397 | Chanelle Koepp           |
|     807584 | Clementina Ryan          |
|     839921 | Gage Huel                |
|     850070 | Ray Armstrong            |
|     850362 | Ford Waelchi             |
|     881210 | Jayme Gutkowski          |
|    1165261 | Allison Kuvalis          |
|    1282036 | Adela Funk               |
...
| 4294957408 | Lyla Nitzsche            |
+------------+--------------------------+
20000 rows in set (0.05 sec)
```

</div>
<div label="Java" value="java">

Javaでは、著者の基本情報を格納するために`Author`クラスを宣言できます。データベース内の[データ型](/data-type-overview.md)と[値の範囲](/data-type-numeric.md)に応じて適切なJavaデータ型を選択する必要があります。例：

- データ型`int`のデータを格納するために`Int`型の変数を使用します。
- データ型`bigint`のデータを格納するために`Long`型の変数を使用します。
- データ型`tinyint`のデータを格納するために`Short`型の変数を使用します。
- データ型`varchar`のデータを格納するために`String`型の変数を使用します。

```java
public class Author {
    private Long id;
    private String name;
    private Short gender;
    private Short birthYear;
    private Short deathYear;

    public Author() {}

     // ゲッターやセッターは省略します。
}
```

```java
public class AuthorDAO {

    // インスタンス変数の初期化は省略します。

    public List<Author> getAuthors() throws SQLException {
        List<Author> authors = new ArrayList<>();

        try (Connection conn = ds.getConnection()) {
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("SELECT id, name FROM authors");
            while (rs.next()) {
                Author author = new Author();
                author.setId(rs.getLong("id"));
                author.setName(rs.getString("name"));
                authors.add(author);
            }
        }
        return authors;
    }
}
```

<CustomContent platform="tidb">

- [JDBCドライバを使用してTiDBに接続](/develop/dev-guide-connect-to-tidb.md#jdbc)した後、`conn.createStatus()`で`Statement`オブジェクトを作成できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- [JDBCドライバを使用してTiDBに接続](/develop/dev-guide-choose-driver-or-orm.md#java-drivers)した後、`conn.createStatus()`で`Statement`オブジェクトを作成できます。

</CustomContent>

- 次に、`stmt.executeQuery("query_sql")`を呼び出してTiDBへのデータベースクエリリクエストを開始します。
- クエリの結果は`ResultSet`オブジェクトに格納されます。`ResultSet`を走査することで、返された結果を`Author`オブジェクトにマッピングできます。

</div>
</SimpleTab>

## 結果をフィルタリングする

クエリ結果をフィルタリングするには、`WHERE`ステートメントを使用できます。

例えば、以下のコマンドは、全著者のうち1998年に生まれた著者をクエリします：

<SimpleTab groupId="language">
<div label="SQL" value="sql">

`WHERE`ステートメントにフィルタ条件を追加します：

```sql
SELECT * FROM authors WHERE birth_year = 1998;
```

</div>
<div label="Java" value="java">

Javaでは、動的パラメータを持つデータクエリリクエストを処理するために同じSQLを使用できます。

これは、パラメータをSQLステートメントに連結することで実行できます。ただし、この方法はアプリケーションのセキュリティに潜在的な[SQLインジェクション](https://en.wikipedia.org/wiki/SQL_injection)のリスクがあります。

このようなクエリに対処するために、通常のステートメントの代わりに[プリペアドステートメント](/develop/dev-guide-prepared-statement.md)を使用してください。

```java
public List<Author> getAuthorsByBirthYear(Short birthYear) throws SQLException {
    List<Author> authors = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        PreparedStatement stmt = conn.prepareStatement("""
        SELECT * FROM authors WHERE birth_year = ?;
        """);
        stmt.setShort(1, birthYear);
        ResultSet rs = stmt.executeQuery();
        while (rs.next()) {
            Author author = new Author();
            author.setId(rs.getLong("id"));
            author.setName(rs.getString("name"));
            authors.add(author);
        }
    }
    return authors;
}
```

</div>
</SimpleTab>

## 結果を並べ替える

クエリ結果を並べ替えるには、`ORDER BY`ステートメントを使用できます。

例えば、次のSQLステートメントは、`birth_year`列に基づいて`authors`テーブルを降順（`DESC`）で並べ替えて、最も若い著者のリストを取得するためのものです。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

```sql
SELECT id, name, birth_year
FROM authors
ORDER BY birth_year DESC;
```

</div>

<div label="Java" value="java">

```java
public List<Author> getAuthorsSortByBirthYear() throws SQLException {
    List<Author> authors = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("""
            SELECT id, name, birth_year
            FROM authors
            ORDER BY birth_year DESC;
            """);

        while (rs.next()) {
            Author author = new Author();
            author.setId(rs.getLong("id"));
            author.setName(rs.getString("name"));
            author.setBirthYear(rs.getShort("birth_year"));
            authors.add(author);
        }
    }
    return authors;
}
```

</div>
</SimpleTab>

結果は以下のようになります：

```
+-----------+------------------------+------------+
| id        | name                   | birth_year |
+-----------+------------------------+------------+
| 83420726  | Terrance Dach          | 2000       |
| 57938667  | Margarita Christiansen | 2000       |
| 77441404  | Otto Dibbert           | 2000       |
| 61338414  | Danial Cormier         | 2000       |
| 49680887  | Alivia Lemke           | 2000       |
| 45460101  | Itzel Cummings         | 2000       |
| 38009380  | Percy Hodkiewicz       | 2000       |
| 12943560  | Hulda Hackett          | 2000       |
| 1294029   | Stanford Herman        | 2000       |
| 111453184 | Jeffrey Brekke         | 2000       |
...
300000 rows in set (0.23 sec)
```

## クエリ結果の数を制限する

クエリ結果の数を制限するには、`LIMIT`ステートメントを使用できます。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

```sql
SELECT id, name, birth_year
FROM authors
ORDER BY birth_year DESC
LIMIT 10;
```

</div>

<div label="Java" value="java">

```java
public List<Author> getAuthorsWithLimit(Integer limit) throws SQLException {
    List<Author> authors = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        PreparedStatement stmt = conn.prepareStatement("""
            SELECT id, name, birth_year
            FROM authors
            ORDER BY birth_year DESC
            LIMIT ?;
            """);
```java
stmt.setInt(1, limit);
ResultSet rs = stmt.executeQuery();
while (rs.next()) {
    Author author = new Author();
    author.setId(rs.getLong("id"));
    author.setName(rs.getString("name"));
    author.setBirthYear(rs.getShort("birth_year"));
    authors.add(author);
}
return authors;
```

</div>
</SimpleTab>

以下是结果：

```
+-----------+------------------------+------------+
| id        | name                   | birth_year |
+-----------+------------------------+------------+
| 83420726  | Terrance Dach          | 2000       |
| 57938667  | Margarita Christiansen | 2000       |
| 77441404  | Otto Dibbert           | 2000       |
| 61338414  | Danial Cormier         | 2000       |
| 49680887  | Alivia Lemke           | 2000       |
| 45460101  | Itzel Cummings         | 2000       |
| 38009380  | Percy Hodkiewicz       | 2000       |
| 12943560  | Hulda Hackett          | 2000       |
| 1294029   | Stanford Herman        | 2000       |
| 111453184 | Jeffrey Brekke         | 2000       |
+-----------+------------------------+------------+
10 rows in set (0.11 sec)
```

在这个示例中，通过使用 `LIMIT` 语句，查询时间从 `0.23 秒` 显著减少到 `0.11 秒`。有关更多信息，请参阅 [TopN 和 Limit](/topn-limit-push-down.md)。

## 聚合查询

为了更好地了解整体数据情况，您可以使用 `GROUP BY` 语句对查询结果进行聚合。

例如，如果您想要知道哪些年份出生的作者更多，您可以通过 `birth_year` 列对 `authors` 表进行分组，然后对每年进行计数：

<SimpleTab groupId="language">
<div label="SQL" value="sql">

```sql
SELECT birth_year, COUNT (DISTINCT id) AS author_count
FROM authors
GROUP BY birth_year
ORDER BY author_count DESC;
```

</div>

<div label="Java" value="java">

```java
public class AuthorCount {
    private Short birthYear;
    private Integer authorCount;

    public AuthorCount() {}

     // 跳过 getters 和 setters。
}

public List<AuthorCount> getAuthorCountsByBirthYear() throws SQLException {
    List<AuthorCount> authorCounts = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("""
            SELECT birth_year, COUNT(DISTINCT id) AS author_count
            FROM authors
            GROUP BY birth_year
            ORDER BY author_count DESC;
            """);

        while (rs.next()) {
            AuthorCount authorCount = new AuthorCount();
            authorCount.setBirthYear(rs.getShort("birth_year"));
            authorCount.setAuthorCount(rs.getInt("author_count"));
            authorCounts.add(authorCount);
        }
    }
    return authorCounts;
}
```

</div>
</SimpleTab>

以下是结果：

```
+------------+--------------+
| birth_year | author_count |
+------------+--------------+
|       1932 |          317 |
|       1947 |          290 |
|       1939 |          282 |
|       1935 |          289 |
|       1968 |          291 |
|       1962 |          261 |
|       1961 |          283 |
|       1986 |          289 |
|       1994 |          280 |
...
|       1972 |          306 |
+------------+--------------+
71 rows in set (0.00 sec)
```

除了 `COUNT` 函数之外，TiDB 还支持其他聚合函数。有关详细信息，请参阅 [聚合 (GROUP BY) 函数](/functions-and-operators/aggregate-group-by-functions.md)。