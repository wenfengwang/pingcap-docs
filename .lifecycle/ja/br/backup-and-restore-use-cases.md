---
title: TiDB バックアップとリストアのユースケース
summary: br コマンドラインツールを使用したデータのバックアップとリストアのユースケースについて学びます。
aliases: ['/ja/docs/dev/br/backup-and-restore-use-cases/','/ja/docs/dev/reference/tools/br/use-cases/','/ja/tidb/dev/backup-and-restore-use-cases-for-maintain/']
---

# TiDB バックアップとリストアのユースケース

[TiDB スナップショットバックアップとリストアガイド](/br/br-snapshot-guide.md)と[TiDB ログバックアップおよびPITRガイド](/br/br-pitr-guide.md)では、TiDBによって提供されるバックアップとリストアソリューション、つまり、スナップショット（完全）バックアップとリストア、ログバックアップと時間指定リカバリ（PITR）について説明します。このドキュメントは、TiDBのバックアップおよびリストアソリューションを特定のユースケースで簡単に始めるのを支援します。

AWS上にTiDB本番クラスタを展開しており、ビジネスチームから以下の要件があるとします：

- データの変更をタイムリーにバックアップします。データベースが災害に遭遇した場合、最小限のデータ損失でアプリケーションを迅速にリカバリできます（数分のデータ損失のみ許容されます）。
- 特定の時点で月次の業務監査を実行します。監査要求が受信された場合、過去1か月間の特定時点でデータをクエリするためのデータベースを提供する必要があります。

PITRを使用すると、上記の要件を満たすことができます。

## TiDBクラスタとBRの展開

PITRを使用するには、TiDBクラスタをv6.2.0以上に展開し、BRをTiDBクラスタと同じバージョンに更新する必要があります。このドキュメントでは、v7.4.0を例に説明します。

以下の表は、TiDBクラスタでPITRを使用するための推奨ハードウェアリソースを示しています。

| コンポーネント | CPU | メモリ | ディスク | AWSインスタンス | インスタンス数 |
| --- | --- | --- | --- | --- | --- |
| TiDB | 8コア以上 | 16 GB以上 | SAS | c5.2xlarge | 2 |
| PD | 8コア以上 | 16 GB以上 | SSD | c5.2xlarge | 3 |
| TiKV | 8コア以上 | 32 GB以上 | SSD | m5.2xlarge | 3 |
| BR | 8コア以上 | 16 GB以上 | SAS | c5.2xlarge | 1 |
| モニタ | 8コア以上 | 16 GB以上 | SAS | c5.2xlarge | 1 |

> **注意:**
>
> - BRがバックアップおよびリストアタスクを実行する際、PDおよびTiKVにアクセスする必要があります。BRがすべてのPDおよびTiKVノードに接続できるようにしてください。
> - BRおよびPDサーバーは同じタイムゾーンを使用する必要があります。

TiUPを使用してTiDBクラスタを展開またはアップグレードします：

- 新しいTiDBクラスタを展開する場合は、[TiDBクラスタの展開](/production-deployment-using-tiup.md)を参照してください。
- TiDBクラスタがv6.2.0より古い場合は、[TiDBクラスタのアップグレード](/upgrade-tidb-using-tiup.md)を参照してアップグレードしてください。

TiUPを使用してBRをインストールまたはアップグレードします：

- インストール：

    ```shell
    tiup install br:v7.4.0
    ```

- アップグレード：

    ```shell
    tiup update br:v7.4.0
    ```

## バックアップストレージ（Amazon S3）を構成する

バックアップタスクを開始する前に、バックアップストレージを準備する必要があります。これには、以下の側面が含まれます：

1. バックアップデータを保存するS3バケットおよびディレクトリを準備します。
2. S3バケットにアクセスするための権限を構成します。
3. 各バックアップデータを保存するサブディレクトリを計画します。

具体的な手順は以下の通りです：

1. S3にディレクトリを作成してバックアップデータを保存します。この例では、ディレクトリは`s3://tidb-pitr-bucket/backup-data`とします。

    1. バケットを作成します。既存のS3を選択してバックアップデータを保存することもできます。ない場合は、[AWSドキュメント: バケットの作成](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)を参照してS3バケットを作成します。この例では、バケット名は`tidb-pitr-bucket`です。
    2. バックアップデータのディレクトリを作成します。バケット（`tidb-pitr-bucket`）内に`backup-data`という名前のディレクトリを作成します。詳細な手順については、[AWSドキュメント: フォルダを使用したAmazon S3コンソール内のオブジェクトの整理](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html)を参照してください。

2. BRおよびTiKVがS3ディレクトリにアクセスするための権限を構成します。S3バケットにアクセスするためにIAMメソッドを使用し、S3バケットに最も安全な方法でアクセスすることをお勧めします。詳細な手順については、[AWSドキュメント: ユーザーポリシーによるバケットへのアクセス制御](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html)を参照してください。必要な権限は以下のとおりです：

    - バックアップクラスタ内のTiKVおよびBRは、`s3://tidb-pitr-bucket/backup-data`ディレクトリの`s3:ListBucket`、`s3:PutObject`、`s3:AbortMultipartUpload`の権限が必要です。
    - リストアクラスタ内のTiKVおよびBRは、`s3://tidb-pitr-bucket/backup-data`ディレクトリの`s3:ListBucket`、`s3:GetObject`、`s3:PutObject`の権限が必要です。

