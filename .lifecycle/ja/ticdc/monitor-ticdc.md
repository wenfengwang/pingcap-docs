---
title: TiCDC監視メトリクスの詳細
summary: Grafana TiCDCダッシュボードに表示される主要メトリクスについて学びます。

# TiCDC監視メトリクスの詳細

TiUPを使用してTiDBクラスターを展開している場合、同時に展開される監視システムのTiCDCのサブダッシュボードを見ることができます。TiCDCの現在のステータスの概要をTiCDCダッシュボードから取得でき、主要メトリクスが表示されます。この文書では、これらの主要メトリクスの詳細な説明を提供します。

この文書のメトリックの説明は、デフォルトの構成を使用してMySQLにデータをレプリケートするリプリケーションタスクの例に基づいています。

```shell
cdc cli changefeed create --server=http://10.0.10.25:8300 --sink-uri="mysql://root:123456@127.0.0.1:3306/" --changefeed-id="simple-replication-task"
```

TiCDCダッシュボードには、4つの監視パネルが含まれています。次のスクリーンショットをご覧ください：

![TiCDCダッシュボード - 概要](/media/ticdc/ticdc-dashboard-overview.png)

各パネルの説明は以下の通りです：

- [**サーバー**](#server): TiDBクラスター内のTiKVノードとTiCDCノードの概要情報
- [**チェンジフィード**](#changefeed): TiCDCリプリケーションタスクの詳細情報
- [**イベント**](#events): TiCDCクラスター内のデータフローに関する詳細情報
- [**TiKV**](#tikv): TiCDCに関連するTiKV情報

## サーバー

以下は**サーバー**パネルの例です：

![TiCDCダッシュボード - サーバーメトリクス](/media/ticdc/ticdc-dashboard-server.png)

**サーバー**パネルの各メトリックの説明は以下の通りです：

- Uptime: TiKVノードとTiCDCノードが稼働している時間
- ゴールーチン数: TiCDCノードのゴールーチン数
- オープンFD数: TiCDCノードが開いているファイルハンドル数
- オーナーシップ: TiCDCクラスター内のノードの現在の状態
- オーナーシップ履歴: TiCDCクラスターのオーナーシップ履歴
- CPU使用率: TiCDCノードのCPU使用率
- メモリ使用率: TiCDCノードのメモリ使用率

## チェンジフィード

以下は**チェンジフィード**パネルの例です：

![TiCDCダッシュボード - チェンジフィードメトリクス1](/media/ticdc/ticdc-dashboard-changefeed-1.png)

- チェンジフィードテーブル数: 各TiCDCノードがレプリケートする必要のあるテーブル数
- プロセッサ解決済みTS: TiCDCクラスターで解決されたタイムスタンプ
- テーブル解決済みTS: レプリケーションタスク内の各テーブルのレプリケーション進捗状況
- チェンジフィードチェックポイント: 下流へのデータレプリケーションの進捗状況。通常、緑色のバーは黄色の線に接続されています
- PD etcdリクエスト/s: TiCDCノードがPDに送信するリクエスト数（秒）
- エラーカウント/m: 分ごとにレプリケーションタスクを中断するエラーの数
- チェンジフィードチェックポイントラグ: 上流と下流間のデータレプリケーションの進捗ラグ（単位: 秒）
- プロセッサ解決済みTSラグ: 上流とTiCDCノード間のデータレプリケーションの進捗ラグ（単位: 秒）

![TiCDCダッシュボード - チェンジフィードメトリクス2](/media/ticdc/ticdc-dashboard-changefeed-2.png)

- シンク書き込み時間: TiCDCが下流にトランザクションの変更を書き込むのにかかる時間のヒストグラム
- シンク書き込み時間パーセンタイル: TiCDCが下流にトランザクションの変更を1秒以内に書き込むのにかかる時間（P95、P99、P999）
- フラッシュシンク時間: TiCDCが下流にデータを非同期でフラッシュするのにかかる時間のヒストグラム
- フラッシュシンク時間パーセンタイル: TiCDCが下流にデータを1秒以内に非同期でフラッシュするのにかかる時間（P95、P99、P999）

![TiCDCダッシュボード - チェンジフィードメトリクス3](/media/ticdc/ticdc-dashboard-changefeed-3.png)

- MySQLシンクコンフリクト検出時間: MySQLシンクコンフリクトを検出するのにかかる時間のヒストグラム
- MySQLシンクコンフリクト検出時間パーセンタイル: MySQLシンクコンフリクトを1秒以内に検出するのにかかる時間（P95、P99、P999）
- MySQLシンクワーカーロード: TiCDCノードのMySQLシンクワーカーのワークロード

![TiCDCダッシュボード - チェンジフィードメトリクス4](/media/ticdc/ticdc-dashboard-changefeed-4.png)

- チェンジフィードキャッチアップETA: レプリケーションタスクが上流クラスターデータに追いつくのに必要な見積もり時間。上流の書き込み速度がTiCDCのレプリケーション速度よりも速い場合、メトリックは非常に大きくなる可能性があります。TiCDCのレプリケーション速度は多くの要因に左右されるため、このメトリックは参考までにし、実際のレプリケーション時間とは異なる場合があります。

## イベント

以下は**イベント**パネルの例です：

![TiCDCダッシュボード - イベントメトリクス1](/media/ticdc/ticdc-dashboard-events-1.png)
![TiCDCダッシュボード - イベントメトリクス2](/media/ticdc/ticdc-dashboard-events-2.png)
![TiCDCダッシュボード - イベントメトリクス3](/media/ticdc/ticdc-dashboard-events-3.png)

**イベント**パネルの各メトリックの説明は以下の通りです：

- イベントフィード数: TiCDCノードのEventfeed RPCリクエストの数
- イベントサイズパーセンタイル: TiCDCがTiKVから1秒以内に受信するイベントサイズ（P95、P99、P999）
- イベントフィードエラー/m: 分ごとにTiCDCノードのEventfeed RPCリクエストで報告されるエラーの数
- KVクライアント受信イベント/s: TiCDCノードのKVクライアントモジュールがTiKVから1秒当たりに受信するイベント数
- Puller受信イベント/s: TiCDCノードのPullerモジュールがKVクライアントから1秒当たりに受信するイベント数
- Puller出力イベント/s: TiCDCノードのPullerモジュールがSorterモジュールに1秒当たりに送信するイベント数
- シンクフラッシュ行数/s: TiCDCノードが1秒当たりに下流に書き込むイベント数
- Pullerバッファサイズ: TiCDCノードがPullerモジュールにキャッシュするイベント数
- Entry sorterバッファサイズ: TiCDCノードがSorterモジュールにキャッシュするイベント数
- プロセッサ/マウンターバッファサイズ: TiCDCノードがProcessorモジュールとMounterモジュールにキャッシュするイベント数
- シンク行バッファサイズ: TiCDCノードがSinkモジュールにキャッシュするイベント数
- Entry sorterソート時間: TiCDCノードがイベントをソートするのにかかる時間のヒストグラム
- Entry sorterソート時間パーセンタイル: TiCDCが1秒以内にイベントをソートするのにかかる時間（P95、P99、P999）
- Entry sorterマージ時間: TiCDCノードがソートされたイベントをマージするのにかかる時間のヒストグラム
- Entry sorterマージ時間パーセンタイル: TiCDCが1秒以内にソートされたイベントをマージするのにかかる時間（P95、P99、P999）
- マウンターアンマーシャル時間: TiCDCノードがイベントをアンマーシャルするのにかかる時間のヒストグラム
- マウンターアンマーシャル時間パーセンタイル: TiCDCが1秒以内にイベントをアンマーシャルするのにかかる時間（P95、P99、P999）
- KVクライアントディスパッチイベント/s: KVクライアントモジュールがTiCDCノード間でディスパッチするイベント数
- KVクライアントバッチ解決サイズ: TiKVがTiCDCに送信する解決タイムスタンプメッセージのバッチサイズ

## TiKV

以下は**TiKV**パネルの例です：

![TiCDCダッシュボード - TiKVメトリクス1](/media/ticdc/ticdc-dashboard-tikv-1.png)
![TiCDCダッシュボード - TiKVメトリクス2](/media/ticdc/ticdc-dashboard-tikv-2.png)

**TiKV**パネルの各メトリックの説明は以下の通りです：

- CDCエンドポイントCPU: TiKVノードのCDCエンドポイントスレッドのCPU使用率
- CDCワーカーCPU: TiKVノードのCDCワーカースレッドのCPU使用率
- 最小解決済みTS: TiKVノードの最小解決タイムスタンプ
- 最小解決済みリージョン: TiKVノードの最小解決タイムスタンプのリージョンID
- 解決済みTSラグ時間パーセンタイル: TiKVノードの最小解決タイムスタンプと現在時刻のラグ
- イニシャルスキャン時間: TiKVノードがTiCDCノードに接続する際の増分スキャンにかかる時間のヒストグラム
- イニシャルスキャン時間パーセンタイル: TiKVノードの増分スキャンにかかる時間（P95、P99、P999）
- ブロックキャッシュを除くメモリ: RocksDBのブロックキャッシュを除くTiKVノードのメモリ使用量
- CDCメモリ内保留バイト数: TiKVノードのCDCモジュールのメモリ使用量
- キャプチャリージョン数: TiKVノードのイベントキャプチャリングリージョン数