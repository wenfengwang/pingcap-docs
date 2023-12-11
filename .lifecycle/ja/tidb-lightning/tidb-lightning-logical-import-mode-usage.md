---
title: 論理インポートモードの使用
summary: TiDB Lightning で論理インポートモードを使用する方法について学びます。

# 論理インポートモードの使用

このドキュメントでは、TiDB Lightning での[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md)の使用方法について、構成ファイルの書き込みやパフォーマンスの調整を含めて紹介します。

## 設定および論理インポートモードの使用

以下の構成ファイルを使用して、データのインポートに論理インポートモードを使用できます。

```toml
[lightning]
# ログ
level = "info"
file = "/tidb-lightning.log"
max-size = 128 # MB
max-days = 28
max-backups = 14

# 開始前にクラスタの最小要件を確認します。
check-requirements = true

[mydumper]
# ローカルのデータソースディレクトリまたは外部ストレージのURI。外部ストレージのURIについての詳細は、https://docs.pingcap.com/tidb/v6.6/backup-and-restore-storages#uri-formatを参照してください。
data-source-dir = "/data/my_database"

[tikv-importer]
# インポートモード。 "tidb" は論理インポートモードを使用することを意味します。
backend = "tidb"

[tidb]
# ターゲットクラスタの情報。クラスタ内の任意のtidb-serverのアドレス。
host = "172.16.31.1"
port = 4000
user = "root"
# TiDBに接続するためのパスワードを構成します。平文またはBase64でエンコードされた形式のいずれか。
password = ""
# tidb-lightningはTiDBライブラリをインポートし、いくつかのログを生成します。
# TiDBライブラリのログレベルを設定します。
log-level = "error"
```

完全な構成ファイルについては、[TiDB Lightning Configuration](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

## 衝突検出

衝突するデータとは、PKまたはUK列で同じデータを持つ2つ以上のレコードを指します。論理インポートモードでは、[`conflict.strategy`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)構成項目を設定することで、衝突するデータの処理戦略を構成できます。戦略に基づき、TiDB Lightningは異なるSQLステートメントでデータをインポートします。

| 戦略 | 衝突するデータのデフォルト動作 | 対応するSQLステートメント |
| :-- | :-- | :-- |
| `"replace"` | 既存のデータを新しいデータで置き換えます。 | `REPLACE INTO ...` |
| `"ignore"` | 既存のデータを保持し、新しいデータを無視します。 | `INSERT IGNORE INTO ...` |
| `"error"` | インポートを一時停止し、エラーを報告します。 | `INSERT INTO ...` |
| `""` | TiDB Lightningは衝突するデータを検出または処理しません。プライマリおよび一意キーの衝突するデータが存在する場合、その後続の手順でエラーが報告されます。 |  なし   |

戦略が`"error"`の場合、衝突するデータによって引き起こされるエラーは直接インポートタスクを中断します。戦略が`"replace"`または`"ignore"`の場合、[`conflict.threshold`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)を構成することで最大許容衝突を制御できます。デフォルト値は `9223372036854775807` で、ほぼすべてのエラーが許容されます。

戦略が`"ignore"`の場合、衝突するデータはダウンストリームの`conflict_records` テーブルに記録されます。詳細については[Error report](/tidb-lightning/tidb-lightning-error-resolution.md#error-report)を参照してください。この場合、[`conflict.max-record-rows`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)を構成することで、制限を超える衝突するデータはスキップされずに記録されません。デフォルト値は `100` です。

## パフォーマンスチューニング

- 論理インポートモードでは、TiDB Lightningのパフォーマンスは主にターゲットのTiDBクラスターの書き込みパフォーマンスに依存します。クラスターがパフォーマンスボトルネックに達した場合は、[Highly Concurrent Write Best Practices](/best-practices/high-concurrency-best-practices.md)を参照してください。

- ターゲットのTiDBクラスターが書き込みのボトルネックに達していない場合は、TiDB Lightning構成の `region-concurrency` の値を増やすことを検討してください。 `region-concurrency` のデフォルト値はCPUコア数です。 `region-concurrency` の意味は、物理インポートモードと論理インポートモードで異なります。論理インポートモードでは、 `region-concurrency` は書き込みの並列性です。

    設定の例:

    ```toml
    [lightning]
    region-concurrency = 32
    ```

- ターゲットのTiDBクラスターの `raftstore.apply-pool-size` および `raftstore.store-pool-size` 構成項目を調整すると、インポート速度が向上する場合があります。