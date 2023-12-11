---
title: データベースを作成する
summary: データベースの作成手順、ルール、および例について学ぶ。
---

# データベースを作成する

このドキュメントでは、SQLおよびさまざまなプログラミング言語を使用してデータベースを作成する方法と、データベース作成のルールについて説明します。このドキュメントでは、データベースの作成手順を説明するために、[書店](/develop/dev-guide-bookshop-schema-design.md)アプリケーションを例に取り上げています。

## 開始する前に

データベースを作成する前に、以下を行ってください:

- [TiDBサーバーレスクラスターの構築](/develop/dev-guide-build-cluster-in-cloud.md)。
- [スキーマ設計の概要](/develop/dev-guide-schema-design-overview.md)を読んでください。

## データベースとは

TiDBの[データベース](/develop/dev-guide-schema-design-overview.md)オブジェクトには、**テーブル**、**ビュー**、**シーケンス**、およびその他のオブジェクトが含まれています。

## データベースの作成

データベースを作成するには、`CREATE DATABASE` ステートメントを使用できます。

例えば、存在しない場合に `bookshop` という名前のデータベースを作成するには、次のステートメントを使用します:

```sql
CREATE DATABASE IF NOT EXISTS `bookshop`;
```

`CREATE DATABASE` ステートメントの詳細および例については、[`CREATE DATABASE`](/sql-statements/sql-statement-create-database.md)ドキュメントを参照してください。

`root`ユーザーとしてライブラリビルドステートメントを実行するには、次のコマンドを実行してください:

```shell
mysql
    -u root \
    -h {host} \
    -P {port} \
    -p {password} \
    -e "CREATE DATABASE IF NOT EXISTS bookshop;"
```

## データベースの表示

クラスター内のデータベースを表示するには、[`SHOW DATABASES`](/sql-statements/sql-statement-show-databases.md) ステートメントを使用します。

例:

```shell
mysql
    -u root \
    -h {host} \
    -P {port} \
    -p {password} \
    -e "SHOW DATABASES;"
```

以下は例の出力です:

```
+--------------------+
| Database           |
+--------------------+
| INFORMATION_SCHEMA |
| PERFORMANCE_SCHEMA |
| bookshop           |
| mysql              |
| test               |
+--------------------+
```

## データベース作成のルール

- [データベースの命名規則](/develop/dev-guide-object-naming-guidelines.md)に従い、データベースに意味のある名前を付けてください。
- TiDBには `test` という名前のデフォルトのデータベースが付属していますが、本番環境で使用しないことをお勧めします。`CREATE DATABASE` ステートメントを使用して独自のデータベースを作成し、SQLセッションで現在のデータベースを変更するために[`USE {databasename};`](/sql-statements/sql-statement-use.md) ステートメントを使用してください。
- データベース、ロール、およびユーザーなどのオブジェクトを作成するには `root` ユーザーを使用してください。ロールとユーザーには必要な権限のみを付与してください。
- データベースのスキーマ変更を実行する際には、ドライバーやORMの代わりに **MySQLコマンドラインクライアント** または **MySQL GUIクライアント** を使用することをお勧めします。

## 次のステップ

データベースを作成した後、それに **テーブル** を追加することができます。詳細については、[テーブルの作成](/develop/dev-guide-create-table.md)を参照してください。