3. スナップショット（完全）バックアップとログバックアップのデータを保存するディレクトリ構造を計画します。

    - すべてのスナップショットバックアップデータは`s3://tidb-pitr-bucket/backup-data/snapshot-${date}`ディレクトリに保存されます。`${date}`はスナップショットバックアップの開始時刻です。例えば、2022/05/12 00:01:30に開始したスナップショットバックアップは`s3://tidb-pitr-bucket/backup-data/snapshot-20220512000130`に保存されます。
    - ログバックアップデータは`s3://tidb-pitr-bucket/backup-data/log-backup/`ディレクトリに保存されます。

## バックアップポリシーを決定する

最小限のデータ損失、迅速なリカバリ、および1か月以内のビジネス監査の要件を満たすために、以下のバックアップポリシーを設定できます：

- データベースのデータ変更を連続してバックアップするためにログバックアップを実行します。
- 2日ごとに00:00にスナップショットバックアップを実行します。
- スナップショットバックアップデータおよびログバックアップデータを30日以内に保持し、30日以上前のバックアップデータを削除します。

## ログバックアップを実行する

ログバックアップタスクが開始されると、ログバックアッププロセスはデータベースのデータ変更をS3ストレージに連続して送信するためにTiKVクラスタで実行されます。ログバックアップタスクを開始するには、次のコマンドを実行します：

```shell
tiup br log start --task-name=pitr --pd="${PD_IP}:2379" \
--storage='s3://tidb-pitr-bucket/backup-data/log-backup'
```

ログバックアップタスクが実行されている間、バックアップの状態をクエリできます：

```shell
tiup br log status --task-name=pitr --pd="${PD_IP}:2379"

● Total 1 Tasks.
> #1 <
    name: pitr
    status: ● NORMAL
    start: 2022-05-13 11:09:40.7 +0800
      end: 2035-01-01 00:00:00 +0800
    storage: s3://tidb-pitr-bucket/backup-data/log-backup
    speed(est.): 0.00 ops/s
checkpoint[global]: 2022-05-13 11:31:47.2 +0800; gap=4m53s
```

## スナップショットバックアップを実行する

crontabなどの自動ツールを使用して、定期的にスナップショットバックアップタスクを実行できます。例えば、2日ごとの00:00にスナップショットバックアップを実行します。

以下に、2つのスナップショットバックアップの例を示します：

- 2022/05/14 00:00:00にスナップショットバックアップを実行します

    ```shell
    tiup br backup full --pd="${PD_IP}:2379" \
    --storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000' \
    --backupts='2022/05/14 00:00:00'
    ```

- 2022/05/16 00:00:00にスナップショットバックアップを実行します

    ```shell
    tiup br backup full --pd="${PD_IP}:2379" \
    --storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220516000000' \
    --backupts='2022/05/16 00:00:00'
    ```

## PITRを実行する

2022/05/15 18:00:00でデータをクエリする必要があると仮定します。PITRを使用して、2022/05/14に実行したスナップショットバックアップとそのスナップショットと2022/05/15 18:00:00の間に実行したログバックアップデータを使用して、クラスタをその時点にリストアできます。

コマンドは以下の通りです：

```shell
tiup br restore point --pd="${PD_IP}:2379" \
--storage='s3://tidb-pitr-bucket/backup-data/log-backup' \
```
```
--full-backup-storage='s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000' \
--restored-ts '2022-05-15 18:00:00+0800'
```

完全なリストア <--------------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
[2022/05/29 18:15:39.132 +08:00] [INFO] [collector.go:69] ["完全なリストア成功の要約"] [total-ranges=12] [ranges-succeed=xxx] [ranges-failed=0] [split-region=xxx.xxxµs] [restore-ranges=xxx] [total-take=xxx.xxxs] [restore-data-size(after-compressed)=xxx.xxx] [Size=xxxx] [BackupTS={TS}] [total-kv=xxx] [total-kv-size=xxx] [average-speed=xxx]
メタファイルの復元 <--------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
KVファイルの復元 <----------------------------------------------------------------------------------------------------------------------------------------------------> 100.00%
[2022/05/29 18:15:39.325 +08:00] [INFO] [collector.go:69] ["リストアログ成功の要約"] [total-take=xxx.xx] [restore-from={TS}] [restore-to={TS}] [total-kv-count=xxx] [total-size=xxx]
```

## 旧データのクリーンアップ

crontabなどの自動ツールを使用して、2日ごとに古いデータをクリーンアップできます。

たとえば、次のコマンドを実行して、古いデータをクリーンアップできます:

- 2022/05/14 00:00:00より前のスナップショットデータを削除する

  ```shell
  rm s3://tidb-pitr-bucket/backup-data/snapshot-20220514000000
  ```

- 2022/05/14 00:00:00より前のログバックアップデータを削除する

  ```shell
  tiup br log truncate --until='2022-05-14 00:00:00 +0800' --storage='s3://tidb-pitr-bucket/backup-data/log-backup'
  ```

## 関連項目

- [バックアップストレージ](/br/backup-and-restore-storages.md)
- [スナップショットのバックアップとリストアコマンドマニュアル](/br/br-snapshot-manual.md)
- [ログバックアップとPITRコマンドマニュアル](/br/br-pitr-manual.md)