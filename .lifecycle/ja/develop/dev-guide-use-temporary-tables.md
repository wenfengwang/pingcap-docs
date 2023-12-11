---
title: 一時テーブル
summary: 一時テーブルの作成、表示、クエリ、削除方法について学びます。
---

# 一時テーブル

一時テーブルは、クエリ結果を再利用するためのテクニックとして考えることができます。

たとえば、[Bookshop](/develop/dev-guide-bookshop-schema-design.md)アプリケーションの最年長の著者について知りたい場合、最年長の著者のリストを使用する複数のクエリを書くことがあります。

例えば、次の文を使用して、`authors` テーブルから最年長の50人の著者を取得できます：

```sql
SELECT a.id, a.name, (IFNULL(a.death_year, YEAR(NOW())) - a.birth_year) AS age
FROM authors a
ORDER BY age DESC
LIMIT 50;
```

結果は以下の通りです：

```
+------------+---------------------+------+
| id         | name                | age  |
+------------+---------------------+------+
| 4053452056 | Dessie Thompson     |   80 |
| 2773958689 | Pedro Hansen        |   80 |
| 4005636688 | Wyatt Keeling       |   80 |
| 3621155838 | Colby Parker        |   80 |
| 2738876051 | Friedrich Hagenes   |   80 |
| 2299112019 | Ray Macejkovic      |   80 |
| 3953661843 | Brandi Williamson   |   80 |
...
| 4100546410 | Maida Walsh         |   80 |
+------------+---------------------+------+
50 rows in set (0.01 sec)
```

後続のクエリの利便性のために、このクエリの結果をキャッシュする必要があります。一般的なテーブルを使用する場合、異なるセッション間でテーブル名の重複問題を回避する方法と、バッチクエリの後に使用されない可能性がある中間結果のクリーンアップの必要性に注意する必要があります。

## 一時テーブルの作成

中間結果をキャッシュするために、TiDB v5.3.0 で一時テーブル機能が導入されました。TiDB は、ローカル一時テーブルがセッション終了後に自動的に削除されるため、増加する中間結果による管理の手間を気にする必要がありません。

### 一時テーブルの種類

TiDB の一時テーブルには、ローカル一時テーブルとグローバル一時テーブルの2種類があります。

- ローカル一時テーブルの場合、テーブル定義やテーブル内のデータは現在のセッションにのみ表示されます。このタイプはセッション中の中間データを一時的に格納するために適しています。
- グローバル一時テーブルの場合、テーブル定義はTiDBクラスタ全体に表示され、テーブル内のデータは現在のトランザクションにのみ表示されます。このタイプはトランザクション中の中間データを一時的に格納するために適しています。

### ローカル一時テーブルの作成

ローカル一時テーブルを作成する前に、現在のデータベースユーザーに対して `CREATE TEMPORARY TABLES` 権限を追加する必要があります。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

`CREATE TEMPORARY TABLE <table_name>` 文を使用して一時テーブルを作成できます。デフォルトのタイプはローカル一時テーブルであり、現在のセッションでのみ表示されます。

```sql
CREATE TEMPORARY TABLE top_50_eldest_authors (
    id BIGINT,
    name VARCHAR(255),
    age INT,
    PRIMARY KEY(id)
);
```

一時テーブルを作成した後、上記のクエリの結果を作成した一時テーブルに挿入するための `INSERT INTO table_name SELECT ...` 文を使用できます。

```sql
INSERT INTO top_50_eldest_authors
SELECT a.id, a.name, (IFNULL(a.death_year, YEAR(NOW())) - a.birth_year) AS age
FROM authors a
ORDER BY age DESC
LIMIT 50;
```

結果は以下の通りです：

```
Query OK, 50 rows affected (0.03 sec)
Records: 50  Duplicates: 0  Warnings: 0
```

</div>
<div label="Java" value="java">

