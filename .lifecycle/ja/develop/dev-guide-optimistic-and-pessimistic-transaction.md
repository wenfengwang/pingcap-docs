---
title: 楽観的トランザクションと悲観的トランザクション
summary: TiDB の楽観的トランザクションと悲観的トランザクションについて学びます。
---

# 楽観的トランザクションと悲観的トランザクション

[楽観的トランザクション](/optimistic-transaction.md)モデルはトランザクションを直接コミットし、競合が発生した場合にロールバックします。一方、[悲観的トランザクション](/pessimistic-transaction.md) モデルは、実際にトランザクションをコミットする前に変更する必要のあるリソースをロックしようとし、トランザクションが成功裏に実行できることを確認した後にのみコミットを開始します。

楽観的トランザクションモデルは競合率が低いシナリオに適しており、直接コミットするため成功する可能性が高いです。しかし、トランザクションの競合が発生すると、ロールバックのコストが比較的高くなります。

悲観的トランザクションモデルの利点は、競合率が高いシナリオでは先にロックをするコストが後でのロールバックよりも少ないことです。さらに、複数の同時トランザクションが競合のためにコミットに失敗する問題を解決できます。ただし、楽観的トランザクションモデルほど競合率が低いシナリオでは効率的ではありません。

悲観的トランザクションモデルはアプリケーション側で直感的で実装が容易です。楽観的トランザクションモデルは複雑なアプリケーション側のリトライメカニズムを必要とします。

以下は[書店](/develop/dev-guide-bookshop-schema-design.md)の例です。本を購入する例を使用し、楽観的トランザクションと悲観的トランザクションの利点と欠点を示します。本の購入プロセスは主に以下の手順から構成されます。

1. 在庫数を更新する
2. 注文を作成する
3. お支払いを行う

これらの操作はすべて成功するか、すべて失敗する必要があります。同時トランザクションが発生した場合には、在庫の過剰販売が発生しないようにする必要があります。

## 悲観的トランザクション

以下のコードは、悲観的トランザクションモードで2つのスレッドを使用して、2人のユーザーが同じ本を購入するプロセスをシミュレートします。書店には10冊の本があり、Bob は6冊購入し、Alice は4冊購入します。彼らはほぼ同時に注文を完了します。その結果、在庫のすべての本が売り切れます。

<SimpleTab groupId="language">

<div label="Java" value="java">

