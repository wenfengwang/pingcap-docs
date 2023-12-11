---
title: TiDBユーザーアカウント管理
summary: TiDBユーザーアカウントの管理方法について学びます。
aliases: ['/docs/dev/user-account-management/','/docs/dev/reference/security/user-account-management/']
---

# TiDBユーザーアカウント管理

このドキュメントでは、TiDBユーザーアカウントの管理方法について説明します。

## ユーザー名とパスワード

TiDBはユーザーアカウントを`mysql.user`システムデータベースのテーブルに保存します。各アカウントはユーザー名とクライアントホストで識別されます。各アカウントにはパスワードが設定されている場合があります。

MySQLクライアントを使用してTiDBサーバーに接続し、指定されたアカウントとパスワードを使用してログインできます。各ユーザー名について、32文字を超えないことを確認してください。

```shell
mysql --port 4000 --user xxx --password
```

または、コマンドラインパラメーターの省略形を使用できます:

```shell
mysql -P 4000 -u xxx -p
```

## ユーザーアカウントの追加

TiDBのアカウントは、次の2つの方法で作成できます:

- `CREATE USER`や`GRANT`などのアカウントを作成し権限を付与するための標準のアカウント管理SQLステートメントを使用する方法
- `INSERT`、`UPDATE`、または`DELETE`などのステートメントを使用して権限テーブルを直接操作する方法。この方法でアカウントを作成することは推奨されていません。不完全な更新が発生する可能性があります。

また、サードパーティのGUIツールを使用してアカウントを作成することもできます。

{{< copyable "sql" >}}

```sql
CREATE USER [IF NOT EXISTS] user [IDENTIFIED BY 'auth_string'];
```

パスワードを割り当てた後、TiDBは`auth_string`を暗号化し、`mysql.user`テーブルに保存します。

{{< copyable "sql" >}}

```sql
CREATE USER 'test'@'127.0.0.1' IDENTIFIED BY 'xxx';
```

TiDBアカウントの名前はユーザー名とホスト名で構成されます。アカウント名の構文は'user_name'@'host_name'です。

- `user_name`は大文字と小文字が区別されます。

- `host_name`はホスト名またはIPアドレスであり、ワイルドカードの`%`または`_`をサポートしています。たとえば、ホスト名`'%'`はすべてのホストに一致し、ホスト名`'192.168.1.%'`はサブネット内のすべてのホストに一致します。

ホストは曖昧一致をサポートしています:

{{< copyable "sql" >}}

```sql
CREATE USER 'test'@'192.168.10.%';
```

`test`ユーザーは`192.168.10`サブネット上のすべてのホストからログインできます。

ホストが指定されていない場合は、ユーザーは任意のIPからログインできます。パスワードが指定されていない場合、デフォルトは空のパスワードです:

{{< copyable "sql" >}}

```sql
CREATE USER 'test';
```

次に等価:

{{< copyable "sql" >}}

```sql
CREATE USER 'test'@'%' IDENTIFIED BY '';
```

指定されたユーザーが存在しない場合、ユーザーを自動的に作成する動作は`sql_mode`に依存します。`sql_mode`に`NO_AUTO_CREATE_USER`が含まれている場合、`GRANT`ステートメントはエラーを返してユーザーを作成しません。

たとえば、`sql_mode`に`NO_AUTO_CREATE_USER`が含まれていないと仮定し、次の`CREATE USER`と`GRANT`ステートメントを使用して4つのアカウントを作成する場合:

{{< copyable "sql" >}}

```sql
CREATE USER 'finley'@'localhost' IDENTIFIED BY 'some_pass';
```

{{< copyable "sql" >}}

```sql
GRANT ALL PRIVILEGES ON *.* TO 'finley'@'localhost' WITH GRANT OPTION;
```

{{< copyable "sql" >}}

```sql
CREATE USER 'finley'@'%' IDENTIFIED BY 'some_pass';
```

{{< copyable "sql" >}}

```sql
GRANT ALL PRIVILEGES ON *.* TO 'finley'@'%' WITH GRANT OPTION;
```

{{< copyable "sql" >}}

```sql
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'admin_pass';
```

{{< copyable "sql" >}}

```sql
GRANT RELOAD,PROCESS ON *.* TO 'admin'@'localhost';
```

{{< copyable "sql" >}}

```sql
CREATE USER 'dummy'@'localhost';
```

アカウントに付与された権限を表示するには、`SHOW GRANTS`ステートメントを使用します:

