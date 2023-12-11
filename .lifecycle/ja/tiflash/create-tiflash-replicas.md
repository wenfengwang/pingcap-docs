---
title: TiFlash レプリカの作成
summary: TiFlash レプリカの作成方法を学びます。
---

# TiFlash レプリカの作成

この文書では、テーブルおよびデータベースのための TiFlash レプリカを作成し、レプリカのスケジューリングに利用可能なゾーンを設定する方法について紹介します。

## テーブルのための TiFlash レプリカの作成

TiFlash が TiKV クラスタに接続された後、デフォルトではデータのレプリケーションは開始されません。特定のテーブルに TiFlash レプリカを作成するために、MySQL クライアントを介して TiDB に DDL ステートメントを送信できます:

```sql
ALTER TABLE table_name SET TIFLASH REPLICA count;
```

上記のコマンドのパラメータについては以下のように説明されています:

- `count` はレプリカの数を示します。値が `0` の場合、レプリカは削除されます。

同じテーブルに複数の DDL ステートメントを実行しても、最後のステートメントのみが効果を持つことが保証されます。以下の例では、テーブル `tpch50` に対して2つの DDL ステートメントが実行されていますが、2番目のステートメント（レプリカの削除）のみが効果を持ちます。

テーブルに2つのレプリカを作成：

```sql
ALTER TABLE `tpch50`.`lineitem` SET TIFLASH REPLICA 2;
```

レプリカを削除：

```sql
ALTER TABLE `tpch50`.`lineitem` SET TIFLASH REPLICA 0;
```

**注記:**

* 上記の DDL ステートメントを通じてテーブル `t` をTiFlashにレプリケートする場合、以下のステートメントを使用して作成されたテーブルも自動的に TiFlash にレプリケートされます:

    ```sql
    CREATE TABLE table_name like t;
    ```

* v4.0.6 より前のバージョンでは、TiDB Lightning を使用してデータをインポートする前に TiFlash レプリカを作成しないとデータのインポートに失敗します。テーブルの TiFlash レプリカを作成する前にデータをテーブルにインポートする必要があります。

* TiDB と TiDB Lightning がいずれも v4.0.6 以上のバージョンである場合、テーブルに TiFlash レプリカがあってもなくても、TiDB Lightning を使用してそのテーブルにデータをインポートすることができます。ただし、これは TiDB Lightning の手順が遅くなる可能性があります。遅延の程度は、ライトニングホストの NIC バンド幅、TiFlash ノードの CPU およびディスク負荷、TiFlash レプリカの数に依存します。

* PD のスケジューリングパフォーマンスを下げないために、1,000 を超えるテーブルをレプリケートしないことをお勧めします。この制限は後のバージョンで解除されます。

* v5.1 以降のバージョンでは、システムテーブルのレプリカ設定を行うことはできなくなりました。クラスタをアップグレードする前に、関連するシステムテーブルのレプリカをクリアする必要があります。そうしないと、クラスタを後のバージョンにアップグレードした後にシステムテーブルのレプリカ設定を変更することはできません。

### レプリケーションの進行状況の確認

