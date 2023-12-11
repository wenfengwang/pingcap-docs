---
title: ドライバーまたはORMの選択
summary: TiDBに接続するためのドライバーまたはORMフレームワークを選択する方法について学びます。

# ドライバーまたはORMの選択

> **注意:**
>
> TiDBでは、以下の2つのサポートレベルがドライバーおよびORMに提供されます:
>
> - **フル (Full)**: TiDBはツールのほとんどの機能と互換性があり、その新しいバージョンとの互換性を維持しています。PingCAPは定期的に[TiDBでサポートされるサードパーティーツール](/develop/dev-guide-third-party-support.md)の最新バージョンとの互換性テストを実施します。
> - **互換性 (Compatible)**: 対応するサードパーティーツールがMySQLに適合しており、TiDBがMySQLプロトコルと高い互換性があるため、TiDBは多くのツールの機能を使用できます。ただし、PingCAPはツールのすべての機能について完全なテストを行っていないため、予期しない動作が発生する可能性があります。
>
> 詳細については、[TiDBでサポートされるサードパーティーツール](/develop/dev-guide-third-party-support.md)を参照してください。

TiDBはMySQLプロトコルと高い互換性がありますが、一部の機能がMySQLと互換性がありません。互換性の違いの完全なリストについては、[MySQLの互換性](/mysql-compatibility.md)を参照してください。

## Java

このセクションでは、JavaでのドライバーやORMフレームワークの使用方法について説明します。

### Javaドライバー

<SimpleTab>
<div label="MySQL-JDBC">

サポートレベル: **フル (Full)**

