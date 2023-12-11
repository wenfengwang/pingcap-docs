---
title: サブクエリ
summary: TiDBでのサブクエリの使用方法を学びます。

# サブクエリ

この文書では、TiDBでのサブクエリ文とカテゴリを紹介します。

## 概要

サブクエリは、別のSQLクエリ内に含まれるクエリです。サブクエリを使用すると、クエリ結果を別のクエリで使用できます。

以下は、[Bookshop](/develop/dev-guide-bookshop-schema-design.md)アプリケーションを例に、サブクエリを紹介します。

## サブクエリ文

ほとんどの場合、以下の5種類のサブクエリがあります。

- スカラーサブクエリ、例: `SELECT (SELECT s1 FROM t2) FROM t1`。
- 派生テーブル、例: `SELECT t1.s1 FROM (SELECT s1 FROM t2) t1`。
- 存在テスト、例: `WHERE NOT EXISTS(SELECT ... FROM t2)`、`WHERE t1.a IN (SELECT ... FROM t2)`。
- 量的比較、例: `WHERE t1.a = ANY(SELECT ... FROM t2)`、`WHERE t1.a = ANY(SELECT ... FROM t2)`。
- 比較演算子のオペランドとしてのサブクエリ、例: `WHERE t1.a > (SELECT ... FROM t2)`。

## サブクエリのカテゴリ

サブクエリは、「相関サブクエリ」と「自己完結型サブクエリ」に分類できます。TiDBではこれら2つのタイプを異なる方法で扱います。

サブクエリが相関しているかどうかは、その外部クエリで使用されている列を参照しているかどうかによります。

### 自己完結型サブクエリ

比較演算子 (`>`, `>=`, `<` , `<=` , `=` , または `! =`) のオペランドとしてサブクエリを使用する自己完結型サブクエリの場合、内部サブクエリは1度だけクエリを実行し、TiDBは実行計画の段階でそれを定数として書き換えます。

たとえば、`authors`テーブルの中で、平均年齢よりも年上の著者をクエリするには、比較演算子のオペランドとしてサブクエリを使用します。

```sql
SELECT * FROM authors a1 WHERE (IFNULL(a1.death_year, YEAR(NOW())) - a1.birth_year) > (
    SELECT
        AVG(IFNULL(a2.death_year, YEAR(NOW())) - a2.birth_year) AS average_age
    FROM
        authors a2
)
```

上記のクエリをTiDBが実行する前に、内部サブクエリが実行されます:

```sql
SELECT AVG(IFNULL(a2.death_year, YEAR(NOW())) - a2.birth_year) AS average_age FROM authors a2;
```

クエリの結果が34であるとします。つまり、平均年齢が34であり、34が元のサブクエリを置き換える定数として使用されます。

```sql
SELECT * FROM authors a1
WHERE (IFNULL(a1.death_year, YEAR(NOW())) - a1.birth_year) > 34;
```

結果は次のとおりです:

```
+--------+-------------------+--------+------------+------------+
| id     | name              | gender | birth_year | death_year |
+--------+-------------------+--------+------------+------------+
| 13514  | Kennith Kautzer   | 1      | 1956       | 2018       |
| 13748  | Dillon Langosh    | 1      | 1985       | NULL       |
| 99184  | Giovanny Emmerich | 1      | 1954       | 2012       |
| 180191 | Myrtie Robel      | 1      | 1958       | 2009       |
| 200969 | Iva Renner        | 0      | 1977       | NULL       |
| 209671 | Abraham Ortiz     | 0      | 1943       | 2016       |
| 229908 | Wellington Wiza   | 1      | 1932       | 1969       |
| 306642 | Markus Crona      | 0      | 1969       | NULL       |
| 317018 | Ellis McCullough  | 0      | 1969       | 2014       |
| 322369 | Mozelle Hand      | 0      | 1942       | 1977       |
| 325946 | Elta Flatley      | 0      | 1933       | 1986       |
| 361692 | Otho Langosh      | 1      | 1931       | 1997       |
| 421294 | Karelle VonRueden | 0      | 1977       | NULL       |
...
```

存在テストや量的比較などの自己完結型サブクエリについては、TiDBはそれらをより良いパフォーマンスのために同等のクエリで書き換え、置換します。詳細については、[サブクエリ関連の最適化](/subquery-optimization.md)を参照してください。

### 相関サブクエリ

相関サブクエリの場合、内部サブクエリは外部クエリの列を参照しているため、各サブクエリは外部クエリの各行ごとに1度ずつ実行されます。つまり、外部クエリが1,000万の結果を取得すると仮定すると、サブクエリも1,000万回実行され、時間とリソースを消費します。

そのため、TiDBは処理の過程で、実行計画レベルで問い合わせ効率を向上させるために[相関サブクエリの非相関化](/correlated-subquery-optimization.md)を試みます。

以下のステートメントは、同じ性別の他の著者の平均年齢よりも年上の著者をクエリするものです。

```sql
SELECT * FROM authors a1 WHERE (IFNULL(a1.death_year, YEAR(NOW())) - a1.birth_year) > (
    SELECT
        AVG(
            IFNULL(a2.death_year, YEAR(NOW())) - IFNULL(a2.birth_year, YEAR(NOW()))
        ) AS average_age
    FROM
        authors a2
    WHERE a1.gender = a2.gender
);
```

TiDBは、これを同等の`join`クエリに書き直します:

```sql
SELECT *
FROM
    authors a1,
    (
        SELECT
            gender, AVG(
                IFNULL(a2.death_year, YEAR(NOW())) - IFNULL(a2.birth_year, YEAR(NOW()))
            ) AS average_age
        FROM
            authors a2
        GROUP BY gender
    ) a2
WHERE
    a1.gender = a2.gender
    AND (IFNULL(a1.death_year, YEAR(NOW())) - a1.birth_year) > a2.average_age;
```

実際の開発では、パフォーマンスの向上が見込める代替クエリが書ける場合は、相関サブクエリを経由して問い合わせるのを避けることをお勧めします。

## 詳細情報

- [サブクエリ関連の最適化](/subquery-optimization.md)
- [相関サブクエリの非相関化](/correlated-subquery-optimization.md)
- [TiDBにおけるサブクエリの最適化](https://en.pingcap.com/blog/subquery-optimization-in-tidb/)