特定のテーブルの TiFlash レプリカの状態は、次のステートメントを使用して確認できます。テーブルは `WHERE` 句を使用して指定されます。`WHERE` 句を削除すると、すべてのテーブルのレプリカの状態を確認できます。

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>' and TABLE_NAME = '<table_name>';
```

上記ステートメントの結果では:

* `AVAILABLE` は、このテーブルの TiFlash レプリカが利用可能かどうかを示します。`1` は利用可能を意味し、`0` は利用できないことを意味します。レプリカが利用可能になると、この状態は変わりません。DDL ステートメントを使用してレプリカの数を変更すると、レプリケーションの状態が再計算されます。
* `PROGRESS` はレプリケーションの進行度を示します。値は `0.0` から `1.0` の間です。`1` は少なくとも1つのレプリカがレプリケートされたことを意味します。

## データベースのための TiFlash レプリカの作成

テーブルの TiFlash レプリカを作成するのと同様に、特定のデータベース内のすべてのテーブルに対して TiFlash レプリカを作成するために、MySQL クライアントを介して TiDB に DDL ステートメントを送信できます:

```sql
ALTER DATABASE db_name SET TIFLASH REPLICA count;
```

このステートメントでは、`count` はレプリカの数を示します。`0` に設定すると、レプリカが削除されます。

例:

- データベース `tpch50` のすべてのテーブルに2つのレプリカを作成:

    ```sql
    ALTER DATABASE `tpch50` SET TIFLASH REPLICA 2;
    ```

- データベース `tpch50` に作成された TiFlash レプリカを削除:

    ```sql
    ALTER DATABASE `tpch50` SET TIFLASH REPLICA 0;
    ```

> **注記:**
>
> - このステートメントは実際にはリソースを多用しますので、途中で中断すると実行された操作はロールバックされず、まだ実行されていない操作は続行されません。
>
> - ステートメントを実行した後、このデータベースのすべてのテーブルがレプリケートされるまで、TiFlash レプリカの数を設定したり、DDL 操作を行ったりしないでください。さもないと、予期しない結果が発生する可能性があります。例えば:
>     - TiFlash レプリカの数を2に設定し、全てのテーブルがレプリケートされる前に数を1に変更した場合、すべてのテーブルの最終的な TiFlash レプリカの数が必ずしも1または2になるとは限りません。
>     - ステートメントを実行した後、このデータベースで新しいテーブルを作成すると、ステートメントの実行が完了する前に TiFlash レプリカが自動的にこれらの新しいテーブルに作成される場合があります。
>     - ステートメントを実行した後、このデータベースのテーブルにインデックスを追加すると、ステートメントがハングし、インデックスが追加された後にのみ再開されます。
>
> - ステートメントの実行完了後、このデータベースで新たにテーブルを作成しても、これらの新しいテーブルのための TiFlash レプリカは自動的に作成されません。
>
> - このステートメントはシステムテーブル、ビュー、一時テーブル、および TiFlash でサポートされていない文字セットを持つテーブルをスキップします。

### レプリケーションの進行状況の確認

テーブルの TiFlash レプリカを作成するのと同様に、DDL ステートメントの正常な実行はリプリケーションの完了を意味しません。次の SQL ステートメントを実行して、対象テーブルのレプリケーションの進行状況を確認できます:

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = '<db_name>';
```

データベース内で TiFlash レプリカのないテーブルを確認するには、次の SQL ステートメントを実行できます:

```sql
SELECT TABLE_NAME FROM information_schema.tables where TABLE_SCHEMA = "<db_name>" and TABLE_NAME not in (SELECT TABLE_NAME FROM information_schema.tiflash_replica where TABLE_SCHEMA = "<db_name>");
```

## TiFlash レプリケーションの高速化

<CustomContent platform="tidb-cloud">

> **注記:**
>
> このセクションは TiDB Cloud には適用されません。

</CustomContent>

TiFlash レプリカが追加される前に、各 TiKV インスタンスはフルテーブルスキャンを実行し、スキャンされたデータを TiFlash に"スナップショット"として送信してレプリカを作成します。デフォルトでは、オンラインサービスへの影響を最小限に抑えるために、TiFlash レプリカがリソースを少なく使用してゆっくりと追加されます。TiKV および TiFlash ノードに余分な CPU およびディスク IO リソースがある場合、次の手順を実行して TiFlash レプリケーションを加速できます。

