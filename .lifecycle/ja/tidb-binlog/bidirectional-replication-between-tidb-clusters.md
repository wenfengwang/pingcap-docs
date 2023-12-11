---
title: TiDBクラスター間の双方向レプリケーション
summary: TiDBクラスター間での双方向レプリケーションの実行方法を学びます。
aliases: ['/docs/dev/tidb-binlog/bidirectional-replication-between-tidb-clusters/','/docs/dev/reference/tidb-binlog/bidirectional-replication/']
---

# TiDBクラスター間の双方向レプリケーション

> **警告:**
>
> - 現在、双方向レプリケーションはまだ実験的な機能です。本番環境での使用は**推奨されていません**。
> - TiDB BinlogはTiDB v5.0で導入された一部の機能と互換性がないため、一緒に使用することはできません。詳細については、[注意事項](/tidb-binlog/tidb-binlog-overview.md#notes)を参照してください。
> - TiDB v7.5.0からは、TiDB Binlogのデータレプリケーション機能の技術サポートはもはや提供されません。データレプリケーションの代替ソリューションとして[TiCDC](/ticdc/ticdc-overview.md)の使用が強く推奨されます。
> - TiDB v7.5.0ではTiDB Binlogのリアルタイムバックアップとリストア機能は引き続きサポートされていますが、このコンポーネントは将来のバージョンで完全に廃止されます。データの復旧の代替ソリューションとして[PITR](/br/br-pitr-guide.md)の使用が推奨されます。

このドキュメントでは、二つのTiDBクラスター間の双方向レプリケーション、レプリケーションの動作方法、有効化方法、およびDDL操作のレプリケーション方法について説明します。

## ユーザーシナリオ

二つのTiDBクラスター間でデータ変更を交換したい場合は、TiDB Binlogを使用してそれを行うことができます。たとえば、クラスターAとクラスターBが互いのデータをレプリケートすることができます。

> **注意:**
>
> これらの二つのクラスターに書き込まれるデータは競合がない必要があります。つまり、二つのクラスターにおいて、同じ主キーまたはテーブルの一意なインデックスの行は変更されていない状態でなければなりません。

ユーザーシナリオは以下の通りです:

![Architect](/media/binlog/bi-repl1.jpg)

## 実装の詳細

![Mark Table](/media/binlog/bi-repl2.png)

クラスターAとクラスターB間で双方向レプリケーションが有効になっている場合、クラスターAに書き込まれたデータはクラスターBにレプリケートされ、そしてこれらのデータの変更はクラスターAに再度レプリケートされます。これにより、レプリケーションの無限ループが発生します。上記の図から、データレプリケーション中にDrainerはbinlogイベントにマークを付け、このようなレプリケーションループを回避するためにマークされたイベントをフィルタリングします。

詳細な実装は以下の通りです:

1. 二つのクラスターそれぞれに対してTiDB Binlogレプリケーションプログラムを起動します。
2. クラスターAのDrainerを通過するレプリケートされるトランザクションは、このDrainerが[`_drainer_repl_mark`テーブル](#mark-table)をトランザクションに追加し、このDMLイベントの更新をマークテーブルに書き込み、そしてこのトランザクションをクラスターBにレプリケートします。
3. クラスターBは、`_drainer_repl_mark`マークテーブルのbinlogイベントをクラスターAに返します。クラスターBのDrainerは、binlogイベントを解析する際に`_drainer_repl_mark`マークテーブルを識別し、このbinlogイベントをクラスターAにレプリケートするのを放棄します。

クラスターBからクラスターAへのレプリケーションプロセスも同様です。二つのクラスターは互いに上流と下流になります。

> **注意:**
>
> *`_drainer_repl_mark`マークテーブルを更新する際には、データ変更が必要であるため、binlogを生成する必要があります。
> * DDL操作はトランザクションではないため、DDL操作をレプリケートするには片方向のレプリケーションメソッドを使用する必要があります。詳細については[DDL操作のレプリケーション](#replicate-ddl-operations)を参照してください。

Drainerは、各ダウンストリームへの接続ごとに一意のIDを使用して競合を回避できます。 `channel_id`は双方向レプリケーションのチャンネルを示すために使用されます。 二つのクラスターは同じ`channel_id`構成を持っている必要があります（同じ値を持っている）。

上流で列を追加または削除すると、ダウンストリームにレプリケートするデータに余分な列があるか、欠落している可能性があります。 Drainerは、余分な列を無視したり、欠落している列にデフォルト値を挿入することによってこの状況を許容します。

## マークテーブル

`_drainer_repl_mark`のマークテーブルは以下の構造を持ちます:

{{< copyable "sql" >}}

```sql
CREATE TABLE `_drainer_repl_mark` (
  `id` bigint(20) NOT NULL,
  `channel_id` bigint(20) NOT NULL DEFAULT '0',
  `val` bigint(20) DEFAULT '0',
  `channel_info` varchar(64) DEFAULT NULL,
  PRIMARY KEY (`id`,`channel_id`)
);
```

Drainerは次のSQLステートメントを使用して`_drainer_repl_mark`を更新し、データ変更とbinlogの生成を保証します:

{{< copyable "sql" >}}

```sql
update drainer_repl_mark set val = val + 1 where id = ? && channel_id = ?;
```

## DDL操作のレプリケーション

DrainerはDDL操作にマークテーブルを追加できないため、DDL操作をレプリケートするには片方向のレプリケーション方法しか使用できません。

例えば、クラスターAからクラスターBへのDDLレプリケーションが有効になっている場合、クラスターBからクラスターAへのレプリケーションは無効になります。つまり全てのDDL操作はクラスターAで実行されます。

> **注意:**
>
> DDL操作は同時に二つのクラスターで実行できません。DDL操作が実行されている場合、同時にDML操作が実行されているか、DML binlogがレプリケートされている場合は、DDLレプリケーションの上流と下流のテーブル構造が一貫性がなくなる可能性があります。

## 双方向レプリケーションの設定と有効化

クラスターAとクラスターB間の双方向レプリケーションのために、全てのDDL操作がクラスターAで実行されると仮定します。クラスターAからクラスターBへのレプリケーションパスには、Drainerに以下の構成を追加します:

{{< copyable "" >}}

```toml
[syncer]
loopback-control = true
channel-id = 1 # レプリケートされる両クラスターに同じIDを構成します。
sync-ddl = true # DDLレプリケーションを実行する必要がある場合は有効にします。

[syncer.to]
# 1はSyncFullColumnで、2はSyncPartialColumnです。
# SyncPartialColumnに設定すると、Drainerは下流テーブル構造に
# レプリケート対象のデータよりも多いまたは少ない列を許可し、
# STRICT_TRANS_TABLESのSQLモードを削除し、少ない列にはゼロ値を挿入します。
sync-mode = 2

# チェックポイントテーブルを無視します。
[[syncer.ignore-table]]
db-name = "tidb_binlog"
tbl-name = "checkpoint"
```

クラスターBからクラスターAへのレプリケーションパスには、Drainerに以下の構成を追加します:

{{< copyable "" >}}

```toml
[syncer]
loopback-control = true
channel-id = 1 # レプリケートされる両クラスターに同じIDを構成します。
sync-ddl = false  # DDLレプリケーションを実行する必要がない場合は無効にします。

[syncer.to]
# 1はSyncFullColumnで、2はSyncPartialColumnです。
# SyncPartialColumnに設定すると、Drainerは下流テーブル構造に
# レプリケート対象のデータよりも多いまたは少ない列を許可し、
# STRICT_TRANS_TABLESのSQLモードを削除し、少ない列にはゼロ値を挿入します。
sync-mode = 2

# チェックポイントテーブルを無視します。
[[syncer.ignore-table]]
db-name = "tidb_binlog"
tbl-name = "checkpoint"
```