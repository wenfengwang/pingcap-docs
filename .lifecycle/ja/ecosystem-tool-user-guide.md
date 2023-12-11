---
title: TiDBツールの概要
summary: ツールと適用シナリオを学ぶ
aliases: ['/docs/dev/ecosystem-tool-user-guide/', '/docs/dev/reference/tools/user-guide/', '/docs/dev/how-to/migrate/from-mysql/', '/docs/dev/how-to/migrate/incrementally-from-mysql/', '/docs/dev/how-to/migrate/overview/']
---

# TiDBツールの概要

TiDBは、TiDBを展開および維持し、データ（データの移行、バックアップ＆リストア、データの比較など）を管理し、TiKV上でSpark SQLを実行するのに役立つ豊富なツールセットを提供しています。必要に応じて適用可能なツールを選択できます。

## 展開および操作ツール

TiDBは、TiUPとTiDB Operatorを提供し、さまざまなシステム環境での展開と運用のニーズに応えます。

### 物理マシンまたは仮想マシン上でTiDBを展開および操作 - TiUP

[TiUP](/tiup/tiup-overview.md) は、物理マシンまたは仮想マシン上でのTiDBパッケージマネージャです。 TiUPは、TiDB、PD、およびTiKVなど、複数のTiDBコンポーネントを管理できます。 TiDBエコシステム内の任意のコンポーネントを起動するには、1行のTiUPコマンドを実行するだけです。

