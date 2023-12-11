---
title: TiKV コントロールユーザーガイド
summary: TiKV クラスタを管理するために TiKV コントロールを使用します。
aliases: ['/docs/dev/tikv-control/','/docs/dev/reference/tools/tikv-control/']
---

# TiKV コントロールユーザーガイド

TiKV コントロール（`tikv-ctl`）は TiKV のコマンドラインツールであり、クラスタを管理するために使用されます。そのインストールディレクトリは以下の通りです：

* クラスタが TiUP を使用して展開されている場合は、`tikv-ctl` ディレクトリは `~/.tiup/components/ctl/{VERSION}/` ディレクトリにあります。

## TiUP で TiKV コントロールを使用する

> **注意:**
>
> 使用するコントロールツールのバージョンはクラスタのバージョンと一致することをお勧めします。

`tiup` コマンドにも `tikv-ctl` が統合されています。次のコマンドを実行して `tikv-ctl` ツールを呼び出します:

```shell
tiup ctl:v<CLUSTER_VERSION> tikv
```

```
Starting component `ctl`: /home/tidb/.tiup/components/ctl/v4.0.8/ctl tikv
TiKV Control (tikv-ctl)
Release Version:   4.0.8
Edition:           Community
Git Commit Hash:   83091173e960e5a0f5f417e921a0801d2f6635ae
Git Commit Branch: heads/refs/tags/v4.0.8
UTC Build Time:    2020-10-30 08:40:33
Rust Version:      rustc 1.42.0-nightly (0de96d37f 2019-12-19)
Enable Features:   jemalloc mem-profiling portable sse protobuf-codec
Profile:           dist_release

TiKV デプロイメントとやり取りするためのツールです。
使用法:
    TiKV コントロール（tikv-ctl）[FLAGS] [OPTIONS] [SUBCOMMAND]
FLAGS:
    -h、--help                    ヘルプ情報を表示
        --skip-paranoid-checks    rocksdb を開くときにパランオイドチェックをスキップ
    -V、--version                 バージョン情報を表示
OPTIONS:
        --ca-path <ca-path>              CA 証明書パスを設定
        --cert-path <cert-path>          証明書パスを設定
        --config <config>                TiKV 設定パス、デフォルトでは <deploy-dir>/conf/tikv.toml です
        --data-dir <data-dir>            TiKV データディレクトリパス、これは <deploy-dir>/scripts/run.sh を確認して取得します
        --decode <decode>                エスケープ形式のキーをデコード
        --encode <encode>                エスケープ形式のキーをエンコード
        --to-hex <escaped-to-hex>        エスケープされたキーを 16 進キーに変換
        --to-escaped <hex-to-escaped>    16 進キーをエスケープされたキーに変換
        --host <host>                    リモートホストを設定
        --key-path <key-path>            プライベートキーパスを設定
        --log-level <log-level>          ログレベルを設定 [default: warn]
        --pd <pd>                        pd のアドレスを設定
サブコマンド:
    bad-regions           すべての破損した raft を持つリージョンを取得
    cluster               クラスタ ID を印刷
    compact               指定された範囲内の列ファミリをコンパクトにする
    compact-cluster       一つまたは複数の列ファミリで指定された範囲内のクラスタ全体をコンパクトにする
    consistency-check     指定されたリージョンで強制的に整合性をチェック
    decrypt-file          暗号化されたファイルを復号化
    diff                  異なる dbs でリージョンキーの差分を計算
    dump-snap-meta        snapshot メタファイルをダンプ
    encryption-meta       暗号化メタデータをダンプ
    fail                  TiKV に障害を注入して復旧
    help                  このメッセージを印刷するか、指定されたサブコマンドのヘルプを印刷
    metrics               メトリクスを印刷
    modify-tikv-config    tikv の構成を変更、例: tikv-ctl --host ip:port modify-tikv-config -n
                          rocksdb.defaultcf.disable-auto-compactions -v true
    mvcc                  mvcc 値を印刷
    print                 生の値を印刷
    raft                  raft ログエントリを印刷
    raw-scan              範囲内のすべての生のキーを印刷
    recover-mvcc          破損したキーを削除してノード上の mvcc データを復旧
    recreate-region       指定されたメタデータでリージョンを再作成し、そのための新しい ID を割り当てる
    region-properties     リージョンプロパティを表示
    scan                  range db 範囲を印刷
    size                  リージョンサイズを印刷
    split-region          リージョンを分割
    store                 ストア ID を印刷
    tombstone             ノード上の一部のリージョンを手動で tombstone に設定
    unsafe-recover        ほとんどのレプリカが失敗したときにクラスタを非安全に復旧
```

