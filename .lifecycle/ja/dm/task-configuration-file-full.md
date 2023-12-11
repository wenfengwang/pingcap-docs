---
title: DMの高度なタスク構成ファイル
aliases: ['/docs/tidb-data-migration/dev/task-configuration-file-full/','/docs/tidb-data-migration/dev/dm-portal/']
---

# DMの高度なタスク構成ファイル

このドキュメントではData Migration（DM）の高度なタスク構成ファイルについて紹介し、[グローバル構成](#global-configuration)と[インスタンス構成](#instance-configuration)を含みます。

## 重要な概念

`source-id`、DMワーカーIDなどの重要な概念については、[重要な概念](/dm/dm-config-overview.md#important-concepts)を参照してください。

## タスク構成ファイルのテンプレート（高度）

以下は**高度な**データマイグレーションタスクを実行するためのタスク構成ファイルのテンプレートです。

```yaml
---

# ----------- グローバル構成 -----------
## ********* 基本構成 *********
name: test                          # タスクの名前。グローバルで一意である必要があります。
task-mode: all                      # タスクモード。`full`（完全なデータのみをマイグレーションする）/ `incremental`（binlogを同期的に複製する）/ `all`（完全なデータと増分のbinlogの両方を複製する）に設定できます。
shard-mode: "pessimistic"           # シャードのマージモード。オプションのモードは`""` / `"pessimistic"` / `"optimistic"`です。`""`モードはデフォルトであり、つまりシャーディングDDLマージは無効になります。タスクがシャードマージタスクである場合は、`"pessimistic"`モードに設定してください。
                                    # "optimistic"モードの原理と制約を理解した上で設定してください。
strict-optimistic-shard-mode: false  # "optimistic"モードでのみ有効です。この構成は"optimistic"モードの動作を制限します。デフォルト値はfalseです。v7.2.0で導入されました。詳細については、こちらを参照してください https://docs.pingcap.com/tidb/v7.2/feature-shard-merge-optimistic
meta-schema: "dm_meta"               # `meta`情報を格納する下流データベース。
timezone: "Asia/Shanghai"            # SQLセッションで使用するタイムゾーン。デフォルトでは、DMはターゲットクラスタのグローバルなタイムゾーン設定を使用し、これにより自動的に正確性が確保されます。カスタマイズされたタイムゾーンはデータマイグレーションには影響しませんが、不要です。
case-sensitive: false                # スキーマ/テーブルが大文字と小文字を区別するかどうかを決定します。
online-ddl: true                     # 上流の"gh-ost"と"pt"の自動処理をサポートします。
online-ddl-scheme: "gh-ost"          # `online-ddl-scheme`は非推奨ですので、`online-ddl`を使用することをお勧めします。
clean-dump-file: true                # データダンプ中に生成されたファイルをクリーンアップするかどうか。これには`metadata`ファイルも含まれます。
collation_compatible: "loose"        # `CREATE` SQLステートメントでデフォルトの照合順序を同期するモード。サポートされる値は`"loose"`（デフォルト）または`"strict"`です。値が`"strict"`の場合、DMはSQLステートメントに対して対応する照合順序を明示的に追加します。 値が`"loose"`の場合、DMはSQLステートメントを変更しません。`"strict"`モードでは、下流が上流のデフォルトの照合順序をサポートしていない場合、下流でエラーが発生する可能性があります。
ignore-checking-items: []             # 無視可能なチェック項目。無視可能なチェック項目の完全なリストについては、DM事前チェックを参照してください：https://docs.pingcap.com/tidb/stable/dm-precheck#ignorable-checking-items。

target-database:                      # 下流データベースインスタンスの構成。
  host: "192.168.0.1"
  port: 4000
  user: "root"
  password: "/Q7B9DizNLLTTfiZHv9WoEAKamfpIUs="  # `dmctl encrypt`で暗号化されたパスワードの使用をお勧めします。
  max-allowed-packet: 67108864                  # DMが内部的にTiDBサーバに接続する際のTiDBクライアント（つまり、最大受け入れパケットの制限）を設定します。単位はバイトです（デフォルトでは67108864）。
                                                # DM v2.0.0以降、この構成項目は非推奨となり、DMは自動的にTiDBから`max_allowed_packet`の値を取得します。
  session:                                       # TiDBのセッション変数。v1.0.6以降でサポートされています。詳細については、こちらを参照してください`https://pingcap.com/docs/stable/system-variables`.
    sql_mode: "ANSI_QUOTES,NO_ZERO_IN_DATE,NO_ZERO_DATE" # DM v2.0.0以降、この項目が構成ファイルに表示されない場合、DMは自動的に下流TiDBから`sql_mode`の適切な値を取得します。この項目の手動構成はより優先されます。
    tidb_skip_utf8_check: 1                     # DM v2.0.0以降、この項目が構成ファイルに表示されない場合、DMは自動的に下流TiDBから`tidb_skip_utf8_check`の適切な値を取得します。この項目の手動構成はより優先されます。
    tidb_constraint_check_in_place: 0
  security:                       # 下流TiDBのTLS構成
    ssl-ca: "/path/to/ca.pem"
    ssl-cert: "/path/to/cert.pem"
    ssl-key: "/path/to/key.pem"


## ******** 機能構成セット **********
# 上流と下流のテーブル間のルーティングマッピングルールセット。
routes:
  route-rule-1:                   # ルーティングマッピングルールの名前。
    schema-pattern: "test_*"      # 上流スキーマ名のパターン。ワイルドカード文字（*?）がサポートされています。
    table-pattern: "t_*"          # 上流テーブル名のパターン。ワイルドカード文字（*?）がサポートされています。
    target-schema: "test"         # 下流スキーマの名前。
    target-table: "t"             # 下流テーブルの名前。
    # オプション。シャードされたスキーマとテーブルのソース情報を抽出し、その情報を下流のユーザー定義列に書き込むために使用されます。これらのオプションを構成する場合は、下流でマージテーブルを手動で作成する必要があります。詳細については、<https://docs.pingcap.com/tidb/dev/dm-table-routing>のテーブルルーティングの説明を参照してください。
    # extract-table:                                        # t_の部分を除いたテーブル名のサフィックスをc-table列に抽出して書き込みます。例）テーブルt_01のために01を抽出してc-table列に書き込みます。
    #   table-regexp: "t_(.*)"
    #   target-column: "c_table"
    # extract-schema:                                       # test_の部分を除いたスキーマ名のサフィックスをc_schema列に抽出して書き込みます。例）スキーマtest_02のために02を抽出してc_schema列に書き込みます。
    #   schema-regexp: "test_(.*)"
    #   target-column: "c_schema"
    # extract-source:                                       # ソースインスタンス情報をc_source列に抽出して書き込みます。例）データソースmysql-replica-01のためにmysql-replica-01を抽出してc_source列に書き込みます。
    #   source-regexp: "(.*)"
    #   target-column: "c_source"
  route-rule-2:
    schema-pattern: "test_*"
    target-schema: "test"

# 上流データベースインスタンスの一致するテーブルのbinlogイベントフィルタルールセット。
filters:
  filter-rule-1:                             # フィルタリングルールの名前。
    schema-pattern: "test_*"                 # 上流スキーマ名のパターン。ワイルドカード文字（*?）がサポートされています。
    table-pattern: "t_*"                     # 上流スキーマ名のパターン。ワイルドカード文字（*?）がサポートされています。
    events: ["truncate table", "drop table"] # 一致するイベントタイプ。
    action: Ignore                           # フィルタリングルールに一致するbinlogをマイグレーション（Do）するか無視（Ignore）するか。
  filter-rule-2:
    schema-pattern: "test_*"
    events: ["all dml"]
    action: Do

expression-filter:                   # データをマイグレーションする際の行の変更のためのフィルタールールを定義します。複数のルールを定義することができます。
  # `expr_filter`.`tbl`に挿入された`c`の値をフィルタリングし、偶数の場合にフィルタリングします。
  even_c:                            # フィルタールールの名前。
    schema: "expr_filter"            # 一致させる上流データベースの名前。ワイルドカード一致または正規一致はサポートされていません。
    table: "tbl"                     # 一致させる上流テーブルの名前。ワイルドカード一致または正規一致はサポートされていません。
    insert-value-expr: "c % 2 = 0"

# 上流データベースインスタンスからマイグレーションするテーブルのフィルタールールセット。同時に複数のルールを設定できます。
block-allow-list:                    # DMバージョンが`v2.0.0-beta.2`以下の場合はblack-white-listを使用します。
  bw-rule-1:                         # ブロック許可リストのルール名。
    do-dbs: ["~^test.*", "user"]     # マイグレーションする必要がある上流スキーマの許可リスト。
    ignore-dbs: ["mysql", "account"] # マイグレーションする必要がある上流スキーマのブロックリスト。
    do-tables:                       # マイグレーションする必要がある上流テーブルの許可リスト。
    - db-name: "~^test.*"
      tbl-name: "~^t.*"
    - db-name: "user"
      tbl-name: "information"
  bw-rule-2:                         # ブロック許可リストのルール名。
    ignore-tables:                   # Upstreamテーブルのブロックリストを移行する必要があります。
    - db-name: "user"
      tbl-name: "log"

# ダンプ処理ユニットの構成引数。
mydumpers:
  global:                            # 処理ユニットの構成の名前。
    threads: 4                       # ダンプ処理ユニットが事前チェックを実行し、アップストリームからデータをエクスポートする際にアクセスするスレッドの数（デフォルトは4）
    chunk-filesize: 64               # ダンプ処理ユニットが生成するファイルのサイズ（デフォルトは64 MB）。
    extra-args: "--consistency none" # ダンプ処理ユニットのその他の引数。`extra-args` でtable-listを手動で設定する必要はありません。これはDMによって自動生成されます。

# ロード処理ユニットの構成引数。
loaders:
  global:                            # 処理ユニットの構成の名前。
    pool-size: 16                    # ロード処理ユニットでダンプされたSQLファイルを並行して実行するスレッドの数（デフォルトは16）。複数のインスタンスが同時にTiDBにデータを移行している場合は、負荷に応じて値をわずかに減らすことをお勧めします。
    # アップストリームからエクスポートされた完全なデータを格納するディレクトリ（デフォルトは "./dumped_data"）。
    # ローカルのファイルシステムパスまたはAmazon S3パスをサポートします。例："s3://dm_bucket/dumped_data?endpoint=s3-website.us-east-2.amazonaws.com&access_key=s3accesskey&secret_access_key=s3secretkey&force_path_style=true"
    dir: "./dumped_data"

    # 完全インポートフェーズ中のインポートモード。以下のモードがサポートされています：
    # - "logical"（デフォルト）。TiDB Lightningの論理インポートモードを使用してデータをインポートします。ドキュメント：https://docs.pingcap.com/tidb/stable/tidb-lightning-logical-import-mode
    # - "physical"。TiDB Lightningの物理インポートモードを使用してデータをインポートします。ドキュメント：https://docs.pingcap.com/zh/tidb/stable/tidb-lightning-physical-import-mode
    #   "physical"モードはまだ実験的な機能であり、本番環境では推奨されません。
    import-mode: "logical"
    # 論理インポートで競合を解決するためのメソッド。
    # - "replace"（デフォルト）。新しいデータを使って既存のデータを置き換えます。
    # - "ignore"。既存のデータを保持し、新しいデータを無視します。
    # - "error"。重複するデータの挿入でエラーを報告し、その後レプリケーションタスクを停止します。
    on-duplicate-logical: "replace"

    # 物理インポートで競合を解決するためのメソッド。
    # - "none"。TiDB Lightningの物理インポートでの競合検出の "none" 戦略に対応します。
    #   (https://docs.pingcap.com/tidb/stable/tidb-lightning-physical-import-mode-usage#conflict-detection)。
    #   このメソッドでは競合するデータは解決されません。 "none" は最もパフォーマンスが良いですが、
    #   ダウンストリームのデータが不整合になる可能性があります。
    # - "manual"。TiDB Lightningの物理インポートでの競合検出の "remove" 戦略に対応します。
    #   (https://docs.pingcap.com/tidb/stable/tidb-lightning-physical-import-mode-usage#conflict-detection)。
    #   インポートが競合すると、DMは対象テーブルから競合するレコードをすべて削除し、データを
    #   `${meta-schema}_${name}.conflict_error_v1` テーブルに記録します。この構成ファイルでは、競合する
    #   データは `dm_meta_test.conflict_error_v1` テーブルに記録されます。完全なインポートフェーズが完了したら、
    #   タスクが一時停止し、このテーブルをクエリして競合を手動で解決する必要があります。タスクを再開し、`resume-task` コマンドを使って増分フェーズに入る必要があります。
    on-duplicate-physical: "none"
    # 物理インポートモードで使用されるローカルKVソートのディレクトリ。この構成のデフォルト値は `dir` 構成と同じです。
    # 詳細は、TiDB Lightningドキュメントを参照してください：https://docs.pingcap.com/tidb/stable/tidb-lightning-physical-import-mode#environment-requirements
    sorting-dir-physical: "./dumped_data"
    # ディスククォータ。TiDB Lightningのディスククォータ構成に対応します。詳細については、TiDB Lightningドキュメントを参照してください：https://docs.pingcap.com/tidb/stable/tidb-lightning-physical-import-mode-usage#configure-disk-quota-new-in-v620
    disk-quota-physical: "0"
    # DMはインポート後に各テーブルで `ADMIN CHECKSUM TABLE <table>` を実行してデータの整合性を検証します。
    # - "required"（デフォルト）：インポート後にadmin checksumを実行します。チェックサムに失敗した場合、
    #   DMはタスクを一時停止し、ユーザーが手動で処理する必要があります。
    # - "optional"：インポート後にadmin checksumを実行します。チェックサムに失敗した場合、DMは警告をログに記録し、
    #   データの移行を継続します。タスクは一時停止しません。
    # - "off"：インポート後にadmin checksumを実行しません。
    # チェックサムに失敗した場合、インポートは異常終了します。つまり、データが整合性がないか失われていることを意味します。
    # したがって、常にチェックサムを有効にすることをお勧めします。
    checksum-physical: "required"
    # 物理インポートの場合にのみ利用可能です。チェックサムプロセスが完了したら、各テーブルの `ANALYZE TABLE <table>` 操作を実行するかどうかを決定します。
    # - "required"（デフォルト）。インポートが完了した後、分析操作が実行されることを示します。分析が失敗した場合、
    #   タスクは一時停止し、ユーザーによる手動処理が必要です。
    # - "optional"。データが完了した後に分析を行います。分析が失敗した場合、警告ログが出力され、タスクは一時停止しません。
    # - "off"。データが完了した後に分析を行いません。
    # 分析は統計データのみに影響し、ほとんどのシナリオではAnalyzeをオフにすることをお勧めします。
    analyze: "off"
    # 物理インポートの場合にのみ利用可能です。TiKVへのKVデータの送信の並列度。dm-workerとTiKV間の直接ネットワーク転送速度が10,000 Mb/sを超える場合に増やすことができます。
    # range-concurrency: 16
    # 物理インポートモードでのみ利用可能です。TiKVへのKVデータの送信時に圧縮を有効にするかどうかを決定します。現在はGzip圧縮のみがサポートされており、"gzip"または"gz"を使用して指定できます。デフォルトでは圧縮は有効になっていません。
    # compress-kv-pairs: ""
    # 1つ以上のPDサーバーアドレス。アドレスが指定されていない場合、デフォルトでTiDBのクエリからPDアドレス情報を使用します。
    # pd-addr: "192.168.0.1:2379"

# 同期処理ユニットの構成引数。
syncers:
  global:                            # 処理ユニットの構成の名前。
    worker-count: 16                 # ローカルに転送されたbinlogを適用する並行スレッド数（デフォルトは16）。このパラメータの調整は、アップストリームのログの取得の並列度に影響を与えませんが、ダウンストリームのデータベースには大きな影響を与えます。
    batch: 100                       # 同期処理ユニットがダウンストリームデータベースに複製するトランザクションバッチ内のSQLステートメントの数（デフォルトは100）。通常、この値は500未満に設定することをお勧めします。
    enable-ansi-quotes: true         # `sql-mode: "ANSI_QUOTES"` が `session` で設定されている場合、この引数を有効にします。

    # trueに設定すると、アップストリームからの `INSERT` ステートメントは `REPLACE` ステートメントに書き換えられ、`UPDATE` ステートメントは `DELETE` および `REPLACE` ステートメントに書き換えられます。これにより、テーブルスキーマにプライマリキーまたはユニークインデックスがある場合でも、データ移行中にDMLステートメントを繰り返しインポートできるようになります。
    safe-mode: false
    # 自動セーフモードの期間。
    # この値が設定されていないか、 "" に設定されている場合、デフォルト値は `checkpoint-flush-interval` の2倍（デフォルトは30秒）である60秒になります。
    # この値が "0s" に設定されている場合、DMは自動的にセーフモードに入るときにエラーを報告します。
    # この値が "1m30s" のような正常な値に設定されている場合、タスクが異常終了した場合や、DMがsafemode_exit_pointを記録できなかった場合、あるいはDMが予期せず終了した場合、自動セーフモードの期間は1分30秒に設定されます。
    safe-mode-duration: "60s"

    # trueに設定すると、DMは可能な限り同じ行の複数のアップストリームステートメントを1つのステートメントにコンパクトにしますが、遅延を増加させません。
    # たとえば、 `INSERT INTO tb(a,b) VALUES(1,1); UPDATE tb SET b=11 WHERE a=1`;` は `INSERT INTO tb(a,b) VALUES(1,11);` に
    # コンパクトされます。「a」がプライマリキーの場合
    # `UPDATE tb SET b=1 WHERE a=1; UPDATE tb(a,b) SET b=2 WHERE a=1;` は `UPDATE tb(a,b) SET b=2 WHERE a=1;` にコンパクトされます。 「a」がプライマリキーの場合
    # `DELETE FROM tb WHERE a=1; INSERT INTO tb(a,b) VALUES(1,1);` は `REPLACE INTO tb(a,b) VALUES(1,1);` にコンパクトされます。 「a」がプライマリキーの場合
    compact: false
```yaml
    multiple-rows: true

validators:
  global:
    mode: full
    worker-count: 4
    row-error-delay: 30m

mysql-instances:
  -
    source-id: "mysql-replica-01"
    meta:
      binlog-name: binlog.000001
      binlog-pos: 4
      binlog-gtid: "03fc0263-28c7-11e7-a653-6c0b84d59f30:1-7041423,05474d3c-28c7-11e7-8352-203db246dd3d:1-170"
    route-rules: ["route-rule-1", "route-rule-2"]
    filter-rules: ["filter-rule-1", "filter-rule-2"]
    block-allow-list: "bw-rule-1"
    expression-filters: ["even_c"]
    mydumper-config-name: "global"
    loader-config-name: "global"
    syncer-config-name: "global"
    validator-config-name: "global"
  -
    source-id: "mysql-replica-02"
    mydumper-thread: 4
    loader-thread: 16
    syncer-thread: 16
```