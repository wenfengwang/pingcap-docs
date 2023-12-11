---
title: TiDBツールの使用事例
summary: TiDBツールの一般的な使用事例とツールの選択方法について学びます。
aliases: ['/docs/dev/ecosystem-tool-user-case/']
---

# TiDBツールの使用事例

このドキュメントでは、TiDBツールの一般的な使用事例とシナリオに適した適切なツールの選択方法について紹介します。

## 物理マシンまたは仮想マシン上でTiDBを展開および運用

物理マシンまたは仮想マシン上でTiDBを展開および運用する必要がある場合、[TiUP](/tiup/tiup-overview.md)をインストールし、TiUPを使用してTiDB、PD、TiKVなどのTiDBコンポーネントを管理できます。

## Kubernetes上でTiDBを展開および運用

Kubernetes上でTiDBを展開および運用する必要がある場合、Kubernetesクラスタを展開し、[TiDB Operator](https://docs.pingcap.com/tidb-in-kubernetes/stable)を展開します。その後、TiDB Operatorを使用してTiDBクラスタを展開および運用できます。

## CSVからTiDBへのデータインポート

他のツールによってエクスポートされた互換性のあるCSVファイルをTiDBにインポートする必要がある場合、[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用します。

## MySQL/Auroraから全データのインポート

MySQL/Auroraから全データをインポートする必要がある場合、まず[Dumpling](/dumpling-overview.md)を使用してデータをSQLダンプファイルとしてエクスポートし、その後[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用してデータをTiDBクラスタにインポートします。

## MySQL/Auroraからデータを移行

MySQL/Auroraから全データと増分データの両方を移行する必要がある場合、[TiDBデータ移行](/dm/dm-overview.md)（DM）を使用して[Amazon AuroraからTiDBへのデータ移行](/migrate-aurora-to-tidb.md)を実行します。

データ量が大きい場合（TBレベル）、まず[Dumpling](/dumpling-overview.md)と[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用して全データ移行を行い、その後、DMを使用して増分データの移行を行うことができます。

## TiDBクラスタのバックアップとリストア

TiDBクラスタをバックアップしたり、バックアップされたデータをクラスタにリストアしたい場合は、[BR](/br/backup-and-restore-overview.md)（Backup & Restore）を使用します。

さらに、BRはTiDBクラスタデータの[増分バックアップ](/br/br-incremental-guide.md#back-up-incremental-data) および[増分リストア](/br/br-incremental-guide.md#restore-incremental-data)を実行するためにも使用できます。

## TiDBへのデータ移行

TiDBクラスタから別のTiDBクラスタにデータを移行する必要がある場合、[Dumpling](/dumpling-overview.md)を使用してTiDBから全データをSQLダンプファイルとしてエクスポートし、その後、[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用して別のTiDBクラスタにデータをインポートします。

増分データの移行も必要な場合は、[TiCDC](/ticdc/ticdc-overview.md)を使用できます。

## TiDBの増分データサブスクリプション

TiDBの増分変更にサブスクライブする必要がある場合、[TiCDC](/ticdc/ticdc-overview.md)を使用できます。