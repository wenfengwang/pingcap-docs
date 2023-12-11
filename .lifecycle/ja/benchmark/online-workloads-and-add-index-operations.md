---
title: オンラインワークロードと`ADD INDEX`操作の相互作用テスト
summary: このドキュメントは、OLTPシナリオにおけるオンラインワークロードと`ADD INDEX`操作の相互作用効果をテストします。
aliases: ['/docs/dev/benchmark/online-workloads-and-add-index-operations/','/docs/dev/benchmark/add-index-with-load/']
---

# オンラインワークロードと`ADD INDEX`操作の相互作用テスト

## テスト目的

このドキュメントは、OLTPシナリオにおけるオンラインワークロードと`ADD INDEX`操作の相互作用効果をテストします。

## テストバージョン、日時、場所

TiDBバージョン: v3.0.1

日時: 2019年7月

場所: 北京

## テスト環境

このテストは、3つのTiDBインスタンス、3つのTiKVインスタンス、および3つのPDインスタンスで展開されたKubernetesクラスタで実行されます。

### バージョン情報

| コンポーネント |                  Gitハッシュ                   |
| :--- | :---------------------------------------- |
| TiDB  | `9e4e8da3c58c65123db5f26409759fe1847529f8` |
| TiKV  | `4151dc8878985df191b47851d67ca21365396133` |
|  PD   | `811ce0b9a1335d1b2a049fd97ef9e186f1c9efc1` |

Sysbenchバージョン: 1.0.17

### TiDBパラメータ構成