[MySQLドキュメント](https://dev.mysql.com/doc/connector-j/en/)に従って、Java JDBCドライバーをダウンロードして構成することができます。TiDB v6.3.0およびそれ以降では、MySQL Connector/J 8.0.33またはそれ以降のバージョンを使用することをお勧めします。

> **ヒント:**
>
> Connector/J 8.0.32より前のバージョンで、TiDB v6.3.0より前のバージョンを使用すると、スレッドがハングする可能性がある[バグ](https://bugs.mysql.com/bug.php?id=106252)があります。この問題を回避するためには、MySQL Connector/J 8.0.32またはそれ以降のバージョン、またはTiDB JDBC（*TiDB-JDBC*タブを参照）を使用することをお勧めします。

完全なアプリケーションの構築例については、[TiDBおよびJDBCでシンプルなCRUDアプリケーションを構築](/develop/dev-guide-sample-application-java-jdbc.md)を参照してください。

</div>
<div label="TiDB-JDBC">

サポートレベル: **フル (Full)**

[TiDB-JDBC](https://github.com/pingcap/mysql-connector-j)は、MySQL 8.0.29を基にしたカスタマイズされたJavaドライバーです。MySQL公式版8.0.29を基にコンパイルされたTiDB-JDBCは、元のJDBCの準備モードでの複数パラメータと複数フィールドEOFのバグを修正し、TiCDCのスナップショットの自動メンテナンスやSM3認証プラグインなどの機能を追加しています。

SM3認証に基づく認証は、TiDBのTiDB-JDBCでのみサポートされています。

Mavenを使用する場合は、`pom.xml`ファイルの`<dependencies></dependencies>`セクションに以下の内容を追加します:

```xml
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.29-tidb-1.0.0</version>
</dependency>
```

SM3認証を有効にする必要がある場合は、`pom.xml`ファイルの`<dependencies></dependencies>`セクションに以下の内容を追加します:

```xml
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.29-tidb-1.0.0</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.67</version>
</dependency>
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcpkix-jdk15on</artifactId>
    <version>1.67</version>
</dependency>
```

Gradleを使用する場合は、`dependencies`に以下の内容を追加します:

```gradle
implementation group: 'io.github.lastincisor', name: 'mysql-connector-java', version: '8.0.29-tidb-1.0.0'
implementation group: 'org.bouncycastle', name: 'bcprov-jdk15on', version: '1.67'
implementation group: 'org.bouncycastle', name: 'bcpkix-jdk15on', version: '1.67'
```

</div>
</SimpleTab>

### Java ORMフレームワーク

> **注意:**
>
> - 現在、Hibernateは[ネストされたトランザクションをサポートしていません](https://stackoverflow.com/questions/37927208/nested-transaction-in-spring-app-with-jpa-postgres)。
>
> - v6.2.0以降、TiDBは[セーブポイント](/sql-statements/sql-statement-savepoint.md)をサポートしています。`@Transactional`で`Propagation.NESTED`トランザクション伝播オプションを使用する場合、つまり、`@Transactional(propagation = Propagation.NESTED)`を設定する場合は、TiDBがv6.2.0以降であることを確認してください。

<SimpleTab>
<div label="Hibernate">

サポートレベル: **フル (Full)**

アプリケーションのさまざまな依存関係を手動で管理する必要がなくなるように、[Gradle](https://gradle.org/install)や[Maven](https://maven.apache.org/install.html)を使用して、アプリケーションのすべての依存関係（間接的なものも含む）を取得できます。なお、TiDBの方言をサポートしているのは`6.0.0.Beta2`以上のHibernateのみです。

**Maven**を使用している場合、以下を`<dependencies></dependencies>`に追加してください:

```xml
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.0.0.CR2</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.49</version>
</dependency>
```

**Gradle**を使用している場合、以下を`dependencies`に追加してください:

```gradle
implementation 'org.hibernate:hibernate-core:6.0.0.CR2'
implementation 'mysql:mysql-connector-java:5.1.49'
```

- ネイティブJavaでHibernateを使用してTiDBアプリケーションを構築する例については、[TiDBおよびHibernateでシンプルなCRUDアプリケーションを構築](/develop/dev-guide-sample-application-java-hibernate.md)を参照してください。
- Spring Data JPAまたはHibernateを使用してSpringでTiDBアプリケーションを構築する例については、[Spring Bootを使用してTiDBアプリケーションを構築](/develop/dev-guide-sample-application-java-spring-boot.md)を参照してください。

さらに、[Hibernateの構成ファイル](https://www.tutorialspoint.com/hibernate/hibernate_configuration.htm)でTiDBの方言を指定する必要があります: `org.hibernate.dialect.TiDBDialect`、これはHibernate `6.0.0.Beta2`以上でのみサポートされています。`Hibernate`のバージョンが`6.0.0.Beta2`より前の場合は、まずアップグレードしてください。

> **注意:**
>
> `Hibernate`のバージョンをアップグレードできない場合は、代わりにMySQL 5.7の方言`org.hibernate.dialect.MySQL57Dialect`を使用してください。ただし、この設定は予測不可能な結果や[シーケンス](/sql-statements/sql-statement-create-sequence.md)など、一部のTiDB固有の機能の不在を引き起こす可能性があります。

</div>
<div label="MyBatis">

サポートレベル: **フル (Full)**

アプリケーションのさまざまな依存関係を手動で管理する必要がなくなるように、[Gradle](https://gradle.org/install)や[Maven](https://maven.apache.org/install.html)を使用して、アプリケーションのすべての依存関係（間接的なものも含む）を取得できます。

Mavenを使用している場合、以下を`<dependencies></dependencies>`に追加してください:

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.9</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.49</version>
</dependency>
```

Gradleを使用している場合、以下を`dependencies`に追加してください:

```gradle
implementation 'org.mybatis:mybatis:3.5.9'
implementation 'mysql:mysql-connector-java:5.1.49'
```

TiDBアプリケーションを構築するためのMyBatisの使用例については、[TiDBおよびMyBatisでシンプルなCRUDアプリケーションを構築](/develop/dev-guide-sample-application-java-mybatis.md)を参照してください。

</div>

</SimpleTab>

### Javaクライアントの負荷分散

**tidb-loadbalance**

サポートレベル: **フル (Full)**

[tidb-loadbalance](https://github.com/pingcap/tidb-loadbalance)はアプリケーション側の負荷分散コンポーネントです。tidb-loadbalanceを使用すると、TiDBサーバーのノード情報を自動的に維持し、tidb-loadbalanceポリシーを使用してクライアントでJDBC接続を自動的にバランスさせることができます。クライアントアプリケーションとTiDBサーバー間で直接JDBC接続を使用する場合、負荷分散コンポーネントを使用するよりも高いパフォーマンスが得られます。

現在、tidb-loadbalanceは以下のポリシーをサポートしています: ラウンドロビン、ランダム、およびウェイト。

> **注意:**
>
```markdown
> tidb-loadbalance must be used with [mysql-connector-j](https://github.com/pingcap/mysql-connector-j).

Mavenを使用する場合は、`pom.xml`ファイルの`<dependencies></dependencies>`要素の本文に次の内容を追加します：

```xml
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.29-tidb-1.0.0</version>
</dependency>
<dependency>
  <groupId>io.github.lastincisor</groupId>
  <artifactId>tidb-loadbalance</artifactId>
  <version>0.0.5</version>
</dependency>
```

Gradleを使用する場合は、`dependencies`に次の内容を追加します：

```gradle
implementation group: 'io.github.lastincisor', name: 'mysql-connector-java', version: '8.0.29-tidb-1.0.0'
implementation group: 'io.github.lastincisor', name: 'tidb-loadbalance', version: '0.0.5'
```

## Golang

このセクションでは、GolangでのドライバとORMフレームワークの使用方法について説明します。

### Golangのドライバ

**go-sql-driver/mysql**

サポートレベル：**フル**

Golangドライバをダウンロードして構成する方法については、[go-sql-driver/mysqlのドキュメント](https://github.com/go-sql-driver/mysql)を参照してください。

完全なアプリケーションの構築例については、[Go-MySQL-Driverを使用してTiDBに接続する](/develop/dev-guide-sample-application-golang-sql-driver.md)を参照してください。

### GolangのORMフレームワーク

**GORM**

サポートレベル：**フル**

GORMはGolangの人気のあるORMフレームワークです。アプリケーションですべての依存関係を取得するには、`go get`コマンドを使用できます。

```shell
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```

GORMを使用してTiDBアプリケーションを構築する例については、[GORMを使用してTiDBに接続する](/develop/dev-guide-sample-application-golang-gorm.md)を参照してください。

## Python

このセクションでは、PythonでのドライバとORMフレームワークの使用方法について説明します。

### Pythonのドライバ

<SimpleTab>
<div label="PyMySQL">

サポートレベル：**互換性**

ドライバのダウンロードと構成については、[PyMySQLのドキュメント](https://pypi.org/project/PyMySQL/)に従ってください。PyMySQL 1.0.2以降のバージョンを使用することをお勧めします。

PyMySQLを使用してTiDBアプリケーションを構築する例については、[PyMySQLを使用してTiDBに接続する](/develop/dev-guide-sample-application-python-pymysql.md)を参照してください。

</div>
<div label="mysqlclient">

サポートレベル：**互換性**

ドライバのダウンロードと構成については、[mysqlclientのドキュメント](https://pypi.org/project/mysqlclient/)に従ってください。mysqlclient 2.1.1以降のバージョンを使用することをお勧めします。

mysqlclientを使用してTiDBアプリケーションを構築する例については、[mysqlclientを使用してTiDBに接続する](/develop/dev-guide-sample-application-python-mysqlclient.md)を参照してください。

</div>
<div label="MySQL Connector/Python">

サポートレベル：**互換性**

ドライバのダウンロードと構成については、[MySQL Connector/Pythonのドキュメント](https://dev.mysql.com/doc/connector-python/en/connector-python-installation-binary.html)に従ってください。Connector/Python 8.0.31以降のバージョンを使用することをお勧めします。

MySQL Connector/Pythonを使用してTiDBアプリケーションを構築する例については、[MySQL Connector/Pythonを使用してTiDBに接続する](/develop/dev-guide-sample-application-python-mysql-connector.md)を参照してください。

</div>
</SimpleTab>

### PythonのORMフレームワーク

<SimpleTab>
<div label="Django">

サポートレベル：**フル**

[Django](https://docs.djangoproject.com/)は人気のあるPythonウェブフレームワークです。TiDBとDjangoの互換性の問題を解決するために、PingCAPはTiDBのダイアレクト `django-tidb`を提供しています。これをインストールするには、[`django-tidb`のドキュメント](https://github.com/pingcap/django-tidb#installation-guide)を参照してください。

Djangoを使用してTiDBアプリケーションを構築する例については、[Djangoを使用してTiDBに接続する](/develop/dev-guide-sample-application-python-django.md)を参照してください。

</div>
<div label="SQLAlchemy">

サポートレベル：**フル**

[SQLAlchemy](https://www.sqlalchemy.org/)はPythonの人気のあるORMフレームワークです。アプリケーションですべての依存関係を取得するには、`pip install SQLAlchemy==1.4.44`コマンドを使用できます。SQLAlchemy 1.4.44以降のバージョンを使用することをお勧めします。

SQLAlchemyを使用してTiDBアプリケーションを構築する例については、[SQLAlchemyを使用してTiDBに接続する](/develop/dev-guide-sample-application-python-sqlalchemy.md)を参照してください。

</div>
<div label="peewee">

サポートレベル：**互換性**

[peewee](http://docs.peewee-orm.com/en/latest/)はPythonの人気のあるORMフレームワークです。アプリケーションですべての依存関係を取得するには、`pip install peewee==3.15.4`コマンドを使用できます。peewee 3.15.4以降のバージョンを使用することをお勧めします。

peeweeを使用してTiDBアプリケーションを構築する例については、[peeweeを使用してTiDBに接続する](/develop/dev-guide-sample-application-python-peewee.md)を参照してください。

</div>
</SimpleTab>

<CustomContent platform="tidb-cloud">

ドライバまたはORMを決定したら、[TiDBクラスタに接続](https://docs.pingcap.com/tidbcloud/connect-to-tidb-cluster)できます。

</CustomContent>
```