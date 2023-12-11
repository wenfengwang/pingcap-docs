---
title: トランザクションの制約
summary: TiDBにおけるトランザクションの制約について学びます。

# トランザクションの制約

このドキュメントではTiDBにおけるトランザクションの制約について簡単に紹介します。

## 分離レベル

TiDBでサポートされている分離レベルは **RC (Read Committed)** および **SI (Snapshot Isolation)** です。ここで、**SI** は基本的に **RR (Repeatable Read)** 分離レベルと同等です。

![分離レベル](/media/develop/transaction_isolation_level.png)

## スナップショット分離は幽霊読み取りを避けることができます

TiDBの`SI`分離レベルは **幽霊読み取り (Phantom Reads)** を避けることができますが、ANSI/ISO SQL標準の`RR`では避けることができません。

以下の2つの例は、**幽霊読み取り**について説明します。

- 例1: **トランザクションA**はまずクエリに従って`n`行を取得し、その後、**トランザクションB**が`n`行以外の`m`行を変更するか、**トランザクションA**のクエリに一致する`m`行を追加します。**トランザクションA**が再びクエリを実行すると、条件に一致する`n+m`行があることがわかります。これは幽霊のようなものであり、それが**幽霊読み取り**と呼ばれる理由です。

- 例2: **管理者A**はデータベース内のすべての学生の成績を特定の点数からABCDEの成績に変更しますが、この時**管理者B**が特定の点数でレコードを挿入します。**管理者A**が変更を終えても、変更されていない（**管理者B**が挿入した）レコードがまだ存在することがあります。これが**幽霊読み取り**です。

## SIはワイスキューを避けることができません

TiDBのSI分離レベルは **ワイスキュー (write skew)** 例外を避けることができません。`SELECT FOR UPDATE`構文を使用して**ワイスキュー**例外を避けることができます。

**ワイスキュー**例外は、2つの並行トランザクションが異なるが関連するレコードを読み取り、それぞれのトランザクションが読み取ったデータを更新し、最終的にトランザクションをコミットすると発生します。これらの関連するレコードには、複数のトランザクションによって同時に変更できない制約が存在する場合、最終的な結果はその制約を破ることになります。

例えば、病院の医師のシフト管理プログラムを作成しているとします。通常、病院では複数の医師が同時に当直になる必要がありますが、最低条件は少なくとも1人の医師が当直であることです。医師は（たとえば、病気のために）自分のシフトをキャンセルすることができますが、そのシフト中に少なくとも1人の医師が当直である必要があります。

ここで、医師`Alice`と`Bob`が当直であり、両方とも病気で休職することになったとします。彼らは同時にボタンをクリックすることになります。以下のプログラムでこのプロセスをシミュレートしてみましょう：

<SimpleTab groupId="language">

<div label="Java" value="java">

```java
// 以下のコードを使用して、TiDBトランザクションに適応します
// 医師のシフト管理に関する簡単な例です
```

</div>

<div label="Golang" value="golang">

