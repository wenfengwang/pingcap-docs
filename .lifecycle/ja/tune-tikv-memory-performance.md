---
title: TiKVメモリパラメータパフォーマンスをチューニングする
summary: 最適なパフォーマンスのためにTiKVパラメータをチューニングする方法を学びます。
aliases: ['/docs/dev/tune-tikv-performance/','/docs/dev/reference/performance/tune-tikv/','/tidb/dev/tune-tikv-performance']
---

# TiKVメモリパラメータパフォーマンスのチューニング

このドキュメントでは、最適なパフォーマンスを得るためにTiKVパラメータを調整する方法について説明します。デフォルトの設定ファイルは[etc/config-template.toml](https://github.com/tikv/tikv/blob/master/etc/config-template.toml) で見つけることができます。設定を変更するには、[TiUPを使用](/maintain-tidb-using-tiup.md#modify-the-configuration)したり、一部の設定項目については[TiKVを動的に変更](/dynamic-config.md#modify-tikv-configuration-dynamically)することができます。完全な構成については、[TiKV設定ファイル](/tikv-configuration-file.md)を参照してください。

TiKVは、アーキテクチャの最下層で永続的なストレージとしてRocksDBを使用しています。そのため、多くのパフォーマンスパラメータはRocksDBに関連しています。TiKVは2つのRocksDBインスタンスを使用します。デフォルトのRocksDBインスタンスはKVデータを格納し、Raft RocksDBインスタンス（RaftDB）はRaftログを格納します。

TiKVはRocksDBから`Column Families` (CF) を実装しています。   

- デフォルトのRocksDBインスタンスは、`default`, `write`, `lock` のCFにKVデータを格納します。

    - `default` CF は実際のデータを格納します。対応するパラメータは `[rocksdb.defaultcf]` にあります。
    - `write` CF はMulti-Version Concurrency Control (MVCC)でのバージョン情報やインデックス関連のデータを格納します。対応するパラメータは `[rocksdb.writecf]` にあります。
    - `lock` CF はロック情報を格納します。システムはデフォルトのパラメータを使用します。

- Raft RocksDB（RaftDB）インスタンスはRaftログを格納します。

    - `default` CF はRaftログを格納します。対応するパラメータは `[raftdb.defaultcf]` にあります。

TiKV 3.0以降、すべてのCFが1つのブロックキャッシュインスタンスを共有します。`capacity` パラメータを `[storage.block-cache]` の下に設定することでキャッシュのサイズを設定できます。ブロックキャッシュが大きいほど、より多くのホットデータをキャッシュでき、データの読み取りが簡単になりますが、その分システムメモリをより多く占有します。

TiKV 3.0以前では、共有ブロックキャッシュはサポートされておらず、各CFごとにブロックキャッシュを設定する必要があります。

それぞれのCFには別々の `write buffer` もあります。`write-buffer-size` パラメータでサイズを設定できます。

## パラメータ仕様

```
# ログレベル：trace, debug, warn, error, info, off.
log-level = "info"

[server]
# リスニングアドレスの設定
# addr = "127.0.0.1:20160"

# gRPC用スレッドプールのサイズ
# grpc-concurrency = 4
# 各TiKVインスタンス間のgRPC接続数
# grpc-raft-conn-num = 10

# TiDBからのほとんどのリクエストはTiKVのcoprocessorに送られます。このパラメータはcoprocessorのスレッド数を設定します。
# 多くの読み取りリクエストが存在する場合、スレッド数を追加し、スレッド数をシステムCPUコア数以内に保ちます。
# たとえば、TiKVをデプロイした32コアのマシンでは、リピータブルリードのシナリオでこのパラメータを30に設定できます。
# このパラメータが設定されていない場合、TiKVは自動的にCPUコア数*0.8に設定します。
# end-point-concurrency = 8

# スケジュールのレプリカにタグ付けするためのTiKVインスタンス
# labels = {zone = "cn-east-1", host = "118", disk = "ssd"}

[storage]
# データディレクトリ
# data-dir = "/tmp/tikv/store"

# ほとんどの場合、デフォルト値を使用できます。データのインポート時は、パラメータを1024000に設定することを推奨します。
# scheduler-concurrency = 102400
# このパラメータは書き込みスレッドの数を制御します。頻繁に書き込み操作が発生する場合、このパラメータ値を高く設定します。
# `top -H -p tikv-pid` を実行し、`sched-worker-pool` と名前の付いたスレッドがビジーである場合、
# パラメータ `scheduler-worker-pool-size` の値を高く設定し、書き込みスレッドの数を増やします。
# scheduler-worker-pool-size = 4

[storage.block-cache]
## すべてのRocksDB Column Familiesに共有ブロックキャッシュを作成するかどうか。
##
## ブロックキャッシュはRocksDBが非圧縮ブロックをキャッシュするために使用されます。大きなブロックキャッシュは読み取りを加速します。
## 共有ブロックキャッシュをオンにすることをお勧めします。設定する必要があるのは総キャッシュサイズだけなので、簡単に構成できます。
## ほとんどの場合、基本的なLRUアルゴリズムを使用してキャッシュの使用を自動でバランスさせることができるはずです。
##
## 共有ブロックキャッシュがオンの場合、storage.block-cacheセッションの残りの設定は有効です。
## v6.6.0以降、`shared` オプションは常に有効であり、無効にすることはできません。
# shared = true

## 共有ブロックキャッシュのサイズ。通常、システムの総メモリの30%〜50%に調整することが推奨されます。
## 設定がされていない場合は、以下のフィールドまたはそれらのデフォルト値の合計によって決定されます。
##   * rocksdb.defaultcf.block-cache-size またはシステムの総メモリの25%
##   * rocksdb.writecf.block-cache-size またはシステムの総メモリの15%
##   * rocksdb.lockcf.block-cache-size またはシステムの総メモリの2%
##   * raftdb.defaultcf.block-cache-size またはシステムの総メモリの2%
##
## 1つの物理マシン上に複数のTiKVノードを展開する場合、このパラメータを明示的に設定します。
## そうしないと、TiKVでOOMの問題が発生する可能性があります。
# capacity = "1GB"

[pd]
# PDアドレス
# endpoints = ["127.0.0.1:2379","127.0.0.2:2379","127.0.0.3:2379"]

[metric]
# Prometheus Pushgatewayにメトリクスをプッシュする間隔
interval = "15s"
# Prometheus Pushgatewayアドレス
address = ""
job = "tikv"

[raftstore]
# Raft RocksDBディレクトリ。デフォルト値は[storage.data-dir]のRaftサブディレクトリです。
# マシンに複数のディスクがある場合、Raft RocksDBのデータを異なるディスクに保存してTiKVのパフォーマンスを向上させます。
# raftdb-path = "/tmp/tikv/store/raft"

# Region内のデータサイズの変更が閾値値よりも大きい場合、TiKVはそのRegionを分割する必要があるかどうかをチェックします。
# チェックプロセスでデータのスキャンコストを削減するため、データのインポートプロセスでは値を32MBに設定します。
# 通常の操作状態では、デフォルト値に設定します。
region-split-check-diff = "32MB"

[coprocessor]
## 範囲[a,e)のRegionのサイズが `region_max_size` の値よりも大きい場合、TiKVはそのRegionを複数のRegionに分割しようとします、
## 例：範囲[a,b), [b,c), [c,d), [d,e) のRegion。
## 分割後、分割されたRegionのサイズは `region_split_size` の値と等しくなります（または `region_split_size` の値よりもわずかに大きくなります）。
# region-max-size = "144MB"
# region-split-size = "96MB"

[rocksdb]
# RocksDBバックグラウンドタスクの最大スレッド数。バックグラウンドタスクにはコンパクションやフラッシュが含まれます。
# コンパクションの理由についての詳細情報はRocksDBに関連する資料を参照してください。データサイズの変更や他の大きな書き込みトラフィックが発生する場合、
# より多くのスレッドを有効にすることを推奨します。ただし、有効にするスレッド数はCPUコア数よりも少ないように設定してください。
# たとえば、データをインポートする際には、32コアCPUのマシンで28に設定します。
# max-background-jobs = 8

# RocksDBが開けるファイルハンドルの最大数
# max-open-files = 40960

# RocksDB MANIFESTのファイルサイズ制限。詳細については、https://github.com/facebook/rocksdb/wiki/MANIFEST を参照してください。
max-manifest-file-size = "20MB"

# RocksDBの書き込み先ログのディレクトリ。マシンに2つのディスクがある場合、RocksDBデータとWALログを別々のディスクに保存してTiKVのパフォーマンスを向上させます。
# wal-dir = "/tmp/tikv/store"

# RocksDBのWALをアーカイブする際に以下の2つのパラメータを使用します。
# 詳細については、https://github.com/facebook/rocksdb/wiki/How-to-persist-in-memory-RocksDB-database%3F を参照してください。
# wal-ttl-seconds = 0
# wal-size-limit = 0

# ほとんどの場合、RocksDB WALログの総合サイズの上限をデフォルト値に設定します。
# max-total-wal-size = "4GB"

# RocksDBコンパクション中のreadahead機能を有効にするためにこのパラメータを使用します。
メカニカルディスクを使用している場合は、少なくとも2MBに設定することをお勧めします。
# compaction-readahead-size = "2MB"

[rocksdb.defaultcf]
# データブロックのサイズ。RocksDBはブロック単位でデータを圧縮します。
```yaml
# 他のデータベースのページと同様に、ブロックはブロックキャッシュでキャッシュされる最小単位です。
block-size = "64KB"

# RocksDBデータの各レイヤーのコンパクションモード。オプションの値には、no、snappy、zlib、bzip2、lz4、lz4hc、zstd
# などがあります。Snappy圧縮ファイルは[公式Snappy形式](https://github.com/google/snappy)である必要があります。他の
# Snappy圧縮のバリエーションはサポートされていません。
# "no:no:lz4:lz4:lz4:zstd:zstd" はレベル0およびレベル1のコンパクションがないことを示し、
# レベル2からレベル4までlz4コンパクションアルゴリズムを使用し、レベル5からレベル6までzstdコンパクションアルゴリズムを
# 使用することを示しています。"no" はコンパクションがないことを意味します。 "lz4" は、中速のコンパクション速度と
# コンパクション比率を持つコンパクションアルゴリズムです。zlibのコンパクション比率は高いです。ストレージスペースに
# フレンドリーですが、コンパクション速度が遅いです。このコンパクションは多くのCPUリソースを占有します。異なるマシンは、
# CPUリソースとI/Oリソースに応じてコンパクションモードを展開します。たとえば、"no:no:lz4:lz4:lz4:zstd:zstd"
# のコンパクションモードを使用して多くのデータを書き込む（インポートする）ときに、システムのI/O圧力が大きいことを
# 発見した場合（iostatコマンドを実行して%utilが100%になる、または多くのiowaitsを見つけるためにtopコマンドを実行する）は、
# レベル0とレベル1を圧縮してCPUリソースをI/Oリソースに交換することができます。"no:no:lz4:lz4:lz4:zstd:zstd"
# のコンパクションモードを使用して多くのデータを書き込むとシステムのI/O圧力が大きくないことがわかった場合でも、
# CPUリソースが不足しているとします。次にtopコマンドを実行し、-Hオプションを選択します。バックグラウンドスレッド
# （つまりRocksDBのコンパクションスレッド）が多く実行されていることがわかった場合、I/OリソースをCPUリソースに交換し、
# "no:no:no:lz4:lz4:zstd:zstd" のコンパクションモードに変更します。要するに、システムの既存リソースを最大限に活用し、
# 現在のリソースに関するTiKVのパフォーマンスを改善することを目指しています。
compression-per-level = ["no", "no", "lz4", "lz4", "lz4", "zstd", "zstd"]

# RocksDBメンテーブルのサイズ
write-buffer-size = "128MB"

# メンテーブルの最大数。RocksDBに書き込まれたデータはまずWALログに記録され、次にメンテーブルに挿入されます。メンテーブルが
# `write-buffer-size` のサイズ制限に達すると、読み取り専用になり、新しい書き込み操作を受け入れる新しいメンテーブルが生成されます。
# RocksDBのフラッシュスレッドは読み取り専用のメンテーブルをディスクにフラッシュしてレベル0のsstファイルになります。
# `max-background-flushes` はフラッシュスレッドの最大数を制御します。フラッシュスレッドがビジーで、ディスクにフラッシュされる
# 待機中のメンテーブルの数が `max-write-buffer-number` の制限に達した場合、RocksDBは新しい操作を停止します。
# "Stall" はRocksDBのフローコントロールメカニズムです。データのインポート時に、 `max-write-buffer-number` の値を10など、
# より高く設定することができます。
max-write-buffer-number = 5

# レベル0のsstファイル数が `level0-slowdown-writes-trigger` の制限に達した場合、RocksDBは
# 書き込み操作を遅延させようとします。なぜなら、レベル0のsstファイルが多すぎるとRocksDBの
# 読み取り圧力が高くなる可能性があるからです。`level0-slowdown-writes-trigger` と `level0-stop-writes-trigger` は
# RocksDBのフローコントロールのためのものです。レベル0のsstファイル数が4（デフォルト値）に達した場合、
# レベル0のsstファイルとそのレベル0のsstファイルと重複するレベル1のsstファイルはコンパクションを実行して、
# 読み取り圧力を緩和します。
level0-slowdown-writes-trigger = 20

# レベル0のsstファイル数が `level0-stop-writes-trigger` の制限に達した場合、RocksDBは新しい書き込み操作を停止します。
level0-stop-writes-trigger = 36

# レベル1のデータサイズが `max-bytes-for-level-base` の制限値に達した場合、レベル1のsstファイルと
# それらのレベル2の重複sstファイルはコンパクションを実行します。 `max-bytes-for-level-base` を設定する際の
# 最も基本的な原則は、 `max-bytes-for-level-base` の値がレベル0のデータボリュームとほぼ等しいように保証することです。
# このために不必要なコンパクションを削減します。たとえば、コンパクションモードが "no:no:lz4:lz4:lz4:lz4:lz4" の場合、
# `max-bytes-for-level-base` の値は write-buffer-size * 4 です。レベル0とレベル1のコンパクションがないため、
# コンパクションのトリガー条件としてのレベル0のsstファイル数が4（デフォルト値）になります。レベル0とレベル1の両方が
# コンパクションを採用する場合、RocksDBログのサイズを分析して、メンテーブルから圧縮されたsstファイルのサイズを
# 知る必要があります。たとえば、ファイルサイズが32MBの場合、 `max-bytes-for-level-base` の提案される値は、
# 32MB * 4 = 128MB です。
max-bytes-for-level-base = "512MB"

# sstファイルサイズ。レベル0のsstファイルサイズは、`write-buffer-size` とレベル0のコンパクションアルゴリズムの影響を受けます。
# `target-file-size-base` は、レベル1〜レベル6の単一のsstファイルのサイズを制御するために使用されます。
target-file-size-base = "32MB"

[rocksdb.writecf]
# `rocksdb.defaultcf.compression-per-level` と同じに設定します。
compression-per-level = ["no", "no", "lz4", "lz4", "lz4", "zstd", "zstd"]

# `rocksdb.defaultcf.write-buffer-size` と同じに設定します。
write-buffer-size = "128MB"
max-write-buffer-number = 5
min-write-buffer-number-to-merge = 1

# `rocksdb.defaultcf.max-bytes-for-level-base` と同じに設定します。
max-bytes-for-level-base = "512MB"
target-file-size-base = "32MB"

[raftdb]
# RaftDBが開くことができる最大ファイルハンドル数
# max-open-files = 40960

# RaftDBコンパクションのリードアヘッド機能を有効にします。機械式ディスクを使用している場合、この値を少なくとも2MBに設定することをお勧めします。
# compaction-readahead-size = "2MB"

[raftdb.defaultcf]
# `rocksdb.defaultcf.compression-per-level` と同じに設定します。
compression-per-level = ["no", "no", "lz4", "lz4", "lz4", "zstd", "zstd"]

# `rocksdb.defaultcf.write-buffer-size` と同じに設定します。
write-buffer-size = "128MB"
max-write-buffer-number = 5
min-write-buffer-number-to-merge = 1

# `rocksdb.defaultcf.max-bytes-for-level-base` と同じに設定します。
max-bytes-for-level-base = "512MB"
target-file-size-base = "32MB"
```

## TiKVメモリ使用量

`ブロックキャッシュ` と `書き込みバッファ` 以外にも、TiKVがシステムメモリを占有するシナリオがあります。

+ システムのページキャッシュとして一部のメモリが予約されます。

+ TiKVは `select * from ...` などの大規模クエリを処理する場合、データを読み取り、対応するデータ構造をメモリに生成し、
  この構造をTiDBに返します。このプロセス中、TiKVは一部のメモリを占有します。

## TiKVの推奨設定

+ 本番環境では、CPUコア数が8未満またはメモリが32GB未満のマシンにTiKVをデプロイすることはお勧めしません。

+ 高い書き込みスループットが必要な場合、スループット容量が高いディスクを使用することをお勧めします。

+ 非常に低い読み書き遅延が必要な場合、IOPSが高いSSDを使用することをお勧めします。
```