```java
public List<Author> getTop50EldestAuthorInfo() throws SQLException {
    List<Author> authors = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        stmt.executeUpdate("""
            CREATE TEMPORARY TABLE top_50_eldest_authors (
                id BIGINT,
                name VARCHAR(255),
                age INT,
                PRIMARY KEY(id)
            );
        """);

        stmt.executeUpdate("""
            INSERT INTO top_50_eldest_authors
            SELECT a.id, a.name, (IFNULL(a.death_year, YEAR(NOW())) - a.birth_year) AS age
            FROM authors a
            ORDER BY age DESC
            LIMIT 50;
        """);

        ResultSet rs = stmt.executeQuery("""
            SELECT id, name FROM top_50_eldest_authors;
        """);

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

### グローバル一時テーブルの作成

<SimpleTab groupId="language">
<div label="SQL" value="sql">

グローバル一時テーブルを作成するには、`GLOBAL` キーワードを追加し、`ON COMMIT DELETE ROWS` で終了させることで、現在のトランザクション終了後にテーブルが削除されることを意味します。

```sql
CREATE GLOBAL TEMPORARY TABLE IF NOT EXISTS top_50_eldest_authors_global (
    id BIGINT,
    name VARCHAR(255),
    age INT,
    PRIMARY KEY(id)
) ON COMMIT DELETE ROWS;
```

グローバル一時テーブルにデータを挿入する際、トランザクションの開始を明示的に宣言するために `BEGIN` を使用する必要があります。そうしないと、`INSERT INTO` 文が実行された後にデータがクリアされます。なぜなら、Auto Commit モードでは、`INSERT INTO` 文が実行された後にトランザクションが自動的にコミットされ、トランザクションが終了するとグローバル一時テーブルがクリアされます。

</div>
<div label="Java" value="java">

グローバル一時テーブルを使用する際には、Auto Commit モードを最初にオフにする必要があります。Java では、`conn.setAutoCommit(false);` 文を使用してこれを行うことができ、`conn.commit();` を使用してトランザクションを明示的にコミットすることができます。トランザクション中にグローバル一時テーブルに追加されたデータは、トランザクションがコミットまたは取り消された後にクリアされます。

```java
public List<Author> getTop50EldestAuthorInfo() throws SQLException {
    List<Author> authors = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        conn.setAutoCommit(false);

        Statement stmt = conn.createStatement();
        stmt.executeUpdate("""
            CREATE GLOBAL TEMPORARY TABLE IF NOT EXISTS top_50_eldest_authors (
                id BIGINT,
                name VARCHAR(255),
                age INT,
                PRIMARY KEY(id)
            ) ON COMMIT DELETE ROWS;
        """);

        stmt.executeUpdate("""
            INSERT INTO top_50_eldest_authors
            SELECT a.id, a.name, (IFNULL(a.death_year, YEAR(NOW())) - a.birth_year) AS age
            FROM authors a
            ORDER BY age DESC
            LIMIT 50;
        """);

        ResultSet rs = stmt.executeQuery("""
            SELECT id, name FROM top_50_eldest_authors;
        """);

        conn.commit();
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

## 一時テーブルの表示

`SHOW [FULL] TABLES` 文を使用することで、現在存在するグローバル一時テーブルのリストを表示できますが、ローカル一時テーブルはリストに表示されません。TiDB には一時テーブル情報を格納するための `information_schema.INNODB_TEMP_TABLE_INFO` システムテーブルが存在しないためです。

例えば、グローバル一時テーブル `top_50_eldest_authors_global` はテーブルリストに表示されますが、`top_50_eldest_authors` テーブルは表示されません。

```
+-------------------------------+------------+
| Tables_in_bookshop            | Table_type |
+-------------------------------+------------+
| authors                       | BASE TABLE |
| book_authors                  | BASE TABLE |
| books                         | BASE TABLE |
| orders                        | BASE TABLE |
| ratings                       | BASE TABLE |
| top_50_eldest_authors_global  | BASE TABLE |
| users                         | BASE TABLE |
+-------------------------------+------------+
9 rows in set (0.00 sec)
```

## 一時テーブルのクエリ

一時テーブルが準備できたら、通常のデータテーブルと同様にクエリできます：

```sql
SELECT * FROM top_50_eldest_authors;
```

一時テーブルからのデータを[複数テーブルの結合クエリ](/develop/dev-guide-join-tables.md)を介してクエリに参照することができます：

```sql
EXPLAIN SELECT ANY_VALUE(ta.id) AS author_id, ANY_VALUE(ta.age), ANY_VALUE(ta.name), COUNT(*) AS books
FROM top_50_eldest_authors ta
LEFT JOIN book_authors ba ON ta.id = ba.author_id
GROUP BY ta.id;
```

[ビュー](/develop/dev-guide-use-views.md)とは異なり、一時テーブルをクエリする場合は、元のデータ挿入に使用されたクエリを実行するのではなく、一時テーブルからデータを直接取得します。いくつかのケースでは、このことがクエリパフォーマンスを向上させることがあります。

## 一時テーブルの削除
```
A local temporary table in a session is automatically dropped after the **session** ends, along with both data and table schema. A global temporary table in a transaction is automatically cleared at the end of the **transaction**, but the table schema remains and needs to be deleted manually.

To manually drop local temporary tables, use the `DROP TABLE` or `DROP TEMPORARY TABLE` syntax. For example:

```sql
DROP TEMPORARY TABLE top_50_eldest_authors;
```

To manually drop global temporary tables, use the `DROP TABLE` or `DROP GLOBAL TEMPORARY TABLE` syntax. For example:

```sql
DROP GLOBAL TEMPORARY TABLE top_50_eldest_authors_global;
```

## Limitation

For limitations of temporary tables in TiDB, see [Compatibility restrictions with other TiDB features](/temporary-tables.md#compatibility-restrictions-with-other-tidb-features).

## Read more

- [Temporary Tables](/temporary-tables.md)
```