{{< copyable "sql" >}}

```sql
SHOW GRANTS FOR 'admin'@'localhost';
```

```
+-----------------------------------------------------+
| Grants for admin@localhost                          |
+-----------------------------------------------------+
| GRANT RELOAD, PROCESS ON *.* TO 'admin'@'localhost' |
+-----------------------------------------------------+
```

## ユーザーアカウントの削除

ユーザーアカウントを削除するには、`DROP USER`ステートメントを使用します:

{{< copyable "sql" >}}

```sql
DROP USER 'test'@'localhost';
```

この操作は、`mysql.user`テーブルのユーザーのレコードと権限テーブルの関連レコードをクリアします。

## 予約されたユーザーアカウント

TiDBはデータベースの初期化時に`'root'@'%'`デフォルトアカウントを作成します。

## アカウントのリソース制限の設定

現在、TiDBはアカウントのリソース制限の設定をサポートしていません。

## アカウントのパスワードの割り当て

TiDBは、パスワードを`mysql.user`システムデータベースに保存します。パスワードを割り当て、更新する操作は、`CREATE USER`権限を持つユーザー、または`mysql`データベースの権限（新しいアカウントを作成するための`INSERT`権限、既存のアカウントを更新するための`UPDATE`権限）を持つユーザーのみ許可されています。

- 新しいアカウントを作成する際にパスワードを割り当てるには、`CREATE USER`を使用し`IDENTIFIED BY`句を含めます:

    ```sql
    CREATE USER 'test'@'localhost' IDENTIFIED BY 'mypass';
    ```

- 既存のアカウントのパスワードを割り当てる、または変更するには、`SET PASSWORD FOR`または`ALTER USER`を使用します:

    ```sql
    SET PASSWORD FOR 'root'@'%' = 'xxx';
    ```

    または:

    ```sql
    ALTER USER 'test'@'localhost' IDENTIFIED BY 'mypass';
    ```

## `root`パスワードを忘れる

1. 設定ファイルを変更します:

    1. tidb-serverのインスタンスが配置されているマシンにログインします。
    2. TiDBノードのデプロイディレクトリの`conf`ディレクトリに移動し、`tidb.toml`設定ファイルを見つけます。
    3. 構成ファイルの`security`セクションに構成項目`skip-grant-table`を追加します。`security`セクションが存在しない場合は、次の2行をtidb.toml構成ファイルの末尾に追加します:

        ```
        [security]
        skip-grant-table = true
        ```

2. tidb-serverプロセスを停止します:

    1. tidb-serverプロセスを表示します:

        ```bash
        ps aux | grep tidb-server
        ```

    2. tidb-serverに対応するプロセスID（PID）を見つけ、`kill`コマンドを使用してプロセスを停止します:

        ```bash
        kill -9 <pid>
        ```

3. 変更した構成を使用してTiDBを起動します:

    > **注意:**
    >
    > TiDBプロセスを開始する前に`skip-grant-table`を設定すると、操作システムのユーザーのチェックが開始されます。操作システムの`root`ユーザーのみがTiDBプロセスを開始できます。

    1. TiDBノードのデプロイディレクトリの`scripts`ディレクトリに移動します。
    2. 操作システムの`root`アカウントに切り替えます。
    3. ディレクトリで`run_tidb.sh`スクリプトをバックグラウンドで実行します。
    4. 新しいターミナルウィンドウで`root`としてログインし、パスワードを変更します。

        ```bash
        mysql -h 127.0.0.1 -P 4000 -u root
        ```

4. `run_tidb.sh`スクリプトの実行を停止し、手順1でTiDB構成ファイルに追加された内容を削除し、tidb-serverの自動的な起動を待ちます。

## `FLUSH PRIVILEGES`

ユーザーと権限に関連する情報はTiKVサーバーに保存され、TiDBはこの情報をプロセス内でキャッシュします。一般に、`CREATE USER`、`GRANT`などのステートメントを使用して関連情報を変更すると、クラスタ全体で変更が速やかに有効になります。一時的に利用できないネットワークなどの要因によって操作が影響を受ける場合は、TiDBはキャッシュ情報を定期的に再読み込みするため、変更は約15分で有効になります。

権限テーブルを直接変更した場合は、次のコマンドを実行してすぐに変更を適用します:

```sql
FLUSH PRIVILEGES;
```

詳細については、[権限管理](/privilege-management.md)を参照してください。