1. [動的設定 SQL ステートメント](https://docs.pingcap.com/tidb/stable/dynamic-config)を使用して、一時的に各 TiKV および TiFlash インスタンスのスナップショット書き込み速度制限を増やします:

    ```sql
    -- 両設定のデフォルト値は 100MiB です。つまり、スナップショットの書き込みに使用される最大ディスク帯域幅は 100MiB/s を超えません。
    SET CONFIG tikv `server.snap-io-max-bytes-per-sec` = '300MiB';
    SET CONFIG tiflash `raftstore-proxy.server.snap-max-write-bytes-per-sec` = '300MiB';
    ```

    これらの SQL ステートメントを実行した後、設定が即座に変更されますが、レプリケーション速度は依然として PD の制限によって制限されるため、現時点では加速が観察できません。

2. [PD Control](https://docs.pingcap.com/tidb/stable/pd-control)を使用して新しいレプリカ速度制限を段階的に緩和します。

    デフォルトの新しいレプリカ速度制限は 30 です。つまり、約30のリージョンが TiFlash レプリカを1分ごとに追加します。以下のコマンドを実行すると、すべての TiFlash インスタンスの制限が 60 に調整され、元の速度が2倍になります:

    ```shell
    tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store limit all engine tiflash 60 add-peer
    ```

    > 上記のコマンドでは、実際のクラスタバージョン（たとえば、`v7.4.0`）に合わせて `v<CLUSTER_VERSION>` を置き換え、PD ノードのアドレスを `<PD_ADDRESS>:2379` に置き換える必要があります。例:

    > ```shell
    > tiup ctl:v7.4.0 pd -u http://192.168.1.4:2379 store limit all engine tiflash 60 add-peer
    > ```

    数分後、TiFlash ノードの CPU およびディスク IO リソースの使用量が大幅に増加し、TiFlash がより速くレプリカを作成することが観察されるはずです。同時に、TiKV ノードの CPU およびディスク IO リソースの使用量も増加します。

    この時点で TiKV および TiFlash ノードにまだ余裕があり、オンラインサービスのレイテンシが著しく増加していない場合は、例えば元の速度の3倍に設定するなど、より制限を緩和することができます:

    ```shell
    tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store limit all engine tiflash 90 add-peer
    ```

```yaml
3. TiFlash レプリケーションが完了したら、オンラインサービスに与える影響を軽減するためにデフォルトの構成に戻します。

    次の PD Control コマンドを実行して、新しいレプリカ速度制限をデフォルトに戻します。

    ```shell
    tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store limit all engine tiflash 30 add-peer
    ```

    次の SQL ステートメントを実行して、デフォルトのスナップショット書き込み速度制限を復元します。

    ```sql
    SET CONFIG tikv `server.snap-io-max-bytes-per-sec` = '100MiB';
    SET CONFIG tiflash `raftstore-proxy.server.snap-max-write-bytes-per-sec` = '100MiB';
    ```

## 利用可能なゾーンを設定

<CustomContent platform="tidb-cloud">

> **注:**
>
> このセクションは TiDB Cloud には適用されません。

</CustomContent>

    レプリカを構成する際に、災害復旧のために TiFlash レプリカを複数のデータセンターに配布する必要がある場合は、以下の手順に従って利用可能なゾーンを構成できます。

1. クラスタ構成ファイルで TiFlash ノードにラベルを指定します。

    ```yaml
    tiflash_servers:
      - host: 172.16.5.81
        config:
          logger.level: "info"
        learner_config:
          server.labels:
            zone: "z1"
      - host: 172.16.5.82
        config:
          logger.level: "info"
        learner_config:
          server.labels:
            zone: "z1"
      - host: 172.16.5.85
        config:
          logger.level: "info"
        learner_config:
          server.labels:
            zone: "z2"
    ```

    以前のバージョンの `flash.proxy.labels` 構成は、利用可能なゾーン名に特殊文字を正しく処理できません。利用可能なゾーンの名前を構成するには、`learner_config` 内の `server.labels` を使用することを推奨します。

2. クラスタを起動した後、レプリカを作成する際にラベルを指定します。

    ```sql
    ALTER TABLE テーブル名 SET TIFLASH REPLICA レプリカ数 LOCATION LABELS ゾーンラベル;
    ```

    例:

    ```sql
    ALTER TABLE t SET TIFLASH REPLICA 2 LOCATION LABELS "zone";
    ```

3. PD はラベルに基づいてレプリカをスケジュールします。この例では、PD は表 `t` の二つのレプリカをそれぞれ二つの利用可能なゾーンにスケジュールします。スケジュールを表示するには pd-ctl を使用できます。

    ```shell
    > tiup ctl:v<CLUSTER_VERSION> pd -u http://<PD_ADDRESS>:2379 store

        ...
        "address": "172.16.5.82:23913",
        "labels": [
          { "key": "engine", "value": "tiflash"},
          { "key": "zone", "value": "z1" }
        ],
        "region_count": 4,

        ...
        "address": "172.16.5.81:23913",
        "labels": [
          { "key": "engine", "value": "tiflash"},
          { "key": "zone", "value": "z1" }
        ],
        "region_count": 5,
        ...

        "address": "172.16.5.85:23913",
        "labels": [
          { "key": "engine", "value": "tiflash"},
          { "key": "zone", "value": "z2" }
        ],
        "region_count": 9,
        ...
    ```

<CustomContent platform="tidb">

ラベルを使用してレプリカをスケジュールする詳細については、[地理トポロジラベルによるスケジュール](/schedule-replicas-by-topology-labels.md)、[1 つの都市に複数のデータセンターを展開](/multi-data-centers-in-one-city-deployment.md)、および [2 つの都市に 3 つのデータセンターを展開](/three-data-centers-in-two-cities-deployment.md) を参照してください。

TiFlash は異なるゾーン向けにレプリカ選択戦略を設定できます。詳細については、[`tiflash_replica_read`](/system-variables.md#tiflash_replica_read-new-in-v730) を参照してください。

</CustomContent>
```