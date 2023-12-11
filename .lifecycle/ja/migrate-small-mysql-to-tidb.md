---
title: MySQLからTiDBへの小規模データセットの移行
summary: MySQLからTiDBへの小規模データセットの移行方法を学びます。
aliases: ['/tidb/dev/usage-scenario-incremental-migration/']
---

# MySQLからTiDBへの小規模データセットの移行

このドキュメントでは、TiDBデータマイグレーション（DM）を使用して、フルマイグレーションモードとインクリメンタルレプリケーションモードでMySQLからTiDBへの小規模データセットの移行方法について説明します。ここでの「小規模データセット」とは、データサイズが1 TiB未満のものを指します。

移行速度は、テーブルスキーマ内のインデックスの数、ハードウェア、ネットワーク環境など、複数の要因に依存し、30 GB/hから50 GB/hの範囲で変動します。<!--DMを使用した移行プロセスは、次の図に示されています。-->

<!--/media/dm/migrate-with-dm.png-->

## 前提条件

- [TiUPを使用してDMクラスタを展開する](/dm/deploy-a-dm-cluster-using-tiup.md)
- [DMのソースデータベースとターゲットデータベースに必要な権限を付与する](/dm/dm-worker-intro.md)

## ステップ1. データソースを作成する

まず、次のように`source1.yaml`ファイルを作成します。

{{< copyable "" >}}

```yaml
# IDはユニークである必要があります。
source-id: "mysql-01"

# DM-workerがグローバルトランザクション識別子（GTID）を使用してbinlogを取得するかどうかを構成します。GTIDを有効にするには、上流のMySQLでGTIDを有効にしている必要があります。上流のMySQLが自動ソース・レプリカの切り替えを持っている場合、GTIDモードが必要です。
enable-gtid: true

from:
  host: "${host}"         # 例: 172.16.10.81
  user: "root"
  password: "${password}" # 平文パスワードはサポートされていますが推奨されません。パスワードを使用する前に、平文パスワードを暗号化するためにdmctl encryptを使用することをお勧めします。
  port: 3306
```

次に、次のコマンドを実行して、`tiup dmctl`を使用してDMクラスタにデータソース構成をロードします。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} operate-source create source1.yaml
```

上記のコマンドで使用されるパラメータについては、以下の通りです。

|パラメータ|説明|
|  :-        |    :-           |
|`--master-addr`  |`dmctl`が接続するクラスタの任意のDM-masterノードの `{advertise-addr}` です。 例: 172.16.10.71:8261。
|`operate-source create`|データソースをDMクラスタにロードします。|

## ステップ2. マイグレーションタスクを作成する

次のように`task1.yaml`ファイルを作成します。

{{< copyable "" >}}

```yaml
# タスク名。同時に実行される複数のタスクごとにユニークな名前が必要です。
name: "test"
# タスクモード。オプションは:
# full: 完全なデータマイグレーションのみを実行します。
# incremental: ビンロッグのリアルタイムレプリケーションのみを実行します。
# all: 完全データマイグレーション + ビンロッグのリアルタイムレプリケーションを実行します。
task-mode: "all"
# ターゲットTiDBデータベースの構成。
target-database:
  host: "${host}"                   # 例: 172.16.10.83
  port: 4000
  user: "root"
  password: "${password}"           # 平文パスワードはサポートされていますが推奨されません。パスワードを使用する前に、平文パスワードを暗号化するためにdmctl encryptを使用することをお勧めします。

# 現在のマイグレーションタスクに必要なソースデータベースのすべてのMySQLインスタンスの構成。
mysql-instances:
-
  # 上流インスタンスまたはレプリケーショングループのID
  source-id: "mysql-01"
  # 移行されるスキーマ名やテーブル名のブロックリストと許可リスト構成の名前。これらの名前は、ブロックリストと許可リストのグローバル構成を参照するために使用されます。グローバル構成については、以下の`block-allow-list`の構成を参照してください。
  block-allow-list: "listA"

# ブロックリストと許可リストのグローバル構成。各インスタンスは構成項目名で参照されます。
block-allow-list:
  listA:                              # 名前
    do-tables:                        # 移行する必要がある上流テーブルの許可リスト
    - db-name: "test_db"              # 移行するテーブルのスキーマ名
      tbl-name: "test_table"          # 移行するテーブルの名前

```

上記は、移行を実行するための最小のタスク構成です。タスクに関するより詳しい構成項目については、[DMタスク完全構成ファイルの紹介](/dm/task-configuration-file-full.md) を参照してください。

## ステップ3. マイグレーションタスクを開始する

マイグレーションタスクを開始する前に、`check-task`コマンドを使用して、構成がDM構成の要件を満たしているかどうかを確認することを推奨します。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} check-task task.yaml
```

次のコマンドを使用して、`tiup dmctl`を使ってマイグレーションタスクを開始します。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} start-task task.yaml
```

上記のコマンドで使用されるパラメータについては、以下の通りです。

|パラメータ|説明|
|     -    |     -     |
|`--master-addr`|`dmctl`が接続するクラスタの任意のDM-masterノードの `{advertise-addr}` です。 例: 172.16.10.71:8261. |
|`start-task`|マイグレーションタスクを開始します。|

タスクが開始に失敗した場合は、返された結果に応じて構成を変更した後、`start-task task.yaml`コマンドを実行してタスクを再起動することができます。問題が発生した場合は、[エラーの処理](/dm/dm-error-handling.md) と[FAQ](/dm/dm-faq.md)を参照してください。

## ステップ4: マイグレーションタスクのステータスを確認する

DMクラスタに実行中のマイグレーションタスクがあるかどうか、タスクのステータスやその他の情報を確認するには、`tiup dmctl`を使用して`query-status`コマンドを実行します。

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} query-status ${task-name}
```

結果の詳細な解釈については、[ステータスのクエリ](/dm/dm-query-status.md) を参照してください。

## ステップ5. タスクを監視し、ログを表示する（オプション）

移行タスクの過去のステータスやその他の内部メトリックスを表示するには、以下の手順を実行してください。

TiUPを使用してDMを展開する際にPrometheus、Alertmanager、およびGrafanaを展開している場合、展開時に指定したIPアドレスとポートを使用してGrafanaにアクセスできます。その後、DMダッシュボードを選択して、DMに関連する監視メトリックスを表示できます。

- DM-masterのログディレクトリ：DM-masterプロセスパラメータ `--log-file` で指定されます。TiUPを使用してDMを展開した場合、ログディレクトリはデフォルトで `/dm-deploy/dm-master-8261/log/` です。
- DM-workerのログディレクトリ：DM-workerプロセスパラメータ `--log-file` で指定されます。TiUPを使用してDMを展開した場合、ログディレクトリはデフォルトで`/dm-deploy/dm-worker-8262/log/`です。

## 次のステップ

- [移行タスクを一時停止する](/dm/dm-pause-task.md)
- [移行タスクを再開する](/dm/dm-resume-task.md)
- [移行タスクを停止する](/dm/dm-stop-task.md)
- [クラスタデータソースとタスク構成のエクスポートとインポート](/dm/dm-export-import-config.md)
- [失敗したDDLステートメントの処理](/dm/handle-failed-ddl-statements.md)