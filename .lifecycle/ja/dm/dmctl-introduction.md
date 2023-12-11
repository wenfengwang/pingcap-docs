---
title: dmctlを使用してDMクラスタを維持する
summary: dmctlを使用してDMクラスタを維持する方法を学びます。
aliases: ['/docs/tidb-data-migration/dev/manage-replication-tasks/']
---

# dmctlを使用してDMクラスタを維持する

> **注意:**
>
> TiUPを使用して展開されたDMクラスタの場合、クラスタを維持するためには直接 [`tiup dmctl`](/dm/maintain-dm-using-tiup.md#dmctl) を使用することをお勧めします。

dmctlは、DMクラスタを維持するためのコマンドラインツールです。インタラクティブモードとコマンドモードの両方をサポートしています。

## インタラクティブモード

インタラクティブモードに入り、DM-masterとやり取りします:

> **注意:**
>
> インタラクティブモードではBash機能はサポートされていません。たとえば、クォートで囲む代わりに、文字列フラグを直接渡す必要があります。

{{< copyable "shell-regular" >}}

```bash
./dmctl --master-addr 172.16.30.14:8261
```

```
ようこそ dmctl へ
リリースバージョン: ${version}
Gitコミットハッシュ: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Gitブランチ: release-x.x
UTCビルド時間: yyyy-mm-dd hh:mm:ss
Goバージョン: go version gox.xx linux/amd64

» help
DMコントロール

使用法:
  dmctl [command]

利用可能なコマンド:
  binlog          binlog操作の管理または表示
  binlog-schema   スキーマトラッカー内のテーブルスキーマの管理または表示
  check-task      タスクの構成ファイルをチェックします
  config          設定操作の管理
  decrypt         暗号文を平文に復号します
  encrypt         平文を暗号文に暗号化します
  help            任意のコマンドに関するヘルプを取得します
  list-member     メンバー情報をリストアップします
  offline-member  閉じられたメンバーをオフラインにします
  operate-leader  リーダーを`追放`/`追放の取り消し`します
  operate-source  上流のMySQL/MariaDBソースを`作成`/`停止`/`表示`します
  pause-relay     DM-workerのリレーユニットを一時停止します
  pause-task      指定された実行中のタスクまたはすべての(サブ)タスクをソースにバインドします
  purge-relay     指定されたファイル名に従ってDM-workerのリレーログファイルを削除します
  query-status    タスクのステータスを問い合わせます
  resume-relay    DM-workerのリレーユニットを再開します
  resume-task     指定された一時停止したタスクまたはすべての(サブ)タスクをソースにバインドします
  shard-ddl-lock  シャード-ddlロック情報の管理または表示
  start-relay     ソースのリレーログを取得するワーカーを開始します
  start-task      構成ファイルで定義されたタスクを開始します
  stop-relay      ソースのリレーログを取得するワーカーを停止します
  stop-task       指定されたタスクまたはすべての(サブ)タスクをソースにバインドします
  transfer-source 上流のMySQL/MariaDBソースを空きワーカーに転送します

フラグ:
  -h, --help             dmctlのヘルプを表示します。
  -s, --source strings   MySQLソースID。

コマンドの詳細については、"dmctl [command] --help" を使用してください
```

## コマンドモード

コマンドモードは、インタラクティブモードと異なり、dmctlコマンドの直後にタスク操作を追加する必要があります。コマンドモードでのタスク操作のパラメータは、インタラクティブモードと同じです。

> **注意:**
>
> + dmctlコマンドの後には、タスク操作は1つだけ記述する必要があります。
> + v2.0.4から、DMは環境変数 `DM_MASTER_ADDR` から `-master-addr` パラメータを読み取ることをサポートします。

{{< copyable "shell-regular" >}}

```bash
./dmctl --master-addr 172.16.30.14:8261 start-task task.yaml
./dmctl --master-addr 172.16.30.14:8261 stop-task task
./dmctl --master-addr 172.16.30.14:8261 query-status

export DM_MASTER_ADDR="172.16.30.14:8261"
./dmctl query-status
```

```
利用可能なコマンド:
  binlog          binlog操作の管理または表示
  binlog-schema   スキーマトラッカー内のテーブルスキーマの管理または表示
  check-task      タスクの構成ファイルをチェックします
  config          設定操作の管理
  decrypt         暗号文を平文に復号します
  encrypt         平文を暗号文に暗号化します
  help            任意のコマンドに関するヘルプを取得します
  list-member     メンバー情報をリストアップします
  offline-member  閉じられたメンバーをオフラインにします
  operate-leader  リーダーを`追放`/`追放の取り消し`します
  operate-source  上流のMySQL/MariaDBソースを`作成`/`停止`/`表示`します
  pause-relay     DM-workerのリレーユニットを一時停止します
  pause-task      指定された実行中のタスクまたはすべての(サブ)タスクをソースにバインドします
  purge-relay     指定されたファイル名に従ってDM-workerのリレーログファイルを削除します
  query-status    タスクのステータスを問い合わせます
  resume-relay    DM-workerのリレーユニットを再開します
  resume-task     指定された一時停止したタスクまたはすべての(サブ)タスクをソースにバインドします
  shard-ddl-lock  シャード-ddlロック情報の管理または表示
  start-relay     ソースのリレーログを取得するワーカーを開始します
  start-task      構成ファイルで定義されたタスクを開始します
  stop-relay      ソースのリレーログを取得するワーカーを停止します
  stop-task       指定されたタスクまたはすべての(サブ)タスクをソースにバインドします
  transfer-source 上流のMySQL/MariaDBソースを空きワーカーに転送します

フラグ:
      --config string        設定ファイルへのパス。
  -h, --help                 dmctlのヘルプ
      --master-addr string   Master APIサーバーアドレス、このパラメータはdm-masterとのやり取り時に必要です
      --rpc-timeout string   RPCタイムアウト、デフォルトは10分 (default "10m")
  -s, --source strings       MySQLソースID。
      --ssl-ca string        接続用の信頼されたSSL CAリストを含むファイルのパス。
      --ssl-cert string      PEM形式のX509証明書を含むファイルのパス。
      --ssl-key string       PEM形式のX509キーを含むファイルのパス。
  -V, --version              バージョンを表示して終了します

コマンドの詳細については、"dmctl [command] --help" を使用してください
```