`tiup ctl:v<CLUSTER_VERSION> tikv` の後に対応するパラメータやサブコマンドを追加できます。

## 一般オプション

`tikv-ctl` には次の 2 つの操作モードがあります：

- リモートモード: `--host` オプションを使用して TiKV のサービスアドレスを引数として受け入れます。

    このモードでは、TiKV で SSL が有効になっている場合、`tikv-ctl` は関連する証明書ファイルを指定する必要があります。例:

    ```shell
    tikv-ctl --ca-path ca.pem --cert-path client.pem --key-path client-key.pem --host 127.0.0.1:20160 <subcommands>
    ```

    ただし、`tikv-ctl` が TiKV の代わりに PD と通信する場合、`--host` の代わりに `--pd` オプションを使用する必要があります。例:

    ```shell
    tikv-ctl --pd 127.0.0.1:2379 compact-cluster
    ```

    ```
    store:"127.0.0.1:20160" compact db:KV cf:default range:([], []) success!
    ```

- ローカルモード:

    * `--data-dir` オプションを使用してローカルの TiKV データディレクトリパスを指定します。
    * `--config` オプションを使用してローカルの TiKV 設定ファイルパスを指定します。

  このモードでは、実行中の TiKV インスタンスを停止する必要があります。

特に指定がない場合、すべてのコマンドはリモートモードとローカルモードの両方をサポートしています。

さらに、`tikv-ctl` には `--to-hex` および `--to-escaped` の 2 つのシンプルなコマンドがあり、キーの形式を簡単に変更するために使用されます。

一般的に、キーの形式として `エスケープ` を使用します。例:

```shell
tikv-ctl --to-escaped 0xaaff
\252\377
tikv-ctl --to-hex "\252\377"
AAFF
```

> **注意:**
>
> コマンドラインで `エスケープ` 形式のキーを指定する場合は、必ず二重引用符で囲む必要があります。さもないと、bash はバックスラッシュを無視し、誤った結果が返されます。

## サブコマンド、一部のオプションおよびフラグ

このセクションでは、`tikv-ctl` が詳細にサポートするサブコマンドについて説明します。一部のサブコマンドは多くのオプションをサポートしています。詳細については、`tikv-ctl --help <subcommand>` を実行してください。

### Raft 状態機械の情報を表示

`raft` サブコマンドを使用して特定の時点で Raft 状態機械の状態を表示します。状態情報には 3 つの構造体（**RegionLocalState**、**RaftLocalState**、**RegionApplyState**）と特定のログの対応する Entries が含まれます。

上記の情報を取得するために `region` および `log` サブコマンドをそれぞれ使用します。両方のサブコマンドは、リモートモードとローカルモードの両方を同時にサポートしています。

`region` サブコマンドの場合:

- 表示するリージョンを指定するには、`-r` オプションを使用します。複数のリージョンは `,` で区切られます。すべてのリージョンを表示するには `--all-regions` オプションを使用します。ただし、`-r` と `--all-regions` は同時に使用できません。
- 印刷するリージョンの数を制限するには `--limit` オプションを使用します（デフォルトは `16`）。
- 特定のキー範囲に含まれるリージョンをクエリするには、`--start` と `--end` オプションを使用します（範囲制限なしの場合、16 進形式で）。

例えば、ID `1239` のリージョンを印刷するには、次のコマンドを使用します:

```shell
tikv-ctl --host 127.0.0.1:20160 raft region -r 1239
```

出力は次のようになります:

```
"region id": 1239
"region state": {
    id: 1239,
    start_key: 7480000000000000FF4E5F728000000000FF1443770000000000FA,
    end_key: 7480000000000000FF4E5F728000000000FF21C4420000000000FA,
    region_epoch: {conf_ver: 1 version: 43},
    peers: [ {id: 1240 store_id: 1 role: Voter} ]
}
"raft state": {
    hard_state {term: 8 vote: 5 commit: 7}
    last_index: 8)
}
"apply state": {
    applied_index: 8 commit_index: 8 commit_term: 8
    truncated_state {index: 5 term: 5}
}
```

特定のキー範囲に含まれるリージョンをクエリするには、次のコマンドを使用します:

