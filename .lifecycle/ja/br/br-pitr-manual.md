---
title: TiDBログバックアップとPITRコマンドマニュアル
summary: TiDBログバックアップとPITRのコマンドについて学びます。
aliases: ['/tidb/dev/br-log-command-line/']
---

# TiDBログバックアップとPITRコマンドマニュアル

このドキュメントでは、TiDBログバックアップとPITR（時系列リカバリ）で使用されるコマンドについて説明します。

ログバックアップとPITRについての詳細は、以下を参照してください。

- [ログバックアップとPITRガイド](/br/br-pitr-guide.md)
- [バックアップとリストアの使用例](/br/backup-and-restore-use-cases.md)

## ログバックアップを実行

`br log`コマンドを使用してログバックアップを開始および管理できます。

```shell
./br log --help

TiDB/TiKVクラスターからログをバックアップします

使用法：
  br log [command]

利用可能なコマンド:
  metadata   log dirのメタデータを取得
  pause      ログバックアップタスクを一時停止
  resume     一時停止されたログバックアップタスクを再開
  start      ログバックアップタスクを開始
  status     ログバックアップタスクのステータスを取得
  stop       ログバックアップタスクを停止してタスクのメタデータを削除
  truncate   特定の時点までログデータを切り捨て
```

各サブコマンドは以下のように説明されています。

- `br log start`: ログバックアップタスクを開始します。
- `br log status`: ログバックアップタスクのステータスをクエリします。
- `br log pause`: ログバックアップタスクを一時停止します。
- `br log resume`: 一時停止されたログバックアップタスクを再開します。
- `br log stop`: ログバックアップタスクを停止し、タスクのメタデータを削除します。
- `br log truncate`: バックアップストレージからログバックアップデータをクリーンアップします。
- `br log metadata`: ログバックアップデータのメタデータをクエリします。

### バックアップタスクを開始

`br log start`コマンドを実行して、ログバックアップタスクを開始できます。このタスクは、TiDBクラスターのバックグラウンドで実行され、KVストレージの変更ログを自動的にバックアップストレージにバックアップします。

ヘルプ情報を表示するには、 `br log start --help`を実行します。

```shell
./br log start --help
ログバックアップタスクを開始

使用法：
  br log start [flags]

Flags:
  -h, --help               startのヘルプ
  --start-ts string        通常は最後の完全バックアップTSと同じで、ログバックアップに使用されます。デフォルト値は現在のtsです。TSOまたは日時をサポートします。例：「400036290571534337」または「2018-05-11 01:42:23+0800」。
  --task-name string       バックアップログタスクのタスク名。

Global Flags:
 --ca string                  TLS接続のためのCA証明書パス
 --cert string                TLS接続のための証明書パス
 --key string                 TLS接続のための秘密鍵パス
 -u, --pd strings             PDアドレス (デフォルト [127.0.0.1:2379])
 -s, --storage string         バックアップストレージのURLを指定します。例：「s3://bucket/path/prefix」

```

例の出力は共通パラメータのみを示しています。これらのパラメータは以下のように説明されています。

- `--start-ts`: ログバックアップの開始タイムスタンプを指定します。このパラメータが指定されていない場合、バックアッププログラムは`start-ts`として現在の時間を使用します。
- `task-name`: ログバックアップのためのタスク名を指定します。この名前は、バックアップタスクのクエリ、一時停止、再開にも使用されます。
- `--ca`、`--cert`、`--key`: TiKVとPDとの通信に使用するmTLS暗号化方式を指定します。
- `--pd`: バックアップクラスターのPDアドレスを指定します。BRは、ログバックアップタスクを開始するためにPDにアクセスする必要があります。
- `--storage`: バックアップストレージのアドレスを指定します。現在、BRはログバックアップのストレージとしてAmazon S3、Google Cloud Storage（GCS）、またはAzure Blob Storageをサポートしています。上記のコマンドはAmazon S3を例としています。詳細については、[外部ストレージサービスのURIフォーマット](/external-storage-uri.md)を参照してください。

使用例:

```shell
./br log start --task-name=pitr --pd="${PD_IP}:2379" \
--storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}"'
```

### バックアップステータスをクエリ

`br log status`コマンドを実行して、バックアップステータスをクエリできます。

ヘルプ情報を表示するには、 `br log status --help`を実行します。

```shell
./br log status --help
ログバックアップタスクのステータスを取得

使用法：
  br log status [flags]

Flags:
  -h, --help           statusのヘルプ
  --json               JSONを出力します。
  --task-name string   バックアップストリームログのタスク名。デフォルトでは全てのタスクのステータスを取得します（デフォルトは"*"）

Global Flags:
 --ca string                  TLS接続のためのCA証明書パス
 --cert string                TLS接続のための証明書パス
 --key string                 TLS接続のための秘密鍵パス
 -u, --pd strings             PDアドレス (デフォルト [127.0.0.1:2379])

```

例の出力では、`task-name`がバックアップタスクの名前を指定するために使用されます。デフォルト値は`*`で、これは全てのタスクのステータスをクエリすることを意味します。

使用例:

