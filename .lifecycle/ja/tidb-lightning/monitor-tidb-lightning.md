---
title: TiDB Lightningのモニタリング
summary: TiDB Lightningのモニター構成と監視メトリクスについて学びます。
aliases: ['/docs/dev/tidb-lightning/monitor-tidb-lightning/','/docs/dev/reference/tools/tidb-lightning/monitor/']
---

# TiDB Lightningのモニタリング

`tidb-lightning`は[Prometheus](https://prometheus.io/)を通じてメトリクスの収集をサポートしています。このドキュメントではTiDB Lightningのモニター構成と監視メトリクスを紹介します。

## モニター構成

TiDB Lightningが手動でインストールされた場合、以下の手順に従います。

`tidb-lightning`のメトリクスは、発見されている限りPrometheusに直接収集することができます。`tidb-lightning.toml`でメトリクスポートを設定できます:

```toml
[lightning]
# HTTP port for debugging and Prometheus metrics pulling (0 to disable)
pprof-port = 8289

...
```

サーバーを発見できるようにPrometheusを構成する必要があります。たとえば、サーバーアドレスを`scrape_configs`セクションに直接追加できます:

```yaml
...
scrape_configs:
  - job_name: 'tidb-lightning'
    static_configs:
      - targets: ['192.168.20.10:8289']
```

## Grafanaダッシュボード

[Grafana](https://grafana.com/)はPrometheusメトリクスをダッシュボードとして視覚化するためのWebインターフェースです。

### 行1: 速度

![最初の行のパネル](/media/lightning-grafana-row-1.png)

| パネル | シリーズ | 説明 |
|:-----|:-----|:-----|
| インポート速度 | TiDB Lightningからの書き込み | TiDB LightningからTiKV ImporterにKVを送信する速度。各テーブルの複雑さに依存します |
| インポート速度 | TiKVへのアップロード | TiKV ImporterからすべてのTiKVレプリカへの総アップロード速度 |
| チャンク処理期間 | | 1つのデータファイルを完全にエンコードするために必要な平均時間 |

インポート速度がゼロになり、他の部分が追いつく余裕があることがあります。これは正常です。

### 行2: 進捗

![2番目の行のパネル](/media/lightning-grafana-row-2.png)

| パネル | 説明 |
|:-----|:-----|
| インポート進捗 | これまでにエンコードされたデータファイルの割合 |
| チェックサム進捗 | 成功裏にインポートされたテーブルの割合 |
| 失敗 | 失敗したテーブルの数と失敗のポイント。通常は空です |

### 行3: リソース

![3番目の行のパネル](/media/lightning-grafana-row-3.png)

| パネル | 説明 |
|:-----|:-----|
| メモリ使用量 | 各サービスが占有するメモリ量 |
| TiDB Lightning Goroutinesの数 | TiDB Lightningによって使用される実行中のゴルーチンの数 |
| CPU% | 各サービスによって使用される論理CPUコアの数 |

### 行4: クォータ

![4番目の行のパネル](/media/lightning-grafana-row-4.png)

| パネル | シリーズ | 説明 |
|:-----|:-----|:-----|
| アイドルワーカー | io | 未使用の`io-concurrency`の数。通常は設定された値に近く（デフォルトは5）0に近いとディスクが遅すぎることを意味します |
| アイドルワーカー | closed-engine | 閉じられていてまだクリーンアップされていないエンジンの数。通常はindex + table-concurrency（デフォルトは8）に近く、0に近いとTiDB LightningがTiKV Importerよりも速いため、TiDB Lightningが停止する可能性があります |
| アイドルワーカー | table | 未使用の`table-concurrency`の数。通常はプロセスの終了まで0です |
| アイドルワーカー | index | 未使用の`index-concurrency`の数。通常はプロセスの終了まで0です |
| アイドルワーカー | region | 未使用の`region-concurrency`の数。通常はプロセスの終了まで0です |
| 外部リソース | KVエンコーダ | アクティブなKVエンコーダの数。通常はプロセスの終了まで`region-concurrency`と同じです |
| 外部リソース | インポーターエンジン | 開かれたエンジンファイルの数。`max-open-engines`設定を超えてはなりません |

### 行5: 読み取り速度

![5番目の行のパネル](/media/lightning-grafana-row-5.png)

| パネル | シリーズ | 説明 |
|:-----|:-----|:-----|
| チャンクパーサーの読み取りブロック期間 | read block | 解析の準備のために1ブロックのバイトを読み取る時間 |
| チャンクパーサーの読み取りブロック期間 | apply worker | アイドルなio-concurrencyを待つために経過した時間 |
| SQL処理期間 | 行のエンコード | 1つの行を解析およびエンコードするのにかかる時間 |
| SQL処理期間 | ブロックの配信 | KVペアのブロックをTiKV Importerに送信するためにかかる時間 |

いずれかの期間が長すぎると、TiDB Lightningが使用しているディスクが遅いかI/Oで使用中であることを示します。

### 行6: ストレージ

![6番目の行のパネル](/media/lightning-grafana-row-6.png)

| パネル | シリーズ | 説明 |
|:-----|:-----|:-----|
| SQL処理速度 | データの配信速度 | データKVペアをTiKV Importerに配信する速度 |
| SQL処理速度 | インデックスの配信速度 | インデックスKVペアをTiKV Importerに配信する速度 |
| SQL処理速度 | 合計配信速度 | 上記2つの速度の合計 |
| 合計バイト | 解析読み取りサイズ | TiDB Lightningによって読み取られているバイト数 |
| 合計バイト | データの配信サイズ | TiKV Importerにすでに配信されたデータKVペアのバイト数 |
| 合計バイト | インデックスの配信サイズ | TiKV Importerにすでに配信されたインデックスKVペアのバイト数 |
| 合計バイト | storage_size / 3 | TiKVクラスタが占有している合計サイズ。デフォルトのレプリカ数である3で割られます |

### 行7: インポート速度

![7番目の行のパネル](/media/lightning-grafana-row-7.png)

| パネル | シリーズ | 説明 |
|:-----|:-----|:-----|
| 配信期間 | 範囲の配信 | TiKVクラスタにKVペアの範囲をアップロードするためにかかる時間 |
| 配信期間 | SSTの配信 | SSTファイルをTiKVクラスタにアップロードするためにかかる時間 |
| SST処理期間 | SSTの分割 | KVペアのストリームをSSTファイルに分割するためにかかる時間 |
| SST処理期間 | SSTのアップロード | アップロードされたSSTファイルを取り込むためにかかる時間 |
| SST処理期間 | SSTのインジェスト | アップロードされたSSTファイルを取り込むためにかかる時間 |
| SST処理期間 | SSTサイズ | SSTファイルのサイズ |

## 監視メトリクス

このセクションでは`tidb-lightning`の監視メトリクスについて説明します。

`tidb-lightning`が提供するメトリクスは、`lightning_*`の名前空間の下にリストされています。

- **`lightning_importer_engine`**（カウンター）

    開かれたエンジンファイルと閉じられたエンジンファイルのカウント。ラベル:

    - **type**:
        * `open`
        * `closed`

- **`lightning_idle_workers`**（ゲージ）

    待機中のワーカーのカウント。ラベル:

    - **name**:
        * `table`：残りの`table-concurrency`。通常はプロセスの終了まで0です
        * `index`：残りの`index-concurrency`。通常はプロセスの終了まで0です
        * `region`：残りの`region-concurrency`。通常はプロセスの終了まで0です
        * `io`：未使用の`io-concurrency`の残りの数。通常は設定値に近く（デフォルトは5）、0に近いとディスクが遅すぎます
        * `closed-engine`：閉じられたがまだクリーンアップされていないエンジンの数。通常はindex + table-concurrency（デフォルトは8）に近いです。値が0に近いと、TiDB LightningがTiKV Importerよりも速いため、TiDB Lightningが停止する可能性があります

- **`lightning_kv_encoder`**（カウンター）

    開かれたKVエンコーダと閉じられたKVエンコーダのカウント。KVエンコーダはSQL `INSERT` ステートメントをKVペアに変換するメモリ内TiDBインスタンスです。ネット値は健全な状況で増える必要があります。ラベル:

    - **type**:
        * `open`
        * `closed`

* **`lightning_tables`**（カウンター）

    処理されたテーブルとそのステータスのカウント。ラベル:

    - **state**：テーブルのステータスで、完了すべきフェーズを示します
        * `pending`：まだ処理されていない
        * `written`：すべてのデータがエンコードされ、送信された
        * `closed`：対応するエンジンファイルがすべて閉じられた
        * `imported`：すべてのエンジンファイルがターゲットクラスタにインポートされました
        * `altered_auto_inc`：AUTO_INCREMENT IDが変更されました
        * `checksum`：チェックサムが実行されました
        * `analyzed`：統計解析が実行されました
        * `completed`：テーブルが完全にインポートされ、検証されました
    - **result**：現在のフェーズの結果
        * `success`：フェーズが正常に完了しました
        * `failure`：フェーズが失敗しました（完了していません）

* **`lightning_engines`**（カウンター）

    処理されたエンジンファイルの数とそのステータスのカウント。ラベル:

    - **state**：エンジンのステータスで、完了すべきフェーズを示します
        * `pending`：まだ処理されていない
        * `written`：すべてのデータがエンコードされ、送信された
        * `closed`：エンジンファイルが閉じられました
        * `imported`：エンジンファイルがターゲットクラスタにインポートされました
        * `completed`：エンジンが完全にインポートされました
    - **result**：現在のフェーズの結果
        * `success`：フェーズが正常に完了しました
        * `failure`：フェーズが失敗しました（完了していません）
- **`lightning_chunks`** (Counter)
    
    チャンクの処理回数とその状態をカウントします。ラベル:

    - **state**: チャンクの状態で、チャンクがどの段階にあるかを示します
        * `estimated`: (状態ではありません) この値は現在のタスクのチャンクの合計数を示します
        * `pending`: ロードされていますがまだ処理されていません
        * `running`: データがエンコードされ送信されています
        * `finished`: 完全なチャンクが処理されました
        * `failed`: 処理中にエラーが発生しました

- **`lightning_import_seconds`** (ヒストグラム)

    テーブルをインポートするために必要な時間のバケット分けされたヒストグラムです。

- **`lightning_row_read_bytes`** (ヒストグラム)

    単一のSQL行のサイズのためのバケット分けされたヒストグラムです。

- **`lightning_row_encode_seconds`** (ヒストグラム)

    単一のSQL行をKVペアにエンコードするために必要な時間のバケット分けされたヒストグラムです。

- **`lightning_row_kv_deliver_seconds`** (ヒストグラム)

    1つの単一のSQL行に対応する一連のKVペアを配信するために必要な時間のバケット分けされたヒストグラムです。

- **`lightning_block_deliver_seconds`** (ヒストグラム)

    KVペアのブロックをImporterに配信するために必要な時間のバケット分けされたヒストグラムです。

- **`lightning_block_deliver_bytes`** (ヒストグラム)

    Importerに配信されるKVペアのブロックの非圧縮サイズのバケット分けされたヒストグラムです。

- **`lightning_chunk_parser_read_block_seconds`** (ヒストグラム)

    データファイルパーサーがブロックを読み取るのに必要な時間のバケット分けされたヒストグラムです。

- **`lightning_checksum_seconds`** (ヒストグラム)

    テーブルのチェックサムを計算するために必要な時間のバケット分けされたヒストグラムです。

- **`lightning_apply_worker_seconds`** (ヒストグラム)

    アイドルワーカーを取得するのに必要な時間のバケット分けされたヒストグラムです（`lightning_idle_workers` ゲージも参照）。ラベル:

    - **name**:
        * `table`
        * `index`
        * `region`
        * `io`
        * `closed-engine`