TiDBトランザクションを適応するための[util](https://github.com/pingcap-inc/tidb-example-golang/tree/main/util)を以下のコードに従って書き込みます：

```go
// 以下のコードを使用して、TiDBトランザクションに適応します
// 医師のシフト管理に関する簡単な例です
```

</div>

```go
    // Txn 1はtxn 2が完了するまで待機する必要があります。
    if goroutineID == 1 {
        <-waitingChan
    }

    txnFunc := func() error {
        queryCurrentOnCall := "SELECT COUNT(*) AS `count` FROM `doctors` WHERE `on_call` = ? AND `shift_id` = ?"
        rows, err := txn.Query(queryCurrentOnCall, true, 123)
        if err != nil {
            return err
        }
        defer rows.Close()
        fmt.Println(txnComment + queryCurrentOnCall + " 成功")

        count := 0
        if rows.Next() {
            err = rows.Scan(&count)
            if err != nil {
                return err
            }
        }
        rows.Close()

        if count < 2 {
            return fmt.Errorf("少なくとも1人の医師が勤務しています")
        }

        shift := "UPDATE `doctors` SET `on_call` = ? WHERE `id` = ? AND `shift_id` = ?"
        _, err = txn.Exec(shift, false, doctorID, 123)
        if err == nil {
            fmt.Println(txnComment + shift + " 成功")
        }
        return err
    }

    err = txnFunc()
    if err == nil {
        txn.Commit()
        fmt.Println("[runTxn] 成功しました")
    } else {
        txn.Rollback()
        fmt.Printf("[runTxn] エラーが発生したため、ロールバック: %+v\n", err)
    }

    // Txn 2が完了しました。Txn 1を再度実行させます。
    if goroutineID == 2 {
        waitingChan <- true
    }

    return nil
}

func prepareData(db *sql.DB) error {
    err := createDoctorTable(db)
    if err != nil {
        return err
    }

    err = createDoctor(db, 1, "Alice", true, 123)
    if err != nil {
        return err
    }
    err = createDoctor(db, 2, "Bob", true, 123)
    if err != nil {
        return err
    }
    err = createDoctor(db, 3, "Carol", false, 123)
    if err != nil {
        return err
    }
    return nil
}

func createDoctorTable(db *sql.DB) error {
    _, err := db.Exec("CREATE TABLE IF NOT EXISTS `doctors` (" +
        "    `id` int(11) NOT NULL," +
        "    `name` varchar(255) DEFAULT NULL," +
        "    `on_call` tinyint(1) DEFAULT NULL," +
        "    `shift_id` int(11) DEFAULT NULL," +
        "    PRIMARY KEY (`id`)," +
        "    KEY `idx_shift_id` (`shift_id`)" +
        "  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin")
    return err
}

func createDoctor(db *sql.DB, id int, name string, onCall bool, shiftID int) error {
    _, err := db.Exec("INSERT INTO `doctors` (`id`, `name`, `on_call`, `shift_id`) VALUES (?, ?, ?, ?)",
        id, name, onCall, shiftID)
    return err
}
```
```java
    + insert.executeUpdate();

    + connection.commit();
} else {
    throw new RuntimeException("At least one doctor is on call");
}
}

// Txn 2 done, let txn 1 run again
if (txnID == 2) {
    txn1Pass.release();
}
} catch (Exception e) {
// If got any error, you should roll back, data is priceless
connection.rollback();
e.printStackTrace();
}
} catch (SQLException e) {
e.printStackTrace();
}
}
}
```

### Golang

```go
package main

import (
"database/sql"
"fmt"
"sync"

"github.com/pingcap-inc/tidb-example-golang/util"

_ "github.com/go-sql-driver/mysql"
)

func main() {
openDB("mysql", "root:@tcp(127.0.0.1:4000)/test", func(db *sql.DB) {
writeSkew(db)
})
}

func openDB(driverName, dataSourceName string, runnable func(db *sql.DB)) {
db, err := sql.Open(driverName, dataSourceName)
if err != nil {
panic(err)
}
defer db.Close()

runnable(db)
}

func writeSkew(db *sql.DB) {
err := prepareData(db)
if err != nil {
panic(err)
}

waitingChan, waitGroup := make(chan bool), sync.WaitGroup{}

waitGroup.Add(1)
go func() {
defer waitGroup.Done()
err = askForLeave(db, waitingChan, 1, 1)
if err != nil {
panic(err)
}
}()

waitGroup.Add(1)
go func() {
defer waitGroup.Done()
err = askForLeave(db, waitingChan, 2, 2)
if err != nil {
panic(err)
}
}()

waitGroup.Wait()
}

func askForLeave(db *sql.DB, waitingChan chan bool, goroutineID, doctorID int) error {
txnComment := fmt.Sprintf("/* txn %d */ ", goroutineID)
if goroutineID != 1 {
txnComment = "\t" + txnComment
}

txn, err := util.TiDBSqlBegin(db, true)
if err != nil {
return err
}
fmt.Println(txnComment + "start txn")

// Txn 1 should be waiting until txn 2 is done.
if goroutineID == 1 {
<-waitingChan
}

txnFunc := func() error {
queryCurrentOnCall := "SELECT COUNT(*) AS `count` FROM `doctors` WHERE `on_call` = ? AND `shift_id` = ?"
rows, err := txn.Query(queryCurrentOnCall, true, 123)
if err != nil {
return err
}
defer rows.Close()
fmt.Println(txnComment + queryCurrentOnCall + " successful")

count := 0
if rows.Next() {
err = rows.Scan(&count)
if err != nil {
return err
}
}
rows.Close()

if count < 2 {
return fmt.Errorf("at least one doctor is on call")
}

shift := "UPDATE `doctors` SET `on_call` = ? WHERE `id` = ? AND `shift_id` = ?"
_, err = txn.Exec(shift, false, doctorID, 123)
if err == nil {
fmt.Println(txnComment + shift + " successful")
}
return err
}

err = txnFunc()
if err == nil {
txn.Commit()
fmt.Println("[runTxn] commit success")
} else {
txn.Rollback()
fmt.Printf("[runTxn] got an error, rollback: %+v\n", err)
}

// Txn 2 is done. Let txn 1 run again.
if goroutineID == 2 {
waitingChan <- true
}

return nil
}

func prepareData(db *sql.DB) error {
err := createDoctorTable(db)
if err != nil {
return err
}

err = createDoctor(db, 1, "Alice", true, 123)
if err != nil {
return err
}
err = createDoctor(db, 2, "Bob", true, 123)
if err != nil {
return err
}
err = createDoctor(db, 3, "Carol", false, 123)
if err != nil {
return err
}
return nil
}

func createDoctorTable(db *sql.DB) error {
_, err := db.Exec("CREATE TABLE IF NOT EXISTS `doctors` (" +
"    `id` int(11) NOT NULL," +
"    `name` varchar(255) DEFAULT NULL," +
"    `on_call` tinyint(1) DEFAULT NULL," +
"    `shift_id` int(11) DEFAULT NULL," +
"    PRIMARY KEY (`id`)," +
"    KEY `idx_shift_id` (`shift_id`)" +
"  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin")
return err
}

func createDoctor(db *sql.DB, id int, name string, onCall bool, shiftID int) error {
_, err := db.Exec("INSERT INTO `doctors` (`id`, `name`, `on_call`, `shift_id`) VALUES (?, ?, ?, ?)",
id, name, onCall, shiftID)
return err
}
```

You can apply for errors like as shown below:

```java
    + throw new RuntimeException("At least one doctor is on call");
```

And further translate the rest of the content.
```
Currently locks are not added to auto-committed `SELECT FOR UPDATE` statements. The effect is shown in the following figure:

![The situation in TiDB](/media/develop/autocommit_selectforupdate_nowaitlock.png)

This is a known incompatibility issue with MySQL. You can solve this issue by using the explicit `BEGIN;COMMIT;` statements.
```