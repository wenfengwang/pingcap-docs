---
title: DM でシャーディング DDL ロックを手動で処理する
summary: DM でシャーディング DDL ロックを手動で処理する方法について学びます。
aliases: ['/docs/tidb-data-migration/dev/feature-manually-handling-sharding-ddl-locks/']
---

# DM でシャーディング DDL ロックを手動で処理する

DM では、操作が正しい順序で実行されることを保証するためにシャーディング DDL ロックを使用します。このロックメカニズムは通常、シャーディング DDL ロックを自動的に解決しますが、異常なシナリオにおいては`shard-ddl-lock`コマンドを使用して異常なDDLロックを手動で処理する必要があります。

> **注意:**
>
> - このドキュメントは、悲観的な調整モードでのシャーディング DDL ロックの処理にのみ適用されます。
> - 本書の「コマンドの使用法」セクションのコマンドは対話モードで使用されます。コマンドラインモードでは、エラーが発生しないようにエスケープ文字を追加する必要があります。
> - コマンド`shard-ddl-lock unlock`は、コマンドがもたらす可能性のある影響を完全に理解し、それを受け入れることができる場合にのみ使用してください。
> - 異常なDDLロックを手動で処理する前に、DMの[シャードマージの原則](/dm/feature-shard-merge-pessimistic.md#principles)をすでに読んでいることを確認してください。

## コマンド

### `shard-ddl-lock`

このコマンドは、DDLロックを表示し、指定されたDDLロックをDM-masterに解放するために使用できます。このコマンドは、DM v6.0以降でのみサポートされています。以前のバージョンでは、`show-ddl-locks`コマンドと`unlock-ddl-locks`コマンドを使用する必要があります。

```bash
shard-ddl-lock -h
```

```
maintain or show shard-ddl locks information
Usage:
  dmctl shard-ddl-lock [task] [flags]
  dmctl shard-ddl-lock [command]
Available Commands:
  unlock      Unlock un-resolved DDL locks forcely
Flags:
  -h, --help   help for shard-ddl-lock
Global Flags:
  -s, --source strings   MySQL Source ID.
Use "dmctl shard-ddl-lock [command] --help" for more information about a command.
```

#### 引数の説明

* `shard-ddl-lock [task] [flags]`: 現在のDM-masterでDDLロック情報を表示します。

+ `shard-ddl-lock [command]`: DM-masterに指定されたDDLロックを解放するための要求です。`[command]`は`unlock`のみが値として受け入れられます。

## 使用例

### `shard-ddl-lock [task] [flags]`

`shard-ddl-lock [task] [flags]`を使用して、現在のDM-masterでDDLロック情報を表示できます。例:

```bash
shard-ddl-lock test
```

<details>
<summary>期待される出力</summary>

```
{
    "result": true,                                        # ロック情報のクエリ結果
    "msg": "",                                             # ロック情報のクエリ失敗またはその他の説明的情報（たとえば、ロックタスクが存在しないなど）の追加メッセージ
    "locks": [                                             # 存在するロック情報のリスト
        {
            "ID": "test-`shard_db`.`shard_table`",         # ロックID。現在のタスク名とDDLに対応するスキーマ/テーブル情報から構成されます
            "task": "test",                                # ロックが属するタスク名
            "mode": "pessimistic"                          # シャードDDLモード。"pessimistic"または"optimistic"に設定できます
            "owner": "mysql-replica-01",                   # ロックの所有者（悲観的モードでこのDDL操作に最初に遭遇した最初のソースのID）。楽観的モードでは常に空です
            "DDLs": [                                      # 悲観的モードでロックに対応するDDL操作のリスト。楽観的モードでは常に空です
                "USE `shard_db`; ALTER TABLE `shard_db`.`shard_table` DROP COLUMN `c2`;"
            ],
            "synced": [                                    # 対応するMySQLインスタンスですべてのシャーディングDDLイベントを受信したソースのリスト
                "mysql-replica-01"
            ],
            "unsynced": [                                  # 対応するMySQLインスタンスでまだすべてのシャーディングDDLイベントを受信していないソースのリスト
                "mysql-replica-02"
            ]
        }
    ]
}
```

</details>

### `shard-ddl-lock unlock`

このコマンドは、`DM-master`に対して指定されたDDLロックを解除するようにアクティブに要求します。所有者に対してDDLステートメントを実行するよう要求し、所有者でないすべての他のDM-workerに対してDDLステートメントをスキップするよう要求し、さらに`DM-master`でのロック情報を削除します。

> **注意:**
>
> 現在、`shard-ddl-lock unlock`は`pessimistic`モードのロックにのみ適用されます。

```bash
shard-ddl-lock unlock -h
```

```
Unlock un-resolved DDL locks forcely

Usage:
  dmctl shard-ddl-lock unlock <lock-id> [flags]

Flags:
  -a, --action string     skip/execの値を受け入れ、DDLをスキップするか実行するかを意味します（デフォルトは"skip"）
  -d, --database string   テーブルのデータベース名
  -f, --force-remove      DDLロックを強制的に削除します
  -h, --help              unlockのヘルプ
  -o, --owner string      デフォルトの所有者を代替するソース
  -t, --table string      テーブル名

Global Flags:
  -s, --source strings   MySQL Source ID.
```

`shard-ddl-lock unlock`は、以下の引数を受け入れます:

+ `-o, --owner`:

    - フラグ; 文字列; オプション
    - 指定されていない場合、このコマンドはデフォルトの所有者（`shard-ddl-lock`の結果での所有者）に対してDDLステートメントの実行を要求します。指定されている場合、このコマンドはDDLステートメントの実行をリクエストするMySQLソース（デフォルトの所有者の代替）に対して要求します。
    - 新しい所有者を指定する場合、元の所有者がクラスタからすでに削除されている場合に限ります。

+ `-f, --force-remove`:

    - フラグ; ブール値; オプション
    - 指定されていない場合、このコマンドは所有者がDDLステートメントを正常に実行した場合にのみロック情報を削除します。指定されている場合、このコマンドは所有者がDDLステートメントに失敗した場合でもロック情報を強制的に削除します（これを行った後はロックに対してクエリや操作を行うことはできません）。

+ `lock-id`:

    - フラグではない; 文字列; 必須
    - `shard-ddl-lock`の結果でのIDを指定します。

以下は`shard-ddl-lock unlock`コマンドの使用例です:

{{< copyable "shell-regular" >}}

```bash
shard-ddl-lock unlock test-`shard_db`.`shard_table`
```

```
{
    "result": true,                                        # アンロック操作の結果
    "msg": "",                                             # ロックのアンロック失敗の追加メッセージ
}
```

## サポートされているシナリオ

現在、`shard-ddl-lock unlock`コマンドは次の2つの異常なシナリオにおけるシャーディングDDLロックの処理のみをサポートしています。

### シナリオ1: いくつかのMySQLソースが削除されている

#### 異常なロックの理由

`DM-master`がシャーディングDDLロックを自動的にアンロックしようとする前に、すべてのMySQLソースがシャーディングDDLイベントを受け取る必要があります（詳細は[シャードマージの原則](/dm/feature-shard-merge-pessimistic.md#principles)を参照）。シャーディングDDLイベントがすでにマイグレーションプロセス中であり、一部のMySQLソースが削除され、リロードされない場合（これらのMySQLソースはアプリケーションの需要に従って削除されています）、すべてのDM-workerがDDLイベントを受信できないため、シャーディングDDLロックを自動的にマイグレーションおよびアンロックすることはできません。

> **注意:**
>
> シャーディングDDLイベントをマイグレーションプロセス中でないときにいくつかのDM-workerをオフラインにする必要がある場合、最初に実行中のタスクを停止するために`stop-task`を使用し、DM-workerをオフラインにさせ、タスク構成ファイルから対応する構成情報を削除し、最後に`start-task`と新しいタスク構成を使用してマイグレーションタスクを再開するという解決策があります。

#### 手動の解決策

上流に2つのインスタンス`MySQL-1` (`mysql-replica-01`) と `MySQL-2` (`mysql-replica-02`) があり、`MySQL-1` には2つのテーブル `shard_db_1`.`shard_table_1` と `shard_db_1`.`shard_table_2` があり、`MySQL-2` には2つのテーブル `shard_db_2`.`shard_table_1` と `shard_db_2`.`shard_table_2` があります。今、これら4つのテーブルをマージし、ダウンストリームのTiDBにテーブル`shard_db`.`shard_table`にマイグレーションする必要があります。

初期のテーブル構造は次のとおりです:

```sql
SHOW CREATE TABLE shard_db_1.shard_table_1;
+---------------+------------------------------------------+
| Table         | Create Table                             |
+---------------+------------------------------------------+
| shard_table_1 | CREATE TABLE `shard_table_1` (
```
      `c1` int(11) NOT NULL,
      PRIMARY KEY (`c1`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+---------------+------------------------------------------+
```

以下のDDL操作が上流のシャードテーブルの構造を変更するために実行されます:

```sql
ALTER TABLE shard_db_*.shard_table_* ADD COLUMN c2 INT;
```

MySQLとDMの操作手順は次のようになります:

1. `mysql-replica-01`の2つのシャードテーブルに対応するDDL操作が実行されて、テーブル構造を変更します。

    ```sql
    ALTER TABLE shard_db_1.shard_table_1 ADD COLUMN c2 INT;
    ```

    ```sql
    ALTER TABLE shard_db_1.shard_table_2 ADD COLUMN c2 INT;
    ```

2. DM-workerは`mysql-replica-01`の2つのシャードテーブルから受け取ったDDL情報をDM-masterに送信し、DM-masterは対応するDDLロックを作成します。
3. `shard-ddl-lock`を使用して、現在のDDLロック情報を確認します。

    ```bash
    » shard-ddl-lock test
    {
        "result": true,
        "msg": "",
        "locks": [
            {
                "ID": "test-`shard_db`.`shard_table`",
                "task": "test",
                "mode": "pessimistic"
                "owner": "mysql-replica-01",
                "DDLs": [
                    "USE `shard_db`; ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `c2` int(11);"
                ],
                "synced": [
                    "mysql-replica-01"
                ],
                "unsynced": [
                    "mysql-replica-02"
                ]
            }
        ]
    }
    ```

4. アプリケーションの要求により、`mysql-replica-02`に対応するデータはもはや下流のTiDBにマイグレーションする必要がなく、`mysql-replica-02`は削除されます。
5. `DM-master`上の```test-`shard_db`.`shard_table````というIDのロックは`mysql-replica-02`のDDL情報を受信できなくなります。

    - `shard-ddl-lock`によって返された`unsynced`の結果は常に`mysql-replica-02`の情報を含んでいます。

6. `shard-ddl-lock unlock`を使用して`DM-master`にDDLロックをアクティブにアンロックするように要求します。

    - DDLロックの所有者がオフラインになった場合、`--owner`パラメータを使用して別のDM-workerを新しい所有者として指定してDDLを実行できます。
    - いずれかのMySQLソースがエラーを報告した場合、`result`は`false`に設定され、その時点で各MySQLソースのエラーが許容範囲内かどうかを注意深く確認する必要があります。

        {{< copyable "shell-regular" >}}

        ```bash
        shard-ddl-lock unlock test-`shard_db`.`shard_table`
        ```

        ```
        {
            "result": true,
            "msg": ""
        ```

7. `shard-ddl-lock`を使用してDDLロックが正常にアンロックされたかどうかを確認します。

    ```bash
    » shard-ddl-lock test
    {
        "result": true,
        "msg": "no DDL lock exists",
        "locks": [
        ]
    }
    ```

8. 下流のTiDBでテーブル構造が正常に変更されたかどうかを確認します。

    ```sql
    mysql> SHOW CREATE TABLE shard_db.shard_table;
    +-------------+--------------------------------------------------+
    | Table       | Create Table                                     |
    +-------------+--------------------------------------------------+
    | shard_table | CREATE TABLE `shard_table` (
      `c1` int(11) NOT NULL,
      `c2` int(11) DEFAULT NULL,
      PRIMARY KEY (`c1`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin |
    +-------------+--------------------------------------------------+
    ```

9. `query-status`を使用して、マイグレーションタスクが正常に実行されていることを確認します。

#### 影響

`shard-ddl-lock unlock`を使用してロックを手動でアンロックした後、次のシャーディングDDLは自動的に正常にマイグレーションされますが、次のDDLイベントを受信した際にロックが自動的にマイグレーションできない可能性があります。

そのため、DDLロックを手動でアンロックした後は、次の操作を実行する必要があります:

1. `stop-task`を使用して実行中のタスクを停止します。
2. タスクの構成ファイルを更新し、構成ファイルからオフラインのMySQLソースに関連する情報を削除します。
3. `start-task`と新しいタスク構成ファイルを使用してタスクを再起動します。

> **注意:**
>
> `shard-ddl-lock unlock`を実行した後、オフラインになったMySQLソースを取り扱わない場合、次回のシャーディングDDLイベントを受信した際にロックが自動的にマイグレーションできない可能性があります。

### シナリオ2: 一部のDM-workerが異常停止またはDDLのアンロックプロセス中にネットワークの障害が発生する場合

#### ロックの異常停止の原因

`DM-master`が全てのDM-workerのDDLイベントを受信すると、自動的に`DDLロックをアンロック`するプロセスは主に以下の手順に従います:

1. ロックの所有者にDDL操作を実行し、対応するシャードテーブルのチェックポイントを更新します。
2. 所有者がDDL操作を正常に実行した後、`DM-master`上で保存されているDDLロック情報を削除します。
3. 他の全ての非所有者にDDLをスキップし、所有者または非所有者がDDL操作を正常に実行した後、対応するシャードテーブルのチェックポイントを更新します。
4. 所有者または非所有者の操作が正常に完了した後に、`DM-master`は対応するDDLロック情報を削除します。

現在、上記のアンロックプロセスはアトミックではありません。非所有者がDDL操作を正常にスキップした場合、非所有者が存在するDM-workerが異常終了したり、下流のTiDBでネットワーク異常が発生した場合、チェックポイントの更新に失敗する可能性があります。

非所有者がデータマイグレーションを復元した際、非所有者は例外が発生する前に協調されたDDL操作をDM-masterに再リクエストし、他のMySQLソースから対応するDDL操作を絶対に受信しません。これにより、DDL操作が対応するロックを自動的にアンロックする可能性があります。

#### 手動解決方法

現在、上流と下流のテーブル構造が同じで、テーブルのマージとマイグレーションに関する需要が手動解決方法にある手動解決方法がある場合、手動解決方法は[シナリオ1: 一部のMySQLソースが削除される](#scenario-1-some-mysql-sources-are-removed)の手動解決方法と同じです。

`DM-master`が自動的にアンロックプロセスを実行する際、所有者(`mysql-replica-01`)はDDLを正常に実行し、マイグレーションプロセスを継続します。ただし、非所有者(`mysql-replica-02`)にDDL操作をスキップするように要求するプロセス中に、非所有者が再起動されたため、対応するDM-workerがDDL操作をスキップした後にチェックポイントの更新に失敗します。

`mysql-replica-02`に対応するデータ移行サブタスクが復元されると、新しいロックが`DM-master`に作成され、他のMySQLソースは既にDDL操作を実行またはスキップし、その後のマイグレーションを実行します。

手順は次のとおりです:

1. `shard-ddl-lock`を使用して`DM-master`に対応するDDLロックが存在するかどうかを確認します。

    `mysql-replica-02`のみが`synced`の状態にあります。

    ```bash
    » shard-ddl-lock
    {
        "result": true,
        "msg": "",
        "locks": [
            {
                "ID": "test-`shard_db`.`shard_table`",
                "task": "test",
                "mode": "pessimistic"
                "owner": "mysql-replica-02",
                "DDLs": [
                    "USE `shard_db`; ALTER TABLE `shard_db`.`shard_table` ADD COLUMN `c2` int(11);"
                ],
                "synced": [
                    "mysql-replica-02"
                ],
                "unsynced": [
                    "mysql-replica-01"
                ]
            }
        ]
    }
    ```

2. `shard-ddl-lock`を使用して、`DM-master`にロックをアンロックするように要求します。

    - アンロックのプロセス中、所有者はダウンストリームにDDLを再実行するようにします(再起動前の元の所有者はダウンストリームに1度DDLを実行しています)。DDL操作が複数回実行できることを確認してください。

        ```bash
        shard-ddl-lock unlock test-`shard_db`.`shard_table`
        {
            "result": true,
            "msg": "",
        }
        ```

3. DDLロックが正常にアンロックされたかどうかを`shard-ddl-lock`を使用して確認します。
4. マイグレーションタスクが正常に実行されているかどうかを`query-status`を使用して確認します。

#### 影響

ロックを手動でアンロックした後、その後のシャーディングDDLは自動的に正常にマイグレーションされるはずですが、次のDDLイベントを受信した際にロックが自動的にマイグレーションできない可能性があります。