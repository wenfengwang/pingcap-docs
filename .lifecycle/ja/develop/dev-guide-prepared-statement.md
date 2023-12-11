---
title: プリペアドステートメント
summary: TiDBのプリペアドステートメントの使用方法について学びます。

# プリペアドステートメント

[プリペアドステートメント](/sql-statements/sql-statement-prepare.md) は、パラメーターのみが異なる複数のSQLステートメントをテンプレート化します。SQLステートメントとパラメーターを分離することができます。次のSQLステートメントの以下の側面を改善するために使用できます。

- **セキュリティ**：パラメーターとステートメントが分離されているため、[SQLインジェクション](https://ja.wikipedia.org/wiki/SQL%E3%82%A4%E3%83%B3%E3%82%B8%E3%82%A7%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3)攻撃のリスクを回避できます。
- **パフォーマンス**：ステートメントがTiDBサーバーで事前に解析されるため、後続の実行に対してパラメーターのみが渡され、全SQLステートメントの解析コスト、SQLステートメント文字列の結合、およびネットワーク転送のコストが節約されます。

ほとんどのアプリケーションでは、SQLステートメントを列挙することができます。アプリケーション全体のデータクエリに対して限られた数のSQLステートメントを使用できます。したがって、プリペアドステートメントの使用はベストプラクティスです。

## SQL構文

このセクションでは、プリペアドステートメントの作成、実行、削除のためのSQL構文について説明します。

### プリペアドステートメントの作成

```sql
PREPARE {prepared_statement_name} FROM '{prepared_statement_sql}';
```

| パラメータ名 | 説明 |
| :-------------------------: | :------------------------------------: |
| `{prepared_statement_name}` | プリペアドステートメントの名前|
| `{prepared_statement_sql}`  | プレースホルダとしてのクエスチョンマークを持つプリペアドステートメントのSQL |

詳細については[PREPAREステートメント](/sql-statements/sql-statement-prepare.md)を参照してください。

### プリペアドステートメントの使用

プリペアドステートメントは**ユーザ変数**のみをパラメーターとして使用できるため、[`SET`ステートメント](/sql-statements/sql-statement-set-variable.md)を使用して[`EXECUTE`ステートメント](/sql-statements/sql-statement-execute.md)がプリペアドステートメントを呼び出す前に変数を設定する必要があります。

```sql
SET @{parameter_name} = {parameter_value};
EXECUTE {prepared_statement_name} USING @{parameter_name};
```

| パラメータ名 | 説明 |
| :-------------------------: | :-------------------------------------------------------------------: |
|     `{parameter_name}`      |                              ユーザ変数の名前                               |
|     `{parameter_value}`     |                              ユーザ変数の値                               |
| `{prepared_statement_name}` | プリプレアドステートメントの名前。これは[プリペアドステートメントの作成](#create-a-prepared-statement)で定義された名前と同じでなければなりません |

詳細については[`EXECUTE`ステートメント](/sql-statements/sql-statement-execute.md)を参照してください。

### プリペアドステートメントの削除

```sql
DEALLOCATE PREPARE {prepared_statement_name};
```

| パラメータ名 | 説明 |
| :-------------------------: | :-------------------------------------------------------------------: |
| `{prepared_statement_name}` | プリプレアドステートメントの名前。これは[プリペアドステートメントの作成](#create-a-prepared-statement)で定義された名前と同じでなければなりません |

詳細については[`DEALLOCATE`ステートメント](/sql-statements/sql-statement-deallocate.md)を参照してください。

## 例

このセクションでは、プリペアドステートメントの2つの例、データの`SELECT`と`INSERT`について説明します。

### `SELECT`の例

たとえば、[`bookshop`アプリケーション](/develop/dev-guide-bookshop-schema-design.md#books-table)の`id = 1`の本をクエリする必要があります。

<SimpleTab groupId="language">

<div label="SQL" value="sql">

```sql
PREPARE `books_query` FROM 'SELECT * FROM `books` WHERE `id` = ?';
```

実行結果：

```
Query OK, 0 rows affected (0.01 sec)
```

```sql
SET @id = 1;
```

実行結果：

```
Query OK, 0 rows affected (0.04 sec)
```

```sql
EXECUTE `books_query` USING @id;
```

実行結果：

```
+---------+---------------------------------+--------+---------------------+-------+--------+
| id      | title                           | type   | published_at        | stock | price  |
+---------+---------------------------------+--------+---------------------+-------+--------+
| 1       | The Adventures of Pierce Wehner | Comics | 1904-06-06 20:46:25 |   586 | 411.66 |
+---------+---------------------------------+--------+---------------------+-------+--------+
1 row in set (0.05 sec)
```

</div>

<div label="Java" value="java">

```java
// dsはcom.mysql.cj.jdbc.MysqlDataSourceのエンティティです
try (Connection connection = ds.getConnection()) {
    PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM `books` WHERE `id` = ?");
    preparedStatement.setLong(1, 1);

    ResultSet res = preparedStatement.executeQuery();
    if(!res.next()) {
        System.out.println("idが1のテーブルに本がありません");
    } else {
        // idが1の本の情報が取得されます
        System.out.println(res.getLong("id"));
        System.out.println(res.getString("title"));
        System.out.println(res.getString("type"));
    }
} catch (SQLException e) {
    e.printStackTrace();
}
```

</div>

</SimpleTab>

### `INSERT`の例

[`books`テーブル](/develop/dev-guide-bookshop-schema-design.md#books-table)を例にすると、`title = TiDB Developer Guide`、`type = Science & Technology`、`stock = 100`、`price = 0.0`、および`published_at = NOW()`（挿入時の現在時刻）の本を挿入する必要があります。`books`テーブルの**主キー**の`AUTO_RANDOM`属性を明示的に指定する必要はないことに注意してください。データを挿入する詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

<SimpleTab groupId="language">

<div label="SQL" value="sql">

```sql
PREPARE `books_insert` FROM 'INSERT INTO `books` (`title`, `type`, `stock`, `price`, `published_at`) VALUES (?, ?, ?, ?, ?);';
```

実行結果：

```
Query OK, 0 rows affected (0.03 sec)
```

```sql
SET @title = 'TiDB Developer Guide';
SET @type = 'Science & Technology';
SET @stock = 100;
SET @price = 0.0;
SET @published_at = NOW();
```

実行結果：

```
Query OK, 0 rows affected (0.04 sec)
```

```sql
EXECUTE `books_insert` USING @title, @type, @stock, @price, @published_at;
```

実行結果：

```
Query OK, 1 row affected (0.03 sec)
```

</div>

<div label="Java" value="java">

```java
try (Connection connection = ds.getConnection()) {
    String sql = "INSERT INTO `books` (`title`, `type`, `stock`, `price`, `published_at`) VALUES (?, ?, ?, ?, ?);";
    PreparedStatement preparedStatement = connection.prepareStatement(sql);

    preparedStatement.setString(1, "TiDB Developer Guide");
    preparedStatement.setString(2, "Science & Technology");
    preparedStatement.setInt(3, 100);
    preparedStatement.setBigDecimal(4, new BigDecimal("0.0"));
    preparedStatement.setTimestamp(5, new Timestamp(Calendar.getInstance().getTimeInMillis()));

    preparedStatement.executeUpdate();
} catch (SQLException e) {
    e.printStackTrace();
}
```

JDBCを使用すると、プリペアドステートメントのライフサイクルを制御するのに役立ち、アプリケーションで手動でプリペアドステートメントを作成、使用、または削除する必要はありません。ただし、TiDBはMySQLと互換性があるため、クライアント側でのデフォルトのMySQL JDBCドライバーの使用構成は**_サーバーサイド_**のプリペアドステートメントオプションを有効にしないものです。クライアント側でプリペアドステートメントを使用するための推奨構成は次のとおりです。

|            パラメーター            |                 意味                  |   推奨シナリオ   | 推奨構成|
| :------------------------: | :-----------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------: | :----------------------: |
|    `useServerPrepStmts`    |    サーバーサイドでプリペアドステートメントを有効にするかどうか    |  プリペアドステートメントを複数回使用する必要がある場合                                                           |          `true`          |
|      `cachePrepStmts`      |       クライアントがプリペアドステートメントをキャッシュするかどうか        |                                                           `useServerPrepStmts=true` とき                                                            |          `true`          |
|  `prepStmtCacheSqlLimit`   |  プリペアドステートメントの最大サイズ（デフォルトでは256文字）  | プリペアドステートメントが256文字を超える場合 | 実際のプリペアドステートメントのサイズに応じて構成されます |
|    `prepStmtCacheSize`     | プリペアドステートメントの最大数（デフォルトでは25個） | プリペアドステートメントの数が25を超える場合  | 実際のプリペアドステートメントの数に応じて構成されます |

次はJDBC接続文字列構成の典型的なシナリオです。ホスト：`127.0.0.1`、ポート：`4000`、ユーザ名：`root`、パスワード：なし、デフォルトデータベース：`test`：

```
jdbc:mysql://127.0.0.1:4000/test?user=root&useConfigs=maxPerformance&useServerPrepStmts=true&prepStmtCacheSqlLimit=2048&prepStmtCacheSize=256&rewriteBatchedStatements=true&allowMultiQueries=true
```

```
      </div>
      
      </SimpleTab>
```
```
      </div>
      
      </SimpleTab>
```