---
title: SQLの診断
summary: TiDBにおけるSQLの診断方法を理解します。
aliases: ['/docs/dev/system-tables/system-table-sql-diagnostics/','/docs/dev/reference/system-databases/sql-diagnosis/','/docs/dev/system-tables/system-table-sql-diagnosis/','/tidb/dev/system-table-sql-diagnostics/','/tidb/dev/check-cluster-status-using-sql-statements','/docs/dev/check-cluster-status-using-sql-statements/','/docs/dev/reference/performance/check-cluster-status-using-sql-statements/']
---

# SQLの診断

SQLの診断はTiDB v4.0で導入された機能です。この機能を使用すると、TiDBの問題をより効率的に特定できます。TiDB v4.0以前では、異なるツールを使用して異なる情報を取得する必要がありました。

SQL診断システムには以下の利点があります:

+ システムの全コンポーネントからの情報を統合できます。
+ システムテーブルを介した一貫したインターフェースを提供します。
+ 監視サマリーと自動診断を提供します。
+ クラスタ情報を簡単にクエリできます。

## 概要

SQL診断システムは主に3つの要素で構成されています:

+ **クラスタ情報テーブル**: SQL診断システムはクラスタ情報テーブルを導入し、各インスタンスの個々の情報を取得する統一された方法を提供します。このシステムはクラスタのトポロジー、ハードウェア情報、ソフトウェア情報、カーネルパラメータ、モニタリング、システム情報、遅いクエリ、ステートメント、クラスタ全体のログなどを完全にテーブルに統合します。そのため、これらの情報をSQLステートメントを使用してクエリできます。

+ **クラスタ監視テーブル**: SQL診断システムはクラスタ監視テーブルを導入しています。これらのテーブルはすべて`metrics_schema`にあり、SQLステートメントを使用して監視情報をクエリできます。v4.0以前の視覚化された監視と比較して、このSQLベースの方法を使用すると、クラスタ全体の監視情報を相関クエリし、異なる時間帯の結果を比較してパフォーマンスのボトルネックを素早く特定できます。TiDBクラスタには多くの監視メトリクスがあるため、SQL診断システムは監視サマリーテーブルも提供しており、異常な監視項目をより簡単に見つけることができます。

+ **自動診断**: 手動でSQLステートメントを実行してクラスタ情報テーブルや監視テーブル、サマリーテーブルをクエリして問題を特定できますが、自動診断を使用すると一般的な問題を素早く特定できます。SQL診断システムは既存のクラスタ情報テーブルと監視テーブルに基づいて自動診断を実行し、関連する診断結果テーブルと診断サマリーテーブルを提供します。

## クラスタ情報テーブル

クラスタ情報テーブルはクラスタ内のすべてのインスタンスとその情報をまとめます。これらのテーブルを使用すると、1つのSQLステートメントだけでクラスタ情報全体を照会できます。以下はクラスタ情報テーブルの一覧です:

+ クラスタトポロジーテーブル[`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md)からは、クラスタの現在のトポロジー情報、各インスタンスのバージョン、バージョンに対応するGit Hash、各インスタンスの起動時間、および実行時間を取得できます。
+ クラスタ構成テーブル[`information_schema.cluster_config`](/information-schema/information-schema-cluster-config.md)からは、クラスタ内のすべてのインスタンスの構成を取得できます。v4.0より前のバージョンでは、これらの構成情報を取得するには、各インスタンスのHTTP APIに1つずつアクセスする必要があります。
+ クラスタハードウェアテーブル[`information_schema.cluster_hardware`](/information-schema/information-schema-cluster-hardware.md)では、クラスタのハードウェア情報を迅速に照会できます。
+ クラスタ負荷テーブル[`information_schema.cluster_load`](/information-schema/information-schema-cluster-load.md)では、クラスタの異なるインスタンスとハードウェアタイプの負荷情報を照会できます。
+ カーネルパラメータテーブル[`information_schema.cluster_systeminfo`](/information-schema/information-schema-cluster-systeminfo.md)では、クラスタ内の異なるインスタンスのカーネル構成情報をクエリできます。現在、TiDBはsysctl情報のクエリをサポートしています。
+ クラスタログテーブル[`information_schema.cluster_log`](/information-schema/information-schema-cluster-log.md)では、クラスタのログを照会できます。クエリの条件を各インスタンスにプッシュダウンすることで、クラスタパフォーマンスへのクエリの影響が`grep`コマンドよりも少なくなります。

TiDB v4.0より前のシステムテーブルでは、現在のインスタンスのみを表示できました。TiDB v4.0では対応するクラスタテーブルを導入し、1つのTiDBインスタンスでクラスタ全体をグローバルに表示できます。これらのテーブルは現在`information_schema`にあり、クエリ方法は他の`information_schema`システムテーブルと同じです。

## クラスタ監視テーブル

異なる時間帯におけるクラスタの状態を動的に観察および比較するために、SQL診断システムはクラスタ監視システムテーブルを導入しています。すべての監視テーブルは`metrics_schema`にあり、SQLステートメントを使用して監視情報をクエリできます。この方法を使用すると、クラスタ全体の監視情報を相関クエリし、異なる時間帯の結果を比較してパフォーマンスのボトルネックを素早く特定できます。

+ [`information_schema.metrics_tables`](/information-schema/information-schema-metrics-tables.md)：現在多くのシステムテーブルが存在するため、`information_schema.metrics_tables`テーブルでこれらの監視テーブルのメタ情報をクエリできます。

TiDBクラスタには多くの監視メトリクスがあるため、TiDBはv4.0で以下の監視サマリーテーブルを提供しています:

+ 監視サマリーテーブル[`information_schema.metrics_summary`](/information-schema/information-schema-metrics-summary.md)は、すべての監視データを要約し、より効率的に各監視メトリクスを確認できます。
+ [`information_schema.metrics_summary_by_label`](/information-schema/information-schema-metrics-summary.md)もすべての監視データを要約します。特に、このテーブルは各監視メトリクスの異なるラベルを使用して統計を集計します。

## 自動診断

上記のクラスタ情報テーブルとクラスタ監視テーブルでは、クラスタの問題を手動でトラブルシューティングする必要があります。TiDB v4.0では自動診断をサポートしています。既存の基本情報テーブルに基づいて診断関連のシステムテーブルを使用することで、診断が自動的に実行されます。以下は自動診断に関連するシステムテーブルです:

+ 診断結果テーブル[`information_schema.inspection_result`](/information-schema/information-schema-inspection-result.md)は、システムの診断結果を表示します。診断は受動的にトリガーされます。`select * from inspection_result`を実行するとすべての診断ルールがシステムを診断し、システム内の障害やリスクが結果に表示されます。
+ 診断サマリーテーブル[`information_schema.inspection_summary`](/information-schema/information-schema-inspection-summary.md)は特定のリンクやモジュールの監視情報を要約します。モジュールやリンク全体のコンテキストに基づいて問題をトラブルシューティングおよび特定できます。