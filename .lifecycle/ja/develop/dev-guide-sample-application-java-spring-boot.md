---
title: Spring Bootを使用してTiDBに接続する
summary: Spring Bootを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Spring Bootを使用してTiDBと共にSpring Data JPAおよびHibernateを使用して以下のタスクを実行するためのJavaサンプルコードスニペットが提供されます。
aliases: ['/tidbcloud/dev-guide-sample-application-spring-boot','/tidb/dev/dev-guide-sample-application-spring-boot']
---

# Spring Bootを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[Spring](https://spring.io/)はJava向けの人気のあるオープンソースのコンテナフレームワークです。このドキュメントでは、Springの使用方法として[Spring Boot](https://spring.io/projects/spring-boot)を使用します。

このチュートリアルでは、次のタスクを実行するためにTiDBを使用する方法について学ぶことができます。

- 環境の設定。
- HibernateとSpring Data JPAを使用してTiDBクラスタに接続する。
- アプリケーションのビルドおよび実行。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- **Java Development Kit (JDK) 17** またはそれ以上。[OpenJDK](https://openjdk.org/) または[Oracle JDK](https://www.oracle.com/hk/java/technologies/downloads/)をビジネスおよび個人の要件に基づいて選択できます。
- [Maven](https://maven.apache.org/install.html) **3.8** またはそれ以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次の手順に従って作成できます。**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従ってローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次の手順に従って作成できます。**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従ってローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリリポジトリをクローンする

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/tidb-samples/tidb-java-springboot-jpa-quickstart.git
cd tidb-java-springboot-jpa-quickstart
```

### ステップ2: 接続情報を構成する

TiDBのデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が環境に合致することを確認します。

    - **エンドポイントタイプ**が`Public`に設定されていること
    - **Connect With**が`General`に設定されていること
    - **オペレーティングシステム**が環境に合致すること

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux (WSL)で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成している場合は、元のパスワードを使用するか、**Reset password**をクリックして新しいパスワードを生成することができます。

5. 次のコマンドを実行して`env.sh.example`をコピーし、`env.sh`に名前を変更します：

    ```shell
    cp env.sh.example env.sh
    ```

6. `env.sh`ファイルに対応する接続文字列をコピーして貼り付けます。次の例は、その結果です：

    ```shell
    export TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: xxxxxx.root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='true'
    ```

    `{}`のプレースホルダを接続ダイアログから取得した接続パラメータに置き換えてください。

    TiDB Serverlessでは安全な接続が必要です。そのため、`USE_SSL`の値を`true`に設定する必要があります。

7. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、次に**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`env.sh.example`をコピーし、`env.sh`に名前を変更します：

    ```shell
    cp env.sh.example env.sh
    ```

5. `env.sh`ファイルに対忲する接続文字列をコピーして貼り付けます。次の例は、その結果です：

    ```shell
    export TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    `{}`のプレースホルダを接続ダイアログから取得した接続パラメータに置き換えてください。

6. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`env.sh.example`をコピーし、`env.sh`に名前を変更します：

    ```shell
    cp env.sh.example env.sh
    ```

2. `env.sh`ファイルに対応する接続文字列をコピーして貼り付けます。次の例は、その結果です：

    ```shell
    export TIDB_HOST='{host}'
    export TIDB_PORT='4000'
    export TIDB_USER='root'
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    `{}`のプレースホルダを接続パラメータに置き換えてください。また、`USE_SSL`を`false`に設定します。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `env.sh`ファイルを保存します。

</div>
</SimpleTab>

### ステップ3: コードを実行して結果を確認する

1. 次のコマンドを実行してサンプルコードを実行します：

    ```shell
    make
    ```

2. 別のターミナルセッションでリクエストスクリプトを実行します：

    ```shell
    make request
    ```

3. [Expected-Output.txt](https://github.com/tidb-samples/tidb-java-springboot-jpa-quickstart/blob/main/Expected-Output.txt)を確認して、出力が一致するか確認します。

## サンプルコードスニペット

次のサンプルコードスニペットを参考にして、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-java-springboot-jpa-quickstart](https://github.com/tidb-samples/tidb-java-springboot-jpa-quickstart)リポジトリを確認してください。

### TiDBに接続する

構成ファイル`application.yml`を編集します：

```yaml
spring:
  datasource:
    url: ${TIDB_JDBC_URL:jdbc:mysql://localhost:4000/test}
    username: ${TIDB_USER:root}
    password: ${TIDB_PASSWORD:}
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    show-sql: true
    database-platform: org.hibernate.dialect.TiDBDialect
    hibernate:
      ddl-auto: create-drop
```

構成後、TiDBクラスターの`TIDB_JDBC_URL`、`TIDB_USER`、`TIDB_PASSWORD`環境変数を実際の値に設定します。構成ファイルにはこれらの環境変数のデフォルト設定が用意されています。環境変数を構成しない場合、デフォルト値は次の通りです:

- `TIDB_JDBC_URL`: `"jdbc:mysql://localhost:4000/test"`
- `TIDB_USER`: `"root"`
- `TIDB_PASSWORD`: `""`

### データ管理: `@Repository`

Spring Data JPAは`@Repository`インターフェースを介してデータを管理します。`JpaRepository`が提供するCRUD操作を使用するには、`JpaRepository`インターフェースを拡張する必要があります:

```java
@Repository
public interface PlayerRepository extends JpaRepository<PlayerBean, Long> {
}
```

その後、`PlayerRepository`が必要な任意のクラスで自動依存性注入を行うために`@Autowired`を使用できます。これによりCRUD関数を直接使用できます。以下は例です:

```java
@Autowired
private PlayerRepository playerRepository;
```

### データの挿入または更新

```java
playerRepository.save(player);
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md) と[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データのクエリ

```java
PlayerBean player = playerRepository.findById(id).orElse(null);
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの削除

```java
playerRepository.deleteById(id);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ

- [Hibernateのドキュメント](https://hibernate.org/orm/documentation)からHibernateの使用方法を学びます。
- 本文書で使用されているサードパーティ製ライブラリおよびフレームワークに関する詳細な使用方法については、以下の公式ドキュメントを参照してください:
    - [Spring Frameworkのドキュメント](https://spring.io/projects/spring-framework)
    - [Spring Bootのドキュメント](https://spring.io/projects/spring-boot)
    - [Spring Data JPAのドキュメント](https://spring.io/projects/spring-data-jpa)
    - [Hibernateのドキュメント](https://hibernate.org/orm/documentation)
- [開発者ガイド](/develop/dev-guide-overview.md)の章を使用して、TiDBアプリケーション開発のベストプラクティスについて学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)
- [専門のTiDB開発者コース](https://www.pingcap.com/education/)を受講し、試験に合格すると[TiDBの認定](https://www.pingcap.com/education/certification/)を取得します。
- [Java開発者向けコース](https://eng.edu.pingcap.com/catalog/info/id:212):『JavaからTiDBと連携』のコースで学びます。

## 必要なヘルプがありますか？

<CustomContent platform="tidb">

[discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>