---
title: データの削除
summary: データ削除のためのSQL構文、ベストプラクティス、および例について学びます。

# データの削除

このドキュメントでは、TiDBでデータを削除するために[DELETE](/sql-statements/sql-statement-delete.md) SQLステートメントを使用する方法について説明します。期限切れのデータを定期的に削除する必要がある場合は、[time to live](/time-to-live.md) 機能を使用してください。

## 開始する前に

このドキュメントを読む前に、以下の準備が必要です。

- [TiDB Serverless Clusterの構築](/develop/dev-guide-build-cluster-in-cloud.md)
- [スキーマ設計の概要](/develop/dev-guide-schema-design-overview.md)、[データベースの作成](/develop/dev-guide-create-database.md)、[テーブルの作成](/develop/dev-guide-create-table.md)、および[二次インデックスの作成](/develop/dev-guide-create-secondary-indexes.md)を読んでください。
- [データの挿入](/develop/dev-guide-insert-data.md)

## SQL構文

`DELETE`ステートメントは、通常、以下の形式で使います。

```sql
DELETE FROM {table} WHERE {filter}
```

| パラメータ名 | 説明 |
| :--------: | :------------: |
| `{table}`  |      テーブル名      |
| `{filter}` | フィルタの一致条件|

この例は`DELETE`の単純な使用例のみを示しています。詳細な情報については、[DELETE構文](/sql-statements/sql-statement-delete.md)を参照してください。

## ベストプラクティス

データを削除する際に従うべきいくつかのベストプラクティスは次のとおりです。

- `DELETE`ステートメントで常に`WHERE`句を指定してください。`WHERE`句が指定されていない場合、TiDBはテーブルの**_すべての行_**を削除します。

<CustomContent platform="tidb">

- 大量の行（例：1万行以上）を削除する場合は、TiDBが単一トランザクションのサイズを制限しているため、[bulk-delete](#bulk-delete)を使用してください（デフォルトで[txn-total-size-limit](/tidb-configuration-file.md#txn-total-size-limit)が100MB）。

</CustomContent>

<CustomContent platform="tidb-cloud">

- 大量の行（例：1万行以上）を削除する場合は、TiDBがデフォルトで単一トランザクションのサイズを100MBに制限しているため、[bulk-delete](#bulk-delete)を使用してください。

</CustomContent>

- テーブル内のすべてのデータを削除する場合は、`DELETE`ステートメントを使用せずに、代わりに[`TRUNCATE`](/sql-statements/sql-statement-truncate.md)ステートメントを使用してください。
- パフォーマンスの考慮事項については、[パフォーマンスの考慮事項](#performance-considerations)を参照してください。
- 大量のデータを削除する必要があるシナリオでは、[非トランザクションのbulk-delete](#non-transactional-bulk-delete)を使用すると、パフォーマンスを大幅に向上させることができます。ただし、この操作は削除のトランザクション性を失い、したがって**取り消すことができなくなります**。適切な操作を選択してください。

## 例

特定の時間枠内でアプリケーションエラーが発生し、この期間内の[ratings](/develop/dev-guide-bookshop-schema-design.md#ratings-table)のすべてのデータを削除する必要があるとします。たとえば、`2022-04-15 00:00:00`から`2022-04-15 00:15:00`までの場合、`SELECT`ステートメントを使用して削除すべきレコードの数を確認できます。

```sql
SELECT COUNT(*) FROM `ratings` WHERE `rated_at` >= "2022-04-15 00:00:00" AND `rated_at` <= "2022-04-15 00:15:00";
```

10,000件以上のレコードが返される場合は、[Bulk-Delete](#bulk-delete)を使用して削除します。

10,000件未満のレコードが返される場合は、以下の例を使用して削除します。

<SimpleTab groupId="language">
<div label="SQL" value="sql">

SQLでは、以下のようになります。

```sql
DELETE FROM `ratings` WHERE `rated_at` >= "2022-04-15 00:00:00" AND `rated_at` <= "2022-04-15 00:15:00";
```

</div>

<div label="Java" value="java">

Javaでは、以下のようになります。

```java
// dsはcom.mysql.cj.jdbc.MysqlDataSourceのエンティティです。

try (Connection connection = ds.getConnection()) {
    String sql = "DELETE FROM `bookshop`.`ratings` WHERE `rated_at` >= ? AND `rated_at` <= ?";
    PreparedStatement preparedStatement = connection.prepareStatement(sql);
    Calendar calendar = Calendar.getInstance();
    calendar.set(Calendar.MILLISECOND, 0);

    calendar.set(2022, Calendar.APRIL, 15, 0, 0, 0);
    preparedStatement.setTimestamp(1, new Timestamp(calendar.getTimeInMillis()));

    calendar.set(2022, Calendar.APRIL, 15, 0, 15, 0);
    preparedStatement.setTimestamp(2, new Timestamp(calendar.getTimeInMillis()));

    preparedStatement.executeUpdate();
} catch (SQLException e) {
    e.printStackTrace();
}
```

</div>

<div label="Golang" value="golang">

Golangでは、以下のようになります。

```go
package main

import (
    "database/sql"
    "fmt"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:4000)/bookshop")
    if err != nil {
        panic(err)
    }
    defer db.Close()

    startTime := time.Date(2022, 04, 15, 0, 0, 0, 0, time.UTC)
    endTime := time.Date(2022, 04, 15, 0, 15, 0, 0, time.UTC)

    bulkUpdateSql := fmt.Sprintf("DELETE FROM `bookshop`.`ratings` WHERE `rated_at` >= ? AND `rated_at` <= ?")
    result, err := db.Exec(bulkUpdateSql, startTime, endTime)
    if err != nil {
        panic(err)
    }
    _, err = result.RowsAffected()
    if err != nil {
        panic(err)
    }
}
```

</div>

<div label="Python" value="python">

Pythonでは、以下のようになります。

```python
import MySQLdb
import datetime
import time
connection = MySQLdb.connect(
    host="127.0.0.1",
    port=4000,
    user="root",
    password="",
    database="bookshop",
    autocommit=True
)
with connection:
    with connection.cursor() as cursor:
        start_time = datetime.datetime(2022, 4, 15)
        end_time = datetime.datetime(2022, 4, 15, 0, 15)
        delete_sql = "DELETE FROM `bookshop`.`ratings` WHERE `rated_at` >= %s AND `rated_at` <= %s"
        affect_rows = cursor.execute(delete_sql, (start_time, end_time))
        print(f'delete {affect_rows} data')
```

</div>

</SimpleTab>

<CustomContent platform="tidb">

`rated_at`フィールドは[日付と時刻の型](/data-type-date-and-time.md)で`DATETIME`型です。これはTiDBでリテラル量として格納されていると仮定できます。一方、`TIMESTAMP`型はタイムスタンプを格納し、異なる[タイムゾーン](/configure-time-zone.md)で異なる時刻文字列を表示します。

</CustomContent>

<CustomContent platform="tidb-cloud">

`rated_at`フィールドは[日付と時刻の型](/data-type-date-and-time.md)で`DATETIME`型です。これはTiDBでリテラル量として格納されていると仮定できます。一方、`TIMESTAMP`型はタイムスタンプを格納し、異なるタイムゾーンで異なる時刻文字列を表示します。

</CustomContent>

> **注意:**
>
> MySQLと同様に、`TIMESTAMP`データ型は[2038年問題](https://en.wikipedia.org/wiki/Year_2038_problem)に影響を受けます。2038年よりも大きな値を保存する場合は、`DATETIME`型を使用することを推奨します。

## パフォーマンスの考慮事項

### TiDB GCメカニズム

TiDBは、`DELETE`ステートメントを実行した後、データをすぐに削除しません。代わりにデータを削除の準備ができたとマークし、その後TiDB GC（Garbage Collection）が古いデータをクリーンアップするのを待ちます。したがって、`DELETE`ステートメントではディスク使用量を**_すぐに_**削減しません。

デフォルトでは、GCは10分ごとにトリガーされます。各GCは**safe_point**という時間点を計算します。この時間点以前のデータは再利用されないため、TiDBはそれを安全にクリーンアップできます。

詳細については、[GCメカニズム](/garbage-collection-overview.md)を参照してください。

### 統計情報の更新

TiDBは、インデックスの選択を決定するために[統計情報](/statistics.md)を使用します。大量のデータが削除された後は、インデックスが正しく選択されない高いリスクがあります。[手動コレクション](/statistics.md#manual-collection)を使用して統計情報を更新できます。これにより、SQLパフォーマンスの最適化のためにTiDBオプティマイザにより正確な統計情報が提供されます。

## Bulk-delete

複数の行データをテーブルから削除する必要がある場合、[`DELETE`の例](#example)を選択し、削除する必要があるデータをフィルタリングするために`WHERE`句を使用できます。

<CustomContent platform="tidb">

ただし、一度に多数の行（1万以上）を削除する必要がある場合は、反復的な方法でデータを削除することをお勧めします。つまり、削除が完了するまで、データの一部を反復して削除します。これは、TiDBが単一トランザクションのサイズを制限しているためです（[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)、デフォルトでは100 MB）。このような操作を実行するためには、プログラムやスクリプトでループを使用できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

ただし、一度に多数の行（1万以上）を削除する必要がある場合は、反復的な方法でデータを削除することをお勧めします。つまり、削除が完了するまで、データの一部を反復して削除します。これは、TiDBが単一トランザクションのサイズをデフォルトで100 MBに制限しているためです。このような操作を実行するためには、プログラムやスクリプトでループを使用できます。

</CustomContent>

このセクションでは、スクリプトを書いて反復的な削除操作を処理する例を提供しており、`SELECT`と`DELETE`の組み合わせを行い、一括削除を完了する方法を示しています。

### 一括削除ループを書く

アプリケーションまたはスクリプトのループ内に`DELETE`文を記述し、`WHERE`句を使用してデータをフィルタリングし、`LIMIT`を使用して単一の文で削除される行数を制限できます。

### 一括削除の例

特定の時間枠内でアプリケーションエラーが見つかったとします。例えば、`2022-04-15 00:00:00`から`2022-04-15 00:15:00`までの[評価](/develop/dev-guide-bookshop-schema-design.md#ratings-table)のすべてのデータを削除する必要があり、15分間で1万件を超えるレコードが書き込まれています。以下のように実行できます。

<SimpleTab groupId="language">
<div label="Java" value="java">

Javaでは、一括削除の例は以下のようになります：

```java
package com.pingcap.bulkDelete;

import com.mysql.cj.jdbc.MysqlDataSource;

import java.sql.*;
import java.util.*;
import java.util.concurrent.TimeUnit;

public class BatchDeleteExample
{
    public static void main(String[] args) throws InterruptedException {
        // 例のデータベース接続を構成します。

        // mysqlデータソースのインスタンスを作成します。
        MysqlDataSource mysqlDataSource = new MysqlDataSource();

        // サーバー名、ポート、データベース名、ユーザー名、パスワードを設定します。
        mysqlDataSource.setServerName("localhost");
        mysqlDataSource.setPortNumber(4000);
        mysqlDataSource.setDatabaseName("bookshop");
        mysqlDataSource.setUser("root");
        mysqlDataSource.setPassword("");

        while (true) {
            batchDelete(mysqlDataSource);
            TimeUnit.SECONDS.sleep(1);
        }
    }

    public static void batchDelete (MysqlDataSource ds) {
        try (Connection connection = ds.getConnection()) {
            String sql = "DELETE FROM `bookshop`.`ratings` WHERE `rated_at` >= ? AND `rated_at` <= ? LIMIT 1000";
            PreparedStatement preparedStatement = connection.prepareStatement(sql);
            Calendar calendar = Calendar.getInstance();
            calendar.set(Calendar.MILLISECOND, 0);

            calendar.set(2022, Calendar.APRIL, 15, 0, 0, 0);
            preparedStatement.setTimestamp(1, new Timestamp(calendar.getTimeInMillis()));

            calendar.set(2022, Calendar.APRIL, 15, 0, 15, 0);
            preparedStatement.setTimestamp(2, new Timestamp(calendar.getTimeInMillis()));

            int count = preparedStatement.executeUpdate();
            System.out.println("delete " + count + " data");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

各反復では、`DELETE`は`2022-04-15 00:00:00`から`2022-04-15 00:15:00`までの最大で1000行を削除します。

</div>

<div label="Golang" value="golang">

Golangでは、一括削除の例は以下のようになります：

```go
package main

import (
    "database/sql"
    "fmt"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:4000)/bookshop")
    if err != nil {
        panic(err)
    }
    defer db.Close()

    affectedRows := int64(-1)
    startTime := time.Date(2022, 04, 15, 0, 0, 0, 0, time.UTC)
    endTime := time.Date(2022, 04, 15, 0, 15, 0, 0, time.UTC)

    for affectedRows != 0 {
        affectedRows, err = deleteBatch(db, startTime, endTime)
        if err != nil {
            panic(err)
        }
    }
}

// deleteBatchは一度に最大1000行を削除します
func deleteBatch(db *sql.DB, startTime, endTime time.Time) (int64, error) {
    bulkUpdateSql := fmt.Sprintf("DELETE FROM `bookshop`.`ratings` WHERE `rated_at` >= ? AND `rated_at` <= ? LIMIT 1000")
    result, err := db.Exec(bulkUpdateSql, startTime, endTime)
    if err != nil {
        return -1, err
    }
    affectedRows, err := result.RowsAffected()
    if err != nil {
        return -1, err
    }

    fmt.Printf("delete %d data\n", affectedRows)
    return affectedRows, nil
}
```

各反復では、`DELETE`は`2022-04-15 00:00:00`から`2022-04-15 00:15:00`までの最大で1000行を削除します。

</div>

<div label="Python" value="python">

Pythonでは、一括削除の例は以下のようになります：

```python
import MySQLdb
import datetime
import time
connection = MySQLdb.connect(
    host="127.0.0.1",
    port=4000,
    user="root",
    password="",
    database="bookshop",
    autocommit=True
)
with connection:
    with connection.cursor() as cursor:
        start_time = datetime.datetime(2022, 4, 15)
        end_time = datetime.datetime(2022, 4, 15, 0, 15)
        affect_rows = -1
        while affect_rows != 0:
            delete_sql = "DELETE FROM `bookshop`.`ratings` WHERE `rated_at` >= %s AND  `rated_at` <= %s LIMIT 1000"
            affect_rows = cursor.execute(delete_sql, (start_time, end_time))
            print(f'delete {affect_rows} data')
            time.sleep(1)
```

各反復では、`DELETE`は`2022-04-15 00:00:00`から`2022-04-15 00:15:00`までの最大で1000行を削除します。

</div>

</SimpleTab>

## 非トランザクショナル一括削除

> **注記:**
>
> TiDB v6.1.0から、[非トランザクショナルDML文](/non-transactional-dml.md)がサポートされています。この機能は、TiDB v6.1.0より前のバージョンでは利用できません。

### 非トランザクショナル一括削除の前提条件

非トランザクショナル一括削除を使用する前に、まず[非トランザクショナルDML文のドキュメント](/non-transactional-dml.md)を読んでください。非トランザクショナル一括削除は、バッチデータ処理シナリオにおけるパフォーマンスと使いやすさが向上しますが、トランザクショナルな原子性と分離性を犠牲にします。

そのため、重大な結果（データの損失など）を避けるためには慎重に使用する必要があります。

### 非トランザクショナル一括削除のためのSQL構文

非トランザクショナル一括削除文のSQL構文は次の通りです。

```sql
BATCH ON {shard_column} LIMIT {batch_size} {delete_statement};
```

| パラメータ名 | 説明 |
| :--------: | :------------: |
| `{shard_column}` | バッチを分割するために使用する列。      |
| `{batch_size}`   | 各バッチのサイズを制御します。 |
| `{delete_statement}` | `DELETE`文。 |

上記の例は、非トランザクショナル一括削除文の簡単な使用例を示しています。詳細については、[非トランザクショナルDML文](/non-transactional-dml.md)を参照してください。

### 非トランザクショナル一括削除の例

一括削除の例と同じシナリオで、非トランザクショナル一括削除を実行するSQL文は以下の通りです。

```sql
BATCH ON `rated_at` LIMIT 1000 DELETE FROM `ratings` WHERE `rated_at` >= "2022-04-15 00:00:00" AND  `rated_at` <= "2022-04-15 00:15:00";
```