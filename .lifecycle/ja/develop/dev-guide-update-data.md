---
title: データの更新
summary: データの更新と一括更新方法について学びます。
---

# データの更新

このドキュメントでは、様々なプログラミング言語を使用してTiDB内のデータを更新するための以下のSQLステートメントの使用方法について説明します。

- [UPDATE](/sql-statements/sql-statement-update.md): 指定したテーブル内のデータを変更するために使用されます。
- [INSERT ON DUPLICATE KEY UPDATE](/sql-statements/sql-statement-insert.md): データの挿入と、主キーまたは一意キーの競合がある場合にこのデータを更新するために使用されます。複数の一意キー（主キーを含む）がある場合は、このステートメントを使用することは**お勧めできません**。なぜなら、このステートメントは一意キー（主キーを含む）の競合を検出するとデータを更新し、複数の行の競合がある場合は1行だけが更新されます。

## 開始する前に

このドキュメントを読む前に、以下を準備する必要があります。

- [TiDB Serverless Clusterの構築](/develop/dev-guide-build-cluster-in-cloud.md)。
- [スキーマ設計の概要](/develop/dev-guide-schema-design-overview.md)、[データベースの作成](/develop/dev-guide-create-database.md)、[テーブルの作成](/develop/dev-guide-create-table.md)、[セカンダリインデックスの作成](/develop/dev-guide-create-secondary-indexes.md)を読みます。
- データを`UPDATE`する場合は、まず[データの挿入](/develop/dev-guide-insert-data.md)が必要です。

## `UPDATE`の使用

テーブル内の既存の行を更新するには、[`UPDATE`ステートメント](/sql-statements/sql-statement-update.md)を使用し、更新する列をフィルタリングするための`WHERE`句を使用する必要があります。

