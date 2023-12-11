---
title: TiDBでサポートされているサードパーティツール
summary: TiDBでサポートされているサードパーティツールについて学びましょう。

# TiDBでサポートされているサードパーティツール

> **注意:**
>
> このドキュメントは、TiDBでサポートされている一般的な[サードパーティツール](https://en.wikipedia.org/wiki/Third-party_source)のみをリストアップしています。他の一部のサードパーティツールはリストされていませんが、これはそれらがサポートされていないためではなく、PingCAPがそのツールがTiDBと互換性のない機能を使用しているかどうかを確信していないためです。

TiDBは[MySQLプロトコルと高い互換性](/mysql-compatibility.md)を持っているため、MySQLドライバ、ORMフレームワーク、およびMySQLに適応する他のツールのほとんどがTiDBと互換性があります。このドキュメントはこれらのツールとTiDBへのサポートレベルに焦点を当てています。

## サポートレベル

PingCAPはコミュニティと協力し、以下のサードパーティツールに対して次のサポートレベルを提供しています:

- **_フル_**: 対応するサードパーティツールのほとんどの機能との互換性があり、最新バージョンのツールとの互換性を維持しています。PingCAPは定期的に最新バージョンのツールとの互換テストを実施します。
- **_互換_**: 対応するサードパーティツールがMySQLに適応されており、TiDBはMySQLプロトコルと高い互換性を持っているため、そのツールの大部分の機能を使用できます。ただし、PingCAPはツールのすべての機能に対する完全なテストを行っておらず、予期せぬ動作が発生する可能性があります。

> **注意:**
>
> 指定されていない限り、**ドライバ**または**ORMフレームワーク**に対する[アプリケーションの再試行とエラーハンドリング](/develop/dev-guide-transaction-troubleshoot.md#application-retry-and-error-handling)のサポートは含まれていません。

このドキュメントにリストされているツールを使用してTiDBに接続する際に問題が発生した場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues/new?assignees=&labels=type%2Fquestion&template=general-question.md)を詳細を含めて提出して、このツールへのサポートを促進してください。

## ドライバ

<table>
   <thead>
      <tr>
         <th>言語</th>
         <th>ドライバ</th>
         <th>最新テストバージョン</th>
         <th>サポートレベル</th>
         <th>TiDBアダプタ</th>
         <th>チュートリアル</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Go</td>
         <td><a href="https://github.com/go-sql-driver/mysql" target="_blank" referrerpolicy="no-referrer-when-downgrade">Go-MySQL-Driver</a></td>
         <td>v1.6.0</td>
         <td>フル</td>
         <td>N/A</td>
         <td><a href="/tidb/dev/dev-guide-sample-application-golang-sql-driver">Go-MySQL-Driverを使用してTiDBに接続する</a></td>
      </tr>
      <tr>
         <td>Java</td>
         <td><a href="https://dev.mysql.com/downloads/connector/j/" target="_blank" referrerpolicy="no-referrer-when-downgrade">JDBC</a></td>
         <td>8.0</td>
         <td>フル</td>
         <td>
            <ul>
               <li><a href="/tidb/dev/dev-guide-choose-driver-or-orm#java-drivers" data-href="/tidb/dev/dev-guide-choose-driver-or-orm#java-drivers">pingcap/mysql-connector-j</a></li>
               <li><a href="/tidb/dev/dev-guide-choose-driver-or-orm#tidb-loadbalance" data-href="/tidb/dev/dev-guide-choose-driver-or-orm#tidb-loadbalance">pingcap/tidb-loadbalance</a></li>
            </ul>
         </td>
         <td><a href="/tidb/dev/dev-guide-sample-application-java-jdbc">JDBCを使用してTiDBに接続する</a></td>
      </tr>
   </tbody>
</table>

## ORM

<table>
   <thead>
      <tr>
         <th>言語</th>
         <th>ORMフレームワーク</th>
         <th>最新テストバージョン</th>
         <th>サポートレベル</th>
         <th>TiDBアダプタ</th>
         <th>チュートリアル</th>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td rowspan="4">Go</td>
         <td><a href="https://github.com/go-gorm/gorm" target="_blank" referrerpolicy="no-referrer-when-downgrade">gorm</a></td>
         <td>v1.23.5</td>
         <td>フル</td>
         <td>N/A</td>
         <td><a href="/tidb/dev/dev-guide-sample-application-golang-gorm">GORMを使用してTiDBに接続する</a></td>
      </tr>
      <tr>
         <td><a href="https://github.com/beego/beego" target="_blank" referrerpolicy="no-referrer-when-downgrade">beego</a></td>
         <td>v2.0.3</td>
         <td>フル</td>
         <td>N/A</td>
         <td>N/A</td>
      </tr>
      <tr>
         <td><a href="https://github.com/upper/db" target="_blank" referrerpolicy="no-referrer-when-downgrade">upper/db</a></td>
         <td>v4.5.2</td>
         <td>フル</td>
         <td>N/A</td>
         <td>N/A</td>
      </tr>
      <tr>
         <td><a href="https://gitea.com/xorm/xorm" target="_blank" referrerpolicy="no-referrer-when-downgrade">xorm</a></td>
         <td>v1.3.1</td>
         <td>フル</td>
         <td>N/A</td>
         <td>N/A</td>
      </tr>
      <tr>
         <td rowspan="4">Java</td>
         <td><a href="https://hibernate.org/orm/" target="_blank" referrerpolicy="no-referrer-when-downgrade">Hibernate</a></td>
         <td>6.1.0.Final</td>
         <td>フル</td>
         <td>N/A</td>
         <td><a href="/tidb/dev/dev-guide-sample-application-java-hibernate">Hibernateを使用してTiDBに接続する</a></td>
      </tr>
      <tr>
         <td><a href="https://mybatis.org/mybatis-3/" target="_blank" referrerpolicy="no-referrer-when-downgrade">MyBatis</a></td>
         <td>v3.5.10</td>
         <td>フル</td>
         <td>N/A</td>
         <td><a href="/tidb/dev/dev-guide-sample-application-java-mybatis">MyBatisを使用してTiDBに接続する</a></td>
      </tr>
      <tr>
         <td><a href="https://spring.io/projects/spring-data-jpa/" target="_blank" referrerpolicy="no-referrer-when-downgrade">Spring Data JPA</a></td>
         <td>2.7.2</td>
         <td>フル</td>
         <td>N/A</td>
         <td><a href="/tidb/dev/dev-guide-sample-application-java-spring-boot">Spring Bootを使用してTiDBに接続する</a></td>
      </tr>
      <tr>
         <td><a href="https://github.com/jOOQ/jOOQ" target="_blank" referrerpolicy="no-referrer-when-downgrade">jOOQ</a></td>
         <td>v3.16.7 (オープンソース)</td>
         <td>フル</td>
         <td>N/A</td>
         <td>N/A</td>
      </tr>
      <tr>
         <td>Ruby</td>
         <td><a href="https://guides.rubyonrails.org/active_record_basics.html" target="_blank" referrerpolicy="no-referrer-when-downgrade">Active Record</a></td>
         <td>v7.0</td>
         <td>フル</td>
         <td>N/A</td>
         <td><a href="/tidb/dev/dev-guide-sample-application-ruby-rails">RailsフレームワークとActiveRecord ORMを使用してTiDBに接続する</a></td>
      </tr>
      <tr>
         <td rowspan="3">JavaScript / TypeScript</td>
         <td><a href="https://sequelize.org/" target="_blank" referrerpolicy="no-referrer-when-downgrade">Sequelize</a></td>
         <td>v6.20.1</td>
         <td>フル</td>
         <td>N/A</td>
         <td><a href="/tidb/dev/dev-guide-sample-application-nodejs-sequelize">Sequelizeを使用してTiDBに接続する</a></td>
      </tr>
      <tr>
         <td><a href="https://www.prisma.io/" target="_blank" referrerpolicy="no-referrer-when-downgrade">Prisma</a></td>
         <td>4.16.2</td>
         <td>フル</td>
         <td>N/A</td>
```
| GUI                                                       | 最新のテスト済みバージョン | サポートレベル | チュートリアル                                                                      |
|-----------------------------------------------------------|-----------------------|---------------|-------------------------------------------------------------------------------|
| [JetBrains DataGrip](https://www.jetbrains.com/datagrip/) | 2023.2.1              | フル          | [JetBrains DataGripを使用したTiDBへの接続方法](/develop/dev-guide-gui-datagrip.md) |
| [DBeaver](https://dbeaver.io/)                            | 23.0.3                | フル          | [DBeaverを使用したTiDBへの接続方法](/develop/dev-guide-gui-dbeaver.md)             |
| [Visual Studio Code](https://code.visualstudio.com/)                            | 1.72.0                | フル          | [Visual Studio Codeを使用したTiDBへの接続方法](/develop/dev-guide-gui-vscode-sqltools.md)             |