TiUPはGolangで書かれたクラスタ管理コンポーネントである[TiUPクラスタ](https://github.com/pingcap/tiup/tree/master/components/cluster)を提供しています。 TiUPクラスタを使用することで、TiDBクラスタのデプロイ、起動、停止、破壊、スケーリング、アップグレード、およびTiDBクラスタパラメーターの管理を簡単に実行できます。

以下は、TiUPの基本です：

- [用語と概念](/tiup/tiup-terminology-and-concepts.md)
- [TiUPを使用したTiDBクラスタの展開](/production-deployment-using-tiup.md)
- [TiUPコマンドを使用したTiUPコンポーネントの管理](/tiup/tiup-component-management.md)
- 適用可能なTiDBバージョン：v4.0以上

### KubernetesでTiDBを展開および操作 - TiDB Operator

[TiDB Operator](https://github.com/pingcap/tidb-operator) は、Kubernetes上でTiDBクラスタを管理するための自動操作システムです。デプロイ、アップグレード、スケーリング、バックアップ、および構成変更を含む、TiDBのフルライフサイクル管理を提供します。 TiDB Operatorを使用すると、TiDBは、パブリッククラウドまたはプライベートクラウド上に展開されたKubernetesクラスタでシームレスに実行できます。

以下は、TiDB Operatorの基本です：

- [TiDB Operatorアーキテクチャ](https://docs.pingcap.com/tidb-in-kubernetes/stable/architecture)
- [Kubernetes上のTiDB Operatorのはじめかた](https://docs.pingcap.com/tidb-in-kubernetes/stable/get-started/)
- 適用可能なTiDBバージョン：v2.1以上

## データ管理ツール

 TiDBは、複数のデータ管理ツールを提供しており、それにはインポートおよびエクスポート、バックアップとリストア、増分データレプリケーション、およびデータ検証が含まれます。

### データ移行 - TiDBデータ移行（DM）

[TiDBデータ移行](/dm/dm-overview.md) （DM）は、MySQL/MariaDBからTiDBへの完全なデータ移行および増分データレプリケーションをサポートするツールです。

以下は、DMの基本です：

- ソース：MySQL/MariaDB
- ターゲット：TiDBクラスタ
- サポートされているTiDBバージョン：すべてのバージョン
- Kubernetesサポート：[TiDB Operator](https://github.com/pingcap/tidb-operator)を使用して、Kubernetes上にTiDB DMを展開します。

データボリュームが1 TB未満の場合は、DMを使用してMySQL/MariaDBからTiDBに直接データを移行することを推奨します。移行プロセスには、完全なデータ移行および増分データレプリケーションが含まれます。

データボリュームが1 TBを超える場合は、次の手順を実行してください。

1. [Dumpling](/dumpling-overview.md) を使用して、MySQL/MariaDBから完全なデータをエクスポートします。
2. [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) を使用して、ステップ1でエクスポートされたデータをTiDBクラスタにインポートします。
3. TiDB DMを使用して、MySQL/MariaDBからTiDBに増分データをレプリケートします。

> **注意：**
>
> Syncerツールはもはやメンテナンスされていません。Syncerに関連するシナリオについては、インクリメンタルレプリケーションを実行するためにDMを使用することをお勧めします。

### フルデータエクスポート - Dumpling

[Dumpling](/dumpling-overview.md) は、MySQLまたはTiDBからの論理的なフルデータエクスポートをサポートしています。

以下は、Dumplingの基本です：

- ソース：MySQL/TiDBクラスタ
- 出力：SQL/CSVファイル
- サポートされているTiDBバージョン：すべてのバージョン
- Kubernetesサポート：いいえ

> **注意：**
>
> PingCAPは以前、TiDBに特化したmydumperプロジェクトのフォークを維持していました。 v7.5.0から、[Mydumper](https://docs.pingcap.com/tidb/v4.0/mydumper-overview) は非推奨となり、そのほとんどの機能が[Dumpling](/dumpling-overview.md)に置き換えられました。mydumperの代わりにDumplingを使用することを強くお勧めします。

### フルデータインポート - TiDB Lightning

[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) は、大容量データセットをTiDBクラスタにフルデータインポートするのをサポートしています。

TiDB Lightningは以下のモードをサポートしています：

- `物理インポートモード`：TiDB Lightningはデータを順序付けられたキー値ペアにパースし、それをTiKVに直接インポートします。このモードは通常、新しいクラスタに大量のデータ（TBレベル）をインポートするために使用されます。インポート中は、クラスタはサービスを提供できません。
- `論理インポートモード`：このモードは、バックエンドにTiDB/MySQLを使用し、`物理インポートモード`よりも遅いですが、オンラインで実行できます。また、MySQLにデータをインポートすることもサポートします。

以下は、TiDB Lightningの基本です：

- データソース：
    - Dumplingの出力ファイル
    - その他の互換性のあるCSVファイル
    - Amazon AuroraまたはApache HiveからエクスポートされたParquetファイル
- サポートされているTiDBバージョン：v2.1以上
- Kubernetesサポート：はい。詳細については、[TiDB Lightningを使用してKubernetes上のTiDBクラスタにデータを迅速に復元する](https://docs.pingcap.com/tidb-in-kubernetes/stable/restore-data-using-tidb-lightning)を参照してください。

> **注意：**
>
> Loaderツールはもはやメンテナンスされていません。Loaderに関連するシナリオについては、代わりに`論理インポートモード`を使用することをお勧めします。

### バックアップおよびリストア - バックアップ＆リストア（BR）

[バックアップ＆リストア](/br/backup-and-restore-overview.md)（BR）は、TiDBクラスタデータの分散バックアップとリストアを行うためのコマンドラインツールです。BRは、巨大なデータ量のTiDBクラスタを効果的にバックアップおよびリストアできます。

以下は、BRの基本です：

- 入力および出力データソース

    - スナップショットバックアップとリストア：[SST + `backupmeta`ファイル](/br/br-snapshot-architecture.md#backup-files)
    - ログバックアップとPITR：[ログバックアップファイル](/br/br-log-architecture.md#log-backup-files)

- サポートされているTiDBバージョン：v4.0以上
- Kubernetesサポート：はい。詳細については、[BRを使用してS3互換ストレージにデータをバックアップ](https://docs.pingcap.com/tidb-in-kubernetes/stable/backup-to-aws-s3-using-br)および[BRを使用してS3互換ストレージからデータをリストア](https://docs.pingcap.com/tidb-in-kubernetes/stable/restore-from-aws-s3-using-br)を参照してください。

### 増分データレプリケーション - TiCDC

[TiCDC](/ticdc/ticdc-overview.md) は、TiKVから変更ログを取得してTiDBの増分データをレプリケートするためのツールです。TiCDCは、上流の任意のTSOに対応した状態にデータを復元できます。また、TiCDCは、他のシステムがデータ変更をサブスクライブできるようにするためのTiCDCオープンプロトコルを提供しています。

以下は、TiCDCの基本です：

- ソース：TiDBクラスタ
- ターゲット：TiDBクラスタ、MySQL、Kafka、およびConfluent
- サポートされているTiDBバージョン：v4.0.6以上

### 増分ログレプリケーション - TiDB Binlog

[TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md) は、TiDBクラスタのためのバイナリログを収集し、ほぼリアルタイムのデータレプリケーションおよびバックアップを提供するツールです。TiDB Binlogを使用して、TiDBクラスタ間での増分データレプリケーションに使用できます。

以下は、TiDB Binlogの基本です：

- ソース：TiDBクラスタ
- ターゲット：TiDBクラスタ、MySQL、Kafka、または増分バックアップファイル
- サポートされているTiDBバージョン：v2.1以上
- Kubernetesサポート：はい。詳細については、[TiDB Binlogクラスタの操作](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-tidb-binlog)および[Kubernetes上のTiDB Binlogドレーナーの構成](https://docs.pingcap.com/tidb-in-kubernetes/stable/configure-tidb-binlog-drainer)を参照してください。

### sync-diff-inspector

[sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) は、MySQLまたはTiDBデータベースに保存されているデータを比較するツールです。さらに、データの不整合が少量あるシナリオでデータを修復するためにも、sync-diff-inspectorを使用できます。

```
The following are the basics of sync-diff-inspector:

- Source: MySQL/TiDB clusters
- Target: MySQL/TiDB clusters
- Supported TiDB versions: all versions

## OLAP Query tool - TiSpark

[TiSpark](/tispark-overview.md) is a product developed by PingCAP to address the complexiy of OLAP queries. It combines strengths of Spark, and the features of distributed TiKV clusters and TiDB to provide a one-stop Hybrid Transactional and Analytical Processing (HTAP) solution.
```


```
以下はsync-diff-inspectorの基本です：

- ソース: MySQL/TiDB クラスタ
- ターゲット: MySQL/TiDB クラスタ
- サポートされているTiDBのバージョン: すべてのバージョン

## OLAPクエリツール - TiSpark

[TiSpark](/tispark-overview.md) は OLAPクエリの複雑さに対処するためにPingCAPが開発した製品です。Sparkの強みと、分散TiKVクラスタとTiDBの機能を組み合わせて、ハイブリッドトランザクショナルおよびアナリティカル処理（HTAP）ソリューションを提供します。
```