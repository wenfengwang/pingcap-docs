---
title: SQLファイルからTiDBへのデータの移行
summary: SQLファイルからTiDBへのデータの移行方法について学びます。
aliases: ['/docs/dev/migrate-from-mysql-mydumper-files/', '/tidb/dev/migrate-from-mysql-mydumper-files/', '/tidb/dev/migrate-from-mysql-dumpling-files/']
---

# SQLファイルからTiDBへのデータの移行

このドキュメントでは、TiDB Lightningを使用してMySQLのSQLファイルからTiDBへデータを移行する方法について説明します。MySQLのSQLファイルの生成方法については、[Dumplingを使用したSQLファイルへのエクスポート](/dumpling-overview.md#export-to-sql-files)を参照してください。

## 前提条件

- [TiUPを使用してTiDB Lightningをインストール](/migration-tools.md)
- [TiDB Lightningのためのターゲットデータベースへの必要な権限を付与](/tidb-lightning/tidb-lightning-faq.md#what-are-the-privilege-requirements-for-the-target-database)

## ステップ1. SQLファイルの準備

すべてのSQLファイルを `/data/my_datasource/` や `s3://my-bucket/sql-backup` のような同じディレクトリに配置します。TiDB Lightningは、このディレクトリとそのサブディレクトリ内のすべての `.sql` ファイルを再帰的に検索します。

## ステップ2. ターゲットテーブルスキーマの定義

TiDBにデータをインポートするためには、ターゲットデータベースのためのテーブルスキーマを作成する必要があります。

Dumplingを使用してデータをエクスポートした場合、テーブルスキーマファイルは自動的にエクスポートされます。他の方法でエクスポートされたデータの場合、次のいずれかの方法でテーブルスキーマを作成できます。

+ **方法1**：TiDB Lightningを使用してターゲットテーブルスキーマを作成する。

    必要なDDLステートメントが含まれるSQLファイルを作成します：

    - `${db_name}-schema-create.sql` ファイルに `CREATE DATABASE` ステートメントを追加します。
    - `${db_name}.${table_name}-schema.sql` ファイルに `CREATE TABLE` ステートメントを追加します。

+ **方法2**：ターゲットテーブルスキーマを手動で作成する。

## ステップ3. 構成ファイルの作成

次の内容で `tidb-lightning.toml` ファイルを作成します：

{{< copyable "" >}}

```toml
[lightning]
# ログ
level = "info"
file = "tidb-lightning.log"

[tikv-importer]
# "local"：デフォルト。大容量のデータ（約1 TiB以上）をインポートするためにローカルバックエンドが使用されます。インポート中、ターゲットのTiDBクラスタはサービスを提供できません。
# "tidb"："tidb" バックエンドは、小容量のデータ（1 TiB未満）をインポートするためにも使用できます。インポート中、ターゲットのTiDBクラスタは通常サービスを提供できます。バックエンドモードの詳細については、https://docs.pingcap.com/tidb/stable/tidb-lightning-backends を参照してください。

backend = "local"
# ソートされたキー値ファイルの一時的な保存ディレクトリを設定します。ディレクトリは空でなければならず、そのストレージスペースはインポートするデータセットのサイズよりも大きくなければなりません。より良いインポートパフォーマンスのためには、`data-source-dir` とは異なるディレクトリを使用し、ディレクトリにフラッシュストレージと排他的I/Oを使用することを推奨します。
sorted-kv-dir = "${sorted-kv-dir}"

[mydumper]
# データソースのディレクトリ
data-source-dir = "${data-path}" # ローカルまたはS3パス、例： 's3://my-bucket/sql-backup'

[tidb]
# ターゲットクラスタの情報
host = ${host}                # 例：172.16.32.1
port = ${port}                # 例：4000
user = "${user_name}"         # 例： "root"
password = "${password}"      # 例： "rootroot"
status-port = ${status-port}  # インポートプロセス中、TiDB Lightningは、TiDBの「ステータスポート」からテーブルスキーマ情報を取得する必要があります。例：10080
pd-addr = "${ip}:${port}"     # クラスタのPDのアドレス。TiDB LightningはPDを介していくつかの情報を取得します。例：172.16.31.3:2379。backend = "local" の場合、status-port と pd-addr を正しく指定する必要があります。そうでない場合、インポートにエラーが発生します。
```

構成ファイルの詳細については、[TiDB Lightning Configuration](/tidb-lightning/tidb-lightning-configuration.md) を参照してください。

## ステップ4. データのインポート

インポートを開始するには、`tidb-lightning` を実行します。プログラムをコマンドラインで起動する場合、プログラムが `SIGHUP` シグナルのために終了することがあります。この場合、`nohup` や `screen` ツールでプログラムを実行することをお勧めします。

S3からデータをインポートする場合、アカウントの `SecretKey` および `AccessKey` を環境変数として渡す必要があります。アカウントにはS3バックエンドストレージへのアクセス権限があります。

{{< copyable "shell-regular" >}}

```shell
export AWS_ACCESS_KEY_ID=${access_key}
export AWS_SECRET_ACCESS_KEY=${secret_key}
nohup tiup tidb-lightning -config tidb-lightning.toml > nohup.out 2>&1 &
```

TiDB Lightningはまた、`~/.aws/credentials` から資格情報ファイルを読み取ることをサポートしています。

インポートが開始されたら、次の方法で進捗状況を確認できます：

- デフォルトでは、5分ごとに更新される `progress` キーワードを `grep` ログで検索します。
- Grafanaダッシュボードを使用します。詳細については、[TiDB Lightning Monitoring](/tidb-lightning/monitor-tidb-lightning.md) を参照してください。
- ウェブインターフェースを使用します。詳細については、[TiDB Lightning Web Interface](/tidb-lightning/tidb-lightning-web-interface.md) を参照してください。

インポートが完了したら、TiDB Lightningは自動的に終了します。`tidb-lightning.log` に最終行に `the whole procedure completed` が含まれているかどうかを確認します。含まれている場合、インポートは成功です。含まれていない場合、インポート中にエラーが発生しています。エラーメッセージに従ってエラーを解決します。

> **注意:**
>
> インポートが成功したかどうかに関係なく、最後の行には `tidb lightning exit` が表示されます。これは単にTiDB Lightningが通常終了したことを意味し、タスクが完了したことを意味するものではありません。
インポートプロセス中に問題が発生した場合は、トラブルシューティングのために[TiDB Lightning FAQ](/tidb-lightning/tidb-lightning-faq.md) を参照してください。