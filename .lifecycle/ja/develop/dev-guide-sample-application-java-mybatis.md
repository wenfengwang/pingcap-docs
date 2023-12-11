---
title: MyBatisを使用してTiDBに接続する
summary: MyBatisを使用してTiDBに接続する方法を学びます。このチュートリアルでは、TiDBをMyBatisと連携させるためのJavaサンプルコードを提供します。
---

# MyBatisを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[MyBatis](https://mybatis.org/mybatis-3/index.html)は人気のあるオープンソースのJava ORMです。

このチュートリアルでは、TiDBとMyBatisを使用して次のタスクを実行する方法を学ぶことができます。

- 環境の設定
- MyBatisを使用してTiDBクラスタに接続する
- アプリケーションのビルドと実行。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携できます。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- **Java開発キット（JDK）17** 以上。ビジネスおよび個人の要件に基づいて、[OpenJDK](https://openjdk.java.net/)または[Oracle JDK](https://www.oracle.com/java/technologies/downloads/)を選択できます。
- [Maven](https://maven.apache.org/install.html) **3.8** 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合、以下のように作成できます。**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合、以下のように作成できます。**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を説明します。

### ステップ1: サンプルアプリのリポジトリをクローンする

ターミナルウィンドウで以下のコマンドを実行して、サンプルコードのリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-java-mybatis-quickstart.git
cd tidb-java-mybatis-quickstart
```

### ステップ2: 接続情報を構成する

TiDBデプロイオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成がオペレーティング環境に一致することを確認します。

    - **エンドポイントタイプ**が`Public`に設定されていること
    - **Connect With**が`General`に設定されていること
    - **オペレーティングシステム**が環境に一致していること

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えます。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するには**Reset password**をクリックします。

5. 以下のコマンドを実行して、`env.sh.example`をコピーして`env.sh`に名前を変更します:

    ```shell
    cp env.sh.example env.sh
    ```

6. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例の結果は以下のとおりです:

    ```shell
    export TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: xxxxxx.root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='true'
    ```

    `{}`のプレースホルダを接続ダイアログから取得した接続パラメータで置き換えてください。

    TiDB Serverlessでは安全な接続が必要なので、`USE_SSL`の値を`true`に設定する必要があります。

7. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可**をクリックし、その後**TiDBクラスタCAのダウンロード**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 以下のコマンドを実行して、`env.sh.example`をコピーして`env.sh`に名前を変更します:

    ```shell
    cp env.sh.example env.sh
    ```

5. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例の結果は以下のとおりです:

    ```shell
    export TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    export TIDB_PORT='4000'
    export TIDB_USER='{user}'  # 例: root
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    `{}`のプレースホルダを接続ダイアログから取得した接続パラメータで置き換えてください。

6. `env.sh`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 以下のコマンドを実行して、`env.sh.example`をコピーして`env.sh`に名前を変更します:

    ```shell
    cp env.sh.example env.sh
    ```

2. 対応する接続文字列を`env.sh`ファイルにコピーして貼り付けます。例の結果は以下のとおりです:

    ```shell
    export TIDB_HOST='{host}'
    export TIDB_PORT='4000'
    export TIDB_USER='root'
    export TIDB_PASSWORD='{password}'
    export TIDB_DB_NAME='test'
    export USE_SSL='false'
    ```

    接続パラメータをプレースホルダ`{}`で置き換えてください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `env.sh`ファイルを保存します。

</div>
</SimpleTab>

### ステップ3: コードを実行して結果を確認する

1. 以下のコマンドを実行して、サンプルコードを実行します:

    ```shell
    make
    ```

2. 出力が一致しているかどうかを確認するために、[Expected-Output.txt](https://github.com/tidb-samples/tidb-java-mybatis-quickstart/blob/main/Expected-Output.txt)をチェックします。

## サンプルコードスニペット

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-java-mybatis-quickstart](https://github.com/tidb-samples/tidb-java-mybatis-quickstart)リポジトリをチェックしてください。

### TiDBに接続する

MyBatisの構成ファイル`mybatis-config.xml`を編集します:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
        <setting name="lazyLoadingEnabled" value="false"/>
        <setting name="aggressiveLazyLoading" value="true"/>
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <environments default="development">
        <environment id="development">
            <!-- JDBCトランザクションマネージャー -->
            <transactionManager type="JDBC"/>
            <!-- データベースプール -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="${tidb_jdbc_url}"/>
                <property name="username" value="${tidb_user}"/>
                <property name="password" value="${tidb_password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
```xml
        <mapper resource="${mapper_location}.xml"/>
    </mappers>
</configuration>
```
`${tidb_jdbc_url}`, `${tidb_user}`,`${tidb_password}`をTiDBクラスターの実際の値に置き換えてください。また、`${mapper_location}`をマッパーのXML構成ファイルのパスに置き換えてください。複数のマッパーXML構成ファイルの場合は、各々に`<mapper/>`タグを追加する必要があります。そして、次の関数を定義してください:

```java
public SqlSessionFactory getSessionFactory() {
    InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
}
```

### データの挿入

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加してください:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <insert id="insert" parameterType="com.pingcap.model.Player">
    insert into player (id, coins, goods)
    values (#{id,jdbcType=VARCHAR}, #{coins,jdbcType=INTEGER}, #{goods,jdbcType=INTEGER})
    </insert>
</mapper>
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加してください。特に、MyBatisクエリ関数の戻り値として`resultMap`を使用する場合は、`<resultMap/>`ノードが正しく構成されていることを確認してください。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <resultMap id="BaseResultMap" type="com.pingcap.model.Player">
        <constructor>
            <idArg column="id" javaType="java.lang.String" jdbcType="VARCHAR" />
            <arg column="coins" javaType="java.lang.Integer" jdbcType="INTEGER" />
            <arg column="goods" javaType="java.lang.Integer" jdbcType="INTEGER" />
        </constructor>
    </resultMap>

    <select id="selectByPrimaryKey" parameterType="java.lang.String" resultMap="BaseResultMap">
    select id, coins, goods
    from player
    where id = #{id,jdbcType=VARCHAR}
    </select>
</mapper>
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加してください:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <update id="updateByPrimaryKey" parameterType="com.pingcap.model.Player">
    update player
    set coins = #{coins,jdbcType=INTEGER},
      goods = #{goods,jdbcType=INTEGER}
    where id = #{id,jdbcType=VARCHAR}
    </update>
</mapper>
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

マッパーXMLにノードを追加し、XML構成ファイルの`mapper.namespace`属性で構成されたインターフェースクラスに同じ名前の関数を追加してください:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.pingcap.model.PlayerMapper">
    <delete id="deleteByPrimaryKey" parameterType="java.lang.String">
    delete from player
    where id = #{id,jdbcType=VARCHAR}
    </delete>
</mapper>
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ

- [MyBatisのドキュメント](http://www.mybatis.org/mybatis-3/)からMyBatisのさらなる使用法を学んでください。
- [開発者ガイド](/develop/dev-guide-overview.md)の各チャプターからTiDBアプリケーション開発のベストプラクティスを学んでください。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などを学んでください。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学んで、試験に合格した後に[TiDBの認定](https://www.pingcap.com/education/certification/)を取得してください。
- Java開発者向けのコースを通じて学んでください: [JavaでTiDBと連携する](https://eng.edu.pingcap.com/catalog/info/id:212)。

## ヘルプが必要ですか?

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問をするか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問をするか、[サポートチケット](https://support.pingcap.com/)を作成してください。

</CustomContent>
```