- キーの範囲がRegionの範囲内にある場合、Region情報が出力されます。
- キーの範囲がRegionの範囲と同じである場合、たとえば、与えられたキーの範囲がRegion`1239`と同じである場合、Regionの範囲が左閉区間であり右開区間であるために、Region`1239`の`end_key`をRegion`1009`の`start_key`として取り、Region`1009`の情報も出力されます。

```shell
tikv-ctl --host 127.0.0.1:20160 raft region --start 7480000000000000FF4E5F728000000000FF1443770000000000FA --end 7480000000000000FF4E5F728000000000FF21C4420000000000FA
```

出力は以下の通りです:

```
"region state": {
    id: 1009
    start_key: 7480000000000000FF4E5F728000000000FF21C4420000000000FA,
    end_key: 7480000000000000FF5000000000000000F8,
    ...
}
"region state": {
    id: 1239
    start_key: 7480000000000000FF4E5F728000000000FF06C6D60000000000FA,
    end_key: 7480000000000000FF4E5F728000000000FF1443770000000000FA,
    ...
}
```

### Regionのサイズを表示

`size`コマンドを使用してRegionのサイズを表示します:

```shell
tikv-ctl --data-dir /path/to/tikv size -r 2
```

出力は以下の通りです:

```
region id: 2
cf default region size: 799.703 MB
cf write region size: 41.250 MB
cf lock region size: 27616
```

### 特定の範囲のMVCCを表示する

`scan`コマンドの`--from`および`--to`オプションは、2つのエスケープされた形式の生のキーを受け入れ、`--show-cf`フラグを使用して必要な列ファミリを指定します。

```shell
tikv-ctl --data-dir /path/to/tikv scan --from 'zm' --limit 2 --show-cf lock,default,write
```

```
key: zmBootstr\377a\377pKey\000\000\377\000\000\373\000\000\000\000\000\377\000\000s\000\000\000\000\000\372
         write cf value: start_ts: 399650102814441473 commit_ts: 399650102814441475 short_value: "20"
key: zmDB:29\000\000\377\000\374\000\000\000\000\000\000\377\000H\000\000\000\000\000\000\371
         write cf value: start_ts: 399650105239273474 commit_ts: 399650105239273475 short_value: "\000\000\000\000\000\000\000\002"
         write cf value: start_ts: 399650105199951882 commit_ts: 399650105213059076 short_value: "\000\000\000\000\000\000\000\001"
```

### 指定されたキーのMVCCを表示する

`scan`コマンドと同様に、`mvcc`コマンドを使用して指定されたキーのMVCCを表示できます。

```shell
tikv-ctl --data-dir /path/to/tikv mvcc -k "zmDB:29\000\000\377\000\374\000\000\000\000\000\000\377\000H\000\000\000\000\000\000\371" --show-cf=lock,write,default
```

```
key: zmDB:29\000\000\377\000\374\000\000\000\000\000\000\377\000H\000\000\000\000\000\000\371
         write cf value: start_ts: 399650105239273474 commit_ts: 399650105239273475 short_value: "\000\000\000\000\000\000\000\002"
         write cf value: start_ts: 399650105199951882 commit_ts: 399650105213059076 short_value: "\000\000\000\000\000\000\000\001"
```

このコマンドでは、キーも生のキーのエスケープされた形式です。

### 生のキーをスキャンする

`raw-scan`コマンドはRocksDBから直接スキャンします。データキーをスキャンするには、キーに`'z'`接頭辞を追加する必要があります。

`--from`および`--to`オプションを使用してスキャンする範囲を指定します（デフォルトでは境界は設定されていません）。`--limit`を使用して最大で何個のキーをプリントアウトするかを制限します（デフォルトは30個です）。`--cf`を使用してスキャンする列ファミリ（`default`、`write`、または`lock`）を指定します。

```shell
tikv-ctl --data-dir /var/lib/tikv raw-scan --from 'zt' --limit 2 --cf default
```

```
key: "zt\200\000\000\000\000\000\000\377\005_r\200\000\000\000\000\377\000\000\001\000\000\000\000\000\372\372b2,^\033\377\364", value: "\010\002\002\002%\010\004\002\010root\010\006\002\000\010\010\t\002\010\n\t\002\010\014\t\002\010\016\t\002\010\020\t\002\010\022\t\002\010\024\t\002\010\026\t\002\010\030\t\002\010\032\t\002\010\034\t\002\010 \t\002\010\"\t\002\010s\t\002\010&\t\002\010(\t\002\010*\t\002\010,\t\002\010.\t\002\0100\t\002\0102\t\002\0104\t\002"
key: "zt\200\000\000\000\000\000\000\377\025_r\200\000\000\000\000\377\000\000\023\000\000\000\000\000\372\372b2,^\033\377\364", value: "\010\002\002&slow_query_log_file\010\004\002P/usr/local/mysql/data/localhost-slow.log"

Total scanned keys: 2
```

