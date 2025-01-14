---
title: リソース制御の主要な監視メトリクス
summary: Grafanaリソース制御ダッシュボードに表示される主要なメトリクスについて学びます。

# リソース制御の主要な監視メトリクス

TiUPを使用してTiDBクラスターをデプロイする場合、モニタリングシステム（Prometheus & Grafana）は同時にデプロイされます。詳細については[モニタリングフレームワークの概要](/tidb-monitoring-framework.md)を参照してください。

Grafanaダッシュボードは、概要、PD、TiDB、TiKV、Node_exporter、Disk Performance、Performance_overviewなどのサブダッシュボードに分かれています。

クラスターで[リソース制御](/tidb-resource-control.md)機能を使用している場合、リソース制御ダッシュボードからリソース消費状況の概要を取得できます。

TiDBはフロー制御に[トークンバケットアルゴリズム](https://en.wikipedia.org/wiki/Token_bucket)を使用しています。[TiDBでのグローバルリソース制御に関するRFC](https://github.com/pingcap/tidb/blob/master/docs/design/2022-11-25-global-resource-control.md#distributed-token-buckets)に記載されているように、TiDBノードには複数のリソースグループがあり、これらはPD側のGAC（グローバルアドミッションコントロール）によってフロー制御されます。各TiDBノードのローカルトークンバケットは、デフォルトでは5秒ごとにPD側のGACと定期的に通信してローカルトークンを再構成します。TiDBでは、ローカルトークンバケットはリソースコントローラクライアントとして実装されています。

このドキュメントでは、リソース制御ダッシュボードに表示されるいくつかの主要な監視メトリクスについて説明します。

## リクエスト単位に関するメトリクス

- RU：各リソースグループの[リクエストユニット（RU）](/tidb-resource-control.md#what-is-request-unit-ru)の消費情報で、リアルタイムで計算されます。`total`はすべてのリソースグループによって消費されたリクエストユニットの合計です。各リソースグループのリクエストユニットの消費量は、その読み取り消費量（Read Request Unit）と書き込み消費量（Write Request Unit）の合計に等しくなければなりません。
- RU Per Query：各SQLステートメントごとに平均で消費されるリクエストユニットの数です。上記のRUメトリクスを1秒あたりに実行されたSQLステートメントの数で割ることによって取得されます。
- RRU：各リソースグループの読み取りリクエストユニットの消費情報で、リアルタイムで計算されます。`total`はすべてのリソースグループによって消費された読み取りリクエストユニットの合計です。
- RRU Per Query：各SQLステートメントごとに平均で消費される読み取りリクエストユニットの数です。上記のRRUメトリクスを1秒あたりに実行されたSQLステートメントの数で割ることによって取得されます。
- WRU：各リソースグループの書き込みリクエストユニットの消費情報で、リアルタイムで計算されます。`total`はすべてのリソースグループによって消費された書き込みリクエストユニットの合計です。
- WRU Per Query：各SQLステートメントごとに平均で消費される書き込みリクエストユニットの数です。上記のWRUメトリクスを1秒あたりに実行されたSQLステートメントの数で割ることによって取得されます。
- 利用可能なRU：各リソースグループのRUトークンバケット内の利用可能なトークンです。 `0`の場合、このリソースグループは`RU_PER_SEC`のレートでトークンを消費し、レート制限されていると見なすことができます。
- クエリ最大期間：リソースグループごとの最大クエリ期間です。

## リソースに関するメトリクス

- KVリクエスト数：各リソースグループに対するKVリクエストの数で、1秒ごとに計算されます。リクエストは読み取りと書き込みのタイプに分かれます。`total`はすべてのリソースグループに対するKVリクエストの合計です。
- KVリクエスト数Perクエリ：各SQLステートメントごとに平均で実行される読み取りおよび書き込みKVリクエストの数です。上記のKVリクエスト数メトリクスを1秒あたりに実行されたSQLステートメントの数で割ることによって取得されます。
- 読み取られたバイト数：各リソースグループによって読み取られたデータの量で、1秒ごとに計算されます。`total`はすべてのリソースグループによって読まれたデータの合計です。
- クエリごとの読み取られたバイト数：各SQLステートメントごとに平均で読まれるデータの量です。上記の読み取られたバイト数メトリクスを1秒あたりに実行されたSQLステートメントの数で割ることによって取得されます。
- 書き込まれたバイト数：各リソースグループによって書き込まれたデータの量で、リアルタイムで計算されます。`total`はすべてのリソースグループによって書かれたデータの合計です。
- クエリごとの書き込まれたバイト数：各SQLステートメントごとに平均で書き込まれるデータの量です。上記の書き込まれたバイト数メトリクスを1秒あたりに実行されたSQLステートメントの数で割ることによって取得されます。
- KV CPU時間：各リソースグループによって消費されたKVレイヤーのCPU時間で、リアルタイムで計算されます。`total`はすべてのリソースグループによって消費されたKVレイヤーのCPU時間の合計です。
- SQL CPU時間：各リソースグループによって消費されたSQLレイヤーのCPU時間で、リアルタイムで計算されます。`total`はすべてのリソースグループによって消費されたSQLレイヤーのCPU時間の合計です。

## リソースコントローラクライアントに関するメトリクス

- アクティブリソースグループ：リアルタイムで計算される各リソースコントローラクライアントのリソースグループの数です。
- 合計KVリクエスト数：各リソースコントローラクライアントに対するKVリクエストの数で、リアルタイムで計算され、リソースグループごとに分かれます。`total`はすべてのリソースコントローラクライアントに対するKVリクエストの合計です。
- 失敗したKVリクエスト数：リアルタイムで計算される各リソースコントローラクライアントに対する失敗したKVリクエストの数です。`total`はすべてのリソースコントローラクライアントに対する失敗したKVリクエストの合計です。
- 成功したKVリクエスト数：リアルタイムで計算される各リソースコントローラクライアントに対する成功したKVリクエストの数です。`total`はすべてのリソースコントローラクライアントに対する成功したKVリクエストの合計です。
- 成功したKVリクエスト待機時間（99/90）：リアルタイムで計算される各リソースコントローラクライアントに対する成功したKVリクエストの待機時間（異なるパーセンタイルで）、リソースグループごとに分かれます。
- トークンリクエスト処理時間（999/99）：サーバーサイドからのトークンリクエストの待機時間（異なるパーセンタイルで）を、リアルタイムで計算される各リソースコントローラクライアントに対して示します。、リソースグループごとに分かれます。
- トークンリクエスト数：サーバーサイドからの各リソースコントローラクライアントに対するトークンリクエストの数です。リアルタイムで計算され、リソースグループごとに分かれます。 `successful`と`failed`はすべてのリソースコントローラクライアントに対する成功したトークンリクエストと失敗したトークンリクエストの合計です。
-