> **注記:**
>
> 大量の行を更新する必要がある場合、例えば1万行を超える場合は、一括更新を一度に行うのではなく、すべての行が更新されるまで反復的に一部を更新することをお勧めします。この操作を繰り返すためのスクリプトやプログラムを記述できます。
> 詳細は[一括更新](#bulk-update)を参照してください。

### `UPDATE`のSQL構文

SQLでは、`UPDATE`ステートメントは一般的に次の形式で記述します：

```sql
UPDATE {table} SET {update_column} = {update_value} WHERE {filter_column} = {filter_value}
```

| パラメータ名 | 説明 |
| :---------------: | :------------------: |
|     `{table}`     |         テーブル名         |
| `{update_column}` |     更新する列の名前     |
| `{update_value}`  |     更新する列の値     |
| `{filter_column}` |     フィルタ条件となる列の名前     |
| `{filter_value}`  |     フィルタ条件となる列の値     |

詳細な情報については、[UPDATEの構文](/sql-statements/sql-statement-update.md)を参照してください。

### `UPDATE`のベストプラクティス

データを更新する際のいくつかのベストプラクティスは次のとおりです：

- `UPDATE`ステートメントには常に`WHERE`句を指定します。`UPDATE`ステートメントに`WHERE`句がない場合、TiDBはテーブル内の**_全ての行_**を更新します。

<CustomContent platform="tidb">

- 行数が多い場合は（例: 1万を超える）、[一括更新](#bulk-update)を使用してください。なぜなら、TiDBは単一トランザクションのサイズを制限しています（[txn-total-size-limit](/tidb-configuration-file.md#txn-total-size-limit)、デフォルトでは100 MB）。一度にデータを大量に更新すると、長時間ロックを保持する（[悲観的トランザクション](/pessimistic-transaction.md)）か、競合を引き起こす（[楽観的トランザクション](/optimistic-transaction.md)）可能性があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- 行数が多い場合は（例: 1万を超える）、[一括更新](#bulk-update)を使用してください。なぜなら、TiDBはデフォルトで単一トランザクションのサイズを100 MBに制限しており、一度にデータを大量に更新すると、長時間ロックを保持する（[悲観的トランザクション](/pessimistic-transaction.md)）か、競合を引き起こす（[楽観的トランザクション](/optimistic-transaction.md)）可能性があります。

</CustomContent>

### `UPDATE`の例

著者が名前を **Helen Haruki** に変更するとします。[authors](/develop/dev-guide-bookshop-schema-design.md#authors-table) テーブルを更新する必要があります。彼女のユニークな `id` が **1** であると仮定し、フィルター条件は `id = 1` とします。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

```sql
UPDATE `authors` SET `name` = "Helen Haruki" WHERE `id` = 1;
```

</div>

<div label="Java" value="java">

```java
// dsは com.mysql.cj.jdbc.MysqlDataSource のエンティティです
try (Connection connection = ds.getConnection()) {
    PreparedStatement pstmt = connection.prepareStatement("UPDATE `authors` SET `name` = ? WHERE `id` = ?");
    pstmt.setString(1, "Helen Haruki");
    pstmt.setInt(2, 1);
    pstmt.executeUpdate();
} catch (SQLException e) {
    e.printStackTrace();
}
```

</div>
</SimpleTab>

## `INSERT ON DUPLICATE KEY UPDATE`の使用

新しいデータをテーブルに挿入する必要があるが、一意キー（主キーも一意キーです）の競合がある場合、最初の競合するレコードが更新されます。`INSERT ... ON DUPLICATE KEY UPDATE ...`ステートメントを使用して挿入または更新が行えます。

### `INSERT ON DUPLICATE KEY UPDATE`のSQL構文

SQLでは、`INSERT ... ON DUPLICATE KEY UPDATE ...`ステートメントは一般的に次の形式で記述します：

```sql
INSERT INTO {table} ({columns}) VALUES ({values})
    ON DUPLICATE KEY UPDATE {update_column} = {update_value};
```

| パラメータ名 | 説明 |
| :---------------: | :--------------: |
|     `{table}`     |         テーブル名         |
|    `{columns}`    |     挿入する列の名前     |
|    `{values}`     |     挿入する列の値     |
| `{update_column}` |     更新する列の名前     |
| `{update_value}`  |     更新する列の値     |

### `INSERT ON DUPLICATE KEY UPDATE`のベストプラクティス

- 1つの一意キーを持つテーブルにのみ`INSERT ON DUPLICATE KEY UPDATE`を使用してください。このステートメントは、競合がある場合にデータを更新します（主キーを含む**_一意キー_**）。複数の行の競合がある場合、1行だけが更新されます。したがって、複数の一意キーを持つテーブルでは、1行の競合が保証されている場合を除いて`INSERT ON DUPLICATE KEY UPDATE`ステートメントの使用はお勧めできません。
- このステートメントはデータの作成または更新時に使用してください。

### `INSERT ON DUPLICATE KEY UPDATE`の例

例として、[ratings](/develop/dev-guide-bookshop-schema-design.md#ratings-table) テーブルを更新して、ユーザーが本に評価をしている場合、新しい評価が作成されます。ユーザーが既に評価をしている場合は、前の評価が更新されます。

次の例では、主キーは `book_id` と `user_id` の複合主キーです。ユーザー `user_id = 1` が本 `book_id = 1000` に対して `5` の評価を行っていると仮定します。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

```sql
INSERT INTO `ratings`
    (`book_id`, `user_id`, `score`, `rated_at`)
VALUES
    (1000, 1, 5, NOW())
ON DUPLICATE KEY UPDATE `score` = 5, `rated_at` = NOW();
```

</div>

<div label="Java" value="java">

```java
// dsは com.mysql.cj.jdbc.MysqlDataSource のエンティティです
try (Connection connection = ds.getConnection()) {
    PreparedStatement p = connection.prepareStatement("INSERT INTO `ratings` (`book_id`, `user_id`, `score`, `rated_at`)
VALUES (?, ?, ?, NOW()) ON DUPLICATE KEY UPDATE `score` = ?, `rated_at` = NOW()");
    p.setInt(1, 1000);
    p.setInt(2, 1);
    p.setInt(3, 5);
    p.setInt(4, 5);
    p.executeUpdate();
} catch (SQLException e) {
    e.printStackTrace();
}
```

</div>
</SimpleTab>

## 一括更新

テーブル内の複数の行のデータを更新する必要がある場合は、[ `INSERT ON DUPLICATE KEY UPDATE`を使用](#use-insert-on-duplicate-key-update)し、`WHERE`句を使用して更新する必要のあるデータをフィルタリングすることができます。

<CustomContent platform="tidb">

ただし、大量の行を更新する必要がある場合（例: 1万行を超える場合）は、データを反復的に更新することをお勧めします。つまり、更新が完了するまでデータの一部のみを反復して更新します。これはTiDBが単一トランザクションのサイズを制限しているためです（[txn-total-size-limit](/tidb-configuration-file.md#txn-total-size-limit)、デフォルトでは100 MB）。一度に大量のデータを更新すると、長時間ロックを保持する（[悲観的トランザクション](/pessimistic-transaction.md)）か、競合を引き起こす（[楽観的トランザクション](/optimistic-transaction.md)）可能性があります。プログラムやスクリプトでループを使用してこの操作を完了させることができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

ただし、多数の行（たとえば1万を超える行）を更新する必要がある場合、データを反復的に更新することをお勧めします。つまり、更新が完了するまでの各反復でデータの一部だけを更新します。これは、TiDBがデフォルトで単一トランザクションのサイズを100 MBに制限しているためです。一度に多くのデータを更新すると、長時間ロックが保持される場合（[悲観的トランザクション](/pessimistic-transaction.md)）、または競合が発生する可能性があります（[楽観的トランザクション](/optimistic-transaction.md)）。プログラムやスクリプトでループを使用して操作を完了することができます。

</CustomContent>

このセクションでは、反復的な更新を処理するスクリプトの例を示します。この例では、`SELECT`と`UPDATE`の組み合わせを使用して大規模な更新を完了する方法を示します。

### 大規模な更新ループを書く

まず、アプリケーションやスクリプトのループ内で`SELECT`クエリを書く必要があります。このクエリの戻り値を使用して更新する必要のある行の主キーを取得できます。この`SELECT`クエリを定義する際には、更新する必要のある行をフィルタリングするために`WHERE`句を使用する必要があります。

### 例

過去1年間にユーザーから多くの書籍の評価を受け取っているが、5点の尺度の元の設計により、書籍の評価がほとんど`3`となってしまった。評価の差別化が足りなくなっているため、5点から10点の尺度に変更することに決めました。

前の5点尺度からのデータを2倍し、評価が更新された行を示す新しいカラムを評価テーブルに追加する必要があります。このカラムを使用して`SELECT`で更新済みの行をフィルタリングすることができ、これによりスクリプトがクラッシュし、同じ行を複数回更新することを防ぎ、不合理なデータを発生させることができます。

たとえば、`ten_point`という名前の[BOOL](/data-type-numeric.md#boolean-type)データ型のカラムを作成し、このカラムを使用して10点尺度であるかどうかを示します。

```sql
ALTER TABLE `bookshop`.`ratings` ADD COLUMN `ten_point` BOOL NOT NULL DEFAULT FALSE;
```

> **注:**
>
> この大規模な更新アプリケーションでは、**DDL**ステートメントを使用してデータテーブルのスキーマを変更しています。TiDBのすべてのDDL変更操作はオンラインで実行されます。詳細については、[ADD COLUMN](/sql-statements/sql-statement-add-column.md)を参照してください。

<SimpleTab groupId="language">
<div label="Golang" value="golang">

Golangでは、大規模な更新アプリケーションは次のようになります。

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/go-sql-driver/mysql"
    "strings"
    "time"
)

func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:4000)/bookshop")
    if err != nil {
        panic(err)
    }
    defer db.Close()

    bookID, userID := updateBatch(db, true, 0, 0)
    fmt.Println("最初のバッチ更新成功")
    for {
        time.Sleep(time.Second)
        bookID, userID = updateBatch(db, false, bookID, userID)
        fmt.Printf("バッチ更新成功、[bookID] %d、[userID] %d\n", bookID, userID)
    }
}

// updateBatch は最大1000行のデータを選択し、スコアを更新します
func updateBatch(db *sql.DB, firstTime bool, lastBookID, lastUserID int64) (bookID, userID int64) {
    // 5点尺度のデータで最大1000個の主キーを選択する
    var err error
    var rows *sql.Rows

    if firstTime {
        rows, err = db.Query("SELECT `book_id`, `user_id` FROM `bookshop`.`ratings` " +
            "WHERE `ten_point` != true ORDER BY `book_id`, `user_id` LIMIT 1000")
    } else {
        rows, err = db.Query("SELECT `book_id`, `user_id` FROM `bookshop`.`ratings` "+
            "WHERE `ten_point` != true AND `book_id` > ? AND `user_id` > ? "+
            "ORDER BY `book_id`, `user_id` LIMIT 1000", lastBookID, lastUserID)
    }

    if err != nil || rows == nil {
        panic(fmt.Errorf("エラーが発生したか、rowsがnilです: %+v", err))
    }

    // すべてのIDをリストで結合
    var idList []interface{}
    for rows.Next() {
        var tempBookID, tempUserID int64
        if err := rows.Scan(&tempBookID, &tempUserID); err != nil {
            panic(err)
        }
        idList = append(idList, tempBookID, tempUserID)
        bookID, userID = tempBookID, tempUserID
    }

    bulkUpdateSql := fmt.Sprintf("UPDATE `bookshop`.`ratings` SET `ten_point` = true, "+
        "`score` = `score` * 2 WHERE (`book_id`, `user_id`) IN (%s)", placeHolder(len(idList)))
    db.Exec(bulkUpdateSql, idList...)

    return bookID, userID
}

// placeHolder はSQLプレースホルダーのフォーマットを行います
func placeHolder(n int) string {
    holderList := make([]string, n/2, n/2)
    for i := range holderList {
        holderList[i] = "(?,?)"
    }
    return strings.Join(holderList, ",")
}
```

各反復では、主キーの順に`SELECT`クエリを実行します。また、まだ10点尺度（`ten_point`が`false`）に更新されていない最大で`1000`行の主キー値を選択します。各`SELECT`ステートメントは、前の`SELECT`結果より大きい主キーを選択するために使用されます。その後、スコアの列を`2`倍にし、`ten_point`を`true`に設定することで、大量更新を行っています。`ten_point`の更新の目的は、クラッシュ後の再起動時に同じ行を繰り返し更新することを防ぎ、データの破損を引き起こすことを避けるためです。ループ内の`time.Sleep(time.Second)`では、更新アプリケーションがハードウェアリソースを過剰に消費することを防ぐために、更新アプリケーションを1秒間一時停止させています。

</div>

<div label="Java (JDBC)" value="jdbc">

Java（JDBC）では、大規模な更新アプリケーションは次のようになるかもしれません。

**コード:**

```java
package com.pingcap.bulkUpdate;

import com.mysql.cj.jdbc.MysqlDataSource;

import java.sql.*;
import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.TimeUnit;

public class BatchUpdateExample {
    static class UpdateID {
        private Long bookID;
        private Long userID;

        public UpdateID(Long bookID, Long userID) {
            this.bookID = bookID;
            this.userID = userID;
        }

        public Long getBookID() {
            return bookID;
        }

        public void setBookID(Long bookID) {
            this.bookID = bookID;
        }

        public Long getUserID() {
            return userID;
        }

        public void setUserID(Long userID) {
            this.userID = userID;
        }

        @Override
        public String toString() {
            return "[bookID] " + bookID + ", [userID] " + userID ;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        // Configure the example database connection.

        // Create a mysql data source instance.
        MysqlDataSource mysqlDataSource = new MysqlDataSource();

        // Set server name, port, database name, username and password.
        mysqlDataSource.setServerName("localhost");
        mysqlDataSource.setPortNumber(4000);
        mysqlDataSource.setDatabaseName("bookshop");
        mysqlDataSource.setUser("root");
        mysqlDataSource.setPassword("");

        UpdateID lastID = batchUpdate(mysqlDataSource, null);

        System.out.println("初回バッチ更新成功");
        while (true) {
            TimeUnit.SECONDS.sleep(1);
            lastID = batchUpdate(mysqlDataSource, lastID);
            System.out.println("バッチ更新成功、[lastID] " + lastID);
        }
    }

    public static UpdateID batchUpdate (MysqlDataSource ds, UpdateID lastID) {
        try (Connection connection = ds.getConnection()) {
            UpdateID updateID = null;

            PreparedStatement selectPs;

            if (lastID == null) {
                selectPs = connection.prepareStatement(
                        "SELECT `book_id`, `user_id` FROM `bookshop`.`ratings` " +
                        "WHERE `ten_point` != true ORDER BY `book_id`, `user_id` LIMIT 1000");
            } else {
                selectPs = connection.prepareStatement(
                        "SELECT `book_id`, `user_id` FROM `bookshop`.`ratings` "+
                            "WHERE `ten_point` != true AND `book_id` > ? AND `user_id` > ? "+
                            "ORDER BY `book_id`, `user_id` LIMIT 1000");

                selectPs.setLong(1, lastID.getBookID());
                selectPs.setLong(2, lastID.getUserID());
            }

            List<Long> idList = new LinkedList<>();
            ResultSet res = selectPs.executeQuery();
            while (res.next()) {
                updateID = new UpdateID(
                        res.getLong("book_id"),
                        res.getLong("user_id")
                );
                idList.add(updateID.getBookID());
```java
                idList.add(updateID.getUserID());
            }

            if (idList.isEmpty()) {
                System.out.println("no data should update");
                return null;
            }

            String updateSQL = "UPDATE `bookshop`.`ratings` SET `ten_point` = true, "+
                    "`score` = `score` * 2 WHERE (`book_id`, `user_id`) IN (" +
                    placeHolder(idList.size() / 2) + ")";
            PreparedStatement updatePs = connection.prepareStatement(updateSQL);
            for (int i = 0; i < idList.size(); i++) {
                updatePs.setLong(i + 1, idList.get(i));
            }
            int count = updatePs.executeUpdate();
            System.out.println("update " + count + " data");

            return updateID;
        } catch (SQLException e) {
            e.printStackTrace();
        }

        return null;
    }

    public static String placeHolder(int n) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < n ; i++) {
            sb.append(i == 0 ? "(?,?)" : ",(?,?)");
        }

        return sb.toString();
    }
}
```

- `hibernate.cfg.xml` configuration:

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>

        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.dialect">org.hibernate.dialect.TiDBDialect</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:4000/movie</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password"></property>
        <property name="hibernate.connection.autocommit">false</property>
        <property name="hibernate.jdbc.batch_size">20</property>

        <!-- Optional: Show SQL output for debugging -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
    </session-factory>
</hibernate-configuration>
```

In each iteration, `SELECT` queries in order of the primary key. It selects primary key values for up to `1000` rows that have not been updated to the 10-point scale (`ten_point` is `false`). Each `SELECT` statement selects primary keys larger than the largest of the previous `SELECT` results to prevent duplication. Then, it uses bulk-update, multiples its `score` column by `2`, and sets `ten_point` to `true`. The purpose of updating `ten_point` is to prevent the update application from repeatedly updating the same row in case of restart after crashing, which can cause data corruption. `TimeUnit.SECONDS.sleep(1);` in each loop makes the update application pause for 1 second to prevent the update application from consuming too many hardware resources.

</div>

</SimpleTab>
```