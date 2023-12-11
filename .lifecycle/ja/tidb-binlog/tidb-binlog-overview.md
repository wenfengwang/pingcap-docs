---
title: TiDB Binlogの概要
summary: TiDB Binlogのクラスターバージョンの概要を学ぶ
aliases: ['/docs/jp/dev/tidb-binlog/tidb-binlog-overview/','/docs/jp/dev/reference/tidb-binlog/overview/','/docs/jp/dev/reference/tidb-binlog-overview/','/docs/jp/dev/reference/tools/tidb-binlog/overview/']
---

# TiDB Binlogクラスターの概要

このドキュメントでは、TiDB Binlogのクラスターバージョンのアーキテクチャとデプロイについて紹介します。

TiDB Binlogは、TiDBからのbinlogデータを収集し、近いリアルタイムのバックアップと下流プラットフォームへのレプリケーションを提供するためのツールです。

TiDB Binlogには以下の機能があります：

- **データレプリケーション:** TiDBクラスター内のデータを他のデータベースにレプリケート
- **リアルタイムバックアップとリストア:** TiDBクラスターのデータをバックアップし、クラスターが障害を起こした場合にTiDBクラスターを復元

> **注意:**
>
> - TiDB BinlogはTiDB v5.0で導入された一部の機能と互換性がないため、一緒に使用できません。詳細については[ノート](#notes)を参照してください。
> - TiDB v7.5.0以降、TiDB Binlogのデータレプリケーション機能の技術サポートは提供されなくなります。データレプリケーションの代替ソリューションとして[TiCDC](/ticdc/ticdc-overview.md)の使用を強く推奨します。
> - TiDB v7.5.0でも、リアルタイムバックアップとリストア機能は引き続きサポートされますが、このコンポーネントは将来のバージョンで完全に廃止される予定です。データの回復の代替ソリューションとして[PITR](/br/br-pitr-guide.md)の使用を推奨します。

## TiDB Binlogのアーキテクチャ

TiDB Binlogのアーキテクチャは次のとおりです:

![TiDB Binlogのアーキテクチャ](/media/tidb-binlog-cluster-architecture.png)

TiDB BinlogクラスターはPumpとDrainerで構成されています。

### Pump

[Pump](https://github.com/pingcap/tidb-binlog/blob/master/pump)は、TiDBで生成されたbinlogを記録し、トランザクションのコミット時間に基づいてbinlogを並べ替え、Drainerにbinlogを送信するために使用されます。

### Drainer

[Drainer](https://github.com/pingcap/tidb-binlog/tree/master/drainer)は、各Pumpからbinlogを収集しマージし、binlogを特定の形式のSQLまたはデータに変換し、そのデータを特定の下流プラットフォームにレプリケートします。

### `binlogctl`ガイド

[`binlogctl`](https://github.com/pingcap/tidb-binlog/tree/master/binlogctl)は、TiDB Binlog用の操作ツールで、次の機能があります:

- TiDBクラスターの現在の`ts o`を取得する
- Pump/Drainerの状態を確認する
- Pump/Drainerの状態を変更する
- Pump/Drainerを一時停止またはクローズする

## 主な機能

- 複数のPumpがクラスターを形成し、水平方向にスケーリングできる
- TiDBは組み込みのPumpクライアントを使用して、各Pumpにbinlogを送信する
- Pumpはbinlogを保存し、順番にDrainerにbinlogを送信する
- Drainerは各Pumpのbinlogを読み取り、マージし並べ替え、次にbinlogを下流に送信します
- Drainerは[relay log](/tidb-binlog/tidb-binlog-relay-log.md)をサポートしており、リレーログにより、Drainerは下流クラスターが一貫した状態になっていることを確認します。

## ノート

* v5.1では、v5.0で導入されたクラスターインデックス機能とTiDB Binlogの非互換性が解消されました。TiDB BinlogとTiDB Serverをv5.1にアップグレードし、TiDB Binlogを有効にした後、TiDBはクラスターインデックスを持つテーブルを作成することをサポートします。クラスターインデックステーブルのデータ挿入、削除、およびアップデートは、TiDB Binlogを介して下流にレプリケートされます。クラスターをv5.1に手動でアップグレードした場合は、TiDBサーバーをv5.1にアップグレードする前に、TiDB binlogがv5.1にアップグレードされていることを確認してください。
また、TiDBクラスター間での構造が一貫するように、システム変数[`tidb_enable_clustered_index`](/system-variables.md#tidb_enable_clustered_index-new-in-v50)を同じ値に設定することをお勧めします。

* TiDB BinlogはTiDB v5.0で導入された以下の機能と互換性がないため、一緒に使用できません。

    - [TiDB Clustered Index](/clustered-indexes.md#limitations): TiDB Binlogを有効にした後、TiDBではプライマリキーとして非単一整数列を持つクラスターインデックスを作成することができません。作成したクラスターインデックステーブルのデータ挿入、削除、およびアップデートはTiDB Binlogを介して下流にレプリケートされません。クラスターインデックスを持つテーブルをレプリケートする必要がある場合は、クラスターをv5.1にアップグレードするか、代わりに[TiCDC](/ticdc/ticdc-overview.md)を使用してください。
    - TiDBシステム変数[tidb_enable_async_commit](/system-variables.md#tidb_enable_async_commit-new-in-v50): TiDB Binlogを有効にした後、このオプションを有効にしてもパフォーマンスが向上しません。TiDB Binlogの代わりに[TiCDC](/ticdc/ticdc-overview.md)を使用することをお勧めします。
    - TiDBシステム変数[tidb_enable_1pc](/system-variables.md#tidb_enable_1pc-new-in-v50): TiDB Binlogを有効にした後、このオプションを有効にしてもパフォーマンスが向上しません。TiDB Binlogの代わりに[TiCDC](/ticdc/ticdc-overview.md)を使用することをお勧めします。

* DrainerはMySQL、TiDB、Kafka、またはローカルファイルにbinlogをレプリケートすることができます。Drainerでサポートされていない他の宛先にbinlogをレプリケートする必要がある場合は、DrainerをKafkaにbinlogをレプリケートするように設定し、binlogコンシューマープロトコルに従ってカスタム処理を行うためにKafkaのデータを読み取ることができます。[Binlog Consumer Client User Guide](/tidb-binlog/binlog-consumer-client.md)をご覧ください。

* 増分データを回復するためにTiDB Binlogを使用する場合は、`db-type`の設定を`file`(protobuf formaのローカルファイル)に設定してください。Drainerはbinlogを指定された[proto buffer format](https://github.com/pingcap/tidb-binlog/blob/master/proto/pb_binlog.proto)のデータに変換し、そのデータをローカルファイルに書き込みます。これにより、[Reparo](/tidb-binlog/tidb-binlog-reparo.md)を使用して増分データを回復することができます。

    `db-type`の値に注意してください:

    - TiDBのバージョンが2.1.9よりも前の場合は、`db-type="pb"`に設定します。
    - TiDBのバージョンが2.1.9以降の場合は、`db-type="file"`または`db-type="pb"`に設定します。

* 下流がMySQL、MariaDB、または別のTiDBクラスターの場合、データレプリケーション後にデータを検証するには[sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)を使用できます。