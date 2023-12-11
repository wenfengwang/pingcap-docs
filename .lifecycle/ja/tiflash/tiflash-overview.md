---
title: TiFlashの概要
summary: TiFlashのアーキテクチャと主要機能について学びます。
aliases: ['/docs/dev/tiflash/tiflash-overview/', '/docs/dev/reference/tiflash/overview/', '/docs/dev/tiflash/use-tiflash/', '/docs/dev/reference/tiflash/use-tiflash/', '/tidb/dev/use-tiflash']
---

# TiFlashの概要

[TiFlash](https://github.com/pingcap/tiflash)は、Essentially a Hybrid Transactional/Analytical Processing (HTAP) databaseを実現するキーコンポーネントです。TiKVのカラムストレージ拡張機能として、TiFlashは、優れた分離レベルと強力な一貫性を提供します。

TiFlashでは、カラム形式のレプリカはRaft Learnerコンセンサスアルゴリズムに従って非同期にレプリケーションされます。これらのレプリカを読み取ると、Snapshot Isolationレベルの一貫性がRaftインデックスとマルチバージョン同時実行制御（MVCC）を検証することで達成されます。

<CustomContent platform="tidb-cloud">

TiDB Cloudを使用すると、HTAPワークロードに応じて1つ以上のTiFlashノードを指定することで、簡単にHTAPクラスタを作成できます。クラスタを作成する際、または後からTiFlashノードの数を指定しない場合は、[クラスタのスケーリング](/tidb-cloud/scale-tidb-cluster.md)でノード数を変更できます。

</CustomContent>

## アーキテクチャ

![TiFlashアーキテクチャ](/media/tidb-storage-architecture-1.png)

上の図は、TiFlashノードを含むHTAP形式のTiDBのアーキテクチャです。

TiFlashは、カラムストレージを提供し、ClickHouseによって効率的に実装されたコプロセッサの層を備えています。TiKVと同様に、TiFlashには多重Raftシステムもあり、領域の単位でデータのレプリケーションと分散がサポートされています（詳細は[データストレージ](https://en.pingcap.com/blog/tidb-internal-data-storage/)を参照）。

TiFlashはTiKVでの書き込みをブロックせずに低コストでデータのリアルタイムレプリケーションを行います。また、TiKVと同じ読み取り一貫性を提供し、最新のデータが読み取られることを保証します。TiFlashの領域レプリカは、論理的にTiKVのものと同じであり、TiKVのリーダーレプリカと同時に分割とマージが行われます。

Linux AMD64アーキテクチャにTiFlashをデプロイするには、CPUはAVX2命令セットをサポートする必要があります。`cat /proc/cpuinfo | grep avx2`を実行して出力があることを確認してください。Linux ARM64アーキテクチャにTiFlashをデプロイするには、CPUはARMv8命令セットアーキテクチャをサポートする必要があります。`cat /proc/cpuinfo | grep 'crc32' | grep 'asimd'`を実行して出力があることを確認してください。これらの命令セット拡張を使用することで、TiFlashのベクトル化エンジンがより優れたパフォーマンスを提供できます。

<CustomContent platform="tidb">

TiFlashは、TiDBとTiSparkの両方と互換性があります。これにより、これら2つのコンピューティングエンジンの選択が自由に行えます。

</CustomContent>

TiKVとは異なるノードにTiFlashをデプロイすることをお勧めし、ワークロードの分離を確保してください。ビジネス分離が必要ない場合は、TiFlashとTiKVを同じノードにデプロイすることも可能です。

現在、データは直接TiFlashに書き込むことはできません。TiKVにデータを書き込んでからTiFlashにレプリケートする必要があります。これは、TiDBクラスタにLearnerの役割で接続されているためです。TiFlashはテーブルのレプリケーションに対応していますが、デプロイ後はデフォルトでデータはレプリケートされません。特定のテーブルのデータをレプリケートするには、[テーブル用のTiFlashレプリカを作成](/tiflash/create-tiflash-replicas.md#テーブル用のtiflashレプリカを作成)を参照してください。

TiFlashには3つのコンポーネントがあります: カラムストレージモジュール、`tiflash proxy`、および`pd buddy`です。`tiflash proxy`はMulti-Raftコンセンサスアルゴリズムを使用した通信に責任があります。`pd buddy`はPDと連携して、テーブルの単位でTiKVからTiFlashへデータをレプリケートします。

TiDBがTiFlashにレプリカを作成するDDLコマンドを受信すると、`pd buddy`コンポーネントはTiDBのステータスポートを介してレプリケートするテーブルの情報を取得し、その情報をPDに送信します。その後、PDは`pd buddy`に提供された情報に基づいて対応するデータスケジューリングを行います。

## 主要機能

TiFlashには、以下の主要機能があります。

- [非同期レプリケーション](#非同期レプリケーション)
- [一貫性](#一貫性)
- [インテリジェントな選択](#インテリジェントな選択)
- [計算の加速](#計算の加速)

### 非同期レプリケーション

TiFlashのレプリカは特別な役割であるRaft Learnerとして非同期にレプリケーションされます。これは、TiFlashノードがダウンした場合やネットワーク遅延が発生した場合でも、TiKVのアプリケーションが通常通り進行できることを意味します。

このレプリケーションメカニズムには、TiKVの自動負荷分散と高い可用性という2つの利点が引き継がれています。

- TiFlashは追加のレプリケーションチャネルに依存せず、多対多でTiKVからデータを直接受信します。
- TiKVでデータが失われない限り、いつでもTiFlashのレプリカを復元できます。

### 一貫性

TiFlashは、TiKVと同じSnapshot Isolationレベルの一貫性を提供し、最新のデータが読み取られることを保証します。つまり、以前にTiKVで書かれたデータを読むことができます。このような一貫性は、データレプリケーションの進行状況を検証することで達成されます。

TiFlashが読み取りリクエストを受信するたびに、領域レプリカは進行状況の検証リクエスト（軽量なRPCリクエスト）をリーダーレプリカに送信します。TiFlashは、現在のレプリケーション進行状況が読み取りリクエストのタイムスタンプでカバーされるデータを含む場合にのみ読み取り操作を行います。

### インテリジェントな選択

TiDBは、TiFlash（カラム形式）またはTiKV（行形式）のいずれか、あるいは両方をクエリ内で自動的に選択して最適なパフォーマンスを確保します。

この選択メカニズムは、TiDBが異なるインデックスを選択してクエリを実行するメカニズムと類似しています。TiDBオプティマイザは、読み取りコストの統計に基づいて適切な選択を行います。

### 計算の加速

TiFlashは、TiDBの計算を次の2つの方法で高速化します。

- カラムストレージエンジンの読み取り操作が効率的に行われます。
- TiFlashは、TiDBの一部の計算作業を共有します。

TiFlashは、TiKV Coprocessorのように計算作業を共有します: TiDBはストレージレイヤで完了できる計算をプッシュダウンします。計算をプッシュダウンできるかどうかは、TiFlashのサポートに依存します。詳細については、[サポートされているプッシュダウン計算](/tiflash/tiflash-supported-pushdown-calculations.md)を参照してください。

## TiFlashの使用

TiFlashをデプロイした後、データのレプリケーションは自動的に開始しません。レプリケーションするテーブルを手動で指定する必要があります。

<CustomContent platform="tidb">

中規模の解析処理にはTiDBを使用してTiFlashレプリカを読み取るか、大規模な解析処理にはTiSparkを使用してTiFlashレプリカを読み取るか、使用目的に応じて以下のセクションを参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

解析処理にはTiDBを使用してTiFlashレプリカを読み取ります。詳細については以下のセクションを参照してください。

</CustomContent>

- [TiFlashレプリカの作成](/tiflash/create-tiflash-replicas.md)
- [TiDBでTiFlashレプリカを読む方法](/tiflash/use-tidb-to-read-tiflash.md)

<CustomContent platform="tidb">

- [TiSparkでTiFlashレプリカを読む方法](/tiflash/use-tispark-to-read-tiflash.md)

</CustomContent>

- [MPPモードの使用](/tiflash/use-tiflash-mpp-mode.md)

<CustomContent platform="tidb">

データをインポートしてからクエリを実行するまでのプロセス全体を体験するには、[TiDB HTAPクイックスタートガイド](/quick-start-with-htap.md)を参照してください。

</CustomContent>

## 関連項目

<CustomContent platform="tidb">

-TiFlashノードを使用して新しいクラスタをデプロイする方法については、[TiUPを使用したTiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を参照してください。
-デプロイ済みクラスタにTiFlashノードを追加する方法については、[TiFlashクラスタをスケールアウトする](/scale-tidb-using-tiup.md#scale-out-a-tiflash-cluster)を参照してください。
-[TiFlashクラスタのメンテナンス](/tiflash/maintain-tiflash.md)。
-[TiFlashのパフォーマンスチューニング](/tiflash/tune-tiflash-performance.md)。
-[TiFlashの構成](/tiflash/tiflash-configuration.md)。
-[TiFlashクラスタのモニタリング](/tiflash/monitor-tiflash.md)。
-[TiFlashアラートルールの学習](/tiflash/tiflash-alert-rules.md)。
-[TiFlashクラスタのトラブルシューティング](/tiflash/troubleshoot-tiflash.md)。
-[TiFlashでサポートされるプッシュダウン計算](/tiflash/tiflash-supported-pushdown-calculations.md)。
-[TiFlashでのデータ検証](/tiflash/tiflash-data-validation.md)。
-[TiFlashの互換性](/tiflash/tiflash-compatibility.md)。

</CustomContent>

<CustomContent platform="tidb-cloud">

-[TiFlashのパフォーマンスチューニング](/tiflash/tune-tiflash-performance.md)。
-[TiFlashでサポートされるプッシュダウン計算](/tiflash/tiflash-supported-pushdown-calculations.md)。
-[TiFlashの互換性](/tiflash/tiflash-compatibility.md)。

</CustomContent>