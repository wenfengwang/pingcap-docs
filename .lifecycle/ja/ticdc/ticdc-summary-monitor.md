---
title: TiCDC モニタリングメトリクス概要
summary: TiCDC のモニタリングメトリクスについて学ぶ
---

# TiCDC モニタリングメトリクス概要

v7.0.0 から、TiUP を使用して Grafana をデプロイする場合、TiCDC サマリーダッシュボードが自動的に Grafana のモニタリングページに追加されます。このダッシュボードを通じて、TiCDC サーバーとチェンジフィードの状態を素早く把握できます。

以下の画像は TiCDC Summary Dashboard のモニタリングパネルを示しています：

![TiCDC Summary Dashboard - Overview](/media/ticdc/ticdc-summary-monitor.png)

各モニタリングパネルは次のように説明されます：

- サーバー: クラスター内の TiCDC ノードの概要。
- Changefeed: TiCDC チェンジフィードのレイテンシと状態情報。
- Dataflow: TiCDC の内部モジュールによるデータ変更の統計。
- トランザクション Sink: 下流の MySQL または TiDB の書き込みレイテンシ。
- MQ Sink: 下流の MQ システムの書き込みレイテンシ。
- Cloud Storage Sink: 下流のクラウドストレージへの書き込み速度。
- Redo: リドゥ機能が有効な場合の書き込みレイテンシ。

## サーバーパネル

**サーバー** パネルは次のようになります：

![TiCDC Summary Dashboard - Server metrics](/media/ticdc/ticdc-summary-monitor-server.png)

- **稼働時間**: TiCDC ノードが稼働している時間。
- **CPU 使用率**: TiCDC ノードの CPU 使用率。
- **メモリ使用量**: TiCDC ノードのメモリ使用量。

## Changefeed パネル

**Changefeed** パネルは次のようになります：

![TiCDC Summary Dashboard - Changefeed metrics](/media/ticdc/ticdc-summary-monitor-changefeed.png)

- **Changefeed チェックポイントラグ**: 上流の TiDB クラスターと下流システム間のデータレプリケーションの遅延を示し、時間で計測されます。通常、このメトリクスはデータレプリケーションタスクの全体的な状態を反映します。通常、ラグが小さいほど、レプリケーションタスクの状態が良好であることを示します。ラグが増加すると、通常は changefeed のレプリケーション能力または下流システムの消費能力が上流の書き込み速度に追いつけないことを示します。

- **Changefeed resolved ts ラグ**: 上流の TiDB クラスターと TiCDC ノード間のデータの遅延を示し、時間で計測されます。このメトリクスは changefeed が上流からデータ変更を取得する能力を反映します。ラグが増加すると、changefeed が上流で生成されたデータ変更を取得できないことを意味します。

## Dataflow パネル

![TiCDC Summary Dashboard - Puller metrics](/media/ticdc/ticdc-summary-monitor-dataflow-puller.png)

- **Puller 出力イベント/s**: TiCDC ノード内の Puller モジュールが Sorter モジュールに対して毎秒出力するデータ変更の数。このメトリックは TiCDC が上流からデータ変更を取得する速度を反映します。
- **Puller 出力イベント**: TiCDC ノード内の Puller モジュールから Sorter モジュールに出力されるデータ変更の合計数。

![TiCDC Summary Dashboard - Sorter metrics](/media/ticdc/ticdc-summary-monitor-dataflow-sorter.png)

- **Sorter 出力イベント/s**: ソーターモジュールが毎秒 Sink モジュールに対して出力するデータ変更の数。ソーターのデータ出力率は Sink モジュールの影響を受けます。したがって、ソーターの出力率が Puller モジュールよりも低い場合、必ずしもソーターモジュールのソート速度が遅いとは限りません。まず Sink モジュールに関連するメトリクスを観察して、Sink モジュールがデータのフラッシュに時間がかかり、ソーターモジュールの出力が減少しているかを確認する必要があります。

- **Sorter 出力イベント**: ソーターモジュールから Sink モジュールに出力されるデータ変更の合計数。

![TiCDC Summary Dashboard - Mounter metrics](/media/ticdc/ticdc-summary-monitor-dataflow-mounter.png)