### 特定のキーの値を表示する

キーの値を出力するには、`print`コマンドを使用します。

### Regionに関するいくつかのプロパティを表示する

Regionの状態の詳細を記録するために、TiKVはRegionsのSSTファイルにいくつかの統計情報を書き込みます。これらのプロパティを表示するには、`tikv-ctl`を`region-properties`サブコマンドとともに実行します:

```shell
tikv-ctl --host localhost:20160 region-properties -r 2
```

```
num_files: 0
num_entries: 0
num_deletes: 0
mvcc.min_ts: 18446744073709551615
mvcc.max_ts: 0
mvcc.num_rows: 0
mvcc.num_puts: 0
mvcc.num_versions: 0
mvcc.max_row_versions: 0
middle_key_by_approximate_size:
```

これらのプロパティは、Regionが健全であるかどうかを確認するために使用できます。もし健全でない場合は、Regionを修復するために使用できます。たとえば、`middle_key_approximate_size`で手動でRegionを分割することができます。

### 各TiKVのデータを手動で圧縮する

各TiKVのデータを手動で圧縮するには、`compact`コマンドを使用します。

- 圧縮範囲を指定するには、エスケープされた生のキー形式で`--from`および`--to`オプションを使用します。指定しない場合、全体の範囲が圧縮されます。
- 特定のRegionの範囲を圧縮するには、`--region`オプションを使用します。指定された場合、`--from`および`--to`は無視されます。
- 列ファミリの名前を指定するには、`-c`オプションを使用します。デフォルト値は`default`です。オプション値は`default`、`lock`、`write`があります。
- 圧縮を行うRocksDBを指定するには、`-d`オプションを使用します。デフォルト値は`kv`です。オプション値は`kv`および`raft`があります。
- `--threads`オプションを使用してTiKVの圧縮の並列度を指定できます。デフォルト値は`8`です。一般的に、より高い並列度はより速い圧縮速度をもたらし、それがサービスに影響を及ぼすかもしれません。シナリオに応じて適切な並列処理数を選択する必要があります。
- TiKVが圧縮を実行する際に最下層のファイルを含めるかどうかを指定するには、`--bottommost`オプションを使用します。オプション値は`default`、`skip`、`force`があります。デフォルト値は`default`です。
- `default`は、Compaction Filter機能が有効になっている場合に、最下層のファイルだけが含まれます。
- `skip`は、TiKVがコンパクションを実行する際に、最下層のファイルが除外されることを意味します。
- `force`は、TiKVがコンパクションを実行する際に、最下層のファイルが常に含まれることを意味します。

- ローカルモードでデータをコンパクトするには、次のコマンドを使用します:

    ```shell
    tikv-ctl --data-dir /path/to/tikv compact -d kv
    ```

- リモートモードでデータをコンパクトするには、次のコマンドを使用します:

    ```shell
    tikv-ctl --host ip:port compact -d kv
    ```

### TiKVクラスタ全体のデータを手動でコンパクトする

`compact-cluster`コマンドを使用して、TiKVクラスタ全体のデータを手動でコンパクトできます。このコマンドのフラグの意味と使用方法は`compact`コマンドと同じです。唯一の違いは以下の通りです:

- `compact-cluster`コマンドでは、`--pd`を使用してPDのアドレスを指定し、`tikv-ctl`がクラスタ内のすべてのTiKVノードをコンパクト対象として特定できるようにします。
- `compact`コマンドでは、`--data-dir`または`--host`を使用して、単一のTiKVをコンパクトの対象として指定します。

### Regionをtombstoneに設定する

`tombstone`コマンドは、通常、同期ログが有効になっていない状況で使用され、Raft状態機械によって書き込まれたいくつかのデータが電源ダウンによって失われた場合に使用されます。

TiKVインスタンスでは、このコマンドを使用していくつかのRegionのステータスをtombstoneに設定できます。その後、インスタンスを再起動すると、これらのRegionはダメージを受けたRaft状態機械による再起動の失敗を避けるためにスキップされます。これらのRegionは、他のTiKVインスタンスに十分な健全なレプリカが存在し、Raftメカニズムを介して読み取りと書き込みを継続できる必要があります。

一般的なケースでは、`remove-peer`コマンドを使用して、このRegionの対応するPeerを削除できます:

