---
title: TiDBデータ移行クイックスタート
summary: バイナリパッケージを使用して迅速にDM（データ移行）クラスタを展開する方法を学びます。
aliases: ['/docs/tidbデータ移行/dev/はじめに/']
---

# TiDBデータ移行クイックスタートガイド

このドキュメントでは、[TiDBデータ移行](https://github.com/pingcap/dm)（DM）を使用してMySQLからTiDBへデータを移行する方法について説明します。このガイドは、DMの機能のクイックデモであり、本番環境では推奨されません。

## ステップ1: DMクラスタを展開する

1. TiUPをインストールし、TiUPを使用して[`dmctl`](/dm/dmctl-introduction.md)をインストールします：

    {{< copyable "shell-regular" >}}

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    tiup install dm dmctl
    ```

2. DMクラスタの最小展開トポロジファイルを生成します：

    {{< copyable "shell-regular" >}}

    ```
    tiup dm template
    ```

3. 出力の構成情報をコピーし、IPアドレスを変更した`topology.yaml`ファイルとして保存します。次に、TiUPを使用して`topology.yaml`ファイルでDMクラスタを展開します：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup dm deploy dm-test 6.0.0 topology.yaml -p
    ```

## ステップ2: データソースを準備する

1つまたは複数のMySQLインスタンスを上流データソースとして使用できます。

1. 以下のように、各データソース用の構成ファイルを作成します：

    {{< copyable "shell-regular" >}}

    ```yaml
    source-id: "mysql-01"
    from:
      host: "127.0.0.1"
      user: "root"
      password: "fCxfQ9XKCezSzuCD0Wf5dUD+LsKegSg="  # `tiup dmctl --encrypt "123456"`で暗号化
      port: 3306
    ```

2. 以下のコマンドを実行して、DMクラスタにソースを追加します。`mysql-01.yaml`は前のステップで作成された構成ファイルです。

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr=127.0.0.1:8261 operate-source create mysql-01.yaml # --master-addrの引数としてmaster_serversの1つを使用します
    ```

テスト用のMySQLインスタンスがない場合は、次の手順に従ってDockerでMySQLインスタンスを作成できます：

1. MySQL構成ファイルを作成します：

    {{< copyable "shell-regular" >}}

    ```shell
    mkdir -p /tmp/mysqltest && cd /tmp/mysqltest

    cat > my.cnf <<EOF
    [mysqld]
    bind-address     = 0.0.0.0
    character-set-server=utf8
    collation-server=utf8_bin
    default-storage-engine=INNODB
    transaction-isolation=READ-COMMITTED
    server-id        = 100
    binlog_format    = row
    log_bin          = /var/lib/mysql/mysql-bin.log
    show_compatibility_56 = ON
    EOF
    ```

2. Dockerを使用してMySQLインスタンスを起動します：

    {{< copyable "shell-regular" >}}

    ```shell
    docker run --name mysql-01 -v /tmp/mysqltest:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d -p 3306:3306 mysql:5.7
    ```

3. MySQLインスタンスが起動したら、インスタンスにアクセスします：

    > **注意:**
    >
    > このコマンドは、データ移行を試すためにのみ適しており、本番環境やストレステストには使用できません。

    {{< copyable "shell-regular" >}}

    ```shell
    mysql -uroot -p -h 127.0.0.1 -P 3306
    ```

## ステップ3: 下流データベースを準備する

データ移行のターゲットとして既存のTiDBクラスタを選択できます。

テスト用のTiDBクラスタのない場合は、次のコマンドを実行して簡単にデモ環境を構築できます:

{{< copyable "shell-regular" >}}

```shell
tiup playground
```

## ステップ4: テストデータを準備する

1つまたは複数のデータソースでテストテーブルとデータを作成します。既存のMySQLデータベースを使用し、データベースに有効なデータが含まれている場合は、このステップをスキップできます。

{{< copyable "sql" >}}

```sql
drop database if exists `testdm`;
create database `testdm`;
use `testdm`;
create table t1 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
create table t2 (id bigint, uid int, name varchar(80), info varchar(100), primary key (`id`), unique key(`uid`)) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
insert into t1 (id, uid, name) values (1, 10001, 'Gabriel García Márquez'), (2, 10002, 'Cien años de soledad');
insert into t2 (id, uid, name) values (3, 20001, 'José Arcadio Buendía'), (4, 20002, 'Úrsula Iguarán'), (5, 20003, 'José Arcadio');
```

## ステップ5: データ移行タスクを作成する

1. タスク構成ファイル`testdm-task.yaml`を作成します：

    {{< copyable "" >}}

    ```yaml
    name: testdm
    task-mode: all

    target-database:
      host: "127.0.0.1"
      port: 4000
      user: "root"
      password: "" # パスワードが空でない場合、dmctlで暗号化することをお勧めします。

    # 1つまたは複数のデータソースの情報を構成します
    mysql-instances:
      - source-id: "mysql-01"
        block-allow-list:  "ba-rule1"

    block-allow-list:
      ba-rule1:
        do-dbs: ["testdm"]
    ```

2. dmctlを使用してタスクを作成します:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr 127.0.0.1:8261 start-task testdm-task.yaml
    ```

`mysql-01`データベースからTiDBへのデータ移行タスクを正常に作成しました。

## ステップ6: タスクのステータスを確認する

タスクを作成した後、`dmctl query-status`コマンドを使用してタスクのステータスを確認できます：

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr 127.0.0.1:8261 query-status testdm
```