---
title: JDBCを使用してTiDBに接続する
summary: JDBCを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Javaのサンプルコードスニペットを使用してTiDBとJDBCを使用する方法について説明します。
---

# JDBCを使用してTiDBに接続

TiDBはMySQL互換のデータベースであり、JDBC（Java Database Connectivity）はJavaのデータアクセスAPIです。[MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/)はJDBCのMySQL実装です。

このチュートリアルでは、TiDBとJDBCを使用して次のタスクを実行する方法を学ぶことができます。

- 環境のセットアップ。
- JDBCを使用してTiDBクラスタに接続する。
- アプリケーションのビルドと実行。オプションで、基本的なCRUD操作に関する[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意：**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件

このチュートリアルを完了するためには、次のものが必要です。

- **Java開発キット（JDK）17**以上。ビジネスおよび個人の要件に応じて、[OpenJDK](https://openjdk.org/)または[Oracle JDK](https://www.oracle.com/hk/java/technologies/downloads/)を選択できます。
- [Maven](https://maven.apache.org/install.html) **3.8**以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次の手順に従って作成できます。**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)や[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次の手順に従って作成できます。**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)や[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/tidb-samples/tidb-java-jdbc-quickstart.git
cd tidb-java-jdbc-quickstart
```

### ステップ2: 接続情報を構成する

TiDBのデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境と一致することを確認します。

    - **Endpoint Type**が`Public`に設定されていること
    - **Connect With**が`General`に設定されていること
    - **Operating System**が環境に一致していること

    > **ヒント：**
    >
    > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えます。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント：**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために**Reset password**をクリックします。

5. 次のコマンドを実行して`env.sh.example`をコピーし、`env.sh`に名前を変更します。

    ```shell
    cp env.sh.example env.sh
    ```

6. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例として次のようになります。

    ```shell
    export TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: xxxxxx.root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='true'
    ```

    プレースホルダ `{}` を接続ダイアログから取得した接続パラメータで置き換えてください。

    TiDB Serverlessではセキュアな接続が必要です。そのため、`USE_SSL`の値を`true`に設定する必要があります。

7. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、その後**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`env.sh.example`をコピーし、`env.sh`に名前を変更します。

    ```shell
    cp env.sh.example env.sh
    ```

5. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例として次のようになります。

    ```shell
    export TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    プレースホルダ `{}` を接続ダイアログから取得した接続パラメータで置き換えてください。

6. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`env.sh.example`をコピーし、`env.sh`に名前を変更します。

    ```shell
    cp env.sh.example env.sh
    ```

2. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例として次のようになります。

    ```shell
    export TIDB_HOST='{host}'
    export TIDB_PORT='4000'
    export TIDB_USER='root'
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    プレースホルダ `{}` を接続パラメータに置き換えてください。`USE_SSL`は`false`に設定し、TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `env.sh`ファイルを保存します。

</div>
</SimpleTab>

### ステップ3: コードを実行して結果を確認する

1. 次のコマンドを実行してサンプルコードを実行します。

    ```shell
    make
    ```

2. [Expected-Output.txt](https://github.com/tidb-samples/tidb-java-jdbc-quickstart/blob/main/Expected-Output.txt)を確認し、出力が一致するかどうかを確認します。

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を行うことができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-java-jdbc-quickstart](https://github.com/tidb-samples/tidb-java-jdbc-quickstart)リポジトリをご覧ください。

### TiDBに接続

```java
public MysqlDataSource getMysqlDataSource() throws SQLException {
    MysqlDataSource mysqlDataSource = new MysqlDataSource();

    mysqlDataSource.setServerName(${tidb_host});
    mysqlDataSource.setPortNumber(${tidb_port});
    mysqlDataSource.setUser(${tidb_user});
    mysqlDataSource.setPassword(${tidb_password});
    mysqlDataSource.setDatabaseName(${tidb_db_name});
    if (${tidb_use_ssl}) {
        mysqlDataSource.setSslMode(PropertyDefinitions.SslMode.VERIFY_IDENTITY.name());
        mysqlDataSource.setEnabledTLSProtocols("TLSv1.2,TLSv1.3");
    }

    return mysqlDataSource;
}
```

この関数を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`をTiDBクラスタの実際の値に置き換える必要があります。

### データの挿入

```java
public void createPlayer(PlayerBean player) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSource();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement preparedStatement = connection.prepareStatement("INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)");
        preparedStatement.setString(1, player.getId());
```java
        preparedStatement.setInt(2, player.getCoins());
        preparedStatement.setInt(3, player.getGoods());

        preparedStatement.execute();
    }
}
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

```java
public void getPlayer(String id) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSourceByEnv();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM player WHERE id = ?");
        preparedStatement.setString(1, id);
        preparedStatement.execute();

        ResultSet res = preparedStatement.executeQuery();
        if(res.next()) {
            PlayerBean player = new PlayerBean(res.getString("id"), res.getInt("coins"), res.getInt("goods"));
            System.out.println(player);
        }
    }
}
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

```java
public void updatePlayer(String id, int amount, int price) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSourceByEnv();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement transfer = connection.prepareStatement("UPDATE player SET goods = goods + ?, coins = coins + ? WHERE id=?");
        transfer.setInt(1, -amount);
        transfer.setInt(2, price);
        transfer.setString(3, id);
        transfer.execute();
    }
}
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

```java
public void deletePlayer(String id) throws SQLException {
    MysqlDataSource mysqlDataSource = getMysqlDataSourceByEnv();
    try (Connection connection = mysqlDataSource.getConnection()) {
        PreparedStatement deleteStatement = connection.prepareStatement("DELETE FROM player WHERE id=?");
        deleteStatement.setString(1, id);
        deleteStatement.execute();
    }
}
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利な注意事項

### ドライバーを使用するか、ORMフレームワークを使用するか？

Javaドライバーはデータベースへの低レベルアクセスを提供しますが、開発者が次のことを手動で行う必要があります。

- データベース接続の手動確立および解除
- データベーストランザクションの手動管理
- データ行をデータオブジェクトに手動でマッピング

複雑なSQL文を書く必要がない場合は、[Hibernate](/develop/dev-guide-sample-application-java-hibernate.md)、[MyBatis](/develop/dev-guide-sample-application-java-mybatis.md)、または[Spring Data JPA](/develop/dev-guide-sample-application-java-spring-boot.md)などの[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワークを使用することをお勧めします。これにより、次のことがでいます。

- 接続およびトランザクションの管理に対する[ボイラープレートコード](https://en.wikipedia.org/wiki/Boilerplate_code)の削減
- 一連のSQL文ではなくデータオブジェクトでデータを操作

## 次のステップ

- [MySQL Connector/Jのドキュメント](https://dev.mysql.com/doc/connector-j/en/)でMySQL Connector/Jのより詳しい使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章でTiDBアプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)など。
- [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。
- Java開発者のためのコース：[JavaからTiDBと連携する](https://eng.edu.pingcap.com/catalog/info/id:212)を受講して学びます。

## 助けが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>
```