```shell
pd-ctl operator add remove-peer <region_id> <store_id>
```

その後、対応するTiKVインスタンスで`tombstone`コマンドを使用して、このRegionをtombstoneに設定して、このRegionの起動時のヘルスチェックをスキップします:

```shell
tikv-ctl --data-dir /path/to/tikv tombstone -p 127.0.0.1:2379 -r <region_id>
```

```
success!
```

ただし、一部のケースでは、このRegionのPeerを簡単にPDから削除できないため、`tikv-ctl`で`--force`オプションを指定してPeerを強制的にtombstoneに設定できます:

```shell
tikv-ctl --data-dir /path/to/tikv tombstone -p 127.0.0.1:2379 -r <region_id>,<region_id> --force
```

```
success!
```

> **注意:**
>
> - `tombstone`コマンドはローカルモードのみをサポートしています。
> - `-p`オプションの引数は、`http`プレフィックスなしでPDエンドポイントを指定します。PDエンドポイントを指定することで、PDがTombstoneに安全に切り替えられるかどうかを問い合わせることができます。

### TiKVに`consistency-check`リクエストを送信する

`consistency-check`コマンドを使用して、特定のRegionのRaft内のレプリカ間で一貫性チェックを実行できます。チェックが失敗すると、TiKV自体がパニックします。`--host`で指定したTiKVインスタンスがRegionのリーダーでない場合、エラーが報告されます。

```shell
tikv-ctl --host 127.0.0.1:20160 consistency-check -r 2
success!
tikv-ctl --host 127.0.0.1:20161 consistency-check -r 2
DebugClient::check_region_consistency: RpcFailure(RpcStatus { status: Unknown, details: Some("StringError(\"Leader is on store 1\")") })
```

> **注意:**
>
> - `consistency-check`コマンドを使用することは**お勧めしません**。これはTiDBのガベージコレクションと互換性がないため、誤ってエラーを報告する可能性があります。
> - このコマンドはリモートモードのみをサポートしています。
> - このコマンドが`success!`を返しても、TiKVがパニックしないかどうかをチェックする必要があります。なぜなら、このコマンドはリーダーの一貫性チェックを要求する提案のみであり、クライアントでは全体のチェックプロセスが成功したかどうかを知ることはできないからです。

### スナップショットメタをダンプする

このサブコマンドは、指定されたパスにあるスナップショットメタファイルを解析し、結果を表示するために使用されます。

### Raft状態機械が壊れたRegionを表示する

TiKVの起動中にリージョンをチェックしないようにするために、`tombstone`コマンドを使用して、Raft状態機械がエラーを報告するリージョンをTombstoneに設定できます。このコマンドを実行する前に、`bad-regions`コマンドを使用してエラーのあるリージョンを調べ、自動処理のために複数のツールを組み合わせることができます。

```shell
tikv-ctl --data-dir /path/to/tikv bad-regions
```

```
all regions are healthy
```

コマンドが正常に実行された場合は、上記の情報が表示されます。コマンドが失敗した場合、悪いリージョンのリストが表示されます。現在、検出できるエラーには、`last index`、`commit index`、`apply index`の不一致、およびRaftログの損失などが含まれます。スナップショットファイルの損傷などのその他の状況については、さらなるサポートが必要です。

### リージョンのプロパティを表示する

- `/path/to/tikv`に展開されたTiKVインスタンス上のリージョン2のプロパティをローカルで表示するには、次のコマンドを使用します:

    ```shell
    tikv-ctl --data-dir /path/to/tikv/data region-properties -r 2
    ```

- `127.0.0.1:20160`で実行中のTiKVインスタンス上のリージョン2のプロパティをオンラインで表示するには、次のコマンドを使用します:

    ```shell
    tikv-ctl --host 127.0.0.1:20160 region-properties -r 2
    ```

### TiKV構成を動的に変更する

