---
title: 物理インポートモードの使用
summary: TiDB Lightningで物理インポートモードを使用する方法について学ぶ
---

# 物理インポートモードの使用

このドキュメントでは、TiDB Lightningでの[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)の使用方法について、構成ファイルの作成、パフォーマンスのチューニング、ディスククォータの設定を含めて紹介します。

物理インポートモードにはいくつかの制限があります。物理インポートモードを使用する前に、[制限事項](/tidb-lightning/tidb-lightning-physical-import-mode.md#limitations)を必ずお読みください。

## 物理インポートモードの構成と使用

以下の構成ファイルを使用して、物理インポートモードを使ったデータのインポートを実行できます。

```toml
[lightning]
# log
level = "info"
file = "tidb-lightning.log"
max-size = 128 # MB
max-days = 28
max-backups = 14

# 開始前にクラスタの最低要件をチェックします。
check-requirements = true

[mydumper]
# ローカルのデータソースディレクトリまたは外部ストレージのURIです。外部ストレージのURIについての詳細は、以下を参照してください。https://docs.pingcap.com/tidb/v6.6/backup-and-restore-storages#uri-format
data-source-dir = "/data/my_database"

[conflict]
# v7.3.0から、競合データを処理する新しいバージョンのストラテジが導入されました。デフォルト値は""です。
# - "": TiDB Lightningは競合データを検出または処理しません。元のファイルに競合するプライマリまたは一意のキーレコードが含まれている場合、後続ステップでエラーが報告されます。
# - "error": インポートされたデータに競合するプライマリまたは一意のキーレコードが検出されると、TiDB Lightningはインポートを終了してエラーを報告します。
# - "replace": 競合するプライマリまたは一意のキーレコードがある場合、TiDB Lightningは新しいデータを保持し、古いデータを上書きします。
# - "ignore": 競合するプライマリまたは一意のキーレコードがある場合、TiDB Lightningは古いデータを保持し、新しいデータを無視します。
# 新しいストラテジはtikv-importer.duplicate-resolution（競合検出の古いバージョン）と同時に使用できません。
strategy = ""
# threshold = 9223372036854775807
# max-record-rows = 100

[tikv-importer]
# Import mode。"local"は物理インポートモードを使用することを意味します。
backend = "local"

# 競合データの解決方法
duplicate-resolution = 'remove'

# ローカルKVソートのディレクトリ
sorted-kv-dir = "./some-dir"

# TiDB Lightningが各TiKVノードにデータを書き込む帯域幅を制限します（物理インポートモードでは0に設定されており、制限はありません）。
# store-write-bwlimit = "128MiB"

# 物理インポートモードでインデックスをSQL経由で追加するかどうかを指定します。デフォルト値は`false`で、これはTiDB Lightningが行データとインデックスデータを両方KVペアにエンコードし、それらを一緒にTiKVにインポートすることを意味します。
# SQL経由でインデックスを追加することの利点は、データとインデックスを別々にインポートし、データをより迅速にインポートできることです。データがインポートされた後、インデックスの追加に失敗しても、インポートされたデータの整合性に影響しません。
# add-index-by-sql = false

[tidb]
# ターゲットクラスタの情報。クラスタ内の任意のtidb-serverのアドレス。
host = "172.16.31.1"
port = 4000
user = "root"
# TiDBに接続するためのパスワードを構成します。平文またはBase64でエンコードされました。
password = ""
# 必須。テーブルスキーマ情報はこのstatus-portを介してTiDBから取得されます。
status-port = 10080
# 必須。クラスタ内の任意のpd-serverのアドレス。
pd-addr = "172.16.31.4:2379"
# tidb-lightningはTiDBライブラリをインポートし、いくつかのログを生成します。
# TiDBライブラリのログレベルを設定します。
log-level = "error"

[post-restore]
# インポート後に各テーブルで`ADMIN CHECKSUM TABLE <table>`を実行するかどうかを指定します。
# 使用可能なオプションは以下の通りです。
# - "required"（デフォルト）：インポート後に管理者チェックサムを実行します。チェックサムに失敗すると、TiDB Lightningは失敗で終了します。
# - "optional"：管理者チェックサムを実行します。チェックサムに失敗しても、TiDB LightningはWARNログを報告し、エラーを無視します。
# - "off"：インポート後にチェックサムを実行しません。
# v4.0.8以降、デフォルト値は"true"から"required"に変更されています。
#
# 注意：
# 1. チェックサムが失敗すると、通常、インポート例外（データの損失またはデータの不整合）を意味するため、常にチェックサムを有効にすることをお勧めします。
# 2. 互換性により、このフィールドには「true」と「false」というbool値も許可されています。
# "true"は"required"に等しく、"false"は"off"に等しいです。
checksum = "required"

# チェックサムが完了した後に各テーブルで`ANALYZE TABLE <table>`を実行するかどうかを指定します。
# このフィールドの使用可能なオプションも`checksum`と同じです。ただし、このフィールドのデフォルト値は"optional"です。
analyze = "optional"
```

完全な構成ファイルについては、[構成ファイルとコマンドラインパラメータ](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

## 競合検出

競合データとは、同じプライマリキーまたは一意キー列データを持つ2つ以上のレコードを指します。データソースが競合データを含み、競合検出機能がオフの場合、テーブルの実際の行数が一意インデックスを使用したクエリで返される総行数と異なる場合があります。

競合検出には、2つのバージョンがあります。

- `conflict`構成項目で制御される新しいバージョンの競合検出。
- `tikv-importer.duplicate-resolution`構成項目で制御される古いバージョンの競合検出。

### 新しいバージョンの競合検出

構成値の意味は次のとおりです。

| Strategy | 競合データのデフォルト動作 | 対応するSQLステートメント |
| :-- | :-- | :-- |
| `"replace"` | 新しいデータで既存のデータを置き換える。 | `REPLACE INTO ...` |
| `"ignore"` | 既存のデータを維持し、新しいデータを無視する。 | `INSERT IGNORE INTO ...` |
| `"error"` | インポートを一時停止し、エラーを報告する。 | `INSERT INTO ...` |
| `""` | TiDB Lightningは競合データを検出または処理しません。元のファイルに競合するプライマリまたは一意のキーレコードが含まれている場合、後続ステップでエラーが報告されます。 |  なし   |

> **注意:**
>
> 物理インポートモードにおける競合検出の結果は、内部実装とTiDB Lightningの制限により、SQLベースのインポートと異なる場合があります。

ストラテジが`"replace"`または`"ignore"`の場合、競合データは[競合エラー](/tidb-lightning/tidb-lightning-error-resolution.md#conflict-errors)として扱われます。 [`conflict.threshold`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)の値が`0`より大きい場合、TiDB Lightningは指定された数の競合エラーを許容します。デフォルト値は`9223372036854775807`で、ほとんどのエラーを許容することを意味します。詳細については、[エラー解決](/tidb-lightning/tidb-lightning-error-resolution.md)を参照してください。

新しいバージョンの競合検出には以下の制限があります。

- インポート前に、TiDB Lightningはすべてのデータを読み取り、エンコードして潜在的な競合データを事前に確認します。検出プロセス中、TiDB Lightningは `tikv-importer.sorted-kv-dir` を使用して一時ファイルを保存します。検出が完了した後、TiDB Lightningはその結果をインポートフェーズのために保持します。これにより、時間とディスクスペースの追加使用、およびデータの読み取りに関するAPI要求が追加されます。
- 新しいバージョンの競合検出は単一ノードでのみ機能し、並列インポートや `disk-quota` パラメータが有効なシナリオには適用されません。
- 新しいバージョン（`conflict`）と古いバージョン（`tikv-importer.duplicate-resolution`）の競合検出は同時に使用できません。構成 [`conflict.strategy`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) が設定されている場合、新しいバージョンの競合検出が有効になります。

古いバージョンの競合検出と比較して、新しいバージョンは大量の競合データを含むインポートデータの場合に時間を節約します。競合データを含むデータがあり、十分なローカルディスクスペースがある場合には、非並列インポートタスクでの新しいバージョンの競合検出の使用をお勧めします。

### 古いバージョンの競合検出

`tikv-importer.duplicate-resolution`が空の文字列でない場合、古いバージョンの競合検出が有効になります。v7.2.0およびそれ以前のバージョンでは、TiDB Lightningはこの競合検出方法のみをサポートしています。

古いバージョンの競合検出では、TiDB Lightningは2つのストラテジを提供します。

- `remove`（推奨）：対象テーブルから競合するレコードを削除して、対象のTiDBで一貫した状態を確保します。
- `none`：重複レコードを検出しません。`none`は2つのストラテジの中で最もパフォーマンスが良く、TiDBでデータの不整合を引き起こす可能性があります。

v5.3以前では、TiDB Lightningは競合検出をサポートしていませんでした。競合するデータがある場合、インポーターはチェックサムステップで失敗します。競合検出が有効な場合、競合するデータがあると、TiDB Lightningはチェックサムステップをスキップします（常に失敗します）。
「order_line」テーブルの次のスキーマがあるとします：

```sql
CREATE TABLE IF NOT EXISTS `order_line` (
  `ol_o_id` int(11) NOT NULL,
  `ol_d_id` int(11) NOT NULL,
  `ol_w_id` int(11) NOT NULL,
  `ol_number` int(11) NOT NULL,
  `ol_i_id` int(11) NOT NULL,
  `ol_supply_w_id` int(11) DEFAULT NULL,
  `ol_delivery_d` datetime DEFAULT NULL,
  `ol_quantity` int(11) DEFAULT NULL,
  `ol_amount` decimal(6,2) DEFAULT NULL,
  `ol_dist_info` char(24) DEFAULT NULL,
  PRIMARY KEY (`ol_w_id`,`ol_d_id`,`ol_o_id`,`ol_number`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

インポート中に競合するデータが検出された場合、次のように`lightning_task_info.conflict_error_v1`テーブルをクエリできます：

```sql
mysql> select table_name,index_name,key_data,row_data from conflict_error_v1 limit 10;
+---------------------+------------+----------+-----------------------------------------------------------------------------+
|  table_name         | index_name | key_data | row_data                                                                    |
+---------------------+------------+----------+-----------------------------------------------------------------------------+
| `tpcc`.`order_line` | PRIMARY    | 21829216 | (2677, 10, 10, 11, 75656, 10, NULL, 5, 5831.97, "HT5DN3EVb6kWTd4L37bsbogj") |
| `tpcc`.`order_line` | PRIMARY    | 49931672 | (2677, 10, 10, 11, 75656, 10, NULL, 5, 5831.97, "HT5DN3EVb6kWTd4L37bsbogj") |
| `tpcc`.`order_line` | PRIMARY    | 21829217 | (2677, 10, 10, 12, 76007, 10, NULL, 5, 9644.36, "bHuVoRfidQ0q2rJ6ZC9Hd12E") |
| `tpcc`.`order_line` | PRIMARY    | 49931673 | (2677, 10, 10, 12, 76007, 10, NULL, 5, 9644.36, "bHuVoRfidQ0q2rJ6ZC9Hd12E") |
| `tpcc`.`order_line` | PRIMARY    | 21829218 | (2677, 10, 10, 13, 85618, 10, NULL, 5, 7427.98, "t3rsesgi9rVAKi9tf6an5Rpv") |
| `tpcc`.`order_line` | PRIMARY    | 49931674 | (2677, 10, 10, 13, 85618, 10, NULL, 5, 7427.98, "t3rsesgi9rVAKi9tf6an5Rpv") |
| `tpcc`.`order_line` | PRIMARY    | 21829219 | (2677, 10, 10, 14, 15873, 10, NULL, 5, 133.21, "z1vH0e31tQydJGhfNYNa4ScD")  |
| `tpcc`.`order_line` | PRIMARY    | 49931675 | (2677, 10, 10, 14, 15873, 10, NULL, 5, 133.21, "z1vH0e31tQydJGhfNYNa4ScD")  |
| `tpcc`.`order_line` | PRIMARY    | 21829220 | (2678, 10, 10, 1, 44644, 10, NULL, 5, 8463.76, "TWKJBt5iJA4eF7FIVxnugNmz")  |
| `tpcc`.`order_line` | PRIMARY    | 49931676 | (2678, 10, 10, 1, 44644, 10, NULL, 5, 8463.76, "TWKJBt5iJA4eF7FIVxnugNmz")  |
+---------------------+------------+----------------------------------------------------------------------------------------+
10 rows in set (0.14 sec)
```

手動で保持する必要があるレコードを識別し、これらのレコードをテーブルに挿入できます。

## インポート中のスケジューリングの一時停止範囲

v6.2.0から、TiDB Lightningはオンラインアプリケーションへのデータインポートの影響を制限するメカニズムを実装しています。新しいメカニズムでは、TiDB Lightningは全体的なスケジューリングを一時停止せず、対象テーブルデータを保持するリージョンのスケジューリングのみを一時停止します。これにより、インポートがオンラインアプリケーションに与える影響が大幅に軽減されます。

v7.1.0からは、TiDB Lightningパラメータ[`pause-pd-scheduler-scope`](/tidb-lightning/tidb-lightning-configuration.md)を使用して、スケジューリングの一時停止範囲を制御できます。既定値は`「table」`で、対象テーブルデータを保持するリージョンのみでスケジューリングが一時停止されます。クラスタにビジネストラフィックがない場合、インポート中に他のスケジューリングからの干渉を避けるために、このパラメータを`「global」`に設定することをお勧めします。

<Note>

TiDB Lightningは、既にデータを含むテーブルにデータをインポートすることはサポートしていません。

TiDBクラスタはv6.1.0以降のバージョンである必要があります。以前のバージョンの場合、TiDB Lightningは古い動作を維持し、全体的なスケジューリングを一時停止し、インポート中にオンラインアプリケーションに重大な影響を与えます。

</Note>

TiDB Lightningは既定でクラスタのスケジューリングを可能な限り最小の範囲で一時停止します。ただし、既定の構成では、クラスタのパフォーマンスは依然として高速なインポートの影響を受ける場合があります。このような場合は、次のオプションを構成してインポート速度やクラスタのパフォーマンスに影響を与えるその他の要因を制御できます。

```toml
[tikv-importer]
# 物理インポートモードでTiDB Lightningが各TiKVノードにデータを書き込む際の帯域幅を制限します。
store-write-bwlimit = "128MiB"

[tidb]
# ChecksumおよびAnalyzeがトランザクションの遅延に与える影響を減らすために、小さな並行性を使用します。
distsql-scan-concurrency = 3
```

## パフォーマンスチューニング

**物理インポートモードのインポートパフォーマンスを向上させる最も直接かつ効果的な方法は次のとおりです:**

- **特にLightningが展開されているノードのハードウェアをアップグレードします。特にCPUと`sorted-key-dir`のストレージデバイスをアップグレードします。**
- **[並列インポート](/tidb-lightning/tidb-lightning-distributed-import.md)機能を使用して水平スケーリングを実現します。**

TiDB Lightningは、物理インポートモードにおけるインポートパフォーマンスに影響を与えるいくつかの並行性関連の構成を提供しています。ただし、長期の経験から、次の4つの構成項目を既定値のままにすることが推奨されています。これらの4つの構成項目を調整しても、大幅なパフォーマンス向上はもたらしません。

```
[lightning]
# エンジンファイルの最大並行性。各テーブルは、インデックスを格納する1つの「インデックスエンジン」と、複数の「データエンジン」に分割されます。これらの設定は、それぞれのエンジンタイプの最大同時実行数を制御します。
# 2つの設定は、2つのエンジンファイルの最大並行性を制御します。
index-concurrency = 2
table-concurrency = 6

# データの並行性。既定値は論理CPUの数です。
region-concurrency =

# I/Oの最大並行性。並行性が高すぎると、ディスクキャッシュが頻繁にリフレッシュされ、キャッシュミスおよび読み取り速度の低下を引き起こす可能性があります。さまざまなストレージメディアについては、このパラメータを調整して最適なパフォーマンスを実現する必要があります。
io-concurrency = 5
```

インポート中、各テーブルは1つの「インデックスエンジン」でインデックスを格納し、複数の「データエンジン」で行データを格納します。

`index-concurrency`は、インデックスエンジンの最大並行性を制御します。`index-concurrency`を調整する場合は、`index-concurrency * 各テーブルのソースファイルの数 > region-concurrency`であり、CPUが十分に利用されることを確認してください。この比率は通常1.5〜2の間です。`index-concurrency`をあまり高く設定しないでください。高すぎる`index-concurrency`では多くのパイプラインが構築され、インデックスエンジンのインポート段階が詰まることがあります。

同様のことが`table-concurrency`にも当てはまります。`table-concurrency * 各テーブルのソースファイルの数 > region-concurrency`であり、CPUが十分に利用されることを確認してください。`region-concurrency * 4 / 各テーブルのソースファイルの数`の推奨値はあり、4より低く設定しないでください。

テーブルが大きい場合、Lightningはテーブルを100 GiBの複数のバッチに分割します。並行性は`table-concurrency`で制御されます。

`index-concurrency`および`table-concurrency`は、インポート速度にほとんど影響しません。既定値のままにしておくことができます。

`io-concurrency`はファイルの読み取りの並行性を制御します。既定値は5です。任意のタイミングで、読み取り操作を実行するハンドルは5つだけです。ファイルの読み取り速度が通常ボトルネックにならないため、この構成を既定値のままにしておくことができます。
```markdown
After the file data is read, Lightning needs to do some post-processing, such as encoding and sorting the data locally. The concurrency of these operations is controlled by `region-concurrency`. The default value is the number of CPU cores. You can leave this configuration in the default value. It is recommended to deploy Lightning on a separate server from other components. If you must deploy Lightning together with other components, you need to lower the value of `region-concurrency` according to the load.

The [`num-threads`](/tikv-configuration-file.md#num-threads) configuration of TiKV can also affect the performance. For new clusters, it is recommended to set `num-threads` to the number of CPU cores.

## Configure disk quota <span class="version-mark">New in v6.2.0</span>

When you import data in the physical import mode, TiDB Lightning creates a large number of temporary files on the local disk to encode, sort, and split the original data. When the local disk space is insufficient, TiDB Lightning reports an error and exits because of write failure.

To avoid this situation, you can configure disk quota for TiDB Lightning. When the size of the temporary files exceeds the disk quota, TiDB Lightning pauses the process of reading the source data and writing temporary files. TiDB Lightning prioritizes writing the sorted key-value pairs to TiKV. After deleting the local temporary files, TiDB Lightning continues the import process.

To enable disk quota, add the following configuration to your configuration file:

```toml
[tikv-importer]
# MaxInt64 by default, which is 9223372036854775807 bytes.
disk-quota = "10GB"
backend = "local"

[cron]
# The interval of checking disk quota. 60 seconds by default.
check-disk-quota = "30s"
```

`disk-quota` limits the storage space used by TiDB Lightning. The default value is MaxInt64, which is 9223372036854775807 bytes. This value is much larger than the disk space you might need for the import, so leaving it as the default value is equivalent to not setting the disk quota.

`check-disk-quota` is the interval of checking disk quota. The default value is 60 seconds. When TiDB Lightning checks the disk quota, it acquires an exclusive lock for the relevant data, which blocks all the import threads. Therefore, if TiDB Lightning checks the disk quota before every write, it significantly slows down the write efficiency (as slow as a single-thread write). To achieve efficient write, disk quota is not checked before every write; instead, TiDB Lightning pauses all the import threads and checks the disk quota every `check-disk-quota` interval. That is, if the value of `check-disk-quota` is set to a large value, the disk space used by TiDB Lightning might exceed the disk quota you set, which leaves the disk quota ineffective. Therefore, it is recommended to set the value of `check-disk-quota` to a small value. The specific value of this item is determined by the environment in which TiDB Lightning is running. In different environments, TiDB Lightning writes temporary files at different speeds. Theoretically, the faster the speed, the smaller the value of `check-disk-quota` should be.
```