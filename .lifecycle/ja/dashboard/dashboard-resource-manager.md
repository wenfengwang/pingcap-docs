---
title: TiDB ダッシュボードのリソースマネージャーページ
summary: リソースマネージャーページを使用して TiDB ダッシュボードでリソース制御に関する情報を表示し、リソースプランニングの前にクラスターの容量を推定し、リソースを効果的に割り当てることができます。

# TiDB ダッシュボードのリソースマネージャーページ

[リソース制御](/tidb-resource-control.md) 機能を使用してリソースの分離を実装するために、クラスター管理者はリソースグループを作成し、各グループのクォータを設定できます。リソースプランニングの前に、クラスターの全容量を把握する必要があります。このドキュメントでは、リソース制御に関する情報を表示することで、リソースプランニングの前にクラスターの容量を推定し、リソースを効果的に割り当てることができます。

## ページへのアクセス

以下のいずれかの方法を使用してリソースマネージャーページにアクセスできます：

* TiDB ダッシュボードにログインした後、左のナビゲーションメニューで **リソースマネージャー** をクリックします。

* ブラウザーで <http://127.0.0.1:2379/dashboard/#/resource_manager> を訪問します。`127.0.0.1:2379` を実際の PD インスタンスのアドレスとポートに置き換えます。

## リソースマネージャーページ

次の図は、リソースマネージャーの詳細ページを示しています：

![TiDB ダッシュボード: リソースマネージャー](/media/dashboard/dashboard-resource-manager-info.png)

リソースマネージャーページには、次の３つのセクションが含まれています：

- 設定: このセクションには TiDB の `RESOURCE_GROUPS` テーブルから得られたデータが表示されます。すべてのリソースグループに関する情報が含まれています。詳細については、[`RESOURCE_GROUPS`](/information-schema/information-schema-resource-groups.md) を参照してください。

- 容量の推定: リソースプランニングの前に、クラスターの全容量を把握する必要があります。次のいずれかの方法を使用できます：

    - [実際のワークロードに基づいて容量を推定する](/sql-statements/sql-statement-calibrate-resource.md#estimate-capacity-based-on-actual-workload)
    - [ハードウェアの展開に基づいて容量を推定する](/sql-statements/sql-statement-calibrate-resource.md#estimate-capacity-based-on-hardware-deployment)

- メトリクス: パネル上のメトリクスを観察することで、クラスターの現在の全体的なリソース消費状況を把握することができます。

## 容量の推定

リソースプランニングの前に、クラスターの全容量を把握する必要があります。TiDB では、現在のクラスターの[リクエストユニット（RU）](/tidb-resource-control.md#what-is-request-unit-ru#what-is-request-unit-ru)の容量を推定するために、次の２つの方法を提供しています：

- [ハードウェアの展開に基づいた容量の推定](/sql-statements/sql-statement-calibrate-resource.md#estimate-capacity-based-on-hardware-deployment)
    
    TiDB は、次のワークロードタイプを受け入れます：
    
    - `tpcc`: データ書き込みが多いワークロードに適用します。これは `TPC-C` に類似したワークロードモデルに基づいて推定されます。
    - `oltp_write_only`: データ書き込みが多いワークロードに適用します。これは `sysbench oltp_write_only` に類似したワークロードモデルに基づいて推定されます。
    - `oltp_read_write`: データの読み書きが均等なワークロードに適用します。これは `sysbench oltp_read_write` に類似したワークロードモデルに基づいて推定されます。
    - `oltp_read_only`: データ読み込みが多いワークロードに適用します。これは `sysbench oltp_read_only` に類似したワークロードモデルに基づいて推定されます。

  ![ハードウェアによる較正](/media/dashboard/dashboard-resource-manager-calibrate-by-hardware.png)

    **ユーザーリソースグループの合計RU** は、`default` リソースグループを除くすべてのユーザーリソースグループのRUの合計量を表します。この値が推定容量よりも少ない場合、システムはアラートをトリガーします。デフォルトでは、システムは事前定義された `default` リソースグループに無制限の使用を割り当てます。すべてのユーザーが `default` リソースグループに属する場合、リソースはリソース制御が無効の場合と同様に割り当てられます。

- [実際のワークロードに基づいた容量の推定](/sql-statements/sql-statement-calibrate-resource.md#estimate-capacity-based-on-actual-workload)

    ![ワークロードによる較正](/media/dashboard/dashboard-resource-manager-calibrate-by-workload.png)

    10分から24時間までの範囲内で推定のための時間枠を選択できます。使用されるタイムゾーンは、フロントエンドユーザーと同じです。

    - 時間枠が10分から24時間の範囲外の場合、次のエラーが表示されます `ERROR 1105 (HY000): the duration of calibration is too short, which could lead to inaccurate output. Please make the duration between 10m0s and 24h0m0s`.

    - 容量の推定に関する監視メトリクスは、`tikv_cpu_quota`、`tidb_server_maxprocs`、`resource_manager_resource_unit`、`process_cpu_usage`を含みます。CPUクォータの監視データが空の場合、対応する監視メトリクス名にエラーが発生します。例えば、`Error 1105 (HY000): There is no CPU quota metrics, metrics 'tikv_cpu_quota' is empty`。

    - 時間枠内のワークロードが低すぎるか、`resource_manager_resource_unit` および `process_cpu_usage` の監視データが欠落している場合、次のエラーが報告されます `Error 1105 (HY000): The workload in selected time window is too low, with which TiDB is unable to reach a capacity estimation; please select another time window with higher workload, or calibrate resource by hardware instead`。さらに、macOS では TiKV が CPU 利用率を監視していないため、実際のワークロードに基づいた容量の推定はサポートされず、このエラーも報告されます。

  [メトリクス](#metrics) セクションで **CPU 使用率** を使用して適切な時間枠を選択できます。

## メトリクス

パネル上のメトリクスを観察することで、クラスターの現在の全体的なリソース消費状況を把握することができます。監視メトリクスとその意味は次のとおりです：

- 合計RU 消費量: リアルタイムでカウントされるリクエストユニットの合計消費量。
- リソースグループによるRU 消費量: リアルタイムでリソースグループによって消費されるリクエストユニットの数。
- TiDB
    - CPU クォータ: TiDB の最大 CPU 使用率。
    - CPU 使用率: すべての TiDB インスタンスの合計 CPU 使用率。
- TiKV
    - CPU クォータ: TiKV の最大 CPU 使用率。
    - CPU 使用率: すべての TiKV インスタンスの合計 CPU 使用率。
    - IO MBps: すべての TiKV インスタンスの合計 I/O スループット。