- **Mounter 出力イベント/s**: マウンターモジュールが毎秒デコードするデータ変更の数。上流のデータ変更に多数のフィールドが関与する場合、マウンターモジュールのデコード速度に影響することがあります。

- **Mounter 出力イベント**: マウンターモジュールによってデコードされたデータ変更の総数。

![TiCDC Summary Dashboard - Sink metrics](/media/ticdc/ticdc-summary-monitor-dataflow-sink.png)

- **Sink フラッシュ行数/s**: Sink モジュールが TiCDC ノード内で下流に毎秒出力するデータ変更の数。このメトリックはデータ変更が下流に複製される速度を反映します。**Sink フラッシュ行数/s** が **Puller 出力イベント/s** よりも低い場合、レプリケーション遅延が増加する可能性があります。

- **Sink フラッシュ行数**: Sink モジュールから TiCDC ノード内の下流に出力されるデータ変更の総数。

## トランザクション Sink パネル

**トランザクション Sink** パネルは、下流が MySQL または TiDB の場合にのみデータを表示します。

![TiCDC Summary Dashboard - Transaction Sink metrics](/media/ticdc/ticdc-summary-monitor-transaction-sink.png)

- **バックエンドフラッシュデュレーション**: TiCDC トランザクション Sink モジュールが下流で SQL ステートメントを実行するのにかかる時間。このメトリックを観察することで、下流のパフォーマンスがレプリケーション速度のボトルネックであるかどうかを判断できます。一般に、p999 の値は 500 ms 未満であるべきです。値がこの制限を超えると、レプリケーション速度に影響が出る可能性があり、Changefeed チェックポイントラグが増加する可能性があります。

- **全体フラッシュデュレーション**: ソーターによる整列から下流への送信までの各トランザクションにかかる合計時間。この値から **バックエンドフラッシュデュレーション** の値を引くことで、トランザクションが下流で実行される前のトランザクションの総待ち時間を得ることができます。待ち時間が長い場合、レプリケーションタスクにより多くのメモリクォータを割り当てることを検討できます。

## MQ Sink パネル

**MQ Sink** パネルは、下流が Kafka の場合にのみデータを表示します。

![TiCDC Summary Dashboard - Transaction Sink metrics](/media/ticdc/ticdc-summary-monitor-mq-sink.png)

- **Worker Send Message Duration Percentile**: TiCDC MQ Sink ワーカーが下流にデータを送信するのにかかる遅延時間。
- **Kafka Ongoing Bytes**: TiCDC MQ Sink が下流にデータを送信する速度。

## Cloud Storage Sink パネル

**Cloud Storage Sink** パネルは、下流が Cloud Storage の場合にのみデータを表示します。

![TiCDC Summary Dashboard - Transaction Sink metrics](/media/ticdc/ticdc-summary-monitor-cloud-storage.png)

- **書き込みバイト数/s**: Cloud Storage Sink モジュールが下流にデータを書き込む速度。
- **ファイル数**: Cloud Storage Sink モジュールによって書き込まれたファイルの総数。

## Redo パネル

**Redo** パネルは、Redo ログ機能が有効な場合にのみデータを表示します。

![TiCDC Summary Dashboard - Transaction Sink metrics](/media/ticdc/ticdc-summary-monitor-redo.png)

- **Redo 書き込み行数/s**: Redo モジュールによって毎秒書き込まれる行数。Redo 機能が有効な場合、レプリケーションタスクのレイテンシが増加した場合、このメトリックと Puller 出力イベント/s の値との間にかなりの差があるかどうかを観察できます。そうであれば、レイテンシの増加は Redo モジュールの書き込み能力不足による可能性があります。
- **Redo 書き込みバイト数/s**: Redo モジュールによって毎秒書き込まれるデータの速度。
- **Redo ログフラッシュデュレーション**: Redo モジュールが下流にデータをフラッシュするのにかかる時間。このメトリックの値が高い場合、この操作がレプリケーション速度に影響を与える可能性があります。
- **Redo 全フラッシュデュレーション**: データ変更が Redo モジュールに滞在する合計時間。