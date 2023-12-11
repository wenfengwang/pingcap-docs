---
title: データマイグレーションを使用したデータの移行
summary: データマイグレーションツールを使用して、フルデータと増分データを移行します。
aliases: ['/docs/tidb-data-migration/dev/replicate-data-using-dm/']
---

# データマイグレーションを使用したデータの移行

このガイドでは、データマイグレーション（DM）ツールを使用してデータを移行する方法を示します。

## ステップ 1: DM クラスタのデプロイ

[DM クラスタを TiUP を使用してデプロイ](/dm/deploy-a-dm-cluster-using-tiup.md)することをお勧めします。試用またはテスト用にバイナリを使用して [DM クラスタをデプロイ](/dm/deploy-a-dm-cluster-using-binary.md) することもできます。

> **注意:**
>
> - すべての DM 構成ファイル内のデータベースパスワードについては、`dmctl` で暗号化されたパスワードを使用することをお勧めします。データベースパスワードが空の場合は、暗号化する必要はありません。[dmctl を使用してデータベースのパスワードを暗号化する](/dm/dm-manage-source.md#encrypt-the-database-password) を参照してください。
> - 上流および下流のデータベースのユーザは、対応する読み取りおよび書き込み権限を持っている必要があります。

## ステップ 2: クラスタ情報の確認

DM クラスタを TiUP を使用してデプロイした後、構成情報は次のようになります。

- DM クラスタ内の関連コンポーネントの構成情報:

    | コンポーネント | ホスト | ポート |
    |------| ---- | ---- |
    | dm_worker1 | 172.16.10.72 | 8262 |
    | dm_worker2 | 172.16.10.73 | 8262 |
    | dm_master | 172.16.10.71 | 8261 |

- 上流および下流のデータベースインスタンスの情報:

    | データベースインスタンス | ホスト | ポート | ユーザ名 | 暗号化されたパスワード |
    | -------- | --- | --- | --- | --- |
    | 上流 MySQL-1 | 172.16.10.81 | 3306 | root | VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU= |
    | 上流 MySQL-2 | 172.16.10.82 | 3306 | root | VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU= |
    | 下流 TiDB | 172.16.10.83 | 4000 | root | |

MySQL ホストで必要な特権のリストについては、[precheck](/dm/dm-precheck.md) ドキュメントを参照してください。

## ステップ 3: データソースの作成

1. MySQL-1 関連情報を `conf/source1.yaml` に記入します:

    ```yaml
    # MySQL1 設定.

    source-id: "mysql-replica-01"
    # これは、DM-worker が Global Transaction Identifier（GTID）を使用してバイナリログを取得するかどうかを示します。この設定項目を使用する前に、上流 MySQL で GTID モードが有効になっていることを確認してください。
    enable-gtid: false

    from:
      host: "172.16.10.81"
      user: "root"
      password: "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="
      port: 3306
    ```

2. ターミナルで次のコマンドを実行し、`tiup dmctl` を使用して MySQL-1 のデータソース構成を DM クラスタに読み込みます:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup dmctl --master-addr 172.16.10.71:8261 operate-source create conf/source1.yaml
    ```

3. MySQL-2 については、構成ファイル内の関連情報を変更し、同じ `dmctl` コマンドを実行します。

## ステップ 4: データマイグレーションタスクの構成

以下の例では、上流 MySQL-1 および MySQL-2 インスタンスの `test_db` データベース内の `test_table` テーブルデータを、下流 TiDB データベースの `test_db` データベース内の `test_table` テーブルにフルデータおよび増分データモードで移行することを前提としています。

以下のように `task.yaml` タスク構成ファイルを編集します:

```yaml
# タスク名。同時に実行される複数のタスクごとに異なる名前を使用する必要があります。
name: "test"
# フルデータおよび増分データ（すべて）移行モード。
task-mode: "all"
# 下流 TiDB の構成情報。
target-database:
  host: "172.16.10.83"
  port: 4000
  user: "root"
  password: ""

# 現在のデータマイグレーションタスクで必要なすべての上流 MySQL インスタンスの構成。
mysql-instances:
-
  # 上流インスタンスまたはマイグレーショングループの ID。 "inventory.ini" ファイルまたは "dm-master.toml" ファイル内の `source_id` の構成を参照できます。
  source-id: "mysql-replica-01"
  # 移行するデータベース/テーブル名のブロックおよび許可リストの構成項目名、グローバルブロック許可リスト構成を引用して使用される、移行対象の名前のブロックおよび許可リストを引用する。
  block-allow-list: "global"  # DM のバージョンが v2.0.0-beta.2 以前である場合は black-white-list を使用してください。
  # ダンプ処理ユニットの構成名、グローバルダンプユニットの構成を引用して使用される。
  mydumper-config-name: "global"

-
  source-id: "mysql-replica-02"
  block-allow-list: "global"  # DM のバージョンが v2.0.0-beta.2 以前である場合は black-white-list を使用してください。
  mydumper-config-name: "global"

# ブロックおよび許可リストのグローバル構成。各インスタンスは、構成項目名でこれを引用できます。
block-allow-list:                     # DM のバージョンが v2.0.0-beta.2 以前である場合は black-white-list を使用してください。
  global:
    do-tables:                        # 移行対象の上流テーブルの許可リスト。
    - db-name: "test_db"              # 移行対象のテーブルのデータベース名。
      tbl-name: "test_table"          # 移行対象のテーブル名。

# ダンプユニットのグローバル構成。各インスタンスは、構成項目名でこれを引用できます。
mydumpers:
  global:
    extra-args: ""
```

## ステップ 5: データマイグレーションタスクの開始

データマイグレーション構成の可能なエラーを事前に検出するために、DM は事前チェック機能を提供します:

- DM は、データマイグレーションタスクを開始する際に、対応する権限および構成を自動的にチェックします。
- 上流 MySQL インスタンスの構成が DM の要件を満たしているかどうかを手動で事前チェックするには、`check-task` コマンドを使用できます。

事前チェック機能の詳細については、[上流 MySQL インスタンスの構成を事前チェック](/dm/dm-precheck.md) を参照してください。

> **注意:**
>
> 初めてデータマイグレーションタスクを開始する前に、上流が設定されている必要があります。それ以外の場合、タスクを開始する際にエラーが報告されます。

`tiup dmctl` コマンドを実行してデータマイグレーションタスクを開始します。編集したのは `task.yaml` 構成ファイルです。

{{< copyable "" >}}

```bash
tiup dmctl --master-addr 172.16.10.71:8261 start-task ./task.yaml
```

- 上記のコマンドが次の結果を返した場合、タスクは正常に開始されたことを示します。

    ```json
    {
        "result": true,
        "msg": "",
        "workers": [
            {
                "result": true,
                "worker": "172.16.10.72:8262",
                "msg": ""
            },
            {
                "result": true,
                "worker": "172.16.10.73:8262",
                "msg": ""
            }
        ]
    }
    ```

- データマイグレーションタスクの開始に失敗した場合は、返されたプロンプトに従って構成を変更し、`start-task task.yaml` コマンドを実行してタスクを再起動してください。

## ステップ 6: データマイグレーションタスクの確認

DM クラスタ内でタスクの状態を確認するか、特定のデータマイグレーションタスクが実行されているかどうかを確認する場合は、`tiup dmctl` で次のコマンドを実行します:

{{< copyable "" >}}

```bash
tiup dmctl --master-addr 172.16.10.71:8261 query-status
```

## ステップ 7: データマイグレーションタスクの停止

データをもう移行する必要がない場合は、`tiup dmctl` で次のコマンドを実行してタスクを停止します:

```bash
tiup dmctl --master-addr 172.16.10.71:8261 stop-task test
```

`test` は `task.yaml` 構成ファイル内の `name` 構成項目で設定したタスク名です。

## ステップ 8: タスクの監視とログの確認

TiUP を使用して DM クラスタをデプロイする際に、Prometheus、Alertmanager、Grafana が正常にデプロイされており、Grafana のアドレスが `172.16.10.71` であると想定しています。DM 関連のアラート情報を表示するには、ブラウザで <http://172.16.10.71:9093> を開き、Alertmanager に入ります。監視メトリックを確認するには、<http://172.16.10.71:3000> に移動し、DM ダッシュボードを選択してください。
```
While the DM cluster is running, DM-master, DM-worker, and dmctl output the monitoring metrics information through logs. The log directory of each component is as follows:

- DM-master log directory: It is specified by the `--log-file` DM-master process parameter. If DM is deployed using TiUP, the log directory is `{log_dir}` in the DM-master node.
- DM-worker log directory: It is specified by the `--log-file` DM-worker process parameter. If DM is deployed using TiUP, the log directory is `{log_dir}` in the DM-worker node.
```