TiDB、TiKV、およびPDはすべてデフォルトの[TiDB Operator](https://github.com/pingcap/tidb-operator)構成を使用します。

### クラスタトポロジー

|                 マシンIP                  |   デプロイインスタンス   |
| :-------------------------------------- | :----------|
|                172.31.8.8                 |  Sysbench |
| 172.31.7.69, 172.31.5.152, 172.31.11.133 |      PD      |
| 172.31.4.172, 172.31.1.155, 172.31.9.210 |     TiKV     |
| 172.31.7.80, 172.31.5.163, 172.31.11.123 |     TiDB     |

### Sysbenchを使用したオンラインワークロードのシミュレーション

Sysbenchを使用して、**データが200万行のテーブル**をKubernetesクラスタにインポートします。

次のコマンドを実行してデータをインポートします:

{{< copyable "shell-regular" >}}

```sh
sysbench oltp_common \
    --threads=16 \
    --rand-type=uniform \
    --db-driver=mysql \
    --mysql-db=sbtest \
    --mysql-host=$tidb_host \
    --mysql-port=$tidb_port \
    --mysql-user=root \
    prepare --tables=1 --table-size=2000000
```

次のコマンドを実行してテストを実行します:

{{< copyable "shell-regular" >}}

```sh
sysbench $testname \
    --threads=$threads \
    --time=300000 \
    --report-interval=15 \
    --rand-type=uniform \
    --rand-seed=$RANDOM \
    --db-driver=mysql \
    --mysql-db=sbtest \
    --mysql-host=$tidb_host \
    --mysql-port=$tidb_port \
    --mysql-user=root \
    run --tables=1 --table-size=2000000
```

## テストプラン1: `ADD INDEX`ステートメントの対象列に対して頻繁に書き込み操作を実行する

1. `oltp_read_write`テストを開始します。
2. ステップ1と同時に、`alter table sbtest1 add index c_idx(c)`を使用してインデックスを追加します。
3. ステップ2の終了時に、インデックスが正常に追加されたら`oltp_read_write`テストを停止します。
4. `alter table ... add index`の所要時間とこの期間中のSysbenchの平均TPSおよびQPSを取得します。
5. 2つのパラメータ`tidb_ddl_reorg_worker_cnt`と`tidb_ddl_reorg_batch_size`の値を徐々に増やし、その後ステップ1〜4を繰り返します。

### テスト結果

#### `ADD INDEX`操作なしの`oltp_read_write`のテスト結果

| sysbench TPS | sysbench QPS    |
| :------- | :-------- |
| 350.31   | 6806 |

#### `tidb_ddl_reorg_batch_size = 32`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 402 |  338.4 |           6776 |
| 2     | 266 |  330.3 |           6001 |
| 4     | 174 |  288.5 |           5769 |
| 8     | 129 |  280.6 |           5612 |
| 16    | 90 |   263.5 |           5273 |
| 32    | 54 |   229.2 |           4583 |
| 48    | 57 |   230.1 |           4601 |

![add-index-load-1-b32](/media/add-index-load-1-b32.png)

#### `tidb_ddl_reorg_batch_size = 64`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 264 |  269.4 |           5388 |
| 2     | 163 |  266.2 |           5324 |
| 4     | 105 |  272.5 |           5430 |
| 8     | 78 |  262.5 |           5228 |
| 16    | 57 |   215.5 |           4308 |
| 32    | 42 |   185.2 |           3715 |
| 48    | 45 |   189.2 |           3794 |

![add-index-load-1-b64](/media/add-index-load-1-b64.png)

#### `tidb_ddl_reorg_batch_size = 128`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 171 |  289.1 |           5779 |
| 2     | 110 |  274.2 |           5485 |
| 4     | 79 |  250.6 |           5011 |
| 8     | 51 |  246.1 |           4922 |
| 16    | 39 |   171.1 |           3431 |
| 32    | 35 |   130.8 |           2629 |
| 48    | 35 |   120.5 |           2425 |

![add-index-load-1-b128](/media/add-index-load-1-b128.png)

#### `tidb_ddl_reorg_batch_size = 256`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 145 |  283.0 |           5659 |
| 2     | 96 |  282.2 |           5593 |
| 4     | 56 |  236.5 |           4735 |
| 8     | 45 |  194.2 |           3882 |
| 16    | 39 |   149.3 |           2893 |
| 32    | 36 |   113.5 |           2268 |
| 48    | 33 |   86.2 |           1715 |

![add-index-load-1-b256](/media/add-index-load-1-b256.png)

#### `tidb_ddl_reorg_batch_size = 512`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 135 |  257.8 |           5147 |
| 2     | 78 |  252.8 |           5053 |
| 4     | 49 |  222.7 |           4478 |
| 8     | 36 |  145.4 |           2904 |
| 16    | 33 |   109 |           2190 |
| 32    | 33 |   72.5 |           1503 |
| 48    | 33 |   54.2 |           1318 |

![add-index-load-1-b512](/media/add-index-load-1-b512.png)

#### `tidb_ddl_reorg_batch_size = 1024`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 376 |  548.9 |           8780 |
| 2     | 212 |  541.5 |           8523 |
| 4     | 135 |  538.6 |           8549 |
| 8     | 114 |  536.7 |           8393 |
| 16    | 77 |   533.9 |           8292 |
| 32    | 46 |   533.4 |           8103 |
| 48    | 46 |   532.2 |           8074 |

![add-index-load-2-b32](/media/add-index-load-2-b32.png)

#### `tidb_ddl_reorg_batch_size = 1024`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 91 |  536.8 |           8316 |
| 2     | 52 |  533.9 |           8165 |
| 4     | 40 |  522.4 |           7947 |
| 8     | 36 |  510 |           7860 |
| 16    | 33 |   485.5 |           7704 |
| 32    | 31 |   467.5 |           7516 |
| 48    | 30 |   562.1 |           7442 |

![add-index-load-2-b1024](/media/add-index-load-2-b1024.png)

#### `tidb_ddl_reorg_batch_size = 4096`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 103 |  502.2 |           7823 |
| 2     | 63 |  486.5 |           7672 |
| 4     | 52 |  467.4 |            7516 |
| 8     | 39 |  452.5 |           7302 |
| 16    | 35 |   447.2 |           7206 |
| 32    | 30 |   441.9 |           7057 |
| 48    | 30 |   440.1 |           7004 |

![add-index-load-2-b4096](/media/add-index-load-2-b4096.png)

### テスト結論

`ADD INDEX`ステートメントの対象列にのみクエリ操作を実行する場合、`ADD INDEX`操作がオンラインワークロードに与える影響は明らかではありません。
| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 372 |  350.4 |           6892 |
| 2     | 207 |  344.2 |           6700 |
| 4     | 140 |  343.1 |           6672 |
| 8     | 121 |  339.1 |           6579 |
| 16    | 76 |   340   |           6607 |
| 32    | 42 |   343.1 |           6695 |
| 48    | 42 |   333.4 |           6454 |

![add-index-load-3-b32](/media/add-index-load-3-b32.png)

#### `tidb_ddl_reorg_batch_size = 1024`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 94 |  352.4 |           6794 |
| 2     | 50 |  332 |           6493 |
| 4     | 45 |  330 |           6456 |
| 8     | 36 |  325.5 |           6324 |
| 16    | 32 |   312.5   |           6294 |
| 32    | 32 |   300.6 |           6017 |
| 48    | 31 |   279.5 |           5612 |

![add-index-load-3-b1024](/media/add-index-load-3-b1024.png)

#### `tidb_ddl_reorg_batch_size = 4096`

| tidb_ddl_reorg_worker_cnt |  add_index_durations(s) | sysbench TPS   | sysbench QPS |
| :------------------------ | :---------------------- | :------------- | :----------- |
| 1     | 116 |  325.5 |           6324 |
| 2     | 65 |  312.5 |           6290 |
| 4     | 50 |  300.6 |           6017 |
| 8     | 37 |  279.5 |           5612 |
| 16    | 34 |   250.4   |           5365 |
| 32    | 32 |   220.2 |           4924 |
| 48    | 33 |   214.8 |           4544 |

![add-index-load-3-b4096](/media/add-index-load-3-b4096.png)

### テスト結論

`ADD INDEX` ステートメントのターゲット列がオンラインのワークロードに関係ない場合、`ADD INDEX` 操作のワークロードへの影響は明らかではありません。

## 要約

- `ADD INDEX` ステートメントのターゲット列に対して頻繁な書き込み操作（`INSERT`、`DELETE`、`UPDATE` 操作を含む）を実行する場合、デフォルトの `ADD INDEX` 構成では比較的頻繁に書き込み競合が発生し、オンラインのワークロードに大きな影響を与えます。同時に、連続したリトライ試行により `ADD INDEX` 操作の完了には長い時間がかかります。このテストでは、`tidb_ddl_reorg_worker_cnt` と `tidb_ddl_reorg_batch_size` の積をデフォルト値の1/32に設定できます。たとえば、パフォーマンスを向上させるために、`tidb_ddl_reorg_worker_cnt` を `4` に、`tidb_ddl_reorg_batch_size` を `256` に設定できます。
- `ADD INDEX` ステートメントのターゲット列に対してクエリ操作のみを実行するか、ターゲット列がオンラインのワークロードと直接関係ない場合、デフォルトの `ADD INDEX` 構成を使用できます。