複数のスレッドを使用して複数のユーザーが同時にデータを挿入する状況をシミュレートするため、安全なスレッドを持つ接続オブジェクトを使用する必要があります。ここでは Java の人気のある接続プール [HikariCP](https://github.com/brettwooldridge/HikariCP) をデモに使用します。

</div>

<div label="Golang" value="golang">

Golang の `sql.DB` は並行処理に安全ですので、サードパーティのパッケージをインポートする必要はありません。

TiDB トランザクションに適応するために、以下のコードに従ってツールキット [util](https://github.com/pingcap-inc/tidb-example-golang/tree/main/util) を記述します。

```go
package util

import (
    "context"
    "database/sql"
)

type TiDBSqlTx struct {
    *sql.Tx
    conn        *sql.Conn
    pessimistic bool
}

func TiDBSqlBegin(db *sql.DB, pessimistic bool) (*TiDBSqlTx, error) {
    ctx := context.Background()
    conn, err := db.Conn(ctx)
    if err != nil {
        return nil, err
    }
    if pessimistic {
        _, err = conn.ExecContext(ctx, "set @@tidb_txn_mode=?", "pessimistic")
    } else {
        _, err = conn.ExecContext(ctx, "set @@tidb_txn_mode=?", "optimistic")
    }
    if err != nil {
        return nil, err
    }
    tx, err := conn.BeginTx(ctx, nil)
    if err != nil {
        return nil, err
    }
    return &TiDBSqlTx{
        conn:        conn,
        Tx:          tx,
        pessimistic: pessimistic,
    }, nil
}

func (tx *TiDBSqlTx) Commit() error {
    defer tx.conn.Close()
    return tx.Tx.Commit()
}

func (tx *TiDBSqlTx) Rollback() error {
    defer tx.conn.Close()
    return tx.Tx.Rollback()
}
```

</div>

<div label="Python" value="python">

スレッドセーフを確保するため、mysqlclient ドライバを使用してスレッド間で共有されない複数の接続を開くことができます。

</div>

</SimpleTab>

### 悲観的トランザクションの例を記述する

<SimpleTab groupId="language">

<div label="Java" value="java">

**設定ファイル**

パッケージを管理する Maven を使用する場合は、`pom.xml` の `<dependencies>` ノードに以下の依存関係を追加して `HikariCP` をインポートし、パッケージングの対象となる JAR パッケージの開始のメインクラスを設定します。以下は `pom.xml` の例です。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.pingcap</groupId>
  <artifactId>plain-java-txn</artifactId>
  <version>0.0.1</version>

  <name>plain-java-jdbc</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.28</version>
    </dependency>

    <dependency>
      <groupId>com.zaxxer</groupId>
      <artifactId>HikariCP</artifactId>
      <version>5.0.1</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
        <configuration>
          <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
          </descriptorRefs>
          <archive>
            <manifest>
              <mainClass>com.pingcap.txn.TxnExample</mainClass>
            </manifest>
          </archive>

        </configuration>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
              <goal>single</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>

</project>
```

**コーディング**

次に、以下のコードを記述します。

```java
package com.pingcap.txn;

import com.zaxxer.hikari.HikariDataSource;

import java.math.BigDecimal;
import java.sql.*;
import java.util.Arrays;
import java.util.concurrent.*;

public class TxnExample {
    public static void main(String[] args) throws SQLException, InterruptedException {
        System.out.println(Arrays.toString(args));
        int aliceQuantity = 0;
        int bobQuantity = 0;

        for (String arg: args) {
            if (arg.startsWith("ALICE_NUM")) {
                aliceQuantity = Integer.parseInt(arg.replace("ALICE_NUM=", ""));
            }

            if (arg.startsWith("BOB_NUM")) {
                bobQuantity = Integer.parseInt(arg.replace("BOB_NUM=", ""));
            }
        }

        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:4000/bookshop?useServerPrepStmts=true&cachePrepStmts=true");
        ds.setUsername("root");
        ds.setPassword("");

        // prepare data
        Connection connection = ds.getConnection();
        createBook(connection, 1L, "Designing Data-Intensive Application", "Science & Technology",
                Timestamp.valueOf("2018-09-01 00:00:00"), new BigDecimal(100), 10);
        createUser(connection, 1L, "Bob", new BigDecimal(10000));
        createUser(connection, 2L, "Alice", new BigDecimal(10000));

        CountDownLatch countDownLatch = new CountDownLatch(2);
        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        final int finalBobQuantity = bobQuantity;
        threadPool.execute(() -> {
            buy(ds, 1, 1000L, 1L, 1L, finalBobQuantity);
            countDownLatch.countDown();
        });
        final int finalAliceQuantity = aliceQuantity;
        threadPool.execute(() -> {
            buy(ds, 2, 1001L, 1L, 2L, finalAliceQuantity);
            countDownLatch.countDown();
        });

        countDownLatch.await(5, TimeUnit.SECONDS);
    }

    public static void createUser(Connection connection, Long id, String nickname, BigDecimal balance) throws SQLException  {
        PreparedStatement insert = connection.prepareStatement(
```go
package main

import (
    "context"
    "database/sql"
    "fmt"
    "time"

    "github.com/go-sql-driver/mysql"
    "github.com/pingcap-inc/tidb-example-golang/util"
    "github.com/shopspring/decimal"
)

type TxnFunc func(txn *util.TiDBSqlTx) error

const (
    ErrWriteConflict      = 9007 // Transactions in TiKV encounter write conflicts.
    ErrInfoSchemaChanged  = 8028 // table schema changes
    ErrForUpdateCantRetry = 8002 // "SELECT FOR UPDATE" commit conflict
    ErrTxnRetryable       = 8022 // The transaction commit fails and has been rolled back
)

const retryTimes = 5

var retryErrorCodeSet = map[uint16]interface{}{
    ErrWriteConflict:      nil,
    ErrInfoSchemaChanged:  nil,
    ErrForUpdateCantRetry: nil,
    ErrTxnRetryable:       nil,
}

func runTxn(db *sql.DB, optimistic bool, optimisticRetryTimes int, txnFunc TxnFunc) {
    txn, err := util.TiDBSqlBegin(db, !optimistic)
    if err != nil {
        panic(err)
    }

    err = txnFunc(txn)
    if err != nil {
        txn.Rollback()
        if mysqlErr, ok := err.(*mysql.MySQLError); ok && optimistic && optimisticRetryTimes != 0 {
            if _, retryableError := retryErrorCodeSet[mysqlErr.Number]; retryableError {
                fmt.Printf("[runTxn] リトライ可能なエラーが発生しました。残りの回数: %d\n", optimisticRetryTimes-1)
                runTxn(db, optimistic, optimisticRetryTimes-1, txnFunc)
                return
            }
        }

        fmt.Printf("[runTxn] エラーが発生しました。ロールバックします: %+v\n", err)
    } else {
        err = txn.Commit()
        if mysqlErr, ok := err.(*mysql.MySQLError); ok && optimistic && optimisticRetryTimes != 0 {
            if _, retryableError := retryErrorCodeSet[mysqlErr.Number]; retryableError {
                fmt.Printf("[runTxn] リトライ可能なエラーが発生しました。残りの回数: %d\n", optimisticRetryTimes-1)
                runTxn(db, optimistic, optimisticRetryTimes-1, txnFunc)
                return
            }
        }

        if err == nil {
            fmt.Println("[runTxn] コミットに成功しました")
        }
    }
}

func prepareData(db *sql.DB, optimistic bool) {
    runTxn(db, optimistic, retryTimes, func(txn *util.TiDBSqlTx) error {
        publishedAt, err := time.Parse("2006-01-02 15:04:05", "2018-09-01 00:00:00")
        if err != nil {
            return err
        }

        if err = createBook(txn, 1, "デザインデータ集約アプリケーション",
            "サイエンス＆テクノロジー", publishedAt, decimal.NewFromInt(100), 10); err != nil {
            return err
        }

        if err = createUser(txn, 1, "ボブ", decimal.NewFromInt(10000)); err != nil {
            return err
        }

        if err = createUser(txn, 2, "アリス", decimal.NewFromInt(10000)); err != nil {
            return err
        }

        return nil
    })
}

func buyPessimistic(db *sql.DB, goroutineID, orderID, bookID, userID, amount int) {
    txnComment := fmt.Sprintf("/* txn %d */ ", goroutineID)
    if goroutineID != 1 {
        txnComment = "\t" + txnComment
    }

    fmt.Printf("\nユーザー %d が %d 冊(id: %d) の本を購入しようとしています\n", userID, amount, bookID)

    runTxn(db, false, retryTimes, func(txn *util.TiDBSqlTx) error {
        time.Sleep(time.Second)

        // 本の価格を読む
        selectBookForUpdate := "select `price` from books where id = ? for update"
        bookRows, err := txn.Query(selectBookForUpdate, bookID)
        if err != nil {
            return err
        }
        fmt.Println(txnComment + selectBookForUpdate + " 成功")
        defer bookRows.Close()

        price := decimal.NewFromInt(0)
        if bookRows.Next() {
            err = bookRows.Scan(&price)
            if err != nil {
                return err
            }
        } else {
            return fmt.Errorf("本のIDが存在しません")
        }
        bookRows.Close()

        // 本の在庫を更新
        updateStock := "update `books` set stock = stock - ? where id = ? and stock - ? >= 0"
        result, err := txn.Exec(updateStock, amount, bookID, amount)
        if err != nil {
            return err
        }
        fmt.Println(txnComment + updateStock + " 成功")

        affected, err := result.RowsAffected()
        if err != nil {
            return err
        }

        if affected == 0 {
            return fmt.Errorf("在庫が足りません。ロールバックします")
        }

        // 注文を挿入
        insertOrder := "insert into `orders` (`id`, `book_id`, `user_id`, `quality`) values (?, ?, ?, ?)"
        if _, err := txn.Exec(insertOrder,
            orderID, bookID, userID, amount); err != nil {
            return err
        }
        fmt.Println(txnComment + insertOrder + " 成功")

        // ユーザーを更新
        updateUser := "update `users` set `balance` = `balance` - ? where id = ?"
        if _, err := txn.Exec(updateUser,
            price.Mul(decimal.NewFromInt(int64(amount))), userID); err != nil {
            return err
        }
        fmt.Println(txnComment + updateUser + " 成功")
```
```go
        関数createBook(txn *util.TiDBSqlTx、id int、title、bookType string、
            publishedAt time.Time、price decimal.Decimal、stock int) エラー{
    _, err := txn.ExecContext(context.Background(),
        "INSERT INTO `books` (`id`, `title`, `type`, `published_at`, `price`, `stock`) values (?, ?, ?, ?, ?, ?)",
        id, title, bookType, publishedAt, price, stock)
    return err
}

関数createUser(txn *util.TiDBSqlTx、id int、nickname string、balance decimal.Decimal) エラー{
    _, err := txn.ExecContext(context.Background(),
        "INSERT INTO `users` (`id`, `nickname`, `balance`) VALUES (?, ?, ?)",
        id, nickname, balance)
    return err
}
```
```python
                    if code in REPEATABLE_ERROR_CODE_SET and optimistic_retry_times > 0:
                        print(f'retry, rest {optimistic_retry_times - 1} times, for {code} {desc}')
                        buy_optimistic(thread_id, order_id, book_id, user_id, amount, optimistic_retry_times - 1)


def buy_pessimistic(thread_id: int, order_id: int, book_id: int, user_id: int, amount: int) -> None:
    connection = create_connection()

    txn_log_header = f"/* txn {thread_id} */"
    if thread_id != 1:
        txn_log_header = "\t" + txn_log_header

    with connection:
        with connection.cursor() as cursor:
            cursor.execute("BEGIN PESSIMISTIC")
            print(f'{txn_log_header} BEGIN PESSIMISTIC')
            time.sleep(1)

            try:
                # read the price of book
                select_book_for_update = "SELECT `price` FROM books WHERE id = %s FOR UPDATE"
                cursor.execute(select_book_for_update, (book_id,))
                book = cursor.fetchone()
                if book is None:
                    raise Exception("book_id not exist")
                price = book[0]
                print(f'{txn_log_header} {select_book_for_update} successful')

                # update book
                update_stock = "update `books` set stock = stock - %s where id = %s and stock - %s >= 0"
                rows_affected = cursor.execute(update_stock, (amount, book_id, amount))
                print(f'{txn_log_header} {update_stock} successful')

                if rows_affected == 0:
                    raise Exception("stock not enough, rollback")

                # insert order
                insert_order = "insert into `orders` (`id`, `book_id`, `user_id`, `quality`) values (%s, %s, %s, %s)"
                cursor.execute(insert_order, (order_id, book_id, user_id, amount))
                print(f'{txn_log_header} {insert_order} successful')

                # update user
                update_user = "update `users` set `balance` = `balance` - %s where id = %s"
                cursor.execute(update_user, (amount * price, user_id))
                print(f'{txn_log_header} {update_user} successful')

            except Exception as err:
                connection.rollback()
                print(f'something went wrong: {err}')
            else:
                connection.commit()


optimistic = os.environ.get('OPTIMISTIC')
alice = os.environ.get('ALICE')
bob = os.environ.get('BOB')

if not (optimistic and alice and bob):
    raise Exception("please use \"OPTIMISTIC=<is_optimistic> ALICE=<alice_num> "
                    "BOB=<bob_num> python3 txn_example.py\" to start this script")

prepare_data()

if bool(optimistic) is True:
    buy_func = buy_optimistic
else:
    buy_func = buy_pessimistic

bob_thread = Thread(target=buy_func, kwargs={
    "thread_id": 1, "order_id": 1000, "book_id": 1, "user_id": 1, "amount": int(bob)})
alice_thread = Thread(target=buy_func, kwargs={
    "thread_id": 2, "order_id": 1001, "book_id": 1, "user_id": 2, "amount": int(alice)})

bob_thread.start()
alice_thread.start()
bob_thread.join(timeout=10)
alice_thread.join(timeout=10)
```
/* txn 1 */ UPDATE `books` SET `stock` = `stock` - 7 WHERE `id` = 1 AND `stock` - 7 >= 0
/* txn 1 */ ROLLBACK
```

`txn 2`がリソースのロックを先取し、在庫を更新したため、`txn 1`の`affected_rows`の戻り値は0となり、`rollback`処理に入ります。

注文の作成、ユーザーの残高の差し引き、および書籍の在庫の差し引きを確認しましょう。アリスは4冊の本を注文し、ボブは7冊の本を注文に失敗し、残りの6冊の本が期待通り在庫にあります。

```sql
mysql> SELECT * FROM books;
+----+--------------------------------------+----------------------+---------------------+-------+--------+
| id | title                                | type                 | published_at        | stock | price  |
+----+--------------------------------------+----------------------+---------------------+-------+--------+
|  1 | Designing Data-Intensive Application | Science & Technology | 2018-09-01 00:00:00 |     6 | 100.00 |
+----+--------------------------------------+----------------------+---------------------+-------+--------+
1 row in set (0.00 sec)

mysql> SELECT * FROM orders;
+------+---------+---------+---------+---------------------+
| id   | book_id | user_id | quality | ordered_at          |
+------+---------+---------+---------+---------------------+
| 1001 |       1 |       1 |       4 | 2022-04-19 11:03:03 |
+------+---------+---------+---------+---------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM users;
+----+----------+----------+
| id | balance  | nickname |
+----+----------+----------+
|  1 | 10000.00 | Bob      |
|  2 |  9600.00 | Alice    |
+----+----------+----------+
2 rows in set (0.01 sec)
```

## 楽観的トランザクション

以下のコードは、2つのユーザーが楽観的なトランザクションで同じ本を購入するプロセスをシミュレートするために2つのスレッドを使用します。悲観的なトランザクションの例と同様に、在庫には10冊の本が残っています。ボブは6冊、アリスは4冊購入します。彼らはほぼ同時に注文を完了します。最終的に在庫には本が残りません。

### 楽観的トランザクションの例を書く

<SimpleTab groupId="language">

<div label="Java" value="java">

**コーディング**

```java
package com.pingcap.txn.optimistic;

import com.zaxxer.hikari.HikariDataSource;

import java.math.BigDecimal;
import java.sql.*;
import java.util.Arrays;
import java.util.concurrent.*;

public class TxnExample {
    public static void main(String[] args) throws SQLException, InterruptedException {
        System.out.println(Arrays.toString(args));
        int aliceQuantity = 0;
        int bobQuantity = 0;

        for (String arg: args) {
            if (arg.startsWith("ALICE_NUM")) {
                aliceQuantity = Integer.parseInt(arg.replace("ALICE_NUM=", ""));
            }

            if (arg.startsWith("BOB_NUM")) {
                bobQuantity = Integer.parseInt(arg.replace("BOB_NUM=", ""));
            }
        }

        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:4000/bookshop?useServerPrepStmts=true&cachePrepStmts=true");
        ds.setUsername("root");
        ds.setPassword("");

        // データの準備
        Connection connection = ds.getConnection();
        createBook(connection, 1L, "Designing Data-Intensive Application", "Science & Technology",
                Timestamp.valueOf("2018-09-01 00:00:00"), new BigDecimal(100), 10);
        createUser(connection, 1L, "Bob", new BigDecimal(10000));
        createUser(connection, 2L, "Alice", new BigDecimal(10000));

        CountDownLatch countDownLatch = new CountDownLatch(2);
        ExecutorService threadPool = Executors.newFixedThreadPool(2);

        final int finalBobQuantity = bobQuantity;
        threadPool.execute(() -> {
            buy(ds, 1, 1000L, 1L, 1L, finalBobQuantity, 5);
            countDownLatch.countDown();
        });
        final int finalAliceQuantity = aliceQuantity;
        threadPool.execute(() -> {
            buy(ds, 2, 1001L, 1L, 2L, finalAliceQuantity, 5);
            countDownLatch.countDown();
        });

        countDownLatch.await(5, TimeUnit.SECONDS);
    }

    public static void createUser(Connection connection, Long id, String nickname, BigDecimal balance) throws SQLException  {
        PreparedStatement insert = connection.prepareStatement(
                "INSERT INTO `users` (`id`, `nickname`, `balance`) VALUES (?, ?, ?)");
        insert.setLong(1, id);
        insert.setString(2, nickname);
        insert.setBigDecimal(3, balance);
        insert.executeUpdate();
    }

    public static void createBook(Connection connection, Long id, String title, String type, Timestamp publishedAt, BigDecimal price, Integer stock) throws SQLException {
        PreparedStatement insert = connection.prepareStatement(
                "INSERT INTO `books` (`id`, `title`, `type`, `published_at`, `price`, `stock`) values (?, ?, ?, ?, ?, ?)");
        insert.setLong(1, id);
        insert.setString(2, title);
        insert.setString(3, type);
        insert.setTimestamp(4, publishedAt);
        insert.setBigDecimal(5, price);
        insert.setInt(6, stock);

        insert.executeUpdate();
    }

    public static void buy (HikariDataSource ds, Integer threadID, Long orderID, Long bookID,
                            Long userID, Integer quantity, Integer retryTimes) {
        String txnComment = "/* txn " + threadID + " */ ";

        try (Connection connection = ds.getConnection()) {
            try {

                connection.setAutoCommit(false);
                connection.createStatement().executeUpdate(txnComment + "begin optimistic");

                // 他のスレッドが 'begin optimistic' ステートメントを実行するのを待ちます
                TimeUnit.SECONDS.sleep(1);

                BigDecimal price = null;

                // 本の価格を読む
                PreparedStatement selectBook = connection.prepareStatement(txnComment + "SELECT * FROM books where id = ? for update");
                selectBook.setLong(1, bookID);
                ResultSet res = selectBook.executeQuery();
                if (!res.next()) {
                    throw new RuntimeException("book not exist");
                } else {
                    price = res.getBigDecimal("price");
                    int stock = res.getInt("stock");
                    if (stock < quantity) {
                        throw new RuntimeException("book not enough");
                    }
                }

                // 本の更新
                String updateBookSQL = "update `books` set stock = stock - ? where id = ? and stock - ? >= 0";
                PreparedStatement updateBook = connection.prepareStatement(txnComment + updateBookSQL);
                updateBook.setInt(1, quantity);
                updateBook.setLong(2, bookID);
                updateBook.setInt(3, quantity);
                updateBook.executeUpdate();

                // 注文の挿入
                String insertOrderSQL = "insert into `orders` (`id`, `book_id`, `user_id`, `quality`) values (?, ?, ?, ?)";
                PreparedStatement insertOrder = connection.prepareStatement(txnComment + insertOrderSQL);
                insertOrder.setLong(1, orderID);
                insertOrder.setLong(2, bookID);
                insertOrder.setLong(3, userID);
                insertOrder.setInt(4, quantity);
                insertOrder.executeUpdate();

                // ユーザーの更新
                String updateUserSQL = "update `users` set `balance` = `balance` - ? where id = ?";
                PreparedStatement updateUser = connection.prepareStatement(txnComment + updateUserSQL);
                updateUser.setBigDecimal(1, price.multiply(new BigDecimal(quantity)));
                updateUser.setLong(2, userID);
                updateUser.executeUpdate();

                connection.createStatement().executeUpdate(txnComment + "commit");
            } catch (Exception e) {
                connection.createStatement().executeUpdate(txnComment + "rollback");
                System.out.println("error occurred: " + e.getMessage());

                if (e instanceof SQLException sqlException) {
                    switch (sqlException.getErrorCode()) {
                        // すべてのエラーコードはhttps://docs.pingcap.com/tidb/stable/error-codesで入手できます
                        case 9007: // Transactions in TiKV encounter...
                        case 8028: // table schema changes
                        case 8002: // "SELECT FOR UPDATE" commit conflict
                        case 8022: // The transaction commit fails and has been rolled back
                            if (retryTimes != 0) {
                                System.out.println("rest " + retryTimes + " times. retry for " + e.getMessage());
                                buy(ds, threadID, orderID, bookID, userID, quantity, retryTimes - 1);
                            }
                    }
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**構成の変更**

`pom.xml`での起動クラスを変更します：

```xml
<mainClass>com.pingcap.txn.TxnExample</mainClass>
```

楽観的トランザクションの例を指すように、以下のように変更してください：

```xml
<mainClass>com.pingcap.txn.optimistic.TxnExample</mainClass>
```

</div>

<div label="Golang" value="golang">

```markdown
The Golang example in the [Write a pessimistic transaction example](#write-a-pessimistic-transaction-example) section already supports optimistic transactions and can be used directly without changes.

</div>

<div label="Python" value="python">

[Write a pessimistic transaction example](#write-a-pessimistic-transaction-example)セクションのPythonの例は、すでに楽観的なトランザクションをサポートしており、変更せずに直接使用できます。

</div>

</SimpleTab>

### オーバーセルを含まない例

サンプルプログラムを実行します:

<SimpleTab groupId="language">

<div label="Java" value="java">

```shell
mvn clean package
java -jar target/plain-java-txn-0.0.1-jar-with-dependencies.jar ALICE_NUM=4 BOB_NUM=6
```

</div>

<div label="Golang" value="golang">

```shell
go build -o bin/txn
./bin/txn -a 4 -b 6 -o true
```

</div>

<div label="Python" value="python">

```shell
OPTIMISTIC=True ALICE=4 BOB=6 python3 txn_example.py
```

</div>

</SimpleTab>

SQLステートメントの実行プロセス：

```sql
    /* txn 2 */ BEGIN OPTIMISTIC
/* txn 1 */ BEGIN OPTIMISTIC
    /* txn 2 */ SELECT * FROM `books` WHERE `id` = 1 FOR UPDATE
    /* txn 2 */ UPDATE `books` SET `stock` = `stock` - 4 WHERE `id` = 1 AND `stock` - 4 >= 0
    /* txn 2 */ INSERT INTO `orders` (`id`, `book_id`, `user_id`, `quality`) VALUES (1001, 1, 1, 4)
    /* txn 2 */ UPDATE `users` SET `balance` = `balance` - 400.0 WHERE `id` = 2
    /* txn 2 */ COMMIT
/* txn 1 */ SELECT * FROM `books` WHERE `id` = 1 for UPDATE
/* txn 1 */ UPDATE `books` SET `stock` = `stock` - 6 WHERE `id` = 1 AND `stock` - 6 >= 0
/* txn 1 */ INSERT INTO `orders` (`id`, `book_id`, `user_id`, `quality`) VALUES (1000, 1, 1, 6)
/* txn 1 */ UPDATE `users` SET `balance` = `balance` - 600.0 WHERE `id` = 1
retry 1 times for 9007 Write conflict, txnStartTS=432618733006225412, conflictStartTS=432618733006225411, conflictCommitTS=432618733006225414, key={tableID=126, handle=1} primary={tableID=114, indexID=1, indexValues={1, 1000, }} [try again later]
/* txn 1 */ BEGIN OPTIMISTIC
/* txn 1 */ SELECT * FROM `books` WHERE `id` = 1 FOR UPDATE
/* txn 1 */ UPDATE `books` SET `stock` = `stock` - 6 WHERE `id` = 1 AND `stock` - 6 >= 0
/* txn 1 */ INSERT INTO `orders` (`id`, `book_id`, `user_id`, `quality`) VALUES (1000, 1, 1, 6)
/* txn 1 */ UPDATE `users` SET `balance` = `balance` - 600.0 WHERE `id` = 1
/* txn 1 */ COMMIT
```

楽観的なトランザクションモードでは、途中の状態が必ずしも正しいとは限らないため、悲観的なトランザクションモードのように `affected_rows` を使用して文が正しく実行されたかどうかを判断することはできません。トランザクション全体を見なければならず、最終的な `COMMIT` ステートメントが例外を返すかどうかを確認して、現在のトランザクションに書き込み競合があるかどうかを判断する必要があります。

上記のSQLログからわかるように、2つのトランザクションが同時に実行され、同じレコードが修正されたため、`txn 1` の COMMIT 後に `9007 Write conflict` 例外がスローされます。楽観的なトランザクションモードにおける書き込み競合では、アプリケーション側で安全にリトライできます。1回のリトライの後に、データは正常にコミットされます。最終的な実行結果は期待通りです。

```sql
mysql> SELECT * FROM books;
+----+--------------------------------------+----------------------+---------------------+-------+--------+
| id | title                                | type                 | published_at        | stock | price  |
+----+--------------------------------------+----------------------+---------------------+-------+--------+
|  1 | Designing Data-Intensive Application | Science & Technology | 2018-09-01 00:00:00 |     0 | 100.00 |
+----+--------------------------------------+----------------------+---------------------+-------+--------+
1 row in set (0.01 sec)

mysql> SELECT * FROM orders;
+------+---------+---------+---------+---------------------+
| id   | book_id | user_id | quality | ordered_at          |
+------+---------+---------+---------+---------------------+
| 1000 |       1 |       1 |       6 | 2022-04-19 03:18:19 |
| 1001 |       1 |       1 |       4 | 2022-04-19 03:18:17 |
+------+---------+---------+---------+---------------------+
2 rows in set (0.01 sec)

mysql> SELECT * FROM users;
+----+---------+----------+
| id | balance | nickname |
+----+---------+----------+
|  1 | 9400.00 | Bob      |
|  2 | 9600.00 | Alice    |
+----+---------+----------+
2 rows in set (0.00 sec)
```

### オーバーセルを防止する例

このセクションでは、オーバーセルを防ぐ楽観的トランザクションの例について説明します。在庫にはまだ本が10冊残っているとします。Bobが7冊購入し、Aliceが4冊購入するとします。彼らはほぼ同時に注文します。何が起こるでしょうか？この要件に対応するために、楽観的トランザクションの例のコードを再利用できます。Bobの購入数を6から7に変更します。

サンプルプログラムを実行します:

<SimpleTab groupId="language">

<div label="Java" value="java">

```shell
mvn clean package
java -jar target/plain-java-txn-0.0.1-jar-with-dependencies.jar ALICE_NUM=4 BOB_NUM=7
```

</div>

<div label="Golang" value="golang">

```shell
go build -o bin/txn
./bin/txn -a 4 -b 7 -o true
```

</div>

<div label="Python" value="python">

```shell
OPTIMISTIC=True ALICE=4 BOB=7 python3 txn_example.py
```

</div>

</SimpleTab>

```sql
/* txn 1 */ BEGIN OPTIMISTIC
    /* txn 2 */ BEGIN OPTIMISTIC
    /* txn 2 */ SELECT * FROM `books` WHERE `id` = 1 FOR UPDATE
    /* txn 2 */ UPDATE `books` SET `stock` = `stock` - 4 WHERE `id` = 1 AND `stock` - 4 >= 0
    /* txn 2 */ INSERT INTO `orders` (`id`, `book_id`, `user_id`, `quality`) VALUES (1001, 1, 1, 4)
    /* txn 2 */ UPDATE `users` SET `balance` = `balance` - 400.0 WHERE `id` = 2
    /* txn 2 */ COMMIT
/* txn 1 */ SELECT * FROM `books` WHERE `id` = 1 FOR UPDATE
/* txn 1 */ UPDATE `books` SET `stock` = `stock` - 7 WHERE `id` = 1 AND `stock` - 7 >= 0
/* txn 1 */ INSERT INTO `orders` (`id`, `book_id`, `user_id`, `quality`) VALUES (1000, 1, 1, 7)
/* txn 1 */ UPDATE `users` SET `balance` = `balance` - 700.0 WHERE `id` = 1
retry 1 times for 9007 Write conflict, txnStartTS=432619094333980675, conflictStartTS=432619094333980676, conflictCommitTS=432619094333980678, key={tableID=126, handle=1} primary={tableID=114, indexID=1, indexValues={1, 1000, }} [try again later]
/* txn 1 */ BEGIN OPTIMISTIC
/* txn 1 */ SELECT * FROM `books` WHERE `id` = 1 FOR UPDATE
Fail -> out of stock
/* txn 1 */ ROLLBACK
```

上記のSQLログからわかるように、最初の実行で書き込み競合が発生したため、アプリケーション側で `txn 1` がリトライされます。最新のスナップショットを比較することで、在庫がなくなっていることがわかります。アプリケーション側は `out of stock` をスローし、異常終了します。
```
```
mysql> SELECT * FROM orders;
+------+---------+---------+---------+---------------------+
| id   | book_id | user_id | quality | ordered_at          |
+------+---------+---------+---------+---------------------+
| 1001 |       1 |       1 |       4 | 2022-04-19 03:41:16 |
+------+---------+---------+---------+---------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM users;
+----+----------+----------+
| id | balance  | nickname |
+----+----------+----------+
|  1 | 10000.00 | Bob      |
|  2 |  9600.00 | Alice    |
+----+----------+----------+
2 rows in set (0.00 sec)
```