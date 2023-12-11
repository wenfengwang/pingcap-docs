---
title: TiDBへの接続
summary: TiDBへの接続方法について学びます。

# TiDBへの接続

TiDBはMySQLプロトコルと高い互換性があります。クライアントリンクパラメータの完全なリストについては、[MySQLクライアントオプション](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html)を参照してください。

TiDBは[MySQLクライアント/サーバープロトコル](https://dev.mysql.com/doc/dev/mysql-server/latest/PAGE_PROTOCOL.html)をサポートしており、ほとんどのクライアントドライバーやORMフレームワークがMySQLに接続する方法と同様にTiDBに接続できます。

## MySQL

個人の好みに応じて、MySQLクライアントまたはMySQL Shellを使用することができます。

<SimpleTab>

<div label="MySQLクライアント">

TiDBにはMySQLクライアントを使用して接続することができます。これはTiDBのためのコマンドラインツールとして使用できます。MySQLクライアントをインストールするには、YUMベースのLinuxディストリビューションに対して以下の手順に従ってください。

```shell
sudo yum install mysql
```

インストール後、以下のコマンドを使用してTiDBに接続できます。

```shell
mysql --host <tidb_server_host> --port 4000 -u root -p --comments
```

</div>

<div label="MySQLシェル">

TiDBにはMySQLシェルを使用して接続することができます。これはTiDBのためのコマンドラインツールとして使用できます。MySQLシェルをインストールするには、[MySQL Shellドキュメント](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install.html)の手順に従ってください。インストール後、以下のコマンドを使用してTiDBに接続できます。

```shell
mysqlsh --sql mysql://root@<tidb_server_host>:4000
```

</div>

</SimpleTab>

## JDBC

[JDBC](https://dev.mysql.com/doc/connector-j/en/)ドライバーを使用してTiDBに接続することができます。それには、`MysqlDataSource`または`MysqlConnectionPoolDataSource`オブジェクト（どちらのオブジェクトも`DataSource`インターフェースをサポートしています）を作成し、`setURL`関数を使用して接続文字列を設定する必要があります。

例：

```java
MysqlDataSource mysqlDataSource = new MysqlDataSource();
mysqlDataSource.setURL("jdbc:mysql://{host}:{port}/{database}?user={username}&password={password}");
```

JDBC接続に関する詳細については、[JDBCドキュメント](https://dev.mysql.com/doc/connector-j/en/)を参照してください。

### 接続パラメータ

| パラメータ名 | 説明 |
| :---: | :----------------------------: |
| `{username}` | TiDBクラスターに接続するためのSQLユーザー |
| `{password}` | SQLユーザーのパスワード |
| `{host}` | TiDBノードの[ホスト](https://en.wikipedia.org/wiki/Host_(network)) |
| `{port}` | TiDBノードがリッスンしているポート |
| `{database}` | 既存のデータベースの名前 |

<CustomContent platform="tidb">

TiDB SQLユーザーに関する詳細については、[TiDBユーザーアカウント管理](/user-account-management.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB SQLユーザーに関する詳細については、[TiDBユーザーアカウント管理](https://docs.pingcap.com/tidb/stable/user-account-management)を参照してください。

</CustomContent>

## Hibernate

[Hibernate ORM](https://hibernate.org/orm/)を使用してTiDBに接続することができます。そのためには、Hibernate構成ファイルの`hibernate.connection.url`を適切なTiDB接続文字列に設定する必要があります。

たとえば、`hibernate.cfg.xml`構成ファイルを使用する場合、`hibernate.connection.url`を以下のように設定します：

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.dialect">org.hibernate.dialect.TiDBDialect</property>
        <property name="hibernate.connection.url">jdbc:mysql://{host}:{port}/{database}?user={user}&amp;password={password}</property>
    </session-factory>
</hibernate-configuration>
```

構成が完了した後、以下のコマンドを使用して構成ファイルを読み込み、`SessionFactory`オブジェクトを取得できます：

```java
SessionFactory sessionFactory = new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();
```

注意点：

- `hibernate.cfg.xml`構成ファイルはXML形式であり、`&`はXMLで特殊な文字であるため、構成ファイルを設定する際に`&`を`&amp;`に変更する必要があります。たとえば、接続文字列`hibernate.connection.url`を`jdbc:mysql://{host}:{port}/{database}?user={user}&password={password}`から`jdbc:mysql://{host}:{ port}/{database}?user={user}&amp;password={password}`に変更する必要があります。
- `hibernate.dialect`を`org.hibernate.dialect.TiDBDialect`に設定することで`TiDB`方言を使用することを推奨します。
- Hibernateは`6.0.0.Beta2`以降からTiDBの方言をサポートしていますので、TiDBに接続する際にはHibernate `6.0.0.Beta2`またはそれ以降のバージョンを使用することをお勧めします。

Hibernate接続パラメータについての詳細については、[Hibernateドキュメント](https://hibernate.org/orm/documentation)を参照してください。

### 接続パラメータ

| パラメータ名 | 説明 |
| :---: | :----------------------------: |
| `{username}` |  TiDBクラスターに接続するためのSQLユーザー  |
| `{password}` | SQLユーザーのパスワード |
| `{host}` | TiDBノードの[ホスト](https://en.wikipedia.org/wiki/Host_(network)) |
| `{port}` | TiDBノードがリッスンしているポート |
| `{database}` |  既存のデータベースの名前  |

<CustomContent platform="tidb">

TiDB SQLユーザーに関する詳細については、[TiDBユーザーアカウント管理](/user-account-management.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB SQLユーザーに関する詳細については、[TiDBユーザーアカウント管理](https://docs.pingcap.com/tidb/stable/user-account-management)を参照してください。

</CustomContent>