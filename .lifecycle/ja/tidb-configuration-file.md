---
title: TiDB構成ファイル
summary: コマンドラインオプションに関与しないTiDB構成ファイルのオプションを学ぶ
aliases: ['/docs/dev/tidb-configuration-file/','/docs/dev/reference/configuration/tidb-server/configuration-file/']
---

<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->

# TiDB構成ファイル

TiDB構成ファイルはコマンドラインパラメータよりも多くのオプションをサポートしています。デフォルトの構成ファイル[`config.toml.example`](https://github.com/pingcap/tidb/blob/master/pkg/config/config.toml.example)をダウンロードし、`config.toml`にリネームしてください。このドキュメントでは、[コマンドラインオプション](/command-line-flags-for-tidb-configuration.md)に関与しないオプションのみを説明しています。

> **ヒント:**
>
> 構成項目の値を調整する必要がある場合は、[構成の変更](/maintain-tidb-using-tiup.md#modify-the-configuration)を参照してください。

### `split-table`

- 各テーブルに独自のリージョンを作成するかどうかを決定します。
- デフォルト値: `true`
- 大量のテーブル（例: 10万を超えるテーブル）を作成する必要がある場合は、`false`に設定することをお勧めします。

### `tidb-max-reuse-chunk` <span class="version-mark">v6.4.0で新規</span>

- チャンク割り当ての最大キャッシュチャンクオブジェクトを制御します。この構成項目をあまりに大きな値に設定すると、OOMのリスクが高まる可能性があります。
- デフォルト値: `64`
- 最小値: `0`
- 最大値: `2147483647`

### `tidb-max-reuse-column` <span class="version-mark">v6.4.0で新規</span>

- 列の割り当ての最大キャッシュ列オブジェクトを制御します。この構成項目をあまりに大きな値に設定すると、OOMのリスクが高まる可能性があります。
- デフォルト値: `256`
- 最小値: `0`
- 最大値: `2147483647`

### `token-limit`

+ 同時に実行可能なリクエストを持つセッションの数。
+ タイプ: 整数
+ デフォルト値: `1000`
+ 最小値: `1`
+ 最大値 (64ビットプラットフォーム): `18446744073709551615`
+ 最大値 (32ビットプラットフォーム): `4294967295`

### `temp-dir` <span class="version-mark">v6.3.0で新規</span>

+ TiDBが一時データを保存するために使用するファイルシステムの場所。TiDBノードでローカルストレージが必要な場合、TiDBは対応する一時データをこの場所に保存します。
+ インデックスの作成中、新しく作成されたインデックスのために補填する必要のあるデータは最初にTiDBローカル一時ディレクトリに保存され、その後TiKVにバッチでインポートされます。これによりインデックスの作成が迅速化されます。このオプションは[`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)が有効になっている場合に適用されます。
+ データのインポートに[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)を使用する場合、ソートされたデータは最初にTiDBローカル一時ディレクトリに保存され、その後TiKVにバッチでインポートされます。
+ デフォルト値: `"/tmp/tidb"`

> **注意:**
>
> ディレクトリが存在しない場合、TiDBは起動時に自動的に作成します。ディレクトリの作成に失敗したり、TiDBがそのディレクトリに対する読み書き権限を持っていない場合、[`Fast Online DDL`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)で予測不能な問題が発生する可能性があります。

### `oom-use-tmp-storage`

> **警告:**
>
> v6.3.0以降、この構成項目は非推奨であり、システム変数[`tidb_enable_tmp_storage_on_oom`](/system-variables.md#tidb_enable_tmp_storage_on_oom)によって置き換えられています。TiDBクラスタをv6.3.0またはそれ以降のバージョンにアップグレードすると、自動的に変数が`oom-use-tmp-storage`の値で初期化されます。その後、`oom-use-tmp-storage`の値を変更しても**効果がありません**。

+ 単一のSQLステートメントでシステム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)で指定されたメモリクォータを超過したとき、一部のオペレータに一時ストレージを有効にするかどうかを制御します。
+ デフォルト値: `true`

### `tmp-storage-path`

+ 単一のSQLステートメントでシステム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)で指定されたメモリクォータを超過したとき、一部のオペレータのための一時ストレージパスを指定します。
+ デフォルト値: `<OSの一時ディレクトリ>/<OSユーザーID>_tidb/MC4wLjAuMDo0MDAwLzAuMC4wLjA6MTAwODA=/tmp-storage`。`MC4wLjAuMDo0MDAwLzAuMC4wLjA6MTAwODA=`は`<host>:<port>/<statusHost>:<statusPort>`の`Base64`エンコード結果です。
+ この構成は、システム変数[`tidb_enable_tmp_storage_on_oom`](/system-variables.md#tidb_enable_tmp_storage_on_oom)が`ON`に設定された場合のみ適用されます。

### `tmp-storage-quota`

+ `tmp-storage-path`内のストレージのクォータを指定します。単位はバイトです。
+ 単一のSQLステートメントで一時ディスクを使用し、TiDBサーバの一時ディスクの総容量がこの構成値を超える場合、現在のSQL操作がキャンセルされ、`Out of Global Storage Quota!`エラーが返されます。
+ この構成の値が`0`未満の場合、上記のチェックと制限は適用されません。
+ デフォルト値: `-1`
+ `tmp-storage-path`の利用可能なストレージが`tmp-storage-quota`で定義された値よりも小さい場合、TiDBサーバは起動時にエラーを報告して終了します。

### `lease`

+ DDLリースのタイムアウト時間。
+ デフォルト値: `45s`
+ 単位: 秒

### `compatible-kill-query`

+ `KILL`ステートメントをMySQL互換に設定するかどうかを決定します。
+ デフォルト値: `false`
+ [`enable-global-kill`](#enable-global-kill-new-in-v610)が`false`に設定されている場合にのみ`compatible-kill-query`が有効です。
+ [`enable-global-kill`](#enable-global-kill-new-in-v610)が`false`に設定されている場合、`compatible-kill-query`は、クエリを終了する際に`TIDB`キーワードを追加する必要があるかどうかを制御します。
    - `compatible-kill-query`が`false`の場合、TiDBでの`KILL xxx`の動作はMySQLと異なります。TiDBでクエリを終了するには、`KILL TIDB xxx`のように`TIDB`キーワードを追加する必要があります。
    - `compatible-kill-query`が`true`の場合、TiDBでクエリを終了するには`TIDB`キーワードを追加する必要はありません。**クライアントが常に同じTiDBインスタンスに接続されることが確実である**場合を除き、`compatible-kill-query`を`true`に設定することは**強くお勧めしません**。デフォルトのMySQLクライアントで<kbd>Control</kbd>+<kbd>C</kbd>を押すと、`KILL`が実行される新しい接続が開かれます。クライアントとTiDBクラスタの間にプロキシがある場合は、新しい接続が誤って異なるTiDBインスタンスにルーティングされ、誤って異なるセッションが終了される可能性があります。
+ [`enable-global-kill`](#enable-global-kill-new-in-v610)が`true`の場合、`KILL xxx`と`KILL TIDB xxx`は同じ効果を持ちますが、<kbd>Control</kbd>+<kbd>C</kbd>を使用してクエリを終了することはサポートされていません。
+ `KILL`ステートメントの詳細については、[KILL [TIDB]](/sql-statements/sql-statement-kill.md)を参照してください。

### `check-mb4-value-in-utf8`

- `utf8mb4`文字チェックを有効にするかどうかを制御します。この機能が有効になっていると、文字セットが`utf8`である場合、`utf8`に`mb4`文字が挿入された場合にエラーが返されます。
- デフォルト値: `false`
- v6.1.0以降、`utf8mb4`文字チェックの有効化はTiDB構成項目`instance.tidb_check_mb4_value_in_utf8`またはシステム変数`tidb_check_mb4_value_in_utf8`によって決定されます。`check-mb4-value-in-utf8`は引き続き有効ですが、`check-mb4-value-in-utf8`と`instance.tidb_check_mb4_value_in_utf8`の両方が設定されている場合は、後者が優先されます。

### `treat-old-version-utf8-as-utf8mb4`

- 古いテーブルの`utf8`文字セットを`utf8mb4`として扱うかどうかを制御します。
- デフォルト値: `true`

### `alter-primary-key`（非推奨）

- 列に主キー制約を追加または削除するかどうかを決定します。
- このデフォルト設定では、プライマリ キー制約の追加または削除はサポートされていません。`alter-primary-key` を `true` に設定することで、この機能を有効にすることができます。ただし、スイッチがオンになる前にテーブルがすでに存在しており、そのプライマリ キー列のデータ型が整数型である場合、この構成項目を `true` に設定しても、その列からプライマリ キーを削除することはできません。

> **注記:**
>
> この構成項目は非推奨となり、現在は`@tidb_enable_clustered_index`の値が`INT_ONLY`の場合にのみ影響を受けます。プライマリ キーを追加または削除する必要がある場合は、テーブルを作成する際に代わりに`NONCLUSTERED`キーワードを使用してください。`CLUSTERED`タイプのプライマリ キーの詳細については、[clustered index](/clustered-indexes.md)を参照してください。

### `server-version`

+ 以下の状況で、TiDBが返すバージョン文字列を変更します。
    - 組込みの`VERSION()`関数が使用された場合。
    - TiDBがクライアントに初期接続を確立し、サーバのバージョン文字列を含む初期ハンドシェイクパケットを返す場合。詳細については、[MySQL Initial Handshake Packet](https://dev.mysql.com/doc/dev/mysql-server/latest/page_protocol_connection_phase.html#sect_protocol_connection_phase_initial_handshake)を参照してください。
+ デフォルト値: ""
+ デフォルトでは、TiDBのバージョン文字列の形式は `5.7.${mysql_latest_minor_version}-TiDB-${tidb_version}` です。

> **注記:**
>
> TiDBノードは`server-version`の値を使用して現在のTiDBバージョンを検証します。したがって、TiDBクラスタをアップグレードする前に、`server-version`の値を空に設定するか、現在のTiDBクラスタの実際のバージョンに設定する必要があります。予期しない動作を回避するために。

### `repair-mode`

- 信頼されていないリペアモードを有効にするかどうかを決定します。`repair-mode` が `true` に設定されている場合、`repair-table-list`の悪いテーブルはロードできません。
- デフォルト値: `false`
- `repair`構文はデフォルトでサポートされません。これはつまり、TiDBが起動されるときにはすべてのテーブルがロードされることを意味します。

### `repair-table-list`

- [`repair-mode`](#repair-mode)が`true`に設定されている場合にのみ有効です。`repair-table-list`はインスタンスで修復する必要のある悪いテーブルのリストです。リストの例: ["db.table1","db.table2"...].
- デフォルト値: []
- デフォルトでは、リストは空です。つまり、修復する必要のある悪いテーブルはありません。

### `new_collations_enabled_on_first_bootstrap`

- 新しい照合順序サポートを有効または無効にします。
- デフォルト値: `true`
- 注: この構成は初めて初期化されるTiDBクラスタにのみ影響します。初期化後、この構成項目を使用して新しい照合順序サポートを有効または無効にすることはできません。

### `max-server-connections`

- TiDBで許可される同時クライアント接続の最大数です。これはリソースを制御するために使用されます。
- デフォルト値: `0`
- デフォルトでは、TiDBは同時クライアント接続の数に制限を設定しません。この構成項目の値が`0`よりも大きく、実際のクライアント接続数がこの値に達した場合、TiDBサーバは新しいクライアント接続を拒否します。
- v6.2.0以降、TiDBの同時クライアント接続の最大数を設定するには、TiDB構成項目[`instance.max_connections`](/tidb-configuration-file.md#max_connections)またはシステム変数[`max_connections`](/system-variables.md#max_connections)を使用します。`max-server-connections`は引き続き有効です。ただし、`max-server-connections`および`instance.max_connections`が同時に設定されている場合、後者が優先されます。

### `max-index-length`

- 新しく作成されるインデックスの最大許容長を設定します。
- デフォルト値: `3072`
- 単位: バイト
- 現在、有効な値の範囲は`[3072, 3072*4]`です。MySQLおよびTiDB（バージョン< v3.0.11）にはこの構成項目はありませんが、両方とも新しく作成されたインデックスの長さを制限しています。MySQLの制限値は`3072`、TiDB（バージョン=< 3.0.7）の制限値は`3072*4`です。TiDB（3.0.7 < バージョン < 3.0.11）の制限値は`3072`です。この構成は、MySQLおよびTiDBの以前のバージョンと互換性を持たせるために追加されています。

### `table-column-count-limit` <span class="version-mark">v5.0で新規</span>

- 単一のテーブル内の列数の制限を設定します。
- デフォルト値: `1017`
- 現在、有効な値の範囲は`[1017, 4096]`です。

### `index-limit` <span class="version-mark">v5.0で新規</span>

- 単一のテーブル内のインデックス数の制限を設定します。
- デフォルト値: `64`
- 現在、有効な値の範囲は`[64, 512]`です。

### `enable-telemetry` <span class="version-mark">v4.0.2で新規</span>

- TiDBでのテレメトリ収集を有効または無効にします。
- デフォルト値: `false`
- この構成がTiDBインスタンスで`true`に設定されている場合、このTiDBインスタンスでのテレメトリ収集が有効になり、[`tidb_enable_telemetry`](/system-variables.md#tidb_enable_telemetry-new-in-v402)システム変数が影響を受けます。
- すべてのTiDBインスタンスでこの構成が`false`に設定されている場合、TiDBでのテレメトリ収集は無効になり、[`tidb_enable_telemetry`](/system-variables.md#tidb_enable_telemetry-new-in-v402)システム変数は影響を受けません。詳細については、[Telemetry](/telemetry.md)を参照してください。

### `deprecate-integer-display-length`

- この構成項目が`true`に設定されている場合、整数型の表示幅を非推奨にします。
- デフォルト値: `false`

### `enable-tcp4-only` <span class="version-mark">v5.0で新規</span>

- TCP4のみでのリスンを有効または無効にします。
- デフォルト値: `false`
- このオプションを有効にすると、LVSを使用してTiDBをロードバランシングするときに便利です。なぜならば、「tcp4」プロトコルによってTCPヘッダから[実際のクライアントIPが正しく解析](https://github.com/alibaba/LVS/tree/master/kernel/net/toa)されるためです。

### `enable-enum-length-limit` <span class="version-mark">v5.0で新規</span>

+ 単一の`ENUM`要素と単一の`SET`要素の最大長を制限するかどうかを決定します。
+ デフォルト値: `true`
+ この構成の値が`true`の場合、単一の`ENUM`要素と単一の`SET`要素の最大長は255文字で、これは[MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/string-type-syntax.html)と互換性があります。この構成の値が`false`の場合、単一要素の長さに制限はありません。これはTiDB（v5.0より前）と互換性があります。

### `graceful-wait-before-shutdown` <span class="version-mark">v5.0で新規</span>

- サーバをシャットダウンする際にTiDBがクライアントが切断されるまで待機する秒数を指定します。
- デフォルト値: `0`
- TiDBがシャットダウンを待っている間（グレース期間）、HTTPステータスは失敗を示し、ロードバランサがトラフィックを再ルーティングすることを可能にします。

### `enable-global-kill` <span class="version-mark">v6.1.0で新規</span>

+ グローバルキル（インスタンス間でクエリまたは接続を終了する機能）を有効または無効にします。
+ デフォルト値: `true`
+ この値が`true`の場合、`KILL`および`KILL TIDB`ステートメントの両方がクエリまたは接続をインスタンス間で終了できるため、クエリまたは接続を誤って終了させる心配はありません。クライアントを使用してTiDBインスタンスに接続し、`KILL`または`KILL TIDB`ステートメントを実行すると、ステートメントは対象のTiDBインスタンスに転送されます。クライアントとTiDBクラスタの間にプロキシがある場合、`KILL`および`KILL TIDB`ステートメントも対象のTiDBインスタンスで実行されます。
+ v7.3.0以降、この構成と[`enable-32bits-connection-id`](#enable-32bits-connection-id-new-in-v730)が`true`に設定されている場合、MySQLのコマンドライン<kbd>Control+C</kbd>を使用してクエリまたは接続を終了できます。詳細については、[`KILL`](/sql-statements/sql-statement-kill.md)を参照してください。

### `enable-32bits-connection-id` <span class="version-mark">v7.3.0で新規</span>

+ 32ビット接続ID機能を有効または無効にします。
+ デフォルト値: `true`
+ この構成項目と[`enable-global-kill`](#enable-global-kill-new-in-v610)が`true`に設定されている場合、TiDBは32ビット接続IDを生成します。これにより、MySQLコマンドライン<kbd>Control+C</kbd>を使用してクエリまたは接続を終了できます。

> **警告:**

> TiDBインスタンスの数がクラスター内で2048を超えるか、単一のTiDBインスタンスの同時接続数が1048576を超えると、32ビット接続IDスペースは不十分となり、自動的に64ビット接続IDにアップグレードされます。アップグレードプロセス中は既存のビジネスと確立された接続に影響はありません。ただし、以降の新しい接続はMySQLのコマンドラインで<kbd>Control+C</kbd>を使用して終了できません。

### `initialize-sql-file` <span class="version-mark">v6.6.0で新規追加</span>

+ TiDBクラスターが初めて起動されるときに実行されるSQLスクリプトを指定します。
+ デフォルト値: `""`
+ このスクリプト内のすべてのSQLステートメントは、特権の確認なしに最高の特権で実行されます。指定されたSQLスクリプトの実行に失敗すると、TiDBクラスターは起動しない可能性があります。
+ この構成項目は、システム変数の値を変更したり、ユーザーを作成したり、権限を付与するなどの操作を行うために使用されます。

### `enable-forwarding` <span class="version-mark">v5.0.0で新規追加</span>

+ TiDBのPDクライアントとTiKVクライアントが、ネットワークの隔離が発生した場合にリーダー経由でフォロワーに要求を転送するかどうかを制御します。
+ デフォルト値: `false`
+ 環境にネットワークの隔離が発生する可能性がある場合、このパラメータを有効にするとサービスの利用できない期間を短縮できます。
+ 隔離、ネットワークの中断、またはダウンタイムが正確に判断できない場合、このメカニズムを使用することで誤判断が発生し、可用性とパフォーマンスが低下します。ネットワーク障害が発生したことがない場合、このパラメータを有効にすることは推奨されません。

### `enable-table-lock` <span class="version-mark">v4.0.0で新規追加</span>

> **注意:**
>
> テーブルロックは実験的な機能です。本番環境で使用することは推奨されません。

+ テーブルロック機能を有効にするかどうかを制御します。
+ デフォルト値: `false`
+ テーブルロックは複数のセッションで同じテーブルへの同時アクセスを調整するために使用されます。現在、`READ`、`WRITE`、`WRITE LOCAL`のロックタイプがサポートされています。構成項目が`false`に設定されている場合、`LOCK TABLES`または`UNLOCK TABLES`ステートメントの実行が無効であり、"LOCK/UNLOCK TABLES is not supported"の警告が返されます。詳細については、[`LOCK TABLES`および`UNLOCK TABLES`](/sql-statements/sql-statement-lock-tables-and-unlock-tables.md)を参照してください。

### `labels`

+ サーバーのラベルを指定します。例えば、 `{ zone = "us-west-1", dc = "dc1", rack = "rack1", host = "tidb1" }`。
+ デフォルト値: `{}`

> **注意:**
>
> - TiDBでは、`zone`ラベルはサーバーの所在地を指定するために特に使用されます。`zone`がヌル以外の値に設定されている場合、該当する値は自動的に[`txn-score`](/system-variables.md#txn_scope)や[`Follower read`](/follower-read.md)などの機能で使用されます。
> - `group`ラベルは、TiDB Operatorにおいて特別な用途があります。[TiDB Operator](/tidb-operator-overview.md)を使用して展開されたクラスターでは、`group`ラベルを手動で指定することは**推奨されていません**。

## ログ

ログに関連する構成項目。

### `level`

+ ログの出力レベルを指定します。
+ 値の選択肢: `debug`、`info`、`warn`、`error`、`fatal`。
+ デフォルト値: `info`

### `format`

- ログの出力形式を指定します。
- 値の選択肢: `json`、`text`。
- デフォルト値: `text`

### `enable-timestamp`

- ログでタイムスタンプ出力を有効にするかどうかを決定します。
- デフォルト値: `null`
- 値を`false`に設定すると、ログはタイムスタンプを出力しません。

> **注意:**
>
> - 互換性を保つために、初期の`disable-timestamp`構成項目は引き続き有効です。ただし、`disable-timestamp`の値が`enable-timestamp`の値と意味的に競合する場合（例: `enable-timestamp`と`disable-timestamp`が両方とも`true`に設定されている場合）、TiDBは`disable-timestamp`の値を無視します。
> - 現在、TiDBはログでタイムスタンプを出力するかどうかを決定するために、`disable-timestamp`を使用しています。この状況では、`enable-timestamp`の値は`null`です。
> - 後のバージョンでは、`disable-timestamp`構成は削除されます。`disable-timestamp`を廃止し、意味的に理解しやすい`enable-timestamp`を使用してください。

### `enable-slow-log`

- 遅いクエリログを有効にするかどうかを決定します。
- デフォルト値: `true`
- 遅いクエリログを有効にするには、`enable-slow-log`を`true`に設定します。それ以外の場合は`false`に設定します。
- v6.1.0以降、遅いクエリログを有効にするかどうかは、TiDBの構成項目[`instance.tidb_enable_slow_log`](/tidb-configuration-file.md#tidb_enable_slow_log)またはシステム変数[`tidb_enable_slow_log`](/system-variables.md#tidb_enable_slow_log)によって決まります。`enable-slow-log`は引き続き有効ですが、`enable-slow-log`と`instance.tidb_enable_slow_log`が同時に設定されている場合は、後者が優先されます。

### `slow-query-file`

- 遅いクエリログのファイル名です。
- デフォルト値: `tidb-slow.log`
- 遅いクエリログのフォーマットはTiDB v2.1.8で更新され、遅いクエリログは別々のファイルに出力されます。v2.1.8以前のバージョンでは、この変数はデフォルトで""に設定されています。
- 設定した場合、遅いクエリログはこのファイルに出力されます。

### `slow-threshold`

- 遅いクエリログで消費時間の閾値値を出力します。
- デフォルト値: `300`
- 単位: ミリ秒
- クエリ内の値がデフォルト値よりも大きい場合、遅いクエリとして遅いクエリログに出力されます。
- v6.1.0以降、遅いクエリログ内での消費時間の閾値値は、TiDBの構成項目[`instance.tidb_slow_log_threshold`](/tidb-configuration-file.md#tidb_slow_log_threshold)またはシステム変数[`tidb_slow_log_threshold`](/system-variables.md#tidb_slow_log_threshold)で指定されます。`slow-threshold`は引き続き有効ですが、`slow-threshold`と`instance.tidb_slow_log_threshold`が同時に設定されている場合は、後者が優先されます。

### `record-plan-in-slow-log`

- 遅いクエリログで実行計画を記録するかどうかを決定します。
- デフォルト値: `1`
- v6.1.0以降、遅いクエリログで実行計画を記録するかどうかは、TiDBの構成項目[`instance.tidb_record_plan_in_slow_log`](/tidb-configuration-file.md#tidb_record_plan_in_slow_log)またはシステム変数[`tidb_record_plan_in_slow_log`](/system-variables.md#tidb_record_plan_in_slow_log)によって決まります。`record-plan-in-slow-log`は引き続き有効ですが、`record-plan-in-slow-log`と`instance.tidb_record_plan_in_slow_log`が同時に設定されている場合は、後者が優先されます。

### `expensive-threshold`

> **警告:**
>
> v5.4.0以降、`expensive-threshold`構成項目は非推奨となり、システム変数[`tidb_expensive_query_time_threshold`](/system-variables.md#tidb_expensive_query_time_threshold)に置き換えられました。

- `expensive`操作の行数の閾値値を出力します。
- デフォルト値: `10000`
- クエリの行数（統計に基づく中間結果を含む）がこの値よりも大きい場合、`expensive`操作として、`[EXPENSIVE_QUERY]`接頭辞付きのログが出力されます。

### `timeout` <span class="version-mark">v7.1.0で新規追加</span>

- TiDBにおけるログ書き込み操作のタイムアウトを設定します。ログの書き込みが失敗し、ディスクの障害が原因でログが書き込めない場合、この構成項目はTiDBプロセスがハングする代わりにパニックを発生させます。
- デフォルト値: `0`（タイムアウトが設定されていないことを示す）
- 単位: 秒
- 一部のユーザーシナリオでは、TiDBのログがホットプラッガブルディスクやネットワーク接続ディスクに保存される場合があり、これらは永続的に利用できなくなることがあります。この場合、TiDBはそのような災害から自動的に回復することができず、ログの書き込み操作が永久的にブロックされます。この構成項目はこのような状況に対処するために設計されています。

## log.file

ログファイルに関連する構成項目。

#### `filename`

- 一般的なログファイルのファイル名です。
- デフォルト値: ""
- 設定した場合、ログはこのファイルに出力されます。

#### `max-size`

- ログファイルのサイズ制限です。
- デフォルト値: 300
- 単位: MB
- 最大値は4096です。

#### `max-days`

- ログが保持される最大日数です。
- デフォルト値: `0`
- ログはデフォルトで保持されます。値を設定すると、期限切れのログは`max-days`後に削除されます。

#### `max-backups`

- 保持されるログの最大数です。
- デフォルト値: `0`
- すべてのログファイルはデフォルトで保持されます。`7` に設定すると、最大で7つのログファイルが保持されます。

## セキュリティ

セキュリティに関連する構成項目。

### `enable-sem`

- セキュリティ強化モード（SEM）を有効にします。
- デフォルト値： `false`
- SEM のステータスは、システム変数 [`tidb_enable_enhanced_security`](/system-variables.md#tidb_enable_enhanced_security) を通じて確認できます。

### `ssl-ca`

- PEM 形式の信頼されたCA証明書のファイルパスです。
- デフォルト値： ""
- このオプションを設定し、同時に `--ssl-cert`、`--ssl-key` を設定した場合、クライアントが証明書を提示した際に、TiDB はこのオプションで指定された信頼されたCAのリストに基づいてクライアント証明書を認証します。認証に失敗した場合、接続が終了します。
- このオプションを設定しているが、クライアントが証明書を提示していない場合、クライアント証明書の認証なしでセキュアな接続が継続します。

### `ssl-cert`

- PEM 形式のSSL証明書のファイルパスです。
- デフォルト値： ""
- このオプションを設定し、同時に `--ssl-key` を設定した場合、TiDB はクライアントがTLSを使用してセキュアにTiDBに接続することを許可します（しかし強制はしません）。
- 指定した証明書または秘密キーが無効な場合、TiDB は通常通りに起動しますが、セキュアな接続を受け取ることはできません。

### `ssl-key`

- PEM 形式のSSL証明書キーのファイルパスです。つまり、`--ssl-cert` で指定された証明書の秘密キーです。
- デフォルト値： ""
- 現在、TiDB はパスワードで保護されている秘密鍵の読み込みをサポートしていません。

### `cluster-ssl-ca`

- TiKV や PD とのTLS接続に使用されるCAルート証明書です。
- デフォルト値： ""

### `cluster-ssl-cert`

- TiKV や PD とのTLS接続に使用されるSSL証明書ファイルのパスです。
- デフォルト値： ""

### `cluster-ssl-key`

- TiKV や PD とのTLS接続に使用されるSSL秘密鍵ファイルのパスです。
- デフォルト値： ""

### `spilled-file-encryption-method`

+ ディスクへの溢れたファイルの保存に使用される暗号化方式を決定します。
+ デフォルト値： `"plaintext"`（暗号化を無効にします）
+ オプション値： `"plaintext"` および `"aes128-ctr"`

### `auto-tls`

- 起動時にTLS証明書を自動生成するかどうかを決定します。
- デフォルト値： `false`

### `tls-version`

- MySQLプロトコル接続のための最小TLSバージョンを設定します。
- デフォルト値： ""（TLSv1.1 以上が許可されます）
- オプション値： `"TLSv1.0"`、`"TLSv1.1"`、`"TLSv1.2"`、`"TLSv1.3"`

### `auth-token-jwks` <span class="version-mark">v6.4.0で新規追加</span>

> **注意:**
>
> `tidb_auth_token` 認証方法は TiDB Cloud の内部操作でのみ使用されます。この構成の値を **変更しないでください**。

- `tidb_auth_token` 認証方法のJSON Web Key Sets（JWKS）のローカルファイルパスを設定します。
- デフォルト値： `""`

### `auth-token-refresh-interval` <span class="version-mark">v6.4.0で新規追加</span>

> **注意:**
>
> `tidb_auth_token` 認証方法は TiDB Cloud の内部操作でのみ使用されます。この構成の値を **変更しないでください**。

- `tidb_auth_token` 認証方法のJWKSリフレッシュ間隔を設定します。
- デフォルト値： `1h`

### `disconnect-on-expired-password` <span class="version-mark">v6.5.0で新規追加</span>

- パスワードの有効期限が切れた場合、TiDBがクライアント接続を切断するかどうかを決定します。
- デフォルト値： `true`
- オプション値： `true`、 `false`
- `true` に設定すると、パスワードの有効期限が切れた場合にクライアント接続が切断されます。`false` に設定すると、クライアント接続が "サンドボックスモード" に制限され、ユーザーはパスワードリセット操作の実行のみが可能です。

### `session-token-signing-cert` <span class="version-mark">v6.4.0で新規追加</span>

> **注意:**
>
> このパラメータによって制御される機能は開発中です。**デフォルト値を変更しないでください**。

+ デフォルト値： ""

### `session-token-signing-key` <span class="version-mark">v6.4.0で新規追加</span>

> **注意:**
>
> このパラメータによって制御される機能は開発中です。**デフォルト値を変更しないでください**。

+ デフォルト値： ""

## パフォーマンス

パフォーマンスに関連する構成項目。

### `max-procs`

- TiDB で使用されるCPUの数です。
- デフォルト値： `0`
- デフォルトの `0` はマシン上のすべてのCPUを使用することを指します。 n に設定することもでき、その場合 TiDB は n 個のCPUを使用します。

### `server-memory-quota` <span class="version-mark">v4.0.9で新規追加</span>

> **注意:**
>
> v6.5.0以降、`server-memory-quota` 構成項目は非推奨となり、システム変数 [`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640) に置き換えられました。

+ tidb-server インスタンスのメモリ使用量制限です。
+ デフォルト値： `0`（バイト単位）、つまりメモリ制限はありません。

### `max-txn-ttl`

- 単一トランザクションがロックを保持できる最長時間です。この時間を超過すると、他のトランザクションによってトランザクションのロックが解放され、このトランザクションは正常にコミットされません。
- デフォルト値： `3600000`
- 単位：ミリ秒
- この時間を超過するロックを保持するトランザクションはコミットまたはロールバックのみ可能です。コミットが成功しない可能性があります。

### `stmt-count-limit`

- 単一のTiDBトランザクション内で許可されるステートメントの最大数です。
- デフォルト値： `5000`
- `stmt-count-limit` を超過するステートメント数でトランザクションがロールバックやコミットを行わない場合、TiDB は `statement count 5001 exceeds the transaction limitation, autocommit = false` エラーを返します。この構成は **再試行可能な楽観的トランザクション** でのみ有効です。**悲観的トランザクション** を使用するかトランザクションの再試行を無効にしている場合、トランザクション内のステートメント数はこの構成によって制限されません。

### `txn-entry-size-limit` <span class="version-mark">v5.0で新規追加</span>

- TiDB における単一行のデータのサイズ制限です。
- デフォルト値： `6291456`（バイト単位）
- トランザクション内の単一のキー値レコードのサイズ制限です。サイズ制限を超えると、TiDB は `entry too large` エラーを返します。この構成項目の最大値は `125829120`（120MB）を超えることはできません。なお、TiKV に同様の制限があります。1つの書き込みリクエストのデータサイズが、デフォルトで8MBの [`raft-entry-max-size`](/tikv-configuration-file.md#raft-entry-max-size) を超えた場合、TiKV はそのリクエストの処理を拒否します。1行のサイズが大きいテーブルを持つ場合、同時に両方の構成を変更する必要があります。
- [`max_allowed_packet`](/system-variables.md#max_allowed_packet-new-in-v610) のデフォルト値（MySQLプロトコルのパケットの最大サイズ）は67108864（64MB）です。行が `max_allowed_packet` を超える場合、行は切り捨てられます。
- [`txn-total-size-limit`](#txn-total-size-limit) のデフォルト値は100MBです。`txn-entry-size-limit` の値を100MBを超えるように設定する場合、それに応じて `txn-total-size-limit` の値も増やす必要があります。

### `txn-total-size-limit`

- TiDB における単一トランザクションのサイズ制限です。
- デフォルト値： `104857600`（バイト単位）
- 単一トランザクション内のキー値レコードの合計サイズはこの値を超えることはできません。このパラメータの最大値は `1099511627776`（1TB）です。なお、ダウンストリームのコンシューマーKafka（`arbiter` クラスターなど）にバイナリログを使用している場合、このパラメータの値は `1073741824`（1GB）を超えてはなりません。なぜなら、Kafkaが処理できる単一メッセージの最大サイズが1GBだからです。この制限を超えるとエラーが返されます。
- TiDB v6.5.0以降のバージョンでは、この構成を推奨しません。トランザクションのメモリサイズはセッションのメモリ使用量に蓄積され、セッションメモリのしきい値を超えたときに [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) 変数が効果を持ちます。以前のバージョンとの互換性を保つため、この構成は次のように動作します（以前のバージョンからTiDB v6.5.0以降にアップグレードする場合）：
    - この構成が設定されていないか、デフォルト値（`104857600`）に設定されている場合は、アップグレード後、トランザクションのメモリサイズはセッションのメモリ使用量に蓄積され、`tidb_mem_quota_query` 変数が効果を持ちます。
- この構成がデフォルト値（`104857600`）で設定されていない場合、そのまま有効となるため、アップグレード前後で単一トランザクションのサイズを制御する動作が変わりません。つまり、トランザクションのメモリサイズは`tidb_mem_quota_query`変数で制御されていないことを意味します。

### `tcp-keep-alive`

- TCPレイヤーで`keepalive`を有効にするかどうかを決定します。
- デフォルト値: `true`

### `tcp-no-delay`

- TCPレイヤーのTCP_NODELAYを有効にするかどうかを決定します。有効にした後、TiDBはTCP/IPプロトコルのNagleアルゴリズムを無効にし、ネットワークレイテンシを減らすために小さなデータパケットを送信することが可能となります。これは、データ量が少ないレイテンシに敏感なアプリケーションに適しています。
- デフォルト値: `true`

### `cross-join`

- デフォルト値: `true`
- TiDBはデフォルトで両方のテーブルの条件（`WHERE`フィールド）なしに`JOIN`文を実行することができます。もし値を`false`に設定すると、そのような`JOIN`文が出現した場合、サーバーは実行を拒否します。

### `stats-lease`

- 統計情報をリロードし、テーブル行数の更新、自動分析の実施が必要かどうかのチェック、フィードバックを使用して統計情報を更新し、カラムの統計情報の読み込みの時間間隔です。
- デフォルト値: `3s`
    - `stats-lease`の間隔で、TiDBは統計情報を更新し、更新があればメモリに更新されます。
    - `20 * stats-lease`の間隔で、TiDBはDMLによって生成された合計行数および変更された行数をシステムテーブルに更新します。
    - `stats-lease`の間隔で、TiDBは自動的に分析する必要があるテーブルやインデックスをチェックします。
    - `stats-lease`の間隔で、TiDBはメモリに読み込む必要があるカラム統計情報をチェックします。
    - `200 * stats-lease`の間隔で、TiDBはメモリにキャッシュされたフィードバックをシステムテーブルに書き込みます。
    - `5 * stats-lease`の間隔で、TiDBはシステムテーブルのフィードバックを読み込み、メモリにキャッシュされた統計情報を更新します。
- `stats-lease`を0秒に設定すると、TiDBは定期的にシステムテーブル内のフィードバックを読み込み、3秒ごとに統計情報を更新します。ただし、TiDBは以下の統計情報関連のシステムテーブルを自動的に変更しません:
    - `mysql.stats_meta`: TiDBはトランザクションによって変更されたテーブル行数を自動記録し、このシステムテーブルに更新しません。
    - `mysql.stats_histograms`/`mysql.stats_buckets`および`mysql.stats_top_n`: TiDBは自動的に分析を行い、統計情報を積極的に更新しません。
    - `mysql.stats_feedback`: TiDBはクエリされたデータの一部に基づいて戻された一部の統計情報に応じて、テーブルやインデックスの統計情報を更新しません。

### `pseudo-estimate-ratio`

- テーブルの修正行数/総行数の比率です。この値を超えると、システムは統計情報が期限切れであると見なし、擬似統計情報が使用されます。
- デフォルト値: `0.8`
- 最小値は`0`、最大値は`1`です。

### `force-priority`

- すべてのステートメントの優先度を設定します。
- デフォルト値: `NO_PRIORITY`
- 値のオプション: デフォルト値である`NO_PRIORITY`は、ステートメントの優先度を強制的に変更しないことを意味します。他のオプションは`LOW_PRIORITY`、`DELAYED`、`HIGH_PRIORITY`で昇順です。
- v6.1.0以降、すべてのステートメントの優先度はTiDB構成項目[`instance.tidb_force_priority`](/tidb-configuration-file.md#tidb_force_priority)またはシステム変数[`tidb_force_priority`](/system-variables.md#tidb_force_priority)によって決定されます。`force-priority`は引き続き有効です。ただし、`force-priority`と`instance.tidb_force_priority`が同時に設定されている場合、後者が優先されます。

### `distinct-agg-push-down`

- 最適化プログラムが`Distinct`を持つ集約関数をCoprocessorsにプッシュダウンする操作を実行するかどうかを決定します。
- デフォルト: `false`
- この変数はシステム変数[`tidb_opt_distinct_agg_push_down`](/system-variables.md#tidb_opt_distinct_agg_push_down)の初期値です。

### `enforce-mpp`

+ オプティマイザのコスト見積もりを無視し、クエリ実行にTiFlashのMPPモードを強制的に使用するかどうかを決定します。  
+ デフォルト値: `false`  
+ この構成項目は、[`tidb_enforce_mpp`](/system-variables.md#tidb_enforce_mpp-new-in-v51)の初期値を制御します。たとえば、この構成項目が`true`に設定されている場合、`tidb_enforce_mpp`のデフォルト値は`ON`となります。

### `enable-stats-cache-mem-quota` <span class="version-mark">v6.1.0で新規</span>

+ 統計情報キャッシュのメモリクォータを有効にするかどうかを制御します。
+ デフォルト値: `true`

### `stats-load-concurrency` <span class="version-mark">v5.4.0で新規</span>

+ TiDBが同期的に処理できる統計情報を読み込む際に使用できる最大のカラム数です。
+ デフォルト値: `5`
+ 現在、有効な値の範囲は`[1, 128]`です。

### `stats-load-queue-size` <span class="version-mark">v5.4.0で新規</span>

+ TiDBが同期的に読み込む統計情報のキャッシュが保持できる列リクエストの最大数です。
+ デフォルト値: `1000`
+ 現在、有効な値の範囲は`[1, 100000]`です。

### `lite-init-stats` <span class="version-mark">v7.1.0で新規</span>

+ TiDB起動時に軽量統計情報の初期化を使用するかどうかを制御します。
+ デフォルト値: v7.2.0以前のバージョンでは`false`、v7.2.0およびそれ以降のバージョンでは`true`
+ `lite-init-stats`の値が`true`の場合、統計情報の初期化はメモリにインデックスやカラムのヒストグラム、TopN、またはCount-Min Sketchを読み込まずに行います。`false`で初期化する場合は、インデックスやプライマリキーのヒストグラム、TopN、およびCount-Min Sketchをメモリに読み込みますが、それ以外のカラムのヒストグラム、TopN、およびCount-Min Sketchはメモリに読み込まれません。オプティマイザが特定のインデックスやカラムのヒストグラム、TopN、およびCount-Min Sketchを必要とする場合、必要な統計情報は同期的または非同期的（[`tidb_stats_load_sync_wait`](/system-variables.md#tidb_stats_load_sync_wait-new-in-v540)で制御）にメモリに読み込まれます。
+ `lite-init-stats`を`true`に設定すると、統計情報の初期化が高速化され、不要な統計情報の読み込みを避けることでTiDBのメモリ使用量が削減されます。詳細は、[統計情報の読み込み](/statistics.md#load-statistics)を参照してください。

### `force-init-stats` <span class="version-mark">v7.1.0で新規</span>

+ TiDB起動時に、サービスを提供する前に統計情報の初期化が完了するのを待つかどうかを制御します。
+ デフォルト値: v7.2.0以前のバージョンでは`false`、v7.2.0およびそれ以降のバージョンでは`true`
+ `force-init-stats`が`true`の場合、TiDBはサービスを提供する前に統計情報の初期化が完了するまで待機する必要があります。大量のテーブルやパーティションがあり、また[`lite-init-stats`](/tidb-configuration-file.md#lite-init-stats-new-in-v710)の値が`false`の場合、`force-init-stats`を`true`に設定すると、TiDBがサービスを提供するまでの時間が延びる可能性があります。
+ `force-init-stats`が`false`の場合、TiDBは統計情報の初期化が完了する前にもサービスを提供できますが、オプティマイザは擬似統計情報を使用して意思決定を行い、最適でない実行プランを引き起こす可能性があります。

## opentracing

opentracingに関連する構成項目です。

### `enable`

+ 一部のTiDBコンポーネントの呼び出しオーバーヘッドをトレースするためにopentracingを有効にします。opentracingを有効にするとパフォーマンスが低下することに注意してください。
+ デフォルト値: `false`

### `rpc-metrics`

+ RPCメトリクスを有効にします。
+ デフォルト値: `false`

## opentracing.sampler

opentracing.samplerに関連する構成項目です。

### `type`

+ opentracingサンプラーのタイプを指定します。文字列の値は大文字小文字を区別しません。
+ デフォルト値: `"const"`
+ 値のオプション: `"const"`, `"probabilistic"`, `"ratelimiting"`, `"remote"`

### `param`

+ opentracingサンプラーのパラメータです。
    - `const`タイプの場合、値は`0`または`1`で、`const`サンプラーを有効にするかどうかを示します。
    - `probabilistic`タイプの場合、パラメータは`0`から`1`の間の浮動小数点数で、サンプリング確率を指定します。
    - `ratelimiting`タイプの場合、パラメータは秒あたりのサンプリングされるスパン数を指定します。
    - `remote`タイプの場合、パラメータは`0`から`1`の間の浮動小数点数で、サンプリング確率を指定します。
+ デフォルト値: `1.0`

### `sampling-server-url`

+ jaeger-agentサンプリングサーバーのHTTP URLです。
+ デフォルト値: `""`

### `max-operations`
+ サンプラーがトレースできる操作の最大数。トレースされない操作は、デフォルトの確率サンプラーが使用されます。
+ デフォルト値：`0`

### `sampling-refresh-interval`

+ Jaegerエージェントのサンプリングポリシーをポーリングする頻度を制御します。
+ デフォルト値：`0`

## opentracing.reporter

opentracing.reporterに関連する構成項目。

### `queue-size`

+ レポーターがメモリ内でスパンを記録するためのキューサイズ。
+ デフォルト値：`0`

### `buffer-flush-interval`

+ レポーターがメモリ内のスパンをストレージにフラッシュする間隔。
+ デフォルト値：`0`

### `log-spans`

+ 送信された全てのスパンに対するログの出力を有効にするかどうかを決定します。
+ デフォルト値：`false`

### `local-agent-host-port`

+ レポーターがスパンをjaeger-agentに送信するアドレス。
+ デフォルト値：`""`

## tikv-client

### `grpc-connection-count`

- 各TiKVと確立される最大接続数。
- デフォルト値：`4`

### `grpc-keepalive-time`

- TiDBとTiKVノード間のRPC接続の`keepalive`時間間隔。指定された時間間隔内にネットワークパケットがない場合、gRPCクライアントはTiKVに`ping`コマンドを実行して生存しているかどうかを確認します。
- デフォルト：`10`
- 単位：秒

### `grpc-keepalive-timeout`

- TiDBとTiKVノード間のRPC`keepalive`チェックのタイムアウト。
- デフォルト値：`3`
- 単位：秒

### `grpc-compression-type`

- TiDBとTiKVノード間のデータ転送に使用される圧縮タイプを指定します。デフォルト値は`"none"`で、つまり圧縮なしを意味します。gzip圧縮を有効にするには、この値を`"gzip"`に設定します。
- デフォルト値：`"none"`
- 値のオプション：`"none"`、`"gzip"`

### `commit-timeout`

- トランザクションのコミットの最大タイムアウト時間。
- デフォルト値：`41s`
- この値はRaftの選挙タイムアウト時間の2倍よりも大きく設定する必要があります。

### `max-batch-size`

- バッチで送信されるRPCパケットの最大数。値が`0`でない場合、`BatchCommands` APIが使用されてTiKVにリクエストが送信され、高い並行性の場合にRPCのレイテンシを低減できます。この値を変更しないことをお勧めします。
- デフォルト値：`128`

### `max-batch-wait-time`

- `max-batch-wait-time`まで待機して、データパケットをまとめて大きなパケットにカプセル化し、TiKVノードに送信する。`tikv-client.max-batch-size`の値が`0`より大きい場合にのみ有効です。この値を変更しないことをお勧めします。
- デフォルト値：`0`
- 単位：ナノ秒

### `batch-wait-size`

- バッチでTiKVに送信されるパケットの最大数。この値を変更しないことをお勧めします。
- デフォルト値：`8`
- 値が`0`の場合、この機能は無効になります。

### `overload-threshold`

- TiKV負荷の閾値。TiKV負荷がこの閾値を超えると、TiKVの負荷を緩和するためにより多くの`batch`パケットが収集されます。`tikv-client.max-batch-size`の値が`0`より大きい場合にのみ有効です。この値を変更しないことをお勧めします。
- デフォルト値：`200`

## tikv-client.copr-cache <span class="version-mark">v4.0.0で新規</span>

このセクションは、コプロセッサキャッシュ機能に関連する構成項目を紹介します。

### `capacity-mb`

- キャッシュされたデータの合計サイズ。キャッシュスペースがいっぱいになると、古いキャッシュエントリが削除されます。値が `0.0` の場合、コプロセッサキャッシュ機能は無効になります。
- デフォルト値：`1000.0`
- 単位：MB
- タイプ：浮動小数点数

## txn-local-latches

ローカルトランザクションラッチに関連する構成です。多くのローカルトランザクション競合が発生する場合は、有効にすることをお勧めします。

### `enabled`

- トランザクションのメモリロックを有効にするかどうかを決定します。
- デフォルト値：`false`

### `capacity`

- Hashに対応するスロットの数で、指数的な2の倍数に自動調整されます。各スロットはメモリの32バイトを占めます。値を小さく設定すると、比較的大きな範囲（例: データのインポートなど）をカバーするデータ書き込みのシナリオで、実行速度が遅くなりパフォーマンスが劣化する可能性があります。
- デフォルト値：`2048000`

## binlog

TiDBバイナリログに関連する構成。

### `enable`

- バイナリログを有効または無効にします。
- デフォルト値：`false`

### `write-timeout`

- バイナリログをPumpに書き込むタイムアウト時間。この値を変更することはお勧めしません。
- デフォルト：`15s`
- 単位：秒

### `ignore-error`

- Pumpにバイナリログを書き込むプロセスでエラーが発生した場合に無視するかどうかを決定します。この値を変更することはお勧めしません。
- デフォルト値：`false`
- 値が`true`に設定されている場合、エラーが発生すると、TiDBはバイナリログの書き込みを停止し、`tidb_server_critical_error_total`モニタリングアイテムのカウントに`1`を加えます。`false`に設定されている場合は、バイナリログの書き込みに失敗し、TiDBサービス全体が停止します。

### `binlog-socket`

- バイナリログがエクスポートされるネットワークアドレス。
- デフォルト値：`""`

### `strategy`

- バイナリログのエクスポート時にPumpの選択方法を指定します。現在、`hash`および`range`メソッドのみがサポートされています。
- デフォルト値：`range`

## status

TiDBサービスのステータスに関連する構成。

### `report-status`

- HTTP APIサービスを有効または無効にします。
- デフォルト値：`true`

### `record-db-qps`

- データベース関連のQPSメトリクスをPrometheusに転送するかどうかを決定します。
- デフォルト値：`false`

## pessimistic-txn

悲観的トランザクションの使用に関連する設定。詳細については[TiDB悲観的トランザクションモード](/pessimistic-transaction.md)を参照してください。

### max-retry-count

- 悲観的トランザクション内の各ステートメントの最大リトライ回数。リトライ回数がこの制限を超えると、エラーが発生します。
- デフォルト値：`256`

### deadlock-history-capacity

+ 各TiDBサーバーの[`INFORMATION_SCHEMA.DEADLOCKS`](/information-schema/information-schema-deadlocks.md)テーブルに記録できるデッドロックイベントの最大数。このテーブルがいっぱいになり、新しいデッドロックイベントが発生すると、テーブル内の最も古いレコードが削除されて最新のエラーのために場所が作られます。
+ デフォルト値：`10`
+ 最小値：`0`
+ 最大値：`10000`

### deadlock-history-collect-retryable

+ [`INFORMATION_SCHEMA.DEADLOCKS`](/information-schema/information-schema-deadlocks.md)テーブルがリトライ可能なデッドロックエラーの情報を収集するかどうかを制御します。リトライ可能なデッドロックエラーの説明については、[リトライ可能なデッドロックエラー](/information-schema/information-schema-deadlocks.md#retryable-deadlock-errors)を参照してください。
+ デフォルト値：`false`

### pessimistic-auto-commit <span class="version-mark">v6.0.0で新規</span>

+ グローバルで悲観的トランザクションモードが有効になっている場合 (`tidb_txn_mode='pessimistic'`)、自動コミットトランザクションが悲観的トランザクションモードを使用するかどうかを決定する。デフォルトでは、悲観的トランザクションモードがグローバルで有効になっていても、自動コミットトランザクションは依然として楽観的トランザクションモードを使用します。`pessimistic-auto-commit` ( `true`に設定)を有効にした場合、自動コミットトランザクションも悲観的モードを使用し、他の明示的にコミットされた悲観的トランザクションと一貫性が取れます。
+ 競合が発生するシナリオでは、この設定を有効にすると、TiDBは自動コミットトランザクションをグローバルなロック待ち管理に含めるため、デッドロックを回避し、デッドロックによって引き起こされる遅延スパイクを和らげることができます。
+ 競合が発生しないシナリオでは、自動コミットトランザクションが多数あり（特定の数は実際のシナリオによって決まります。たとえば、自動コミットトランザクションの数がアプリケーションの総数の半分以上を占める場合）、単一トランザクションで大容量のデータを操作する場合、この設定を有効にすると性能が低下する可能性があります。例: 自動コミットの`INSERT INTO SELECT`文。
+ デフォルト値：`false`

### constraint-check-in-place-pessimistic <span class="version-mark">v6.4.0で新規</span>

+ システム変数[`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630)のデフォルト値を制御します。
+ デフォルト値：`true`

## isolation-read

読み取りアイソレーションに関連する構成項目。

### `engines`

- TiDBがデータを読み取ることを許可するエンジンを制御します。
- デフォルト値：["tikv", "tiflash", "tidb"]、つまりエンジンはオプティマイザによって自動的に選択されます。
- 値のオプション："tikv"、"tiflash"、"tidb"の任意の組み合わせ。例: ["tikv", "tidb"]または["tiflash", "tidb"]

## instance

### `tidb_enable_collect_execution_info`

- スロークエリログに各オペレータの実行情報を記録するかどうかを制御します。
- デフォルト値：`true`
- v6.1.0以前、この構成は`enable-collect-execution-info`によって設定されていました。

### `tidb_enable_slow_log`

- この構成は遅いクエリのフィーチャを有効にするかどうかを制御するために使用されます。
- デフォルト値: `true`
- 値のオプション: `true` または `false`
- v6.1.0以前、この構成は`enable-slow-log`によって設定されていました。

### `tidb_slow_log_threshold`

- この構成は遅いクエリによって消費された時間の閾値値を出力するために使用されます。クエリによって消費された時間がこの値よりも大きい場合、このクエリは遅いクエリと見なされ、そのログは遅いクエリログに出力されます。
- デフォルト値: `300`
- レンジ: `[-1, 9223372036854775807]`
- 単位: ミリ秒
- v6.1.0以前、この構成は`slow-threshold`によって設定されていました。

### `in-mem-slow-query-topn-num` <span class="version-mark">v7.3.0で新規追加</span>

+ この構成はメモリ内にキャッシュされる最も遅いクエリの数を制御します。
+ デフォルト値: 30

### `in-mem-slow-query-recent-num` <span class="version-mark">v7.3.0で新規追加</span>

+ この構成はメモリ内にキャッシュされる最近使用された遅いクエリの数を制御します。
+ デフォルト値: 500

### `tidb_expensive_query_time_threshold`

- この構成は高コストなクエリログを出力するかどうかを設定するために使用されます。高コストなクエリログと遅いクエリログの違いは以下の通りです:
    - 遅いログは文が実行された後に出力されます。
    - 高コストなクエリログは、閾値値を超える実行時間を持つ実行中の文とその関連情報を出力します。
- デフォルト値: `60`
- レンジ: `[10, 2147483647]`
- 単位: 秒
- v5.4.0以前、この構成は`expensive-threshold`によって設定されていました。

### `tidb_record_plan_in_slow_log`

- この構成は遅いクエリログに遅いクエリの実行計画を含めるかどうかを制御するために使用されます。
- デフォルト値: `1`
- 値のオプション: `1` (有効、デフォルト) または `0` (無効)。
- この構成の値は、システム変数[`tidb_record_plan_in_slow_log`](/system-variables.md#tidb_record_plan_in_slow_log)の値を初期化します。
- v6.1.0以前、この構成は`record-plan-in-slow-log`によって設定されていました。

### `tidb_force_priority`

- この構成は、TiDBサーバーで実行される文のデフォルト優先度を変更するために使用されます。
- デフォルト値: `NO_PRIORITY`
- デフォルト値`NO_PRIORITY`は、文の優先度が強制的に変更されないことを意味します。他のオプションは、昇順に`LOW_PRIORITY`、`DELAYED`、`HIGH_PRIORITY`です。
- v6.1.0以前、この構成は`force-priority`によって設定されていました。

### `max_connections`

- 単一のTiDBインスタンスに許可される最大接続数です。リソース制御に使用できます。
- デフォルト値: `0`
- レンジ: `[0, 100000]`
- デフォルト値`0`は制限がないことを意味します。この変数の値が`0`よりも大きい場合、そして接続数がその値に達した場合、TiDBサーバーはクライアントからの新しい接続を拒否します。
- この構成の値は、システム変数[`max_connections`](/system-variables.md#max_connections)の値を初期化します。
- v6.2.0以前、この構成は`max-server-connections`によって設定されていました。

### `tidb_enable_ddl`

- この構成は、対応するTiDBインスタンスがDDL所有者になるかどうかを制御します。
- デフォルト値: `true`
- 可能な値: `OFF`、`ON`
- この構成の値は、システム変数[`tidb_enable_ddl`](/system-variables.md#tidb_enable_ddl-new-in-v630)を初期化します。
- v6.3.0以前、この構成は`run-ddl`によって設定されていました。

### `tidb_stmt_summary_enable_persistent` <span class="version-mark">v6.6.0で新規追加</span>

> **警告:**
>
> ステートメントサマリの永続性は実験的な機能です。本番環境での使用は推奨されません。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ ステートメントサマリの永続性を有効にするかどうかを制御します。
+ デフォルト値: `false`
+ 詳細については、[ステートメントサマリの永続化](/statement-summary-tables.md#persist-statements-summary)を参照してください。

### `tidb_stmt_summary_filename` <span class="version-mark">v6.6.0で新規追加</span>

> **警告:**
>
> ステートメントサマリの永続性は実験的な機能です。本番環境での使用は推奨されません。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ ステートメントサマリの永続性が有効にされている場合、この構成は永続データが書き込まれるファイルを指定します。
+ デフォルト値: `tidb-statements.log`

### `tidb_stmt_summary_file_max_days` <span class="version-mark">v6.6.0で新規追加</span>

> **警告:**
>
> ステートメントサマリの永続性は実験的な機能です。本番環境での使用は推奨されません。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ ステートメントサマリの永続性が有効にされている場合、この構成は永続データファイルを保持する最大日数を指定します。
+ デフォルト値: `3`
+ 単位: day
+ データの保持要件とディスクスペースの使用状況に基づいて値を調整できます。

### `tidb_stmt_summary_file_max_size` <span class="version-mark">v6.6.0で新規追加</span>

> **警告:**
>
> ステートメントサマリの永続性は実験的な機能です。本番環境での使用は推奨されません。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ ステートメントサマリの永続性が有効にされている場合、この構成は永続データファイルの最大サイズを指定します。
+ デフォルト値: `64`
+ 単位: MiB
+ データの保持要件とディスクスペースの使用状況に基づいて値を調整できます。

### `tidb_stmt_summary_file_max_backups` <span class="version-mark">v6.6.0で新規追加</span>

> **警告:**
>
> ステートメントサマリの永続性は実験的な機能です。本番環境での使用は推奨されません。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ ステートメントサマリの永続性が有効にされている場合、この構成は永続化できるデータファイルの最大数を指定します。`0`はファイル数に制限がないことを意味します。
+ デフォルト値: `0`
+ データの保持要件とディスクスペースの使用状況に基づいて値を調整できます。

## proxy-protocol

PROXYプロトコルに関連する構成項目。

### `networks`

- PROXYプロトコルを使用してTiDBに接続を許可するプロキシサーバのIPアドレスのリストです。 [PROXYプロトコル](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)を参照してください。
- デフォルト値: ""
- 一般的なケースでは、リバースプロキシの背後にあるTiDBにアクセスする場合、TiDBはリバースプロキシサーバのIPアドレスをクライアントのIPアドレスとして取ります。PROXYプロトコルを有効にすることで、このプロトコルをサポートするHAProxyなどのリバースプロキシは実際のクライアントIPアドレスをTiDBに渡すことができます。
- このパラメータを構成した後、TiDBは構成されたソースIPアドレスがPROXYプロトコルを使用してTiDBに接続することを許可します。PROXY以外のプロトコルを使用した場合、この接続は拒否されます。このパラメータが空の場合、IPアドレスはPROXYプロトコルを使用してTiDBに接続することはできません。値はIPアドレス（192.168.1.50）またはCIDR（192.168.1.0/24）で、セパレーターとして`,`を使用します。`*`は任意のIPアドレスを意味します。

> **警告:**
>
> `*`を使用する際は注意してください。これにより、任意のIPアドレスのクライアントが自身のIPアドレスを報告することでセキュリティリスクが発生する可能性があります。さらに、`*`を使用すると、TiDBに直接接続する内部のコンポーネント（TiDB Dashboardなど）が利用できなくなる可能性があります。

### `fallbackable` <span class="version-mark">v6.5.1で新規追加</span>

+ PROXYプロトコルのフォールバックモードを有効にするかどうかを制御します。この構成項目が`true`に設定されている場合、TiDBは`proxy-protocol.networks`に属するクライアントがPROXYプロトコル仕様を使用せずにTiDBに接続した場合、またはPROXYプロトコルヘッダーを送信しない場合に受け入れることができます。デフォルトでは、TiDBは`proxy-protocol.networks`に属するクライアントの接続かつPROXYプロトコルヘッダーを送信するクライアント接続のみを受け入れます。
+ デフォルト値: `false`
```
      + {R}
      + {R}
    + {R}
  + {R}
```

```
      + {T}
      + {T}
    + {T}
  + {T}
```