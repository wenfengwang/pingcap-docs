---
title: TiDB Lightning用語集
summary: TiDB Lightningで使用される特別な用語のリスト。
aliases: ['/docs/dev/tidb-lightning/tidb-lightning-glossary/','/docs/dev/reference/tools/tidb-lightning/glossary/']
---

# TiDB Lightning用語集

このページでは、TiDB Lightningのログ、モニタリング、構成、およびドキュメントで使用される特別な用語について説明します。

<!-- A -->

## A

### 解析する

TiDBテーブルの[統計情報](/statistics.md)を再構築する操作で、[`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md) ステートメントを実行します。

TiDB LightningはTiDBを経由せずにデータをインポートするため、統計情報は自動的に更新されません。そのため、TiDB Lightningはインポート後、明示的に各テーブルを解析します。このステップは、`post-restore.analyze`構成を`false`に設定することで省略できます。

### `AUTO_INCREMENT_ID`

各テーブルには、自動増分列のデフォルト値を提供する`AUTO_INCREMENT_ID`カウンターが関連付けられています。TiDBでは、このカウンターは行IDの割り当てにも使用されます。

TiDB LightningはTiDBを経由せずにデータをインポートするため、`AUTO_INCREMENT_ID`カウンターは自動的に更新されません。そのため、TiDB Lightningは常に`AUTO_INCREMENT_ID`を有効な値に明示的に変更します。このステップは、テーブルに自動増分列がない場合でも常に実行されます。

<!-- B -->

## B

### バックエンド

バックエンドは、TiDB Lightningが解析した結果を送信する宛先です。"backend"としても綴りが用いられます。

詳細については、[TiDB Lightningアーキテクチャ](/tidb-lightning/tidb-lightning-overview.md)を参照してください。

<!-- C -->

## C

### チェックポイント

TiDB Lightningはインポート中に進捗状況を連続的にローカルファイルまたはリモートデータベースに保存します。これにより、途中でクラッシュしても中間状態から再開できます。詳細については[チェックポイント](/tidb-lightning/tidb-lightning-checkpoints.md)セクションを参照してください。

### チェックサム

TiDB Lightningでは、テーブルのチェックサムはそのテーブルの各KVペアの内容から計算された3つの数字のセットです。これらの数値は次のとおりです。

* KVペアの数
* すべてのKVペアの総バイト数
* それぞれのペアの[CR 64-ECMA](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)値のビットごとのXOR

TiDB Lightningは[ローカル](/tidb-lightning/tidb-lightning-glossary.md#local-checksum)および[リモートのチェックサム](/tidb-lightning/tidb-lightning-glossary.md#remote-checksum)を比較することで、インポートされたデータの整合性を検証します。プログラムは、一致しないペアがある場合に停止します。`post-restore.checksum`構成を`false`に設定することで、このチェックをスキップできます。

チェックサムの不一致をどのように適切に処理するかについては、[FAQ](/tidb-lightning/troubleshoot-tidb-lightning.md#checksum-failed-checksum-mismatched-remote-vs-local)も参照してください。

### チャンク

通常、ソースデータの連続する範囲で、1つのファイルに相当します。

ファイルが大きすぎる場合、TiDB Lightningは1つのファイルを複数のチャンクに分割することがあります。

### 圧縮

複数の小さなSSTファイルを1つの大きなSSTファイルにマージし、削除されたエントリをクリーンアップする操作です。TiKVは、TiDB Lightningのインポート中にバックグラウンドでデータを自動的に圧縮します。

> **注意:**
>
> 伝統的な理由から、TiDB Lightningを使用してテーブルをインポートするたびに明示的に圧縮をトリガーするように構成することはまだ可能ですが、これは推奨されません。また、それに対応する設定はデフォルトで無効になっています。

技術的な詳細については、RocksDBの[Compactionのウィキページ](https://github.com/facebook/rocksdb/wiki/Compaction)を参照してください。

<!-- D -->

## D

### データエンジン

実際の行データをソートするための[エンジン](/tidb-lightning/tidb-lightning-glossary.md#engine)です。

テーブルが非常に大きい場合、そのデータは複数のデータエンジンに配置され、タスクのパイプライン化とTiKV Importerのスペースの節約を向上させます。デフォルトでは、100 GBのSQLデータごとに新しいデータエンジンが開かれるようになっており、`mydumper.batch-size`設定を通じて構成できます。

TiDB Lightningは複数のデータエンジンを同時に処理します。これは`lightning.table-concurrency`設定によって制御されます。

<!-- E -->

## E

### エンジン

TiKV Importerでは、エンジンはKVペアをソートするためのRocksDBインスタンスです。

TiDB Lightningはエンジンを介してデータをTiKV Importerに転送します。まずエンジンを開き、その後特定の順序なしにKVペアを送信し、最後にエンジンを閉じます。エンジンは閉じられた後に受信したKVペアをソートします。これらの閉じられたエンジンはさらにTiKVストアにアップロードされ、取り込まれます。

エンジンは一時的なストレージとしてTiKV Importerの`import-dir`を使用し、これは時々 "エンジンファイル"と呼ばれることがあります。

[data engine](/tidb-lightning/tidb-lightning-glossary.md#data-engine)および[index engine](/tidb-lightning/tidb-lightning-glossary.md#index-engine)も参照してください。

<!-- F -->

## F

### フィルター

インポートまたは除外するテーブルを指定する構成リストです。

詳細については、[テーブルフィルター](/table-filter.md)を参照してください。

<!-- I -->

## I

### インポートモード

TiKVの書き込みを最適化する構成であり、読み取り速度と空間利用を低下させます。

TiDB Lightningは実行中に自動的にインポートモードへの切り替えと解除を行います。ただし、TiKVがインポートモードで停止する場合は、`tidb-lightning-ctl`を使用して[通常モード](/tidb-lightning/tidb-lightning-glossary.md#normal-mode)に[強制的に復帰](/tidb-lightning/troubleshoot-tidb-lightning.md#the-tidb-cluster-uses-lots-of-cpu-resources-and-runs-very-slowly-after-using-tidb-lightning)することができます。

### インデックスエンジン

索引をソートするための[エンジン](/tidb-lightning/tidb-lightning-glossary.md#engine)です。

インデックスの数に関係なく、すべてのテーブルには必ず1つのインデックスエンジンが対応しています。

TiDB Lightningは複数のインデックスエンジンを同時に処理します。これは`lightning.index-concurrency`設定によって制御されます。すべてのテーブルが1つのインデックスエンジンを持つため、これによって同時に処理するテーブルの最大数も設定できます。

### 取り込み

[SSTファイル](/tidb-lightning/tidb-lightning-glossary.md#sst-file)の全内容をRocksDB（TiKV）ストアに挿入する操作です。

一度にKVペアを1つずつ挿入するよりも非常に高速な操作です。この操作はTiDB Lightningのパフォーマンスの決定要因です。

技術的な詳細については、RocksDBの[Creating and Ingesting SST files](https://github.com/facebook/rocksdb/wiki/Creating-and-Ingesting-SST-files)のウィキページを参照してください。

<!-- K -->

## K

### KVペア

"key-value pair"の略です。

### KVエンコーダー

SQLまたはCSVの行を解析してKVペアに変換する手順です。複数のKVエンコーダーが並行して実行され、処理を高速化しています。

<!-- L -->

## L

### ローカルチェックサム

TiDB Lightning自体がTiKV ImporterにKVペアを送信する前に計算される、テーブルの[チェックサム](/tidb-lightning/tidb-lightning-glossary.md#checksum)です。

<!-- N -->

## N

### 通常モード

[インポートモード](/tidb-lightning/tidb-lightning-glossary.md#import-mode)が無効になっているモードです。

<!-- P -->

## P

### ポストプロセス

データソース全体が解析され、TiKV Importerに送信された後の期間です。TiDB Lightningは、TiKV Importerが[SSTファイル](/tidb-lightning/tidb-lightning-glossary.md#sst-file)をアップロードし、[取り込む](/tidb-lightning/tidb-lightning-glossary.md#ingest)のを待っています。

<!-- R -->

## R

### リモートチェックサム

TiDBによってインポート後に計算される、テーブルの[チェックサム](/tidb-lightning/tidb-lightning-glossary.md#checksum)です。

<!-- S -->

## S

### スキャッタリング

[Region](/glossary.md#regionpeerraft-group)のリーダーとピアをランダムに再割り当てする操作です。スキャッタリングにより、インポートされたデータがTiKVストア全体に均等に分散されるため、PDにかかる負荷が軽減されます。

### 分割

エンジンは通常非常に大きいです（約100 GB）、これを1つの[region](/glossary.md#regionpeerraft-group)として扱うとTiKVには使い勝手がありません。そのため、TiKV Importerはエンジンを複数の小さな[SSTファイル](/tidb-lightning/tidb-lightning-glossary.md#sst-file)に分割し（TiKV Importerの`import.region-split-size`設定で構成可能）、アップロードします。

### SSTファイル

"SST"は"sorted string table"の略です。SSTファイルは、RocksDB（およびTiKV）のキーと値のペアのコレクションのネイティブストレージ形式です。
```
TiKV Importer produces SST files from a closed [engine](/tidb-lightning/tidb-lightning-glossary.md#engine). These SST files are uploaded and then [ingested](/tidb-lightning/tidb-lightning-glossary.md#ingest) into TiKV stores.
```