---
title: TiFlashクラスターのトラブルシューティング
summary: TiFlashクラスターのトラブルシューティング時の一般的な操作方法を学びます。
aliases: ['/docs/dev/tiflash/troubleshoot-tiflash/']
---

# TiFlashクラスターのトラブルシューティング

このセクションでは、TiFlashの使用中に遭遇する一般的な問題、その原因、および解決方法について説明します。

## TiFlashが起動しない

この問題はさまざまな理由で発生する可能性があります。以下の手順に従ってトラブルシューティングすることをお勧めします。

1. システムがRedHat Enterprise Linux 8であるかどうかを確認します。

    RedHat Enterprise Linux 8には`libnsl.so`システムライブラリがありません。次のコマンドで手動でインストールできます：

    {{< copyable "shell-regular" >}}

    ```shell
    dnf install libnsl
    ```

2. システムの`ulimit`パラメータ設定を確認します。

    {{< copyable "shell-regular" >}}

    ```shell
    ulimit -n 1000000
    ```

3. PD Controlツールを使用して、ノード（同じIPとポート）でオフラインにならなかったTiFlashインスタンスがあるかどうかをチェックし、インスタンスを強制的にオフラインにします。詳細な手順については、[Scale in a TiFlash cluster](/scale-tidb-using-tiup.md#scale-in-a-tiflash-cluster)を参照してください。

上記の手法で問題が解決しない場合は、TiFlashログファイルを保存して、PingCAPまたはコミュニティから[サポートを受けて](/support.md)ください。

## TiFlashのレプリカが常に利用できない

これはTiFlashが構成エラーや環境の問題により異常な状態にあるためです。次の手順に従って故障しているコンポーネントを特定します。

1. PDが`Placement Rules`機能を有効にしているかどうかを確認します：

    {{< copyable "shell-regular" >}}

    ```shell
    echo 'config show replication' | /path/to/pd-ctl -u http://${pd-ip}:${pd-port}
    ```

    - `true`が返された場合、次の手順に進みます。
    - `false`が返された場合、[Placement Rules機能を有効に](/configure-placement-rules.md#enable-placement-rules)し、次の手順に進みます。

2. TiFlashプロセスが正常に動作しているかどうかを、TiFlash-Summary監視パネルで`UpTime`を表示して確認します。

3. `pd-ctl`を使用して、TiFlashプロキシの状態が正常かどうかを確認します。

    {{< copyable "shell-regular" >}}

    ```shell
    echo "store" | /path/to/pd-ctl -u http://${pd-ip}:${pd-port}
    ```

    TiFlashプロキシの`store.labels`には、`{"key": "engine", "value": "tiflash"}`などの情報が含まれています。この情報を確認してTiFlashプロキシであることを確認できます。

4. `pd buddy`が正しくログを出力できるかどうかを確認します（ログパスは[flash.flash_cluster]構成項目の`log`の値で、デフォルトのログパスはTiFlash構成ファイルで設定された`tmp`ディレクトリの下にあります）。

5. 構成されたレプリカの数がクラスターのTiKVノードの数以下であるかどうかを確認します。そうでない場合、PDはデータをTiFlashにレプリケートできません：

    {{< copyable "shell-regular" >}}

    ```shell
    echo 'config placement-rules show' | /path/to/pd-ctl -u http://${pd-ip}:${pd-port}
    ```

    `default: count`の値を再確認します。

    > **注意:**
    >
    > [placement rules](/configure-placement-rules.md)機能が有効になった後、以前に構成された`max-replicas`および`location-labels`は効果を持たなくなります。レプリカポリシーを調整するには、placement rulesに関連するインターフェースを使用してください。

6. マシン（TiFlashノードの`store`がある場所）の残りのディスク容量が十分であるかどうかを確認します。デフォルトでは、残りのディスク容量が`store`容量の20%未満の場合（`low-space-ratio`パラメータで制御されます）、PDはこのTiFlashノードにデータをスケジュールできません。

## 一部のクエリが`Region Unavailable`エラーを返す

TiFlashへの負荷が高すぎてTiFlashのデータレプリケーションが遅れ、一部のクエリが`Region Unavailable`エラーを返すことがあります。

この場合、追加のTiFlashノードを追加することで負荷を均衡させることができます。

## データファイルの破損

データファイルの破損を処理するには、次の手順を実行します：

1. [Take a TiFlash node down](/scale-tidb-using-tiup.md#scale-in-a-tiflash-cluster)を参照して、対応するTiFlashノードを停止します。
2. TiFlashノードの関連データを削除します。
3. クラスター内でTiFlashノードを再デプロイします。

## TiFlash解析が遅い

ステートメントにMPPモードでサポートされていない演算子または関数が含まれている場合、TiDBはMPPモードを選択しません。そのため、ステートメントの解析が遅くなります。この場合、`EXPLAIN`ステートメントを実行して、MPPモードでサポートされていない演算子または関数をチェックできます。

{{< copyable "sql" >}}

```sql
create table t(a datetime);
alter table t set tiflash replica 1;
insert into t values('2022-01-13');
set @@session.tidb_enforce_mpp=1;
explain select count(*) from t where subtime(a, '12:00:00') > '2022-01-01' group by a;
show warnings;
```

この例では、警告メッセージにより、TiDB 5.4およびそれ以前のバージョンが`subtime`関数をサポートしていないため、TiDBはMPPモードを選択しないことが示されます。

```
+---------+------+-----------------------------------------------------------------------------+
> | Level   | Code | Message                                                                     |
+---------+------+-----------------------------------------------------------------------------+
| Warning | 1105 | Scalar function 'subtime'(signature: SubDatetimeAndString, return type: datetime) is not supported to push down to tiflash now.       |
+---------+------+-----------------------------------------------------------------------------+
```

## データがTiFlashにレプリケートされない

TiFlashノードをデプロイし、ALTER操作を実行してレプリケーションを開始した後、データがレプリケートされない場合があります。この場合、次の手順に従って問題を特定し、対処します。

1. `ALTER table <tbl_name> set tiflash replica <num>`コマンドを実行し、出力を確認してレプリケーションが成功したかどうかを確認します。

    - 出力がある場合、次の手順に進みます。
    - 出力がない場合、`SELECT * FROM information_schema.tiflash_replica`コマンドを実行してTiFlashレプリカが作成されたかどうかを確認します。それでも作成されていない場合は、`ALTER table ${tbl_name} set tiflash replica ${num}`コマンドを再度実行し、他のステートメント（たとえば、`add index`）が実行されたか、またはDDLの実行が成功したかを確認します。

2. TiFlashリージョンレプリケーションが正常に実行されているかどうかを確認します。

   `progress`に変更があるかどうかを確認します：

   - ある場合、TiFlashレプリケーションは正常に実行されています。
   - ない場合、TiFlashレプリケーションが異常です。`tidb.log`で`Tiflash replica is not available`というログを検索し、対応するテーブルの`progress`が更新されているかどうかを確認します。更新されていない場合は、`tiflash log`の詳細情報を確認します。たとえば、`lag_region_info`を検索して、遅れているリージョンを特定します。

3. [Placement Rules](/configure-placement-rules.md)機能が有効にされているかどうかをpd-ctlを使用して確認します：

    {{< copyable "shell-regular" >}}

    ```shell
    echo 'config show replication' | /path/to/pd-ctl -u http://<pd-ip>:<pd-port>
    ```

    - `true`が返された場合、次の手順に進みます。
    - `false`が返された場合、[Placement Rules機能を有効に](/configure-placement-rules.md#enable-placement-rules)し、次の手順に進みます。

4. `max-replicas`の構成が正しいかどうかを確認します：

    - `max-replicas`の値がクラスターのTiKVノードの数を超えていない場合は、次の手順に進みます。
    - `max-replicas`の値がクラスターのTiKVノードの数を超えている場合、PDはデータをTiFlashノードにレプリケートしません。この問題を解決するには、`max-replicas`をクラスターのTiKVノードの数以下の整数に変更してください。

    > **注意:**
    >
    > `max-replicas`のデフォルト値は3です。本番環境では、この値は通常、TiKVノードの数よりも少なくなります。テスト環境では、この値は1になります。

    {{< copyable "shell-regular" >}}

    ```shell
        curl -X POST -d '{
            "group_id": "pd",
            "id": "default",
            "start_key": "",
            "end_key": "",
            "role": "voter",
            "count": 3,
            "location_labels": [
            "host"
            ]
        }' <http://172.16.x.xxx:2379/pd/api/v1/config/rule>
    ```

5. TiDBがテーブルに対して配置規則を作成しているかどうかを確認します。
TiDB DDL オーナーのログを検索し、TiDB が PD に配置規則の追加を要請したかどうかを確認します。パーティションされていないテーブルの場合は、「ConfigureTiFlashPDForTable」を検索します。パーティションされたテーブルの場合は、「ConfigureTiFlashPDForPartitions」を検索します。

- キーワードが見つかった場合は、次のステップに進みます。
- 見つからない場合は、対応するコンポーネントのログを収集し、トラブルシューティングを行います。

6. PD がテーブルに対して配置規則を設定しているかどうかを確認します。

    現在の PD 上で全ての TiFlash 配置規則を表示するには、`curl http://<pd-ip>:<pd-port>/pd/api/v1/config/rules/group/tiflash` コマンドを実行します。ID が `table-<table_id>-r` の規則が見つかった場合、PD は配置規則を正常に設定しています。

7. PD のスケジュールが適切かどうかを確認します。

    `pd.log` ファイルを検索し、`table-<table_id>-r` キーワードや `add operator` のようなスケジュール動作を確認します。

    - キーワードが見つかった場合、PD のスケジュールは適切です。
    - 見つからない場合、PD のスケジュールが適切でない可能性があります。

## データ複製が停滞しています

TiFlash 上のデータ複製が通常どおり開始されているが、ある期間経過後にすべてまたは一部のデータが複製に失敗する場合は、次の手順に従って問題を確認または解決できます。

1. ディスクスペースを確認します。

    ディスク使用率が `low-space-ratio` の値よりも高いかどうかを確認します（デフォルトは 0.8 です。ノードのスペース使用量が 80% を超えると、PD はそのノードにデータを移行しなくなり、ディスクスペースの尽きを避けます）。

    - ディスク使用率が `low-space-ratio` の値以上である場合、ディスクスペースが不足しています。ディスクスペースを確保するために、`${data}/flash/` フォルダーの `space_placeholder_file` などの不要なファイルを削除します（必要な場合は、ファイルを削除後に `reserve-space` を 0MB に設定します）。
    - ディスク使用率が `low-space-ratio` の値未満である場合、ディスクスペースは十分です。次のステップに進みます。

2. `down peer` があるかどうかを確認します（`down peer` があると複製が停滞する可能性があります）。

    `pd-ctl region check-down-peer` コマンドを実行して、`down peer` があるかどうかを確認します。もしあれば、`pd-ctl operator add remove-peer <region-id> <tiflash-store-id>` コマンドを実行して削除します。

## データ複製が遅い

原因はさまざまです。次の手順を実行して問題を解決できます。

1. [`store limit`](/configure-store-limit.md#usage) を増やして複製を高速化します。

2. TiFlash 上の負荷を調整します。

    TiFlash 上の過剰な負荷も複製を遅くする原因となります。Grafana の **TiFlash-Summary** パネルで TiFlash の負荷指標を確認できます。

    - `Applying snapshots Count`：`TiFlash-summary` > `raft` > `Applying snapshots Count`
    - `Snapshot Predecode Duration`：`TiFlash-summary` > `raft` > `Snapshot Predecode Duration`
    - `Snapshot Flush Duration`：`TiFlash-summary` > `raft` > `Snapshot Flush Duration`
    - `Write Stall Duration`：`TiFlash-summary` > `Storage Write Stall` > `Write Stall Duration`
    - `generate snapshot CPU`：`TiFlash-Proxy-Details` > `Thread CPU` > `Region task worker pre-handle/generate snapshot CPU`

    サービスの優先度に基づいて、負荷を適切に調整して最適なパフォーマンスを実現してください。