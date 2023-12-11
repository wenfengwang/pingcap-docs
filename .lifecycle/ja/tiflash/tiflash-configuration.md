---
title: TiFlashの構成
summary: TiFlashの設定方法について学びます。
aliases: ['/docs/dev/tiflash/tiflash-configuration/','/docs/dev/reference/tiflash/configuration/']
---

# TiFlashの構成

このドキュメントでは、TiFlashの展開および使用に関連する構成パラメータについて紹介します。

## PDスケジューリングパラメータ

[PD制御](/pd-control.md)を使用してPDスケジューリングパラメータを調整できます。Tiupを使用してクラスタを展開および管理する際には、`pd-ctl -u <pd_ip:pd_port>`を置き換えるために`tiup ctl:v<CLUSTER_VERSION> pd`を使用できます。

- [`replica-schedule-limit`](/pd-configuration-file.md#replica-schedule-limit): レプリカ関連のオペレーターが生成される速度を決定します。このパラメータは、ノードをオフラインにする操作やレプリカを追加する操作などに影響を与えます。

  > **注意:**
  >
  > このパラメータの値は、`region-schedule-limit`の値よりも小さくする必要があります。そうしないと、通常のTiKVノード間でのリージョンスケジューリングに影響が出ます。

- `store-balance-rate`: 各TiKV/TiFlashストアのリージョンのスケジューリング速度を制限します。このパラメータは、ストアがクラスタに新たに参加した場合にのみ効果があります。既存のストアの設定を変更する場合は、以下のコマンドを使用してください。

  > **注意:**
  >
  > v4.0.2以降、`store-balance-rate`パラメータは非推奨となり、`store limit`コマンドに変更が加えられました。詳細は[store-limit](/configure-store-limit.md)を参照してください。

    - `pd-ctl -u <pd_ip:pd_port> store limit <store_id> <value>`コマンドを実行して、指定されたストアのスケジューリング速度を設定します（`store_id`は`pd-ctl -u <pd_ip:pd_port> store`コマンドを実行して取得できます）。
    - 特定のストアのリージョンのスケジューリング速度を設定しない場合は、このストアは`store-balance-rate`の設定を継承します。
    - `pd-ctl -u <pd_ip:pd_port> store limit`コマンドを実行して、`store-balance-rate`の現在の設定値を表示できます。

- [`replication.location-labels`](/pd-configuration-file.md#location-labels): TiKVインスタンスの地理的配置に関する情報を示します。キーの順序は、異なるラベルの階層関係を示します。TiFlashが有効になっている場合は、デフォルト値を設定するために[`pd-ctl config placement-rules`](/pd-control.md#config-show--set-option-value--placement-rules)を使用する必要があります。詳細については、[geo-distributed-deployment-topology](/geo-distributed-deployment-topology.md)を参照してください。

## TiFlash構成パラメータ

このセクションでは、TiFlashの構成パラメータについて紹介します。

> **ヒント:**
>
> 構成項目の値を調整する必要がある場合は、[構成の変更](/maintain-tidb-using-tiup.md#modify-the-configuration)を参照してください。

### `tiflash.toml`ファイルの構成

```toml
## TPC/HTTPなどのサポートサービスのリスニングホスト。このマシンのすべてのIPアドレスでリスニングするように設定することを推奨します。
listen_host = "0.0.0.0"
## TiFlash TCPサービスポート。このポートは内部テストに使用され、デフォルトでは9000に設定されます。TiFlash v7.1.0以前は、このポートはデフォルトで有効になっており、セキュリティリスクがあります。セキュリティを強化するためには、このポートにアクセス制御を適用し、ホワイトリスト指定されたIPアドレスからのアクセスのみを許可することを推奨します。TiFlash v7.1.0からは、このポートの構成をコメントアウトすることでセキュリティリスクを回避できます。TiFlash構成ファイルでこのポートの設定がされていない場合は、このポートは無効になります。
## いかなるTiFlash展開においても、このポートを設定することは**推奨されません**（注: TiFlash v7.1.0以降、TiUP >= v1.12.5またはTiDB Operator >= v1.5.0によってデフォルトでポートが無効になり、より安全になります）。
# tcp_port = 9000
## データブロックのメタデータのキャッシュサイズ制限。通常、この値を変更する必要はありません。
mark_cache_size = 1073741824
## データブロックの最小最大インデックスのキャッシュサイズ制限。通常、この値を変更する必要はありません。
minmax_index_cache_size = 1073741824
## DeltaIndexのキャッシュサイズ制限。デフォルト値は0で、制限なしを示します。
delta_index_cache_size = 0

## TiFlashデータのストレージパス。複数のディレクトリがある場合は、各ディレクトリをカンマで区切って指定します。
## pathおよびpath_realtime_modeは、v4.0.9以降で非推奨です。マルチディスク展開シナリオでのパフォーマンスを向上させるために、[storage]セクションの構成を使用してください。
## TiDB v5.2.0以降、storage.io_rate_limit構成を使用する必要がある場合は、TiFlashデータのストレージパスを同時にstorage.main.dirに設定する必要があります。
## [storage]構成が存在する場合、pathおよびpath_realtime_modeの構成は無視されます。
# path = "/tidb-data/tiflash-9000"
## または
# path = "/ssd0/tidb-data/tiflash,/ssd1/tidb-data/tiflash,/ssd2/tidb-data/tiflash"
## デフォルト値はfalseです。trueに設定し、pathで複数のディレクトリが設定されている場合、最新のデータは最初のディレクトリに保存され、古いデータは残りのディレクトリに保存されます。
# path_realtime_mode = false

## TiFlash一時ファイルが保存されるパス。デフォルトでは、pathの最初のディレクトリ、またはstorage.latest.dirに"/tmp"が追加されたものです。
# tmp_path = "/tidb-data/tiflash-9000/tmp"

## v4.0.9以降、ストレージパス設定が有効になります
[storage]

    ## DTFileフォーマット
    ## * format_version = 2、バージョン<v6.0.0のデフォルトフォーマットです。
    ## * format_version = 3、v6.0.0およびv6.1.xのデフォルトフォーマットで、より多くのデータ検証機能を提供します。
    ## * format_version = 4、v6.2.0からv7.3.0のデフォルトフォーマットで、書き込み増幅とバックグラウンドタスクリソース消費を削減します。
    ## * format_version = 5、v7.4.0およびそれ以降のデフォルトフォーマット（v7.3.0で導入）で、小さなファイルをマージして物理ファイルの数を減らします。
    # format_version = 5

    [storage.main]
    ## メインデータを保存するディレクトリのリスト。合計データの90%以上がディレクトリリストに保存されます。
    ## dir = [ "/tidb-data/tiflash-9000" ]
    ## または
    # dir = [ "/ssd0/tidb-data/tiflash", "/ssd1/tidb-data/tiflash" ]

    ## storage.main.dirの各ディレクトリの最大ストレージ容量。
    ## 設定されていない場合やゼロに設定されている場合、実際のディスク（ディレクトリがあるディスク）の容量が使用されます。
    ## "10GB"などの人が読める数字はまだサポートされていません。
    ## 容量はバイトで指定されます。
    ## 容量リストのサイズは、dirサイズと同じである必要があります。
    ## 例えば:
    # capacity = [ 10737418240, 10737418240 ]

    [storage.latest]
    ## 最新のデータを保存するディレクトリのリスト。合計データの10%程度がディレクトリリストに保存されます。こちらのディレクトリ（またはディレクトリ）は、storage.main.dirよりも高いIOPSメトリックを必要とします。
    ## 設定されていない場合（デフォルト）、storage.main.dirの値が使用されます。
    # dir = [ ]
    ## storage.latest.dirの各ディレクトリの最大ストレージ容量。
    ## 設定されていない場合やゼロに設定されている場合、実際のディスク（ディレクトリがあるディスク）の容量が使用されます。
    # capacity = [ 10737418240, 10737418240 ]

    ## [storage.io_rate_limit]の設定はv5.2.0で新しくなりました。
    [storage.io_rate_limit]
    ## この構成項目は、I/Oトラフィックを制限するかどうかを決定します。デフォルトでは無効になっています。TiFlashにおけるこのトラフィック制限は、ディスクの帯域幅が小さくて特定のサイズであるクラウドストレージに適しています。
    ## ディスク読み取りと書き込みの合計I/O帯域幅。単位はバイトで、デフォルト値は0で、デフォルトではI/Oトラフィックは制限されません。
    # max_bytes_per_sec = 0
    ## max_read_bytes_per_secおよびmax_write_bytes_per_secは、max_bytes_per_secと同様の意味を持っています。max_read_bytes_per_secはディスク読み取りのためのI/O帯域幅の合計を意味し、max_write_bytes_per_secはディスク書き込みのためのI/O帯域幅の合計を意味します。
    ## これらの構成項目は、ディスク読み取りと書き込みのI/O帯域幅を個別に制限します。Google Cloudが提供するPersistent Diskなど、ディスク読み取りと書き込みのI/O帯域幅の制限を個別に計算するクラウドストレージ用に使用できます。
    ## max_bytes_per_secの値が0でない場合、max_bytes_per_secが優先されます。
    ## <br>
    # max_read_bytes_per_sec = 0
    # max_write_bytes_per_sec = 0

    ## 以下のパラメータは、異なるI/Oトラフィックタイプに割り当てられた帯域幅の重みを制御します。一般的に、これらのパラメータを調整する必要はありません。
    ## TiFlashはI/Oリクエストを前景書き込み、バックグラウンド書き込み、前景読み込み、バックグラウンド読み込みの4つのタイプに内部分割します。
    ## I/Oトラフィックリミットが初期化されると、TiFlashは以下の重み比率に従って帯域幅を割り当てます。
    ## 以下のデフォルト設定は、各トラフィックタイプが25％の重みを得ることを示しています（25 /（25 + 25 + 25 + 25）= 25％）。
    ## 重みが0に設定されている場合、対応するI/Oトラフィックは制限されません。
    # foreground_write_weight = 25
    # background_write_weight = 25
    # foreground_read_weight = 25
    # background_read_weight = 25
    ## TiFlashは、現在のI/O負荷に応じて異なるI/Oタイプのトラフィックリミットを自動調整できます。調整された帯域幅は、上記で設定された重み比率を超える場合があります。
    ## auto_tune_secは自動調整の間隔を示します。単位は秒です。 auto_tune_secの値が0の場合、自動チューニングは無効になります。
    # auto_tune_sec = 5

    ## 以下の設定は、TiFlashの分離されたストレージおよびコンピュートアーキテクチャモードにのみ影響します。詳細については、https://docs.pingcap.com/tidb/dev/tiflash-disaggregated-and-s3のドキュメントを参照してください。
    # [storage.s3]
    # endpoint: http://s3.{region}.amazonaws.com # S3エンドポイントアドレス
    # bucket: mybucket                           # TiFlashはこのバケットにすべてのデータを保存します
    # root: /cluster1_data                       # S3バケット内でデータが保存されているルートディレクトリ
    # access_key_id: {ACCESS_KEY_ID}             # ACCESS_KEY_IDを使用してS3にアクセス
    # secret_access_key: {SECRET_ACCESS_KEY}     # SECRET_ACCESS_KEYを使用してS3にアクセス
    # [storage.remote.cache]
    # dir: /data1/tiflash/cache        # コンピュートノードのローカルデータキャッシュディレクトリ
    # capacity: 858993459200           # 800 GiB

[flash]
    tidb_status_addr = TiDBのステータスポートとアドレス # 複数のアドレスはコンマで区切られます。
    service_addr = TiFlash Raftサービスおよびコプロセッササービスのリスニングアドレス。

    ## v7.4.0で導入された機能。現在のRaftステートマシンによって適用された`applied_index`と前回のディスクスパイリング時の`applied_index`の間隔が`compact_log_min_gap`を超える場合、TiFlashはTiKVから`CompactLog`コマンドを実行し、データをディスクにスパイルします。このギャップを増やすと、TiFlashのディスクスパイリング頻度が低下し、ランダム書き込みシナリオでの読み取り遅延が低減される可能性がありますが、メモリオーバーヘッドが増加する可能性もあります。このギャップを減らすと、TiFlashのディスクスパイリング頻度が増加し、TiFlash内のメモリ圧力が緩和される可能性があります。ただし、現時点では、このギャップが0に設定されていても、TiFlashのディスクスパイリング頻度はTiKVよりも高くなりません。
    ## デフォルト値を使用することをお勧めします。
    # compact_log_min_gap = 200
    ## v5.0で導入された機能。TiFlashがキャッシュしたリージョンの行数またはサイズが、以下のいずれかの閾値を超えると、TiFlashはTiKVから`CompactLog`コマンドを実行し、データをディスクにスパイルします。
    ## デフォルト値を使用することをお勧めします。
    # compact_log_min_rows = 40960 # 40k
    # compact_log_min_bytes = 33554432 # 32MB

    ## 以下の構成項目は、TiFlashの分離されたストレージおよびコンピュートアーキテクチャモードにのみ影響します。詳細については、https://docs.pingcap.com/tidb/dev/tiflash-disaggregated-and-s3のドキュメントを参照してください。
    # disaggregated_mode = tiflash_write # サポートされているモードは `tiflash_write` または `tiflash_compute` です。

## 複数のTiFlashノードは、PDに配置ルールを追加または削除するためにマスターを選出し、flash.flash_cluster内の構成がこのプロセスを制御します。
[flash.flash_cluster]
    refresh_interval = マスターが定期的に有効期間を更新します。
    update_rule_interval = マスターが定期的にTiFlashレプリカのステータスを取得し、PDと対話します。
    master_ttl = 選出されたマスターの有効期間。
    cluster_manager_path = pdバディディレクトリの絶対パス。
    log = pdバディログパス。

[flash.proxy]
    addr = プロキシのリスニングアドレス。空白の場合、デフォルトで127.0.0.1:20170が使用されます。
    advertise-addr = addrの外部アクセスアドレス。空白の場合、デフォルトで「addr」が使用されます。
    data-dir = プロキシのデータ保存パス。
    config = プロキシの構成ファイルパス。
    log-file = プロキシのログパス。
    log-level = プロキシのログレベル。デフォルトでは "info" が使用されます。
    status-addr = プロキシがメトリクス|ステータス情報をプルするリスニングアドレス。空白の場合、デフォルトで127.0.0.1:20292が使用されます。
    advertise-status-addr = status-addrの外部アクセスアドレス。空白の場合、デフォルトで「status-addr」が使用されます。

[logger]
    ## ログレベル（使用可能なオプション：「trace」、「debug」、「info」、「warn」、「error」）。デフォルト値は「debug」です。
    level = "debug"
    log = TiFlashログパス
    errorlog = TiFlashエラーログパス
    ## 1つのログファイルのサイズ。デフォルト値は「100M」です。
    size = "100M"
    ## 保存されるログファイルの最大数。デフォルト値は「10」です。
    count = 10

[raft]
    ## PDサービスアドレス。複数のアドレスはコンマで区切られます。
    pd_addr = "10.0.1.11:2379,10.0.1.12:2379,10.0.1.13:2379"

[status]
    ## Prometheusがメトリクス情報をプルするポート。デフォルト値は8234です。
    metrics_port = 8234

[profiles]

[profiles.default]
    ## デフォルト値はfalseです。このパラメータは、DeltaTreeストレージエンジンのセグメントが論理分割を使用するかどうかを決定します。
    ## 論理的な分割を使用すると、書き込みの増幅が低減される可能性があります。ただし、これはディスクスペースの浪費につながります。
    ## v6.2.0以降のバージョンでは、デフォルト値を `false` に維持し、`true`に変更しないよう強くお勧めします。詳細については、既知の問題[#5576](https://github.com/pingcap/tiflash/issues/5576)を参照してください。
    # dt_enable_logical_split = false

    ## 単一のクエリで生成される中間データのメモリ使用量制限。
    ## 値が整数の場合はバイト単位です。たとえば、34359738368は32 GiBのメモリ制限を意味し、0は制限がありません。
    ## 値が[0.0、1.0)の範囲の浮動小数点数の場合、許可されたメモリ使用量の全体メモリに対する比率を意味します。たとえば、0.8は総メモリの80％を意味し、0.0は制限がありません。
    ## デフォルト値は0で、制限がありません。
    ## この制限を超えるメモリをクエリが使用しようとすると、クエリは中止され、エラーが報告されます。
    max_memory_usage = 0

    ## すべてのクエリで生成される中間データのメモリ使用量制限。
    ## 値が整数の場合はバイト単位です。たとえば、34359738368は32 GiBのメモリ制限を意味し、0は制限がありません。
    ## 値が[0.0、1.0)の範囲の浮動小数点数の場合、許可されたメモリ使用量の全体メモリに対する比率を意味します。たとえば、0.8は総メモリの80％を意味し、0.0は制限がありません。
    ## デフォルト値は0.8で、総メモリの80％を意味します。
    ## この制限を超えるメモリをクエリが使用しようとすると、クエリは中止され、エラーが報告されます。
    max_memory_usage_for_all_queries = 0.8

    ## v5.0で導入された機能。この項目は、TiFlashコプロセッサが同時に実行することができる最大のコプリクエスト数を指定します。リクエスト数が指定された値を超えると、余分なリクエストはキューに入れられます。構成値が0に設定されているか設定されていない場合、2倍の物理コア数がデフォルト値として使用されます。
    cop_pool_size = 0
    ## v5.0で導入された機能。この項目は、TiFlashコプロセッサが同時に実行することができる最大のバッチリクエスト数を指定します。リクエスト数が指定された値を超えると、余分なリクエストはキューに入れられます。構成値が0に設定されているか設定されていない場合、2倍の物理コア数がデフォルト値として使用されます。
    batch_cop_pool_size = 0
    ## v6.1.0で導入された機能。この項目は、TiFlashがTiDBからALTER TABLE ... COMPACTを受け取った際に同時に処理できるリクエスト数を指定します。
    ## 値が0に設定されている場合、デフォルト値1が適用されます。
```toml
manual_compact_pool_size = 1
## v5.4.0で新しく追加された項目です。この項目は、TiFlashの高同時実行シナリオにおいてCPUの利用率を大幅に向上させるエラスティックスレッドプール機能を有効または無効にします。デフォルト値はtrueです。
enable_elastic_threadpool = true
## TiFlashストレージエンジンの圧縮アルゴリズムです。値はLZ4、zstd、またはLZ4HCのいずれかで、大文字小文字を区別しません。デフォルトではLZ4が使用されます。
dt_compression_method = "LZ4"
## TiFlashストレージエンジンの圧縮レベルです。デフォルト値は1です。
## dt_compression_methodがLZ4の場合、この値を1に設定することを推奨します。
## dt_compression_methodがzstdの場合、この値を-1（圧縮率が小さく、読み取み性能が向上する）または1に設定することを推奨します。
## dt_compression_methodがLZ4HCの場合、この値を9に設定することを推奨します。
dt_compression_level = 1

## v6.2.0で新しく追加された項目です。この項目は、PageStorageデータファイルにおける有効データの最小比率を指定します。PageStorageデータファイルにおける有効データの比率がこの設定値よりも低い場合、ファイル内のデータを整理するためにGCがトリガーされます。デフォルト値は0.5です。
dt_page_gc_threshold = 0.5

## v7.0.0で新しく追加された項目です。この項目は、グループ化キーによるHashAggregation演算子に対してディスクスピルがトリガーされる前に使用可能な最大メモリを指定します。メモリ使用量が閾値を超えると、HashAggregationはディスクにスピルしてメモリ使用量を減らします。この項目はデフォルトで0に設定されており、メモリ使用量が制限されず、HashAggregationに対してディスクスピルが使用されないことを意味します。
max_bytes_before_external_group_by = 0

## v7.0.0で新しく追加された項目です。この項目は、ソートまたはtopN演算子に対してディスクスピルがトリガーされる前に使用可能な最大メモリを指定します。メモリ使用量が閾値を超えると、ソートまたはtopN演算子はディスクにスピルしてメモリ使用量を減らします。この項目はデフォルトで0に設定されており、メモリ使用量が制限されず、ソートまたはtopNに対してディスクスピルが使用されないことを意味します。
max_bytes_before_external_sort = 0

## v7.0.0で新しく追加された項目です。この項目は、EquiJoinによるHashJoin演算子に対してディスクスピルがトリガーされる前に使用可能な最大メモリを指定します。メモリ使用量が閾値を超えると、HashJoinはディスクにスピルしてメモリ使用量を減らします。この項目はデフォルトで0に設定されており、メモリ使用量が制限されず、EquiJoinに対してディスクスピルが使用されないことを意味します。
max_bytes_before_external_join = 0

## v7.4.0で新しく追加された項目です。この項目は、TiFlashリソース制御機能を有効にするかどうかを制御します。trueに設定すると、TiFlashはパイプライン実行モデルを使用します。
enable_resource_control = true

## v4.0.5からセキュリティ設定が適用されます。
[security]
## v5.0で新しく追加された構成項目です。この構成項目は、ログのレダクションを有効または無効にします。trueに設定すると、ログ内のすべてのユーザーデータが?に置き換えられます。
## また、tiflash-learner.tomlのtiflash-learnerのロギングに対して security.redact-info-log も設定する必要があります
# redact_info_log = false

## 信頼されたSSL CAのリストが含まれるファイルのパスです。この設定がされている場合、cert_pathとkey_pathの設定も必要です。
# ca_path = "/path/to/ca.pem"
## PEM形式のX509証明書ファイルのパスです。
# cert_path = "/path/to/tiflash-server.pem"
## PEM形式のX509鍵ファイルのパスです。
# key_path = "/path/to/tiflash-server-key.pem"
```

### `tiflash-learner.toml`ファイルを設定する

```toml
[server]
    engine-addr = TiFlashコプロセッサーサービスの外部アクセスアドレス
[raftstore]
    ## Raftデータをストレージにフラッシュするスレッドプールで許可されるスレッド数です。
    apply-pool-size = 4

    ## Raftを処理するスレッドの数で、Raftstoreスレッドプールのサイズです。
    store-pool-size = 4

    ## スナップショットを処理するスレッドの数です。
    ## デフォルト数は2です。
    ## 0に設定すると、マルチスレッドの最適化が無効になります。
    snap-handle-pool-size = 2

    ## RaftストアがWALを永続化する最短インターバルです。
    ## 適切にレイテンシを増加させることでIOPS使用量を減らすことができます。
    ## デフォルト値は「4ms」です。
    ## 0msに設定すると、最適化が無効になります。
    store-batch-retry-recv-timeout = "4ms"
[security]
    ## v5.0で新しく追加された構成項目です。この構成項目は、ログのレダクションを有効または無効にします。
    ## trueに設定すると、ログ内のすべてのユーザーデータが?に置き換えられます。デフォルト値はfalseです。
    redact-info-log = false

[security.encryption]
    ## データファイルの暗号化方法です。
    ## 値のオプション：「aes128-ctr」、「aes192-ctr」、「aes256-ctr」、「sm4-ctr」（v6.4.0以降対応）、および「plaintext」。
    ## デフォルト値：「plaintext」はデフォルトでは暗号化が無効になっています。「plaintext」以外の値は暗号化が有効になり、その場合はマスターキーを指定する必要があります。
    data-encryption-method = "aes128-ctr"
    ## データ暗号化キーがローテーションされる頻度を指定します。デフォルト値：`7d`。
    data-key-rotation-period = "168h" # 7 days

[security.encryption.master-key]
    ## 暗号化が有効になっている場合はマスターキーを指定します。マスターキーの設定方法については、「暗号化の設定」を参照してください：https://docs.pingcap.com/tidb/dev/encryption-at-rest#configure-encryption 。

[security.encryption.previous-master-key]
    ## 新しいマスターキーをローテーションする際の古いマスターキーを指定します。構成フォーマットは「master-key」と同じです。「暗号化の設定」を参照して、古いマスターキーの設定方法について学んでください：https://docs.pingcap.com/tidb/dev/encryption-at-rest#configure-encryption 。
```
複数のTiFlashノードでI/Oメトリクスが類似しているディスクが複数ある場合は、`storage.main.dir`リストで対応するディレクトリを指定し、`storage.latest.dir`を空のままにすることを推奨します。TiFlashはI/O負荷とデータをすべてのディレクトリに均等に分散します。

TiFlashノードでI/Oメトリクスが異なる複数のディスクがある場合は、`storage.latest.dir`リストでメトリクスが高いディレクトリを指定し、`storage.main.dir`リストでメトリクスが低いディレクトリを指定することを推奨します。たとえば、NVMe-SSDが1つとSATA-SSDが2つある場合、`storage.latest.dir`を`["/nvme_ssd_a/data/tiflash"]`に設定し、`storage.main.dir`を`["/sata_ssd_b/data/tiflash", "/sata_ssd_c/data/tiflash"]`に設定できます。TiFlashはそれぞれのディレクトリリスト間でI/O負荷とデータを均等に分散します。この場合、`storage.latest.dir`の容量は総計画容量の10%として計画する必要があります。

> **警告：**
>
> `[storage]`構成は、TiUP v1.2.5以降でサポートされています。TiDBクラスタのバージョンがv4.0.9以降の場合、TiUPのバージョンがv1.2.5以降であることを確認してください。それ以外の場合、`[storage]`で定義されたデータディレクトリはTiUPによって管理されません。