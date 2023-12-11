---
title: ハイバネートを使用して TiDB に接続する
summary: Hibernateを使用してTiDBに接続する方法を学びます。このチュートリアルでは、TiDBをHibernateと組み合わせて使用するためのJavaのサンプルコードスニペットが紹介されています。
---

# ハイバネートを使用して TiDB に接続する

TiDB は MySQL 互換のデータベースであり、[Hibernate](https://hibernate.org/orm/) は人気のあるオープンソースの Java ORM です。バージョン `6.0.0.Beta2` 以降、Hibernate は TiDB ダイアレクトをサポートしており、これは TiDB の特徴に適しています。

このチュートリアルでは、次のタスクを実行するための TiDB と Hibernate の使用方法を学ぶことができます:

- 環境のセットアップ。
- Hibernate を使用して TiDB クラスタに接続。
- アプリケーションをビルドして実行する。オプションで、基本的な CRUD 操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB サーバーレス、TiDB 専用、および TiDB セルフホストと共に動作します。

## 前提条件

このチュートリアルを完了するには、以下が必要です:

- **Java 開発キット (JDK) 17** 以上。[OpenJDK](https://openjdk.java.net/) または [Oracle JDK](https://www.oracle.com/java/technologies/downloads/) のいずれかを、ビジネスおよび個人の要件に基づいて選択できます。
- [Maven](https://maven.apache.org/install.html) バージョン **3.8** 以上。
- [Git](https://git-scm.com/downloads)。
- TiDB クラスタ。

<CustomContent platform="tidb">

**TiDB クラスタをお持ちでない場合は、次の手順に従って作成できます:**

- (お勧め) [TiDB Serverless クラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md) を参照して、独自の TiDB クラウド クラスタを作成します。
- ローカル クラスタを作成するには、[ローカル テスト TiDB クラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster) または [本番向け TiDB クラスタのデプロイ](/production-deployment-using-tiup.md) を参照してください。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDB クラスタをお持ちでない場合は、次の手順に従って作成できます:**

- (お勧め) [TiDB Serverless クラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md) を参照して、独自の TiDB クラウド クラスタを作成します。
- ローカル クラスタを作成するには、[ローカル テスト TiDB クラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) または [本番向け TiDB クラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) を参照してください。

</CustomContent>

## サンプルアプリを実行して TiDB に接続する

このセクションでは、サンプル アプリケーション コードを実行し、TiDB に接続する方法を示します。

### ステップ 1: サンプル アプリのリポジトリをクローンする

以下のコマンドをターミナル ウィンドウで実行して、サンプル コードのリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-java-hibernate-quickstart.git
cd tidb-java-hibernate-quickstart
```

### ステップ 2: 接続情報を構成する

TiDB のデプロイメント オプションに応じて、TiDB クラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が環境に合致することを確認します。

    - **Endpoint Type** が `Public` に設定されていること
    - **Connect With** が `General` に設定されていること
    - **Operating System** が環境に一致していること

    > **ヒント:**
    >
    > プログラムが Windows Subsystem for Linux (WSL) で実行されている場合は、対応する Linux ディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するには、 **Create password** をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合、元のパスワードを使用するか、新しいパスワードを生成するには **Reset password** をクリックできます。

5. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します:

    ```shell
    cp env.sh.example env.sh
    ```

6. `env.sh` ファイルに対応する接続文字列をコピーして貼り付けます。例を次に示します:

    ```shell
    export TIDB_HOST='{ホスト}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{ユーザー}'  # 例: xxxxxx.root
    export TIDB_PASSWORD='{パスワード}'
    export TIDB_DB_NAME='test'
    export USE_SSL='true'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

    TiDB サーバーレスでは安全な接続が必要です。そのため、`USE_SSL` の値を `true` に設定する必要があります。

7. `env.sh` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Anywhere からのアクセスを許可** をクリックし、次に **TiDB クラスタ CA のダウンロード** をクリックして CA 証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated 標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します:

    ```shell
    cp env.sh.example env.sh
    ```

5. `env.sh` ファイルに対応する接続文字列をコピーして貼り付けます。例を次に示します:

    ```shell
    export TIDB_HOST='{ホスト}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{ユーザー}'  # 例: root
    export TIDB_PASSWORD='{パスワード}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

6. `env.sh` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `env.sh.example` をコピーし、`env.sh` に名前を変更します:

    ```shell
    cp env.sh.example env.sh
    ```

2. `env.sh` ファイルに対応する接続文字列をコピーして貼り付けます。例を次に示します:

    ```shell
    export TIDB_HOST='{ホスト}'
    export TIDB_PORT='4000'
    export TIDB_USER='root'
    export TIDB_PASSWORD='{パスワード}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    接続パラメータに対応するプレースホルダ `{}` を置き換えてください。TiDB をローカルで実行している場合、デフォルトのホスト アドレスは `127.0.0.1` であり、パスワードは空です。

3. `env.sh` ファイルを保存します。

</div>
</SimpleTab>

### ステップ 3: コードを実行して結果を確認する

1. 次のコマンドを実行してサンプル コードを実行します:

    ```shell
    make
    ```

2. 出力が一致するかどうかを確認するには、[Expected-Output.txt](https://github.com/tidb-samples/tidb-java-hibernate-quickstart/blob/main/Expected-Output.txt) を確認してください。

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプル コードおよびその実行方法については、[tidb-samples/tidb-java-hibernate-quickstart](https://github.com/tidb-samples/tidb-java-hibernate-quickstart) リポジトリを参照してください。

### TiDB への接続

Hibernate 構成ファイル `hibernate.cfg.xml` を編集します:

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>

        <!-- データベース接続設定 -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.dialect">org.hibernate.dialect.TiDBDialect</property>
        <property name="hibernate.connection.url">${tidb_jdbc_url}</property>
        <property name="hibernate.connection.username">${tidb_user}</property>
        <property name="hibernate.connection.password">${tidb_password}</property>
        <property name="hibernate.connection.autocommit">false</property>

        <!-- 'PlayerDAO' クラスからテーブルを作成できるようにするために必要 -->
        <property name="hibernate.hbm2ddl.auto">create-drop</property>

```xml
<!-- オプション: デバッグ用のSQL出力を表示 -->
<property name="hibernate.show_sql">true</property>
<property name="hibernate.format_sql">true</property>
</session-factory>
</hibernate-configuration>
```

`${tidb_jdbc_url}`、`${tidb_user}`、`${tidb_password}`をTiDBクラスターの実際の値に置き換えてください。次に、次の関数を定義します：

```java
public SessionFactory getSessionFactory() {
    return new Configuration()
            .configure("hibernate.cfg.xml")
            .addAnnotatedClass(${your_entity_class})
            .buildSessionFactory();
}
```

この関数を使用する際には、`${your_entity_class}`を独自のデータエンティティクラスに置き換える必要があります。複数のエンティティクラスを使用する場合、各エンティティクラス用に`.addAnnotatedClass(${your_entity_class})`ステートメントを追加する必要があります。前述の関数はHibernateを構成するための1つの方法にすぎません。構成で問題が発生した場合やHibernateについてもっと学びたい場合は、[Hibernate公式ドキュメント](https://hibernate.org/orm/documentation)を参照してください。

### データの挿入または更新

```java
try (Session session = sessionFactory.openSession()) {
    session.persist(new PlayerBean("id", 1, 1));
}
```
詳細については、[データの挿入](/develop/dev-guide-insert-data.md)および[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データのクエリ

```java
try (Session session = sessionFactory.openSession()) {
    PlayerBean player = session.get(PlayerBean.class, "id");
    System.out.println(player);
}
```
詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの削除

```java
try (Session session = sessionFactory.openSession()) {
    session.remove(new PlayerBean("id", 1, 1));
}
```
詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ

- [Hibernateのドキュメント](https://hibernate.org/orm/documentation)からHibernateのさらなる使用方法について学びましょう。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びましょう。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)など。
- [PingCAPのプロフェッショナルTiDB開発者コース](https://www.pingcap.com/education/)を受講して、試験に合格すると[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得できます。
- Java開発者向けのコース：[JavaからTiDBを使用する方法](https://eng.edu.pingcap.com/catalog/info/id:212)を通じて学びましょう。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>
```