---
title: データ移行タスクの作成
summary: TiDBデータ移行でデータ移行タスクを作成する方法を学びます。

# データ移行タスクの作成

`start-task` コマンドを使用してデータ移行タスクを作成できます。データ移行タスクが開始されると、DMは[特権と構成の事前チェック](/dm/dm-precheck.md)を実行します。

{{< copyable "" >}}

```bash
help start-task
```

```
設定ファイルで定義されたタスクを開始します

使用法:
  dmctl start-task [-s source ...] [--remove-meta] <config-file> [flags]

フラグ:
  -h, --help                start-task のヘルプを表示します
      --remove-meta         タスクのメタデータを削除するかどうか
      --start-time string   バイナリログのレプリケーションの開始時間を指定します。例: '2021-10-21 00:01:00' または 2021-10-21T00:01:00

グローバルフラグ:
      --config string        設定ファイルへのパス
      --master-addr string   マスターAPIサーバーアドレス。dm-masterとの対話時にこのパラメータが必要です
      --rpc-timeout string   RPCタイムアウト。デフォルトは10分です。 (デフォルト "10m")
  -s, --source strings       MySQLソースID。
      --ssl-ca string        接続用の信頼されたSSL CAリストを含むファイルのパス
      --ssl-cert string      接続用のPEM形式のX509証明書を含むファイルのパス
      --ssl-key string       接続用のPEM形式のX509キーを含むファイルのパス
  -V, --version              バージョンを表示して終了します。
```

## 使用例

{{< copyable "" >}}

```bash
start-task [ -s "mysql-replica-01"] ./task.yaml
```

## フラグの説明

- `-s`: (オプション) `task.yaml`を実行するMySQLソースを指定します。設定されている場合、このコマンドは指定されたタスクのサブタスクのみをMySQLソースで開始します。
- `config-file`: (必須) `task.yaml`のファイルパスを指定します。
- `remove-meta`: (オプション) タスクを開始する際に、タスクの以前のメタデータを削除するかどうかを指定します。
- `start-time`: (オプション) バイナリログのレプリケーションの開始時間を指定します。
    - フォーマット: `'2021-10-21 00:01:00'` または `2021-10-21T00:01:00`。
    - 増分タスクの場合、このフラグを使用してタスクのおおよその開始地点を指定できます。このフラグはタスク構成ファイル内のバイナリログポジションおよびダウンストリームのチェックポイントのバイナリログポジションよりも優先されます。
    - タスクに既にチェックポイントがある場合、このフラグを使用してタスクを開始すると、DMはチェックポイントを通過するまでレプリケーションに安全モードを自動的に有効にします。これは、タスクを以前の位置にリセットすることによるデータ重複エラーを回避するためです。
        - タスクを以前の位置にリセットすると、その時点でのテーブルスキーマが現在の時点のダウンストリームと異なる場合、タスクはエラーを報告する可能性があります。
        - タスクをより新しい位置にリセットする場合、スキップされたバイナリログにはダウンストリームに残された不正なデータがあるかもしれませんので注意してください。
    - より早い開始時間を指定すると、DMは利用可能な最初のバイナリログ位置から移行を開始します。
    - より遅い開始時間を指定すると、DMはエラーを報告します: `start-time {input-time} is too late, no binlog location matches it`.

## 返される結果

{{< copyable "" >}}

```bash
start-task task.yaml
```

```
{
    "result": true,
    "msg": "",
    "sources": [
        {
            "result": true,
            "msg": "",
            "source": "mysql-replica-01",
            "worker": "worker1"
        }
    ]
}
```