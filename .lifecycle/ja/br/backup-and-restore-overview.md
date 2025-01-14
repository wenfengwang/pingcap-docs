---
title: TiDBバックアップ＆リストアの概要
summary: TiDBバックアップ＆リストアの定義と機能について学びます。
aliases: ['/docs/dev/br/backup-and-restore-tool/','/docs/dev/reference/tools/br/br/','/docs/dev/how-to/maintain/backup-and-restore/br/','/tidb/dev/backup-and-restore-tool/','/tidb/dev/point-in-time-recovery/']
---

# TiDBバックアップ＆リストアの概要

Raftプロトコルと適切なデプロイメントトポロジに基づいて、TiDBはクラスタの高可用性を実現します。クラスタ内のいくつかのノードが障害になった場合でも、クラスタは利用可能なままです。この基盤の上で、さらなるデータの安全性を確保するために、TiDBはバックアップ＆リストア（BR）機能を提供しています。これは、自然災害やミス操作からデータを回復する最後の手段となります。

BRは、以下の要件を満たします：

- 災害シナリオでのデータ損失を減らすため、クラスタデータを災害復旧（DR）システムに5分間隔でバックアップすることができます。
- アプリケーションの誤操作による問題を扱うため、エラーイベントの前の時間ポイントにデータをロールバックすることができます。
- 司法監督の要件を満たすために、過去のデータ監査を実行できます。
- トラブルシューティングやパフォーマンスチューニング、シミュレーションテストなどに便利な、プロダクション環境のクローンを作成できます。

## 使用する前に

このセクションでは、TiDBバックアップおよびリストアの使用に必要な前提条件について説明します。制約事項、使用上のヒント、互換性の問題などが含まれます。

### 制約事項

- PITRは**空のクラスタ**にのみデータをリストアできます。
- PITRはクラスタ単位でのリストアにのみ対応し、データベース単位またはテーブル単位のリストアはサポートしていません。
- PITRは、ユーザーテーブルや権限テーブルのデータをシステムテーブルからリストアすることをサポートしていません。
- BRは、**同時に複数のバックアップタスクをクラスタ上で実行することはサポートしていません**。
- PITRが実行中の場合、ログバックアップタスクを実行したり、TiCDCを使用してデータをダウンストリームクラスタにレプリケートすることはできません。

### 注意事項

スナップショットバックアップ：

- アプリケーションに対する影響を最小限に抑えるために、オフピーク時にバックアップ操作を実行することをおすすめします。
- 複数のバックアップまたはリストアタスクを1つずつ実行することをおすすめします。複数のバックアップタスクを並行して実行すると、パフォーマンスが低下する可能性があります。さらに、複数のタスクが連携しない可能性があるため、タスクの失敗が発生し、クラスタのパフォーマンスに影響を与える場合があります。

スナップショットリストア：

- BRは、可能な限りターゲットクラスタのリソースを使用します。したがって、データを新しいクラスタまたはオフラインクラスタにリストアすることをおすすめします。本番クラスタにデータをリストアすると、アプリケーションに必ず影響が出ます。

バックアップストレージとネットワークの設定：

- バックアップデータをAmazon S3、GCS、またはAzure Blob Storageと互換性のあるストレージシステムに保存することをおすすめします。
- BR、TiKV、およびバックアップストレージシステムが十分なネットワーク帯域幅を持ち、バックアップおよびリストア中に十分な読み書きパフォーマンス（IOPS）を提供できるようにする必要があります。これらがパフォーマンスのボトルネックになる可能性があります。

## バックアップとリストアの使用

BRの使用方法は、TiDBのデプロイ方法によって異なります。このドキュメントでは、オンプレミスデプロイメントでbrコマンドラインツールを使用してTiDBクラスタデータをバックアップおよびリストアする方法について説明します。

他のデプロイシナリオでこの機能を使用する方法については、以下のドキュメントを参照してください：

