---
title: TiDBのグローバルソート
summary: TiDB Global Sortの使用事例、制限、使用法、および実装原則について学びます。

# TiDBのグローバルソート

> **警告:**
>
> この機能は実験的です。本番環境で使用することはお勧めしません。この機能は事前に通知なく変更される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

<CustomContent platform="tidb-cloud">

> **Note:**
>
> 現在、この機能はTiDB専用クラスターにのみ適用されます。TiDBサーバーレスクラスターでは使用できません。

</CustomContent>

## 概要

TiDBのグローバルソート機能は、データのインポートおよびDDL（Data Definition Language）操作の安定性と効率を向上させます。これは、クラウド上でグローバルソートサービスを提供する分散実行フレームワークで一般的な演算子として機能します。

グローバルソート機能は現在、クラウドストレージとしてAmazon S3のみをサポートしています。将来のリリースでは、POSIXなどの複数の共有ストレージインターフェースをサポートし、異なるストレージシステムとシームレスに統合できるように拡張されます。この柔軟性により、さまざまなユースケースで効率的かつ適応可能なデータソーティングが可能となります。

## 使用事例

グローバルソート機能は、`IMPORT INTO`および`CREATE INDEX`の安定性と効率を向上させます。タスクで処理されるデータをグローバルでソートすることで、TiKVへのデータの書き込みの安定性、制御性、および拡張性が向上します。これにより、データのインポートやDDLタスクにおけるユーザーエクスペリエンスが向上し、より高品質なサービスが提供されます。

グローバルソート機能は、統一された分散並列実行フレームワーク内でタスクを実行し、グローバル規模で効率的かつ並列にデータをソートします。

## 制限事項

現在、グローバルソート機能はクエリ結果のソートに責任を持つクエリ実行プロセスのコンポーネントとして使用されていません。

## 使用法

グローバルソートを有効にするには、次の手順に従ってください。

1. [`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710)の値を`ON`に設定して、分散実行フレームワークを有効にします。

    ```sql
    SET GLOBAL tidb_enable_dist_task = ON;
    ```

<CustomContent platform="tidb">

2. [`tidb_cloud_storage_uri`](/system-variables.md#tidb_cloud_storage_uri-new-in-v740)を正しいクラウドストレージパスに設定します。[例を参照](/br/backup-and-restore-storages.md)。

> **Note:**
>
> [`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)に対して、[`CLOUD_STORAGE_URI`](/sql-statements/sql-statement-import-into.md#withoptions)オプションを使用してクラウドストレージパスを指定することもできます。有効なクラウドストレージパスが`tidb_cloud_storage_uri`および`CLOUD_STORAGE_URI`の両方で構成されている場合、`IMPORT INTO`に対して`CLOUD_STORAGE_URI`の構成が有効になります。

</CustomContent>
<CustomContent platform="tidb-cloud">

2. [`tidb_cloud_storage_uri`](/system-variables.md#tidb_cloud_storage_uri-new-in-v740)を正しいクラウドストレージパスに設定します。[例を参照](https://docs.pingcap.com/tidb/stable/backup-and-restore-storages)。

</CustomContent>

    ```sql
    SET GLOBAL tidb_cloud_storage_uri = 's3://my-bucket/test-data?role-arn=arn:aws:iam::888888888888:role/my-role'
    ```

## 実装原則

グローバルソート機能のアルゴリズムは次のようになります：

![グローバルソートのアルゴリズム](/media/dist-task/global-sort.jpeg)

詳細な実装原則は次のとおりです：

### ステップ1: データのスキャンと準備

1. TiDBノードが特定のデータ範囲をスキャンした後（データソースはCSVデータまたはTiKV内のテーブルデータのいずれかです）：

    1. TiDBノードはそれらをキーと値のペアにエンコードします。
    2. TiDBノードはキーと値のペアを複数のブロックデータセグメントにソートします（データセグメントはローカルでソートされたものです）。各セグメントは1つのファイルであり、クラウドストレージにアップロードされます。

2. TiDBノードはまた、各セグメントの実際のキーと値の範囲（統計ファイルと呼ばれる）を記録し、スケーラブルなソート実装のための鍵の準備を行います。これらのファイルは、実際のデータとともにクラウドストレージにアップロードされます。

### ステップ2: データのソートと配布

ステップ1から、グローバルソートプログラムはソートされたブロックのリストとそれに対応する統計ファイルを取得します。これはローカルでソートされたブロックの数を提供します。プログラムはまた、PDが分割および散布に使用できる実際のデータ範囲を持っています。次の手順が実行されます：

1. 統計ファイルのレコードをソートして、ほぼ同じサイズの範囲に分割し、並列で実行されるサブタスクに分割します。
2. サブタスクをTiDBノードに配布して実行します。
3. 各TiDBノードは、サブタスクのデータを独立して範囲に並べ替え、重複しないようにTiKVに取り込みます。