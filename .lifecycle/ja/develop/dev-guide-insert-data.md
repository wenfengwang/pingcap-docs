---
title: データの挿入
summary: データの挿入方法について学びます。
---

<!-- markdownlint-disable MD029 -->

# データの挿入

本書では、異なるプログラミング言語を使用してSQL言語を使い、TiDBにデータを挿入する方法について説明します。

## 開始する前に

本書を読む前に、以下を準備する必要があります。

- [TiDB Serverless Clusterの構築](/develop/dev-guide-build-cluster-in-cloud.md)。
- [スキーマデザインの概要](/develop/dev-guide-schema-design-overview.md)、[データベースの作成](/develop/dev-guide-create-database.md)、[テーブルの作成](/develop/dev-guide-create-table.md)、および[二次インデックスの作成](/develop/dev-guide-create-secondary-indexes.md)を読む。

## 行の挿入

データを複数行挿入するには、2つの方法があります。例えば、**3**人分のプレイヤーデータを挿入する必要がある場合、

- **複数行の挿入文**を使用します：

    {{< copyable "sql" >}}

    ```sql
    INSERT INTO `player` (`id`, `coins`, `goods`) VALUES (1, 1000, 1), (2, 230, 2), (3, 300, 5);
    ```

- 複数の**単一行の挿入文**を使用します：

    {{< copyable "sql" >}}

    ```sql
    INSERT INTO `player` (`id`, `coins`, `goods`) VALUES (1, 1000, 1);
    INSERT INTO `player` (`id`, `coins`, `goods`) VALUES (2, 230, 2);
    INSERT INTO `player` (`id`, `coins`, `goods`) VALUES (3, 300, 5);
    ```

通常、`複数行の挿入文`の方が、複数の`単一行の挿入文`よりも高速に実行されます。

<SimpleTab>
<div label="SQL">

```sql
CREATE TABLE `player` (`id` INT, `coins` INT, `goods` INT);
INSERT INTO `player` (`id`, `coins`, `goods`) VALUES (1, 1000, 1), (2, 230, 2);
```