- [TiDB クラウド上でのバックアップとリストア](https://docs.pingcap.com/tidbcloud/backup-and-restore): [TiDB クラウド](https://www.pingcap.com/tidb-cloud/?from=en) で TiDB クラスタを作成することをおすすめします。TiDB クラウドでは、完全管理されたデータベースを提供し、アプリケーションに集中できます。
- [TiDB Operator を使用してのデータのバックアップとリストア](https://docs.pingcap.com/tidb-in-kubernetes/stable/backup-restore-overview): Kubernetes 上で TiDB Operator を使用して TiDB クラスタをデプロイする場合は、Kubernetes CustomResourceDefinition (CRD) を使用してデータをバックアップおよびリストアすることをおすすめします。

## BRの機能

TiDB BRは、以下の機能を提供します：

- クラスタデータのバックアップ：指定した時点でクラスタのすべてのデータ（フルバックアップ）をバックアップしたり、TiDBのデータの変更（ログバックアップ）をバックアップしたりすることができます（ログとは、TiKVのKV変更を指します）。

- バックアップデータのリストア：

    - フルバックアップをリストアすることができます。
    - バックアップデータ（フルバックアップおよびログバックアップ）を基にして、バックアップクラスタの任意の時点までターゲットクラスタをリストアすることができます。このタイプのリストアは、時間ポイントリカバリ（PITR）と呼ばれます。

### クラスタデータのバックアップ

フルバックアップは、特定の時点でクラスタのすべてのデータをバックアップします。TiDBでは、以下のフルバックアップの方法をサポートしています：

- クラスタスナップショットのバックアップ：TiDBクラスタのスナップショットには、特定の時点でトランザクション的に一貫性のあるデータが含まれます。詳細については、[スナップショットバックアップ](/br/br-snapshot-guide.md#back-up-cluster-snapshots)を参照してください。

フルバックアップは、多くのストレージスペースを占有し、特定の時点のクラスタデータのみを含んでいます。必要に応じてリストアポイントを選択する（つまり、時間ポイントリカバリ（PITR）を実行する）場合は、以下の2つのバックアップ方法を同時に使用することができます：

- [ログバックアップ](/br/br-pitr-guide.md#start-log-backup)を開始します。ログバックアップが開始されると、タスクはすべてのTiKVノードで実行され、指定されたストレージに定期的に小バッチでTiDBの増分データをバックアップします。
- 定期的にスナップショットバックアップを実行します。毎日AM 0:00にクラスタ全体のデータをバックアップストレージにバックアップします。

#### バックアップのパフォーマンスとTiDBクラスタへの影響

- クラスタのCPUとI/Oリソースが十分な場合、スナップショットバックアップはTiDBクラスタにほぼ20%を超えない影響を与えます。適切なTiDBクラスタの設定を行うことで、この影響をさらに10%またはそれ以下に軽減することもできます。リソースが不足している場合、TiDBクラスタのTiKV設定項目[`backup.num-threads`](/tikv-configuration-file.md#num-threads-1)を調整して、バックアップタスクがTiDBクラスタに与える影響を軽減することができます。TiKVノードのバックアップ速度はスケーラブルであり、50 MB/sから100 MB/sまでの範囲です。詳細については、[バックアップのパフォーマンスと影響](/br/br-snapshot-guide.md#performance-and-impact-of-snapshot-backup)を参照してください。
- ログバックアップタスクのみの場合、クラスタへの影響は約5%です。ログバックアップでは、最後のリフレッシュ後に生成されたすべての変更を3-5分ごとにバックアップストレージにフラッシュし、**5分間隔のリカバリポイントオブジェクティブ（RPO）が達成**されます。

### バックアップデータのリストア

バックアップ機能に対応して、フルリストアとPITRの2種類のリストアを実行することができます。

- フルバックアップのリストア

    - クラスタスナップショットバックアップをリストアすることができます。スナップショットバックアップデータを空のクラスタまたはデータの競合がないクラスタ（同じスキーマまたはテーブルを持つ）にリストアすることができます。詳細については、「[スナップショットバックアップのリストア](/br/br-snapshot-guide.md#restore-cluster-snapshots)」を参照してください。また、バックアップデータから特定のデータベースやテーブルをリストアし、不要なデータをフィルタリングすることもできます。詳細については、「[バックアップデータから特定のデータベースやテーブルをリストアする](/br/br-snapshot-guide.md#restore-a-database-or-a-table)」を参照してください。

- 任意のタイムポイント（PITR）へのデータのリストア

    - `br restore point`コマンドを実行することで、リカバリ時刻ポイントより前の最新のスナップショットバックアップデータとログバックアップデータを指定の時間にリストアすることができます。BRは自動的にリストア範囲を決定し、バックアップデータにアクセスし、順番にデータをターゲットクラスタにリストアします。

#### リストアのパフォーマンスとTiDBクラスタへの影響

- データのリストアは、スケーラブルな速度で実行されます。一般的には、TiKVノードあたりの速度は100 MiB/sです。`br`はデータを新しいクラスタにのみリストアし、可能な限りターゲットクラスタのリソースを使用します。詳細については、「[リストアのパフォーマンスと影響](/br/br-snapshot-guide.md#performance-and-impact-of-snapshot-restore)」を参照してください。
- 各TiKVノードでは、PITRではリストア速度は30 GiB/hです。詳細については、「[PITRのパフォーマンスと影響](/br/br-pitr-guide.md#performance-and-impact-of-pitr)」を参照してください。

## バックアップストレージ

TiDBは、Amazon S3、Google Cloud Storage（GCS）、Azure Blob Storage、NFS、およびその他のS3互換のファイルストレージサービスにデータをバックアップすることをサポートしています。詳細については、以下のドキュメントを参照してください：

- [URIでバックアップストレージを指定する](/external-storage-uri.md)
- [バックアップストレージへのアクセス特権の設定](/br/backup-and-restore-storages.md#authentication)

## 互換性

### 他の機能との互換性

バックアップおよびリストアは、いくつかのTiDB機能が有効または無効になっている場合に問題が発生する場合があります。これらの機能がバックアップおよびリストア中に一貫して有効または無効になっていない場合、互換性の問題が発生する可能性があります。

| 機能 | 問題 | 解決策 |
|  ----  | ----  | ----- |
| GBK文字セット || v5.4.0以前のバージョンのBRは、「charset=GBK」のテーブルのリストアをサポートしていません。BRのどのバージョンも、「charset=GBK」のテーブルをv5.4.0以前のTiDBクラスタにリカバリすることはできません。 |
| クラスタードインデックス | [#565](https://github.com/pingcap/br/issues/565) | リストア時の `tidb_enable_clustered_index` グローバル変数の値がバックアップ時と一貫していることを確認してください。そうでないと、`default not found` エラーやデータインデックスの不整合などのデータ不整合が発生する可能性があります。 |
| 新しい照合順序 | [#352](https://github.com/pingcap/br/issues/352)       | リストア時の `new_collations_enabled_on_first_bootstrap` 変数の値がバックアップ時と一貫していることを確認してください。そうでないと、データインデックスの不整合やチェックサムが合格しない可能性があります。詳細については、[FAQ - BRが`new_collations_enabled_on_first_bootstrap`の不一致を報告するのはなぜですか？](/faq/backup-and-restore-faq.md#why-is-new_collations_enabled_on_first_bootstrap-mismatch-reported-during-restore)を参照してください。 |
| グローバル一時テーブル | | バックアップとリストアの際に、BRのバージョンをv5.3.0またはそれ以降のバージョンを使用していることを確認してください。そうでないと、バックアップされたグローバル一時テーブルの定義でエラーが発生します。 |
| TiDB Lightning物理インポート| | 上流データベースがTiDB Lightningの物理インポートモードを使用している場合、ログバックアップでデータをバックアップできません。データインポート後に完全バックアップを実行することをお勧めします。詳細については、[上流データベースがTiDB Lightningを物理インポートモードでデータをインポートすると、なぜログバックアップ機能が利用できなくなるのですか？](/faq/backup-and-restore-faq.md#when-the-upstream-database-imports-data-using-tidb-lightning-in-the-physical-import-mode-the-log-backup-feature-becomes-unavailable-why)を参照してください。|

### バージョンの互換性

> **ノート:**
>
> バックアップおよびリストアには TiDB クラスターのメジャーバージョンと同じバージョンの BR を使用することをお勧めします。

バックアップおよびリストアを実行する前に、BR は TiDB クラスターのバージョンとその互換性を比較します。バージョンが互換性がない場合、BR はエラーを報告して終了します。バージョンチェックを強制的にスキップするには、`--check-requirements=false` を設定できます。バージョンチェックをスキップすると、データの互換性に非互換性が生じる可能性があることに注意してください。

v7.0.0 から、TiDB は SQL 文を介してバックアップとリストア操作を逐次的にサポートしています。そのため、クラスターデータのバックアップとリストア操作を実行する際には、TiDB クラスターと同じメジャーバージョンの BR ツールを使用することを強くお勧めし、メジャーバージョンをまたいだデータのバックアップとリストア操作を避けるようにしてください。これにより、リストア操作のスムーズな実行とデータの一貫性が確保されます。

TiDB v6.6.0 より前の BR の互換性情報は次のとおりです:

| バックアップバージョン（縦） \ リストアバージョン（横）   | TiDB v6.0 にリストア | TiDB v6.1 にリストア | TiDB v6.2 にリストア | TiDB v6.3、v6.4、または v6.5 にリストア | TiDB v6.6 にリストア |
|  ----  |  ----  | ---- | ---- | ---- | ---- |
| TiDB v6.0、v6.1、v6.2、v6.3、v6.4、または v6.5 のスナップショットバックアップ | 互換性あり（既知の問題 [#36379](https://github.com/pingcap/tidb/issues/36379): バックアップデータに空のスキーマが含まれている場合、BR はエラーを報告するかもしれません。） | 互換性あり | 互換性あり | 互換性あり | 互換性あり（BR は v6.6 である必要があります） |
| TiDB v6.3、v6.4、v6.5、または v6.6 のログバックアップ| 非互換 | 非互換 | 非互換 | 互換性あり | 互換性あり |

## 関連項目

- [TiDB スナップショットのバックアップとリストアガイド](/br/br-snapshot-guide.md)
- [TiDB ログのバックアップとPITRガイド](/br/br-pitr-guide.md)
- [バックアップストレージ](/br/backup-and-restore-storages.md)