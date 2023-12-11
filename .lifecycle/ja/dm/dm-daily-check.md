---
title: TiDBデータ移行の日次チェック
summary: TiDBデータ移行（DM）の日次チェックについて学びます。
aliases: ['/docs/tidb-data-migration/dev/daily-check/']
---

# TiDBデータ移行の日次チェック

このドキュメントは、TiDBデータ移行（DM）の日次チェックの実施方法をまとめたものです。

+ 方法1: `query-status`コマンドを実行してタスクの実行状況とエラー出力（ある場合）を確認します。詳細は[Query Status](/dm/dm-query-status.md)を参照してください。

+ 方法2: TiUPを使用してDMクラスタをデプロイする際にPrometheusとGrafanaが正しくデプロイされている場合、GrafanaでDM監視メトリクスを表示できます。例えば、Grafanaのアドレスが`172.16.10.71`であるとします。 <http://172.16.10.71:3000>にアクセスしてGrafanaダッシュボードに入り、DMダッシュボードを選択してDMの監視メトリクスを確認します。これらのメトリクスの詳細については、[DM Monitoring Metrics](/dm/monitor-a-dm-cluster.md)を参照してください。

+ 方法3: DMの実行状況およびエラー（ある場合）をログファイルを使用して確認します。

    - DM-masterのログディレクトリ: `--log-file`DM-masterプロセスパラメータで指定されます。TiUPを使用してDMをデプロイした場合、ログディレクトリはDM-masterノード内の`{log_dir}`です。
    - DM-workerのログディレクトリ: `--log-file`DM-workerプロセスパラメータで指定されます。TiUPを使用してDMをデプロイした場合、ログディレクトリはDM-workerノード内の`{log_dir}`です。