`modify-tikv-config`コマンドを使用して、構成引数を動的に変更できます。現在、動的に変更可能なTiKVの構成項目と詳細な変更は、SQL文を使用して構成を変更するのと一致しています。詳細については、[動的にTiKVの構成を変更する](/dynamic-config.md#modify-tikv-configuration-dynamically)を参照してください。

- `-n`は構成項目のフルネームを指定するために使用されます。動的に変更可能な構成項目のリストについては、[動的にTiKVの構成を変更する](/dynamic-config.md#modify-tikv-configuration-dynamically)を参照してください。
- `-v`は構成値を指定するために使用されます。

`shared block cache`のサイズを設定します:

```shell
tikv-ctl --host ip:port modify-tikv-config -n storage.block-cache.capacity -v 10GB
```

```
success
```

`shared block cache`が無効になっている場合、`write` CFの`block cache size`を設定します:

```shell
tikv-ctl --host ip:port modify-tikv-config -n rocksdb.writecf.block-cache-size -v 256MB
```

```
success
```

```shell
tikv-ctl --host ip:port modify-tikv-config -n raftdb.defaultcf.disable-auto-compactions -v true
```

```
success
```

```shell
tikv-ctl --host ip:port modify-tikv-config -n raftstore.sync-log -v false
```

```
success
```

コンパクションのレート制限が蓄積されたコンパクション保留バイトを引き起こす場合、`rate-limiter-auto-tuned`モードを無効にするか、コンパクションフローの制限をより高く設定します:

```shell
tikv-ctl --host ip:port modify-tikv-config -n rocksdb.rate-limiter-auto-tuned -v false
```

```
success
```

```shell
tikv-ctl --host ip:port modify-tikv-config -n rocksdb.rate-bytes-per-sec -v "1GB"
```

```
success
```

### 複数のレプリカの障害からリージョンにサービスを回復させる

> **警告:**
>
> この機能を使用することはお勧めしません。代わりに、停止サービスなどの余分な操作が必要ない、オンラインの不安定なリカバリを提供する`pd-ctl`の使用ができます。詳細については[オンラインの不安定なリカバリ](/online-unsafe-recovery.md)を参照してください。

`unsafe-recover remove-fail-stores`コマンドを使用して、リージョンのピアリストから失敗したマシンを削除できます。このコマンドを実行する前に、対象のTiKVストアのサービスを停止してファイルロックを解放する必要があります。

`-s`オプションは、複数の`store_id`をコンマで区切って受け入れ、`-r`フラグを使用して関連するリージョンを指定します。特定のストアのすべてのリージョンでこの操作を実行する必要がある場合は、単純に`--all-regions`を指定することができます。

> **警告:**
>
> - 何らかの誤操作が行われると、クラスタを回復することが困難になる可能性があります。潜在的なリスクに注意し、本機能を本番環境で使用しないでください。
> - If the `--all-regions` option is used, you are expected to run this command on all the remaining stores connected to the cluster. You need to ensure that these healthy stores stop providing services before recovering the damaged stores. Otherwise, the inconsistent peer lists in Region replicas will cause errors when you run `split-region` or `remove-peer`. This further causes inconsistency between other metadata, and finally, the Regions will become unavailable.
> - Once you have run `remove-fail-stores`, you cannot restart the removed nodes or add these nodes to the cluster. Otherwise, the metadata will be inconsistent, and finally, the Regions will be unavailable.

```shell
tikv-ctl --data-dir /path/to/tikv unsafe-recover remove-fail-stores -s 3 -r 1001,1002
```

```
success!
```

```shell
tikv-ctl --data-dir /path/to/tikv unsafe-recover remove-fail-stores -s 4,5 --all-regions
```

その後、TiKVを再起動したら、残りの健全なレプリカでRegionsは引き続きサービスを提供できます。このコマンドは、複数のTiKVストアに損傷が発生したり削除されたりした場合に一般的に使用されます。

> **ノート:**
>
> - 指定されたRegionsのペアが存在するすべてのストアでこのコマンドを実行することが期待されています。
> - このコマンドはローカルモードのみをサポートしています。正常に実行された場合は `success!` が出力されます。

### MVCCデータの損傷からの回復

MVCCデータの損傷によりTiKVが正常に実行できない場合に、`recover-mvcc`コマンドを使用します。これにより、異なる種類の不整合から回復するために、"default"、"write"、"lock"の3つのCF（Column Family）がクロスチェックされます。

- `-r`オプションを使用して`region_id`ごとに関連するRegionsを指定します。
- `-p`オプションを使用してPDエンドポイントを指定します。

```shell
tikv-ctl --data-dir /path/to/tikv recover-mvcc -r 1001,1002 -p 127.0.0.1:2379
success!
```

> **ノート:**
>
> - このコマンドはローカルモードのみをサポートしています。正常に実行された場合は `success!` が出力されます。
> - `-p`オプションの引数は、PDエンドポイントを `http` の接頭辞なしで指定するものです。指定された `region_id` が有効かどうかをクエリするために、PDエンドポイントを指定する必要があります。
> - 指定されたRegionsのペアが存在するすべてのストアでこのコマンドを実行する必要があります。

### Ldbコマンド

`ldb`コマンドラインツールは、複数のデータアクセスおよびデータベース管理コマンドを提供します。以下にいくつかの例を示します。詳細については、`tikv-ctl ldb`を実行するか、RocksDBのドキュメントを参照してください。

データアクセスシーケンスの例：

既存のRocksDBをHEX形式でダンプするには:

```shell
tikv-ctl ldb --hex --db=/tmp/db dump
```

既存のRocksDBのマニフェストをダンプするには:

```shell
tikv-ctl ldb --hex manifest_dump --path=/tmp/db/MANIFEST-000001
```

クエリ対象となるカラムファミリを指定するには、`--column_family=<string>`コマンドラインを使用できます。

`--try_load_options`は、データベースオプションファイルをロードしてデータベースを開くためのオプションです。データベースをデフォルトオプションで開くと、LSMツリーが台無しになる可能性があるため、データベースが実行中である場合は、常にこのオプションをオンにすることをお勧めします。LSMツリーの状態は自動的に回復されない可能性があります。

### 暗号化メタデータのダンプ

`encryption-meta`サブコマンドを使用して暗号化メタデータをダンプします。このサブコマンドでは、データファイルの暗号化情報と使用されているデータ暗号化キーのリストの2種類のメタデータをダンプすることができます。

データファイルの暗号化情報をダンプするには、`encryption-meta dump-file`サブコマンドを使用します。TiKVデプロイメントの`data-dir`を指定するためにTiKV構成ファイルを作成する必要があります:

```
# conf.toml
[storage]
data-dir = "/path/to/tikv/data"
```

`--path`オプションを使用して、対象のデータファイルへの絶対または相対パスを指定することができます。データファイルが暗号化されていない場合、コマンドは空の出力を生成する場合があります。`--path`が指定されていない場合、すべてのデータファイルについての暗号化情報が出力されます。

```shell
tikv-ctl --config=./conf.toml encryption-meta dump-file --path=/path/to/tikv/data/db/CURRENT
```

```
/path/to/tikv/data/db/CURRENT: key_id: 9291156302549018620 iv: E3C2FDBF63FC03BFC28F265D7E78283F method: Aes128Ctr
```

データ暗号化キーをダンプするには、`encryption-meta dump-key`サブコマンドを使用します。`data-dir`に加えて、構成ファイルで使用中のマスターキーを指定する必要があります。マスターキーの構成方法については、[Encryption-At-Rest](/encryption-at-rest.md)を参照してください。また、このコマンドを使用する際には、`security.encryption.previous-master-key`構成は無視され、マスターキーのローテーションはトリガーされません。

```
# conf.toml
[storage]
data-dir = "/path/to/tikv/data"

[security.encryption.master-key]
type = "kms"
key-id = "0987dcba-09fe-87dc-65ba-ab0987654321"
region = "us-west-2"
```

マスターキーがAWS KMSキーの場合、`tikv-ctl`がKMSキーにアクセスできる必要があります。AWS KMSキーへの`tiikv-ctl`のアクセス許可は、環境変数、AWSデフォルト構成ファイル、または適切なものを使用したIAMロールを通じて与えられます。使用法についてはAWSドキュメントを参照してください。

`--ids`オプションを使用して、カンマで区切られたデータ暗号化キーIDのリストを指定できます。`--ids`が指定されていない場合、すべてのデータ暗号化キーが、最新の有効なデータ暗号化キーのIDである現在のキーIDと共に出力されます。

このコマンドを使用する際に、機密情報を公開する警告が表示されます。続行するには "I consent" と入力してください。

```shell
tikv-ctl --config=./conf.toml encryption-meta dump-key
```

```
This action will expose encryption key(s) as plaintext. Do not output the result in file on disk.
Type "I consent" to continue, anything else to exit: I consent
current key id: 9291156302549018620
9291156302549018620: key: 8B6B6B8F83D36BE2467ED55D72AE808B method: Aes128Ctr creation_time: 1592938357
```

```shell
tikv-ctl --config=./conf.toml encryption-meta dump-key --ids=9291156302549018620
```

```
This action will expose encryption key(s) as plaintext. Do not output the result in file on disk.
Type "I consent" to continue, anything else to exit: I consent
9291156302549018620: key: 8B6B6B8F83D36BE2467ED55D72AE808B method: Aes128Ctr creation_time: 1592938357
```

> **ノート:**
> このコマンドはデータ暗号化キーを平文で公開します。本番環境では、出力をファイルにリダイレクトしないでください。後で出力ファイルを削除しても、ディスクから内容を消去し切れない可能性があります。

### 損傷したSSTファイルに関連する情報の表示

TiKVの損傷したSSTファイルはTiKVプロセスがパニックする原因となる可能性があります。TiDB v6.1.0以前では、これらのファイルによりTiKVは直ちにパニックする可能性があります。TiDB v6.1.0以降では、損傷したSSTファイルにより、TiKVプロセスは損傷から1時間後にパニックする可能性があります。

損傷したSSTファイルをクリーンアップするには、TiKV Controlで`bad-ssts`コマンドを実行して必要な情報を表示できます。以下は、例示されたコマンドと出力です。

> **ノート:**
> このコマンドを実行する前に、実行中のTiKVインスタンスを停止してください。

```shell
tikv-ctl --data-dir </path/to/tikv> bad-ssts --pd <endpoint>
```

```
--------------------------------------------------------
corruption info:
data/tikv-21107/db/000014.sst: Corruption: Bad table magic number: expected 9863518390377041911, found 759105309091689679 in data/tikv-21107/db/000014.sst

sst meta:
14:552997[1 .. 5520]['0101' seq:1, type:1 .. '7A7480000000000000FF0F5F728000000000FF0002160000000000FAFA13AB33020BFFFA' seq:2032, type:1] at level 0 for Column family "default"  (ID 0)
it isn't easy to handle local data, start key:0101

overlap region:
RegionInfo { region: id: 4 end_key: 7480000000000000FF0500000000000000F8 region_epoch { conf_ver: 1 version: 2 } peers { id: 5 store_id: 1 }, leader: Some(id: 5 store_id: 1) }

suggested operations:
tikv-ctl ldb --db=data/tikv-21107/db unsafe_remove_sst_file "data/tikv-21107/db/000014.sst"
```
```
tikv-ctl --db=data/tikv-21107/db tombstone -r 4 --pd <endpoint>
--------------------------------------------------------
corruption analysis has completed
```

上記の出力から、壊れたSSTファイルの情報が最初にプリントされ、その後メタ情報がプリントされていることがわかります。

+ `sst meta` パートでは、`14` はSSTファイルの番号を示し、`552997` はファイルサイズを示し、その後に最小および最大のシーケンス番号およびその他のメタ情報が続きます。
+ `overlap region` パートでは、関連するリージョンの情報が表示されます。この情報はPDサーバーを通じて取得されます。
+ `suggested operations` パートでは、壊れたSSTファイルをクリーンアップするための提案が提供されます。この提案を受けてファイルをクリーンアップし、TiKVインスタンスを再起動することができます。

### リージョンの `RegionReadProgress` の状態を取得する

v6.5.4 および v7.3.0 から、TiKVは `get-region-read-progress` サブコマンドを導入し、リゾルバと `RegionReadProgress` の最新の詳細を取得します。Region ID と TiKV を指定する必要がありますが、これは Grafana（`Min Resolved TS Region` および `Min Safe TS Region`）から取得するか、`DataIsNotReady` ログから取得できます。

- `--log`（オプション）: 指定されている場合、TiKVはこのTiKVのリゾルバ内の最小の `start_ts` のロックを `INFO` レベルでログに残します。このオプションを使用すると、リゾルバを先読みする可能性のあるロックを識別するのに役立ちます。

- `--min-start-ts`（オプション）: 指定されている場合、TiKVはログにおいてこの値よりも小さい `start_ts` のロックをフィルタリングします。これを使用してログの興味のあるトランザクションを指定できます。デフォルト値は `0` であり、フィルターは適用されません。

以下は例です:

```
./tikv-ctl --host 127.0.0.1:20160 get-region-read-progress -r 14 --log --min-start-ts 0
```

出力は以下のようになります:

```
Region read progress:
    exist: true,
    safe_ts: 0,
    applied_index: 92,
    pending front item (oldest) ts: 0,
    pending front item (oldest) applied index: 0,
    pending back item (latest) ts: 0,
    pending back item (latest) applied index: 0,
    paused: false,
Resolver:
    exist: true,
    resolved_ts: 0,
    tracked index: 92,
    number of locks: 0,
    number of transactions: 0,
    stopped: false,
```

このサブコマンドは、Stale Read および safe-ts に関連する問題の診断に役立ちます。詳細については、[Understanding Stale Read and safe-ts in TiKV](/troubleshoot-stale-read.md) を参照してください。