```shell
./br log status --task-name=pitr --pd="${PD_IP}:2379"
```

期待される出力:

```shell
● Total 1 Tasks.
> #1 <
              name: pitr
            status: ● NORMAL
             start: 2022-07-14 20:08:03.268 +0800
               end: 2090-11-18 22:07:45.624 +0800
           storage: s3://backup-101/logbackup
       speed(est.): 0.82 ops/s
checkpoint[global]: 2022-07-25 22:52:15.518 +0800; gap=2m52s
```

出力フィールドは以下のように説明されています。

- `status`: バックアップタスクのステータス。「NORMAL」、「ERROR」、「PAUSE」のいずれかです。
- `start`: バックアップタスクの開始時間。これはバックアップタスクが開始された際に指定された`start-ts`の値です。
- `storage`: バックアップストレージのアドレス。
- `speed`: バックアップタスクの合計QPS。QPSは1秒あたりにバックアップされるログの数を意味します。
- `checkpoint[global]`: このチェックポイントまでのすべてのデータがバックアップストレージにバックアップされています。これはバックアップデータを復元するための最新のタイムスタンプです。
- `error[store]`: ストレージノードでログバックアッププログラムが遭遇したエラー。

### バックアップタスクを一時停止および再開

実行中のバックアップタスクを一時停止するには`br log pause`コマンドを実行できます。

ヘルプ情報を表示するには、 `br log pause --help`を実行します。

```shell
./br log pause --help
ログバックアップタスクを一時停止

使用法：
  br log pause [flags]

Flags:
  --gc-ttl int         PDがBRのGC安全点を保持するTTL（秒単位）（デフォルト 86400）
  -h, --help           statusのヘルプ
  --task-name string   バックアップストリームログのタスク名。

Global Flags:
 --ca string                  TLS接続のためのCA証明書パス
 --cert string                TLS接続のための証明書パス
 --key string                 TLS接続のための秘密鍵パス
 -u, --pd strings             PDアドレス (デフォルト [127.0.0.1:2379])
```

> **注意:**
>
> - ログバックアップタスクを一時停止した後、変更ログを生成するMVCCデータが削除されるのを防ぐために、バックアッププログラムは自動的に最新のバックアップチェックポイントをサービス安全点として設定します。これにより、最新の24時間以内のMVCCデータが保持されます。バックアップタスクが24時間を超えて一時停止されると、対応するデータはガベージコレクションされ、バックアップされません。
> - 過剰なMVCCデータを保持すると、TiDBクラスターのストレージ容量とパフォーマンスに悪影響があります。そのため、バックアップタスクは適切なタイミングで再開することが推奨されます。

使用例:

```shell
./br log pause --task-name=pitr --pd="${PD_IP}:2379"
```

一時停止されたバックアップタスクを再開するには、`br log resume`コマンドを実行できます。

ヘルプ情報を表示するには、 `br log resume --help`を実行します。

```shell
./br log resume --help
ログバックアップタスクを再開

使用法：
  br log resume [flags]

Flags:
  -h, --help           statusのヘルプ
  --task-name string   バックアップストリームログのタスク名。

Global Flags:
 --ca string                  TLS接続のためのCA証明書パス
 --cert string                TLS接続のための証明書パス
 --key string                 TLS接続のための秘密鍵パス
 -u, --pd strings             PDアドレス (デフォルト [127.0.0.1:2379])
```

バックアップタスクが24時間を超えて一時停止された場合、`br log resume`を実行するとエラーが報告され、BRはバックアップデータが失われたことを示します。このエラーに対処するには、[バックアップとリストアのFAQ](/faq/backup-and-restore-faq.md#what-should-i-do-if-the-error-message-errbackupgcsafepointexceeded-is-returned-when-using-the-br-log-resume-command-to-resume-a-suspended-task)を参照してください。

使用例:

```shell
./br log resume --task-name=pitr --pd="${PD_IP}:2379"
```

### バックアップタスクの停止と再開
```
      + {R}
      + {R}
    + {R}
  + {R}
```
```
      + {T}
      + {T}
    + {T}
  + {T}
```
```
***完全な復元成功の概要*** ****** [total-take=3.112928252s] [restore-data-size(after-compressed)=5.056kB] [Size=5056] [BackupTS=434693927394607136] [total-kv=4] [total-kv-size=290B] [average-speed=93.16B/s]
メタファイルの復元<--------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
KVファイルの復元<----------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
"ログの復元成功の概要"] [total-take=192.955533ms] [restore-from=434693681289625602] [restore-to=434693753549881345] [total-kv-count=33] [total-size=21551]
```
```
***完全な復元成功の概要*** ****** [total-take=3.112928252s] [restore-data-size(after-compressed)=5.056kB] [Size=5056] [BackupTS=434693927394607136] [total-kv=4] [total-kv-size=290B] [average-speed=93.16B/s]
メタファイルの復元<--------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
KVファイルの復元<----------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
"ログの復元成功の概要"] [total-take=192.955533ms] [restore-from=434693681289625602] [restore-to=434693753549881345] [total-kv-count=33] [total-size=21551]
```