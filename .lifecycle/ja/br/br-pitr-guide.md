---
title: TiDBログバックアップおよびPITRガイド
summary: TiDBでのログバックアップおよびPITRの方法について学びます。
aliases: ['/tidb/dev/pitr-usage']
---

# TiDBログバックアップおよびPITRガイド

完全バックアップ（スナップショットバックアップ）は特定の時点での完全なクラスターデータを含みますが、TiDBログバックアップはアプリケーションによって書き込まれたデータを適時に指定されたストレージにバックアップできます。復元ポイントを必要に応じて選択する場合、つまり、時間指定リカバリ（PITR）を行う場合は、[ログバックアップを開始](#start-log-backup)し、[定期的に完全バックアップを実行](#run-full-backup-regularly)してください。

brコマンドラインツール（以下、 `br`という）を使用してデータをバックアップまたは復元する前に、[ `br`のインストール](/br/br-use-overview.md#deploy-and-use-br)が必要です。

## TiDBクラスターのバックアップ

### ログバックアップを開始

> **注:**
>
> - 次の例では、Amazon S3アクセスキーおよびシークレットキーを使用してアクセス権を承認することを前提としています。IAMロールを使用してアクセス権を承認する場合は、`--send-credentials-to-tikv`を`false`に設定する必要があります。
> - 他のストレージシステムや承認方法を使用してアクセス権を承認する場合は、[バックアップストレージ](/br/backup-and-restore-storages.md)に従ってパラメータ設定を調整する必要があります。

ログバックアップを開始するには、 `br log start`を実行します。クラスターは一度に1つのログバックアップタスクのみを実行できます。

```shell
tiup br log start --task-name=pitr --pd "${PD_IP}:2379" \
--storage 's3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}"'
```

ログバックアップタスクが開始された後、TiDBクラスターのバックグラウンドで実行され、手動で停止するまで続きます。このプロセス中、TiDB変更ログは定期的に指定されたストレージに小さなバッチでバックアップされます。ログバックアップタスクのステータスをクエリするには、次のコマンドを実行します。

```shell
tiup br log status --task-name=pitr --pd "${PD_IP}:2379"
```

期待される出力：

```
● Total 1 Tasks.
> #1 <
    name: pitr
    status: ● NORMAL
    start: 2022-05-13 11:09:40.7 +0800
      end: 2035-01-01 00:00:00 +0800
    storage: s3://backup-101/log-backup
    speed(est.): 0.00 ops/s
checkpoint[global]: 2022-05-13 11:31:47.2 +0800; gap=4m53s
```

### 定期的に完全バックアップを実行

スナップショットバックアップは完全バックアップの手段として使用できます。クラスタースナップショットをバックアップストレージに固定のスケジュール（たとえば、2日ごと）でバックアップするには、`br backup full`を実行できます。

```shell
tiup br backup full --pd "${PD_IP}:2379" \
--storage 's3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}"'
```

## PITRの実行

バックアップ保持期間内の任意の時点にクラスターを復元するには、 `br restore point`を使用できます。このコマンドを実行すると、**復元したい時刻点**、時刻点の前の**最新のスナップショットバックアップデータ**、および**ログバックアップデータ**を指定する必要があります。 BR は、自動的にリストアに必要なデータを判断し、それらのデータを指定されたクラスターに順番にリストアします。

```shell
br restore point --pd "${PD_IP}:2379" \
--storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}"' \
--full-backup-storage='s3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}"' \
--restored-ts '2022-05-15 18:00:00+0800'
```

データのリストア中、ターミナルのプログレスバーで進捗を表示できます。リストアは、フルリストアとログリストア（メタファイルのリストアおよび KVファイルのリストア）の2つのフェーズに分かれます。各フェーズが完了すると、`br`はリストア時間やデータサイズなどの情報を出力します。

```shell
Full Restore <---------------------------------------------------------------------------------------------------------------------------------------=> 100.00%
*** ["Full Restore success summary"] ****** [total-take=xxx.xxxs] [restore-data-size(after-compressed)=xxx.xxx] [Size=xxxx] [BackupTS={TS}] [total-kv=xxx] [total-kv-size=xxx] [average-speed=xxx]
Restore Meta Files <--------------------------------------------------------------------------------------------------------------------------------=> 100.00%
Restore KV Files <----------------------------------------------------------------------------------------------------------------------------------=> 100.00%
*** ["restore log success summary"] [total-take=xxx.xx] [restore-from={TS}] [restore-to={TS}] [total-kv-count=xxx] [total-size=xxx]
```

## 期限切れのデータのクリーンアップ

[TiDBバックアップおよびリストアの使用概要](/br/br-use-overview.md)に記載されているように：

PITRを実行するには、復元ポイントの前の完全バックアップと、完全バックアップポイントと復元ポイントの間のログバックアップをリストアする必要があります。したがって、バックアップ保持期間を超過したログバックアップについては、`br log truncate`を使用して指定した時点より前のバックアップを削除できます。**フルスナップショットの前のログバックアップだけを削除することをお勧めします**。

次の手順は、バックアップ保持期間を超過するバックアップデータをクリーンアップする方法を説明しています：

1. バックアップ保持期間外の**直近の完全バックアップ**を取得します。
2. `validate`コマンドを使用して、バックアップに対応する時刻点を取得します。2022/09/01以前のバックアップデータをクリーンアップする必要があると仮定すると、この時点の前の最後の完全バックアップを見つけ、それが削除されないようにします。

    ```shell
    FULL_BACKUP_TS=`tiup br validate decode --field="end-version" --storage "s3://backup-101/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}"| tail -n1`
    ```

3. スナップショットバックアップ `FULL_BACKUP_TS` より前のログバックアップデータを削除します：

    ```shell
    tiup br log truncate --until=${FULL_BACKUP_TS} --storage='s3://backup-101/logbackup?access-key=${access-key}&secret-access-key=${secret-access-key}"'
    ```

4. スナップショットバックアップ `FULL_BACKUP_TS` より前のスナップショットデータを削除します：

    ```shell
    rm -rf s3://backup-101/snapshot-${date}
    ```

## PITRのパフォーマンスおよび影響

### 機能

- 各TiKVノードでは、PITRはスナップショットデータを時速280GBおよびログデータを時速30GBでリストアできます。
- BRは、時速600GBで期限切れのログバックアップデータを削除できます。

> **注:**
>
> 上記の仕様は、次の2つのテストシナリオのテスト結果に基づいています。実際のデータは異なる場合があります。
>
> - スナップショットデータのリストア速度 = スナップショットデータサイズ / （期間 * TiKVノードの数）
> - ログデータのリストア速度 = リストアされたログデータサイズ / （期間 * TiKVノードの数）
>
> スナップショットデータサイズは、単一のレプリカのすべてのKVの論理サイズを指し、実際のリストアデータの量ではありません。BRはクラスターに構成されたレプリカの数に応じてすべてのレプリカをリストアします。レプリカが多いほど、実際にリストアできるデータも多くなります。
> テストのすべてのクラスターのデフォルトレプリカ数は3です。
> 全体のリストアパフォーマンスを向上させるために、TiKVの構成ファイルの [`import.num-threads`](/tikv-configuration-file.md#import)アイテムと、BRコマンドの [`concurrency`](/br/use-br-command-line-tool.md#common-options)オプションを変更できます。

テストシナリオ1（[TiDB Cloud](https://tidbcloud.com)で実行）：

- TiKVノードの数（8コア、16GBメモリ）：21
- TiKV構成項目 `import.num-threads`：8
- BRコマンドオプション `concurrency`：128
- リージョンの数：183,000
- クラスターで作成された新しいログデータ：時速10GB
- 書き込み（INSERT/UPDATE/DELETE）QPS：10,000

テストシナリオ2（TiDB Self-Hostedで実行）：

- TiKVノードの数（8コア、64GBメモリ）：6
- TiKV構成項目 `import.num-threads`：8
- BRコマンドオプション `concurrency`：128
- リージョンの数：50,000
- クラスターで作成された新しいログデータ：時速10GB
- 書き込み（INSERT/UPDATE/DELETE）QPS：10,000

## 関連項目

* [TiDBバックアップおよびリストアの使用事例](/br/backup-and-restore-use-cases.md)
* [brコマンドラインマニュアル](/br/use-br-command-line-tool.md)
* [ログバックアップおよびPITRアーキテクチャ](/br/br-log-architecture.md)