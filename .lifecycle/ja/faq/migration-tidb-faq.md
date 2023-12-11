---
title: マイグレーションFAQ
summary: データマイグレーションに関連するFAQについて学びます。
---

# マイグレーションFAQ

このドキュメントは、TiDBデータマイグレーションに関連するよくある質問（FAQ）を要約しています。

マイグレーション関連ツールに関するよくある質問については、以下のリンクをクリックしてください。

- [バックアップ＆リストアFAQ](/faq/backup-and-restore-faq.md)
- [TiDBバイナリログFAQ](/tidb-binlog/tidb-binlog-faq.md)
- [TiDB LightningのFAQ](/tidb-lightning/tidb-lightning-faq.md)
- [TiDBデータマイグレーション（DM）FAQ](/dm/dm-faq.md)
- [TiCDCのFAQ](/ticdc/ticdc-faq.md)

## フルデータのエクスポートとインポート

### MySQLで実行中のアプリケーションをTiDBに移行する方法は？

TiDBはほとんどのMySQL構文をサポートしているため、一般的にはほとんどの場合、コードを一行も変更せずにアプリケーションをTiDBに移行できます。

### データのインポートとエクスポートが遅く、各コンポーネントのログに再試行とEOFエラーが多く表示される

論理エラーが発生していない場合、再試行とEOFエラーはネットワークの問題による可能性があります。まずネットワーク接続をチェックするためにツールを使用することを推奨します。次の例では、[iperf](https://iperf.fr/)がトラブルシューティングに使用されます。

+ 再試行とEOFエラーが発生するサーバサイドノードで次のコマンドを実行します：

    {{< copyable "shell-regular" >}}

    ```shell
    iperf3 -s
    ```

+ 再試行とEOFエラーが発生するクライアントサイドノードで次のコマンドを実行します：

    {{< copyable "shell-regular" >}}

    ```shell
    iperf3 -c <server-IP>
    ```

以下の例は、ネットワーク接続が良好なクライアントノードの出力です：

```shell
$ iperf3 -c 192.168.196.58
ホスト 192.168.196.58 に接続中、ポート 5201
[  5] ローカル 192.168.196.150 ポート 55397 が 192.168.196.58 ポート 5201 に接続しました
[ ID] インターバル           転送量     ビットレート
[  5]   0.00-1.00   秒  18.0 MBytes   150 Mbits/sec
[  5]   1.00-2.00   秒  20.8 MBytes   175 Mbits/sec
[  5]   2.00-3.00   秒  18.2 MBytes   153 Mbits/sec
[  5]   3.00-4.00   秒  22.5 MBytes   188 Mbits/sec
[  5]   4.00-5.00   秒  22.4 MBytes   188 Mbits/sec
[  5]   5.00-6.00   秒  22.8 MBytes   191 Mbits/sec
[  5]   6.00-7.00   秒  20.8 MBytes   174 Mbits/sec
[  5]   7.00-8.00   秒  20.1 MBytes   168 Mbits/sec
[  5]   8.00-9.00   秒  20.8 MBytes   175 Mbits/sec
[  5]   9.00-10.00  秒  21.8 MBytes   183 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] インターバル           転送量     ビットレート
[  5]   0.00-10.00  秒   208 MBytes   175 Mbits/sec                  送信元
[  5]   0.00-10.00  秒   208 MBytes   174 Mbits/sec                  受信者

iperf 完了。
```

出力に低いネットワーク帯域幅と高い帯域幅の変動が表示される場合、各コンポーネントのログに多数の再試行とEOFエラーが表示される可能性があります。この場合は、ネットワークサービスプロバイダに相談してネットワーク品質を改善する必要があります。

各メトリックの出力が良好な場合は、各コンポーネントを更新してみてください。更新後も問題が解消されない場合は、PingCAPまたはコミュニティから[サポート](/support.md)を受けてください。

### MySQLのユーザーテーブルを誤ってTiDBにインポートしたり、パスワードを忘れてログインできない場合、どのように対処すればよいですか？

TiDBサービスを再起動し、構成ファイルに`-skip-grant-table=true`パラメータを追加します。パスワードなしでクラスタにログインし、ユーザを再作成するか、`mysql.user`テーブルを再作成します。特定のテーブルスキーマについては、公式ドキュメンテーションを参照してください。

### TiDBでデータをエクスポートする方法は？

TiDBでデータをエクスポートするには、以下の方法を使用できます：

- `mysqldump`と`WHERE`句を使用してデータをエクスポートする。
- MySQLクライアントを使用して`select`の結果をファイルにエクスポートする。

### DB2またはOracleからTiDBに移行する方法は？

DB2またはOracleからデータをすべて移行したり、増分的に移行する場合、以下のソリューションを参照してください：

- OGG、Gateway、CDC（Change Data Capture）などのOracleの公式移行ツールを使用する。
- データのインポートとエクスポートのためのプログラムを開発する。
- スプールをテキストファイルとしてエクスポートし、Load infileを使用してデータをインポートする。
- サードパーティのデータマイグレーションツールを使用する。

現時点では、OGGの使用が推奨されています。

### Sqoopを使用してTiDBに`バッチ`でデータを書き込む際に、`java.sql.BatchUpdateExecption:statement count 5001 exceeds the transaction limitation`エラーが表示される場合

Sqoopでは、`--batch`は各`バッチ`で100の`ステートメント`をコミットすることを意味しますが、デフォルトでは各`ステートメント`には100のSQLステートメントが含まれています。したがって、100 * 100 = 10000のSQLステートメントがあり、これは単一のTiDBトランザクションで許可されるステートメントの最大数である5000を超えています。

2つの解決策があります：

- 次のように`-Dsqoop.export.records.per.statement=10`オプションを追加します：

    {{< copyable "shell-regular" >}}

    ```bash
    sqoop export \
        -Dsqoop.export.records.per.statement=10 \
        --connect jdbc:mysql://mysql.example.com/sqoop \
        --username sqoop ${user} \
        --password ${passwd} \
        --table ${tab_name} \
        --export-dir ${dir} \
        --batch
    ```

- 単一のTiDBトランザクションでのステートメントの制限数を増やすこともできますが、これによりより多くのメモリを消費することになります。

### Dumplingが`ローカルディスクの空き容量が不足しています`エラーを返すか、エクスポート中に上流データベースのメモリが不足することがありますが、これはなぜですか？

この問題には以下の原因があります：

+ データベースの主キーが均等に分散されていない場合（例えば、[`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md)を有効にしている場合）。
+ 上流データベースがTiDBであり、エクスポート対象のテーブルがパーティション化されたテーブルである場合。

上記のケースでは、Dumplingは非常に大きなデータチャンクをエクスポートし、非常に大きな結果を持つクエリを送信します。この問題を解決するためには、最新バージョンのDumplingを入手できます。

### TiDBにおいてOracleのFlashbackクエリのような機能はありますか？DDLもサポートされていますか？

はい、あります。また、DDLもサポートされています。詳細は、[TiDBが履歴バージョンからデータを読む方法](/read-historical-data.md)を参照してください。

## データをオンラインでマイグレーションする

### TiDBからHBaseやElasticsearchなどの他のデータベースにデータをレプリケートする現在の解決策はありますか？

いいえ。現在、データレプリケーションはアプリケーションそのものに依存しています。

## トラフィックをマイグレーションする

### トラフィックを迅速にマイグレーションする方法は？

[最適なネットワーク構成](/dm/dm-overview.md)を行い、ネットワーク構成を編集して必要に応じて読み書きトラフィックをバッチごとにマイグレーションするために、MySQLからTiDBにアプリケーションデータを[「TiDBデータマイグレーション」](/dm/dm-overview.md)ツールを使用することを推奨します。上位レイヤーに安定したネットワークLB（HAProxy、LVS、F5、DNSなど）を展開し、直接ネットワーク構成を編集することで透過的なマイグレーションを実現してください。

### TiDBの総書き込みおよび読み取り容量には制限がありますか？

総読み取り容量に制限はありません。TiDBサーバを追加することで読み取り容量を増やすことができます。一般的に書き込み容量にも制限はありません。TiKVノードを追加することで書き込み容量を増やすことができます。

### `transaction too large`エラーメッセージが表示される

基礎ストレージエンジンの制限により、TiDBの各キー-値エントリ（1行）は6MBを超えてはいけません。[`txn-entry-size-limit`](/tidb-configuration-file.md#txn-entry-size-limit-new-in-v50)の構成値を最大で120MBまで調整することができます。

```
      Distributed transactions need two-phase commit and the bottom layer performs the Raft replication. If a transaction is very large, the commit process would be quite slow and the write conflict is more likely to occur. Moreover, the rollback of a failed transaction leads to an unnecessary performance penalty. To avoid these problems, we limit the total size of key-value entries to no more than 100MB in a transaction by default. If you need larger transactions, modify the value of `txn-total-size-limit` in the TiDB configuration file. The maximum value of this configuration item is up to 10G. The actual limitation is also affected by the physical memory of the machine.

There are [similar limits](https://cloud.google.com/spanner/docs/limits) on Google Cloud Spanner.

### How to import data in batches?

When you import data, insert in batches and keep the number of rows within 10,000 for each batch.

### Does TiDB release space immediately after deleting data?

None of the `DELETE`, `TRUNCATE` and `DROP` operations release data immediately. For the `TRUNCATE` and `DROP` operations, after the TiDB GC (Garbage Collection) time (10 minutes by default), the data is deleted and the space is released. For the `DELETE` operation, the data is deleted but the space is not released according to TiDB GC. When subsequent data is written into RocksDB and executes `COMPACT`, the space is reused.

### Can I execute DDL operations on the target table when loading data?

No. None of the DDL operations can be executed on the target table when you load data, otherwise the data fails to be loaded.

### Does TiDB support the `replace into` syntax?

Yes.

### Why does the query speed getting slow after deleting data?

Deleting a large amount of data leaves a lot of useless keys, affecting the query efficiency. Currently the Region Merge feature is in development, which is expected to solve this problem. For details, see the [deleting data section in TiDB Best Practices](https://en.pingcap.com/blog/tidb-best-practice/#write).

### What is the most efficient way of deleting data?

When deleting a large amount of data, it is recommended to use `Delete from t where xx limit 5000;`. It deletes through the loop and uses `Affected Rows == 0` as a condition to end the loop, so as not to exceed the limit of transaction size. With the prerequisite of meeting business filtering logic, it is recommended to add a strong filter index column or directly use the primary key to select the range, such as `id >= 5000*n+m and id < 5000*(n+1)+m`.

If the amount of data that needs to be deleted at a time is very large, this loop method will get slower and slower because each deletion traverses backward. After deleting the previous data, lots of deleted flags remain for a short period (then all will be processed by Garbage Collection) and influence the following Delete statement. If possible, it is recommended to refine the Where condition. See [details in TiDB Best Practices](https://en.pingcap.com/blog/tidb-best-practice/#write).

### How to improve the data loading speed in TiDB?

- The [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) tool is developed for distributed data import. It should be noted that the data import process does not perform a complete transaction process for performance reasons. Therefore, the ACID constraint of the data being imported during the import process cannot be guaranteed. The ACID constraint of the imported data can only be guaranteed after the entire import process ends. Therefore, the applicable scenarios mainly include importing new data (such as a new table or a new index) or the full backup and restoring (truncate the original table and then import data).
- Data loading in TiDB is related to the status of disks and the whole cluster. When loading data, pay attention to metrics like the disk usage rate of the host, TiClient Error, Backoff, Thread CPU and so on. You can analyze the bottlenecks using these metrics.
```