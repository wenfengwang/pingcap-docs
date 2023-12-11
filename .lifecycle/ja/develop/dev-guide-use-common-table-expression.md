---
title: 共通テーブル式
summary: TiDBのCTE機能について学び、SQLステートメントをより効率的に記述する方法を理解しましょう。
---

# 共通テーブル式

取引シナリオの中には、アプリケーションの複雑さから、最大2,000行にも及ぶ単一のSQLステートメントを書く必要があるものがあります。そのステートメントには、多くの集計や多層サブクエリのネスティングが含まれることがあります。このような長いSQLステートメントをメンテナンスすることは、開発者にとって悪夢になり得ます。

これほど長いSQLステートメントを回避するためには、クエリを単純化するために[ビュー](/develop/dev-guide-use-views.md)を使うか、[一時テーブル](/develop/dev-guide-use-temporary-tables.md)を使用して中間クエリ結果をキャッシュすることができます。

この文書では、TiDBにおける共通テーブル式(CTE)の構文を紹介します。これは、クエリ結果の再利用をより便利にする方法です。

TiDB v5.1以降、TiDBはANSI SQL99標準のCTEと再帰をサポートしています。CTEを利用することで、複雑なアプリケーションロジックのためのSQLステートメントをより効率的に書くことができ、コードのメンテナンスもずっと簡単になります。

## 基本的な使用法

共通テーブル式(CTE)は、SQLステートメント内で複数回参照することができる一時的な結果セットです。これにより、ステートメントの可読性と実行効率が向上します。CTEを使用するには`WITH`文を適用します。

共通テーブル式には、非再帰CTEと再帰CTEの二つのタイプがあります。

### 非再帰CTE

非再帰CTEは、以下の構文を使用して定義できます:

```sql
WITH <クエリ名> AS (
    <クエリ定義>
)
SELECT ... FROM <クエリ名>;
```

たとえば、50人の最も古い著者それぞれが何冊の本を書いたかを知りたい場合は、次のステップを踏みます:

<SimpleTab groupId="language">
<div label="SQL" value="sql">

一時テーブル(/develop/dev-guide-use-temporary-tables.md)のステートメントを以下のように変更します:

```sql
WITH 最古の50人の著者CTE AS (
    SELECT a.id, a.name, (IFNULL(a.death_year, YEAR(NOW())) - a.birth_year) AS 年齢
    FROM 著者 a
    ORDER BY 年齢 DESC
    LIMIT 50
)
SELECT
    ANY_VALUE(ta.id) AS 著者ID,
    ANY_VALUE(ta.年齢) AS 著者年齢,
    ANY_VALUE(ta.name) AS 著者名,
    COUNT(*) AS 本の数
FROM 最古の50人の著者CTE ta
LEFT JOIN book_authors ba ON ta.id = ba.author_id
GROUP BY ta.id;
```

結果は以下の通りです:

```
+------------+------------+---------------------+-------+
| 著者ID     | 著者年齢   | 著者名              | 本の数 |
+------------+------------+---------------------+-------+
| 1238393239 |         80 | Araceli Purdy       |     1 |
|  817764631 |         80 | Ivory Davis         |     3 |
| 3093759193 |         80 | Lysanne Harris      |     1 |
| 2299112019 |         80 | Ray Macejkovic      |     4 |
...
+------------+------------+---------------------+-------+
50行 (0.01秒)
```

</div>
<div label="Java" value = "java">

```java
public List<Author> getTop50EldestAuthorInfoByCTE() throws SQLException {
    List<Author> 著者 = new ArrayList<>();
    try (Connection conn = ds.getConnection()) {
        Statement stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("""
            WITH 最古の50人の著者CTE AS (
                SELECT a.id, a.name, (IFNULL(a.death_year, YEAR(NOW())) - a.birth_year) AS 年齢
                FROM 著者 a
                ORDER BY 年齢 DESC
                LIMIT 50
            )
            SELECT
                ANY_VALUE(ta.id) AS 著者ID,
                ANY_VALUE(ta.name) AS 著者名,
                ANY_VALUE(ta.年齢) AS 著者年齢,
                COUNT(*) AS 本の数
            FROM 最古の50人の著者CTE ta
            LEFT JOIN book_authors ba ON ta.id = ba.author_id
            GROUP BY ta.id;
        """);
        while (rs.next()) {
            Author 著者 = new Author();
            著者.setId(rs.getLong("著者ID"));
            著者.setName(rs.getString("著者名"));
            著者.setAge(rs.getShort("著者年齢"));
            著者.setBooks(rs.getInt("本の数"));
            著者.add(著者);
        }
    }
    return 著者;
}
```

</div>
</SimpleTab>

著者"Ray Macejkovic"が4冊の本を書いたことがわかります。CTEクエリを使って、これら4冊の本の順序と評価情報を以下のようにさらに取得できます:

```sql
WITH レイ・マセコビッチ氏の著書として (
    SELECT *
    FROM 本 b
    LEFT JOIN 本の著者 ba ON b.id = ba.book_id
    WHERE 著者ID = 2299112019
), 平均評価付きの本 AS (
    SELECT
        b.id AS 本ID,
        AVG(r.score) AS 平均評価
    FROM レイ・マセコビッチ氏の著書として b
    LEFT JOIN 評価 r ON b.id = r.book_id
    GROUP BY b.id
), 注文付きの本 AS (
    SELECT
        b.id AS 本ID,
        COUNT(*) AS 注文数
    FROM レイ・マセコビッチ氏の著書として b
    LEFT JOIN 注文 o ON b.id = o.book_id
    GROUP BY b.id
)
SELECT
    b.id AS `本ID`,
    b.title AS `本のタイトル`,
    br.平均評価 AS `平均評価`,
    bo.注文数 AS `注文数`
FROM
    レイ・マセコビッチ氏の著書として b
    LEFT JOIN 平均評価付きの本 br ON b.id = br.本ID
    LEFT JOIN 注文付きの本 bo ON b.id = bo.本ID
;
```

結果は以下の通りです:

```
+------------+-------------------------+----------------+--------+
| 本ID       | 本のタイトル            | 平均評価       | 注文数 |
+------------+-------------------------+----------------+--------+
|  481008467 | The Documentary of goat |         2.0000 |     16 |
| 2224531102 | Brandt Skiles           |         2.7143 |     17 |
| 2641301356 | Sheridan Bashirian      |         2.4211 |     12 |
| 4154439164 | Karson Streich          |         2.5833 |     19 |
+------------+-------------------------+----------------+--------+
4行 (0.06秒)
```

このSQLステートメントでは、`,`により分割された3つのCTEブロックが定義されています。

まず、CTEブロック`レイ・マセコビッチ氏の著書として`で著者（IDは`2299112019`）によって書かれた本をチェックします。次に、これらの本の平均評価と注文をそれぞれ`平均評価付きの本`と`注文付きの本`で見つけます。最後に、`JOIN`文によって結果を集約します。

これらのクエリの中で、`レイ・マセコビッチ氏の著書として`のクエリは一度だけ実行され、その結果はTiDBによって一時スペースにキャッシュされます。`平均評価付きの本`と`注文付きの本`のクエリが`レイ・マセコビッチ氏の著書として`を参照するとき、TiDBはこの一時スペースから直接結果を得ます。

> **ヒント:**
>
> デフォルトのCTEクエリの効率が良くない場合は、[`MERGE()`](/optimizer-hints.md#merge)ヒントを使用してCTEサブクエリを外部クエリへ展開し、効率を向上させることができます。

### 再帰CTE

再帰CTEは、以下の構文を使用して定義できます:

```sql
WITH RECURSIVE <クエリ名> AS (
    <クエリ定義>
)
SELECT ... FROM <クエリ名>;
```

典型的な例は、再帰CTEを使用して一連の[フィボナッチ数](https://en.wikipedia.org/wiki/Fibonacci_number)を生成することです:

```sql
WITH RECURSIVE フィボナッチ (n, fib_n, next_fib_n) AS
(
  SELECT 1, 0, 1
  UNION ALL
  SELECT n + 1, next_fib_n, fib_n + next_fib_n FROM フィボナッチ WHERE n < 10
)
SELECT * FROM フィボナッチ;
```

結果は以下の通りです:

```
+------+-------+------------+
| n    | fib_n | next_fib_n |
+------+-------+------------+
|    1 |     0 |          1 |
|    2 |     1 |          1 |
|    3 |     1 |          2 |
|    4 |     2 |          3 |
|    5 |     3 |          5 |
|    6 |     5 |          8 |
|    7 |     8 |         13 |
|    8 |    13 |         21 |
|    9 |    21 |         34 |
|   10 |    34 |         55 |
+------+-------+------------+
10行 (0.00秒)
```

## 詳細を読む

- [WITH](/sql-statements/sql-statement-with.md)