このSQLの使用方法については、[TiDBクラスターへの接続](/develop/dev-guide-build-cluster-in-cloud.md#step-2-connect-to-a-cluster)を参照し、クライアントを使用してTiDBクラスターに接続した後、SQL文を入力する手順に従ってください。

</div>

<div label="Java">

```java
// dsはcom.mysql.cj.jdbc.MysqlDataSourceのエンティティです
try (Connection connection = ds.getConnection()) {
    connection.setAutoCommit(false);

    PreparedStatement pstmt = connection.prepareStatement("INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)"))

    // 1つ目のプレイヤー
    pstmt.setInt(1, 1);
    pstmt.setInt(2, 1000);
    pstmt.setInt(3, 1);
    pstmt.addBatch();

    // 2つ目のプレイヤー
    pstmt.setInt(1, 2);
    pstmt.setInt(2, 230);
    pstmt.setInt(3, 2);
    pstmt.addBatch();

    pstmt.executeBatch();
    connection.commit();
} catch (SQLException e) {
    e.printStackTrace();
}
```

デフォルトのMySQL JDBCドライバーの設定により、良好なバルク挿入のパフォーマンスを得るために、いくつかのパラメータを変更する必要があります。

|            パラメーター           |                 意味                  |   推奨シナリオ   | 推奨構成|
| :------------------------: | :-----------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------: | :----------------------: |
|    `useServerPrepStmts`    |    サーバーサイドでプリペアドステートメントを有効にするかどうか    |  プリペアドステートメントを複数回使用する必要がある場合                                                             |          `true`          |
|      `cachePrepStmts`      |       クライアントがプリペアドステートメントをキャッシュするかどうか        |                                                           `useServerPrepStmts=true`の場合                                                           |          `true`          |
|  `prepStmtCacheSqlLimit`   |  プリペアドステートメントの最大サイズ (デフォルトでは256文字)  | プリペアドステートメントが256文字を超える場合 | 実際のプリペアドステートメントのサイズに応じて構成する |
|    `prepStmtCacheSize`     | プリペアドステートメントキャッシュの最大数 (デフォルトでは25) | プリペアドステートメントの数が25を超える場合  | 実際のプリペアドステートメントの数に応じて構成する |
| `rewriteBatchedStatements` |          バッチ操作を書き換えるかどうか          | バッチ操作が必要な場合 |          `true`          |
|    `allowMultiQueries`     |             バッチ操作を開始するかどうか              | **Batched** ステートメントを書き換える`rewriteBatchedStatements = true` かつ `useServerPrepStmts = true`の設定をするには、[client bug](https://bugs.mysql.com/bug.php?id=96623)により必要になる |          `true`          |

MySQL JDBCドライバーは、`useConfigs`という統合された構成を提供します。これを`maxPerformance`で構成すると、一連の構成を構成することになります。例えば、`mysql:mysql-connector-java:8.0.28`を取ると、`useConfigs=maxPerformance`は以下を含みます。

```properties
cachePrepStmts=true
cacheCallableStmts=true
cacheServerConfiguration=true
useLocalSessionState=true
elideSetAutoCommits=true
alwaysSendSetIsolation=false
enableQueryTimeouts=false
connectionAttributes=none
useInformationSchema=true
```

対応するMySQL JDBCドライバーのバージョンに含まれる`useConfigs=maxPerformance`の構成を取得するには、`mysql-connector-java-{version}.jar!/com/mysql/cj/configurations/maxPerformance.properties`を確認できます。

次のはJDBC接続文字列構成の典型的なシナリオです。この例では、ホスト: `127.0.0.1`、ポート: `4000`、ユーザー名: `root`、パスワード: null、デフォルトデータベース: `test`です：

```
jdbc:mysql://127.0.0.1:4000/test?user=root&useConfigs=maxPerformance&useServerPrepStmts=true&prepStmtCacheSqlLimit=2048&prepStmtCacheSize=256&rewriteBatchedStatements=true&allowMultiQueries=true
```

Javaの完全な例については、次を参照してください:
- [JDBCを使用してTiDBに接続](/develop/dev-guide-sample-application-java-jdbc.md)
- [Hibernateを使用してTiDBに接続](/develop/dev-guide-sample-application-java-hibernate.md)
- [Spring Bootを使用してTiDBに接続](/develop/dev-guide-sample-application-java-spring-boot.md)

</div>

<div label="Golang">

```go
package main

import (
    "database/sql"
    "strings"

    _ "github.com/go-sql-driver/mysql"
)

type Player struct {
    ID    string
    Coins int
    Goods int
}

func bulkInsertPlayers(db *sql.DB, players []Player, batchSize int) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }

    stmt, err := tx.Prepare(buildBulkInsertSQL(batchSize))
    if err != nil {
        return err
    }

    defer stmt.Close()

    for len(players) > batchSize {
        if _, err := stmt.Exec(playerToArgs(players[:batchSize])...); err != nil {
            tx.Rollback()
            return err
        }

        players = players[batchSize:]
    }

    if len(players) != 0 {
        if _, err := tx.Exec(buildBulkInsertSQL(len(players)), playerToArgs(players)...); err != nil {
            tx.Rollback()
            return err
        }
    }

    if err := tx.Commit(); err != nil {
        tx.Rollback()
        return err
    }

    return nil
}

func playerToArgs(players []Player) []interface{} {
    var args []interface{}
    for _, player := range players {
        args = append(args, player.ID, player.Coins, player.Goods)
    }
    return args
}

func buildBulkInsertSQL(amount int) string {
    return "INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)" + strings.Repeat(",(?,?,?)", amount-1)
}
```

Golangの完全な例については、次を参照してください:
- [Go-MySQL-Driverを使用してTiDBに接続](/develop/dev-guide-sample-application-golang-sql-driver.md)
- [GORMを使用してTiDBに接続](/develop/dev-guide-sample-application-golang-gorm.md)

</div>

<div label="Python">

```python
import MySQLdb
connection = MySQLdb.connect(
    host="127.0.0.1",
    port=4000,
    user="root",
    password="",
    database="bookshop",
    autocommit=True
)

with get_connection(autocommit=True) as connection:
    with connection.cursor() as cur:
        player_list = random_player(1919)
        for idx in range(0, len(player_list), 114):
            cur.executemany("INSERT INTO player (id, coins, goods) VALUES (%s, %s, %s)", player_list[idx:idx + 114])
```

Pythonの完全な例については、次を参照してください:
- [PyMySQLを使用してTiDBに接続](/develop/dev-guide-sample-application-python-pymysql.md)
- [mysqlclientを使用してTiDBに接続](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart)
- [MySQL Connector/Pythonを使用してTiDBに接続](/develop/dev-guide-sample-application-python-mysql-connector.md)
- [SQLAlchemyを使用してTiDBに接続](/develop/dev-guide-sample-application-python-sqlalchemy.md)
- [peewee を使用して TiDB に接続する](/develop/dev-guide-sample-application-python-peewee.md)

</div>

</SimpleTab>

## バルクインサート

TiDB クラスタに大量のデータを迅速にインポートする必要がある場合、データ移行に関連して **PingCAP** が提供するツール範囲を使用することをお勧めします。 `INSERT` ステートメントを使用するのは効率的ではなく、例外およびその他の問題を自力で処理する必要があるため、最適な方法ではありません。

以下は、バルクインサートに関して推奨されるツールです。

- データエクスポート: [Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview)。MySQL や TiDB データをローカルまたは Amazon S3 にエクスポートできます。

<CustomContent platform="tidb">

- データインポート: [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)。エクスポートされた **Dumpling** データ、**CSV** ファイル、または [Amazon Aurora から TiDB へのデータ移行](/migrate-aurora-to-tidb.md) をインポートできます。また、ローカルディスクまたは Amazon S3 クラウドディスクからデータを読み取ることもサポートしています。
- データレプリケーション: [TiDB データ移行](/dm/dm-overview.md)。MySQL、MariaDB、Amazon Aurora データベースを TiDB にレプリケートできます。また、ソースデータベースからの分割されたインスタンスおよびテーブルのマージおよび移行もサポートしています。
- データバックアップとリストア: [バックアップ＆リストア (BR)](/br/backup-and-restore-overview.md)。**Dumpling** に比べて、**BR** は**_ビッグデータ_**シナリオにより適しています。

</CustomContent>

<CustomContent platform="tidb-cloud">

- データインポート: [TiDB クラウドコンソール](https://tidbcloud.com/) の [Create Import](/tidb-cloud/import-sample-data.md) ページ。エクスポートした **Dumpling** データ、ローカル **CSV** ファイル、または [Amazon S3 や GCS から TiDB クラウドへの CSV ファイルのインポート](/tidb-cloud/import-csv-files.md) をサポートしています。また、ローカルディスク、Amazon S3 クラウドディスク、または GCS クラウドディスクからデータを読み取ることもサポートしています。
- データレプリケーション: [TiDB データ移行](https://docs.pingcap.com/tidb/stable/dm-overview)。MySQL、MariaDB、Amazon Aurora データベースを TiDB にレプリケートできます。また、ソースデータベースからの分割されたインスタンスおよびテーブルのマージおよび移行もサポートしています。
- データバックアップとリストア: TiDB Cloud コンソールの [Backup](/tidb-cloud/backup-and-restore.md) ページ。**Dumpling** に比べて、バックアップとリストアは**_ビッグデータ_**シナリオにより適しています。

</CustomContent>

## ホットスポットを回避する

テーブルを設計する際に、大量の挿入操作があるかどうかを考慮する必要があります。その場合は、テーブル設計中にホットスポットを回避する必要があります。[主キーの選択](/develop/dev-guide-create-table.md#select-primary-key)セクションを参照し、[主キーを選択する際のルール](/develop/dev-guide-create-table.md#guidelines-to-follow-when-selecting-primary-key)に従ってください。

<CustomContent platform="tidb">

ホットスポットの問題を処理する方法の詳細については、[ホットスポット問題のトラブルシューティング](/troubleshoot-hot-spot-issues.md)を参照してください。

</CustomContent>

## `AUTO_RANDOM` 主キーを持つテーブルにデータを挿入する

挿入するテーブルの主キーに `AUTO_RANDOM` 属性がある場合、デフォルトでは主キーを指定できません。例えば、[`bookshop`](/develop/dev-guide-bookshop-schema-design.md) データベースの [`users` テーブル](/develop/dev-guide-bookshop-schema-design.md#users-table) の `id` フィールドに `AUTO_RANDOM` 属性が含まれていることが分かります。

この場合、以下のような SQL を使用して挿入することは**できません**:

```sql
INSERT INTO `bookshop`.`users` (`id`, `balance`, `nickname`) VALUES (1, 0.00, 'nicky');
```

以下のエラーが発生します:

```
ERROR 8216 (HY000): Invalid auto random: Explicit insertion on auto_random column is disabled. Try to set @@allow_auto_random_explicit_insert = true.
```

挿入時に `AUTO_RANDOM` 列を手動で指定することはお勧めしません。

このエラーを処理するための2つの解決策があります:

- (推奨) この列を挿入ステートメントから削除し、TiDB が初期化した `AUTO_RANDOM` の値を使用します。これは `AUTO_RANDOM` のセマンティクスと一致しています。

    {{< copyable "sql" >}}

    ```sql
    INSERT INTO `bookshop`.`users` (`balance`, `nickname`) VALUES (0.00, 'nicky');
    ```

- もしもこの列を指定する必要があると確信している場合は、[`SET` ステートメント](https://docs.pingcap.com/ja/tidb/stable/sql-statement-set-variable)を使用してユーザー変数を変更し、挿入時に `AUTO_RANDOM` 列を指定できるようにすることができます。

    {{< copyable "sql" >}}

    ```sql
    SET @@allow_auto_random_explicit_insert = true;
    INSERT INTO `bookshop`.`users` (`id`, `balance`, `nickname`) VALUES (1, 0.00, 'nicky');
    ```

## HTAP を使用する

TiDB では、HTAP 機能によりデータを挿入する際に追加の操作が発生することがありません。追加の挿入ロジックはありません。TiDB はデータの整合性を自動的に保証します。テーブルを作成した後、[カラム指向レプリカ同期を有効にする](/develop/dev-guide-create-table.md#use-htap-capabilities)必要があり、カラム指向レプリカを使用してクエリを直接高速化するだけです。