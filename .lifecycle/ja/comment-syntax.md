---
title: コメント構文
summary: 本書はTiDBでサポートされているコメント構文について紹介します。
aliases: ['/docs/dev/comment-syntax/','/docs/dev/reference/sql/language-structure/comment-syntax/']
---

# コメント構文

本書ではTiDBでサポートされているコメント構文について説明します。

TiDBは以下の3つのコメントスタイルをサポートしています:

- 行をコメントアウトするには `#` を使用します:

    {{< copyable "sql" >}}

    ```sql
    SELECT 1+1;     # comments
    ```

    ```
    +------+
    | 1+1  |
    +------+
    |    2 |
    +------+
    1 row in set (0.00 sec)
    ```

- 行をコメントアウトするには `--` を使用します:

    {{< copyable "sql" >}}

    ```sql
    SELECT 1+1;     -- comments
    ```

    ```
    +------+
    | 1+1  |
    +------+
    |    2 |
    +------+
    1 row in set (0.00 sec)
    ```
    
    また、このスタイルでは `--` の後に少なくとも1つの空白を必要とします:

   {{< copyable "sql" >}}

    ```sql
    SELECT 1+1--1;
    ```

    ```
    +--------+
    | 1+1--1 |
    +--------+
    |      3 |
    +--------+
    1 row in set (0.01 sec)
    ```

- ブロックや複数行をコメントアウトするには `/* */` を使用します:

   {{< copyable "sql" >}}

    ```sql
    SELECT 1 /* this is an in-line comment */ + 1;
    ```

    ```
    +--------+
    | 1  + 1 |
    +--------+
    |      2 |
    +--------+
    1 row in set (0.01 sec)
    ```

    {{< copyable "sql" >}}

    ```sql
    SELECT 1+
    /*
    /*> this is a
    /*> multiple-line comment
    /*> */
        1;
    ```

    ```
    +-------------------+
    | 1+
            1 |
    +-------------------+
    |                 2 |
    +-------------------+
    1 row in set (0.001 sec)
    ```

## MySQL互換のコメント構文

MySQLと同様に、TiDBはCコメントスタイルの変種をサポートしています:

```
/*! 特定のコード */
```

または

```
/*!50110 特定のコード */
```

このスタイルではTiDBはコメント内のステートメントを実行します。

例:

```sql
SELECT /*! STRAIGHT_JOIN */ col1 FROM table1,table2 WHERE ...
```

TiDBでは、次のバージョンも使えます:

```sql
SELECT STRAIGHT_JOIN col1 FROM table1,table2 WHERE ...
```

コメント内でサーババージョン番号が指定されている場合、例えば `/*!50110 KEY_BLOCK_SIZE=1024 */` のように、MySQLではこのコメントの内容はMySQLのバージョンが5.1.10以上の場合のみ処理されます。しかし、TiDBではMySQLのバージョン番号は効果がなく、コメント内のすべての内容が処理されます。

## TiDB固有のコメント構文

TiDBには固有のコメント構文（すなわち、TiDB固有のコメント構文）があり、以下の2つの種類に分類できます:

* `/*T! 特定のコード */`: この構文はTiDBによってのみ解析および実行され、他のデータベースでは無視されます。
* `/*T![feature_id] 特定のコード */`: この構文はTiDBの異なるバージョン間の互換性を確保するために使用されます。TiDBは現在のバージョンで`feature_id`の対応機能を実装している場合にのみ、このコメント内のSQLフラグメントを解析できます。例えば、`AUTO_RANDOM`機能がv3.1.1で導入されているため、このバージョンのTiDBは`/*T![auto_rand] auto_random */` を `auto_random` に解析できます。しかし、`AUTO_RANDOM`機能がv3.0.0には実装されていないため、前述のSQLステートメントのフラグメントは無視されます。**`/*T![` の中にスペースを残さないでください**。

## オプティマイザコメント構文

もう一つのコメントの種類は、特にオプティマイザヒントとして扱われます:

{{< copyable "sql" >}}

```sql
SELECT /*+ hint */ FROM ...;
```

TiDBがサポートしているオプティマイザヒントの詳細については、[オプティマイザヒント](/optimizer-hints.md)を参照してください。

> **注意:**
>
> MySQLクライアントでは、TiDB独自のコメント構文はコメントとして扱われ、デフォルトでクリアされます。MySQLクライアントでは5.7.7より前のバージョンでは、ヒントもコメントとして扱われ、デフォルトでクリアされます。クライアントを起動する際に`--comments`オプションを使用することをお勧めします。例えば、 `mysql -h 127.0.0.1 -P 4000 -uroot --comments`。 

詳細については、[Comment Syntax](https://dev.mysql.com/doc/refman/8.0/en/comments.html)を参照してください。
