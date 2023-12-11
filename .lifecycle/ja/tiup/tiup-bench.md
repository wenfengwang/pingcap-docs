---
title: TiUPベンチコンポーネントを使用したTiDBのストレステスト
summary: TiUPを使用してTPC-C、TPC-H、CH、RawSQL、およびYCSBのワークロードを使用してTiDBをストレステストする方法について学びます。
aliases: ['/docs/dev/tiup/tiup-bench/','/docs/dev/reference/tools/tiup/bench/']
---

# TiUPベンチコンポーネントを使用したTiDBのストレステスト

データベースのパフォーマンスをテストする際、データベースをストレステストすることがよくあります。そのために、TiUPには複数のストレステスト用のワークロードを提供するベンチコンポーネントが統合されています。以下のコマンドでこれらのワークロードにアクセスできます。

```bash
tiup bench tpcc   # TPC-Cを使用してデータベースをベンチマーク
tiup bench tpch   # TPC-Hを使用してデータベースをベンチマーク
tiup bench ch     # CH-benCHmarkを使用してデータベースをベンチマーク
tiup bench ycsb   # YCSBを使用してデータベースをベンチマーク
tiup bench rawsql # 任意のSQLファイルを使用してデータベースをベンチマーク
```

`tpcc`、`tpch`、`ch`、`rawsql`では以下の共通のコマンドフラグが共有されています。ただし、`ycsb`は主に`.properties`ファイルによって構成されます。これについては[使用ガイド](https://github.com/pingcap/go-ycsb#usage)で説明されています。

```plaintext
  -t, --acThreads int         OLAPクライアントの同時実行数、CH-benCHmark専用（デフォルトは1）
      --conn-params string    tidb_isolation_read_engines='tiflash'のようなセッション変数、PostgreSQL接続時の`sslmode=disable`のような設定など
      --count int             合計実行回数 (0は無制限)
  -D, --db string             データベース名（デフォルトは"test"）
  -d, --driver string         データベースドライバ: mysql、postgres (デフォルトは"mysql")
      --dropdata              準備前に履歴データをクリーンアップ
  -H, --host strings          データベースホスト（デフォルトは[127.0.0.1]）
      --ignore-error          ワークロード実行時のエラーを無視
      --interval duration     出力間隔時間 (デフォルトは10s)
      --isolation int         分離レベル (0: デフォルト; 1: ReadUncommitted; 2: ReadCommitted; 3: WriteCommitted; 4: RepeatableRead; 5: Snapshot; 6: Serializable; 7: Linerizable)
      --max-procs int         Golangのruntime.GOMAXPROCS、使用できるコア数上限
      --output string         出力スタイル。有効な値は { plain | table | json }（デフォルトは"plain"）
  -p, --password string       データベースパスワード
  -P, --port ints             データベースポート（デフォルトは[4000]）
      --pprof string          pprofエンドポイントのアドレス
      --silence               ワークロード実行時のエラーを出力しない
  -S, --statusPort int        データベースステータスポート（デフォルトは10080）
  -T, --threads int           スレッド同時実行数（デフォルトは1）
      --time duration         合計実行時間（デフォルトは2562047時間47分16.854775807秒）
  -U, --user string           データベースユーザ（デフォルトは"root")
```

- `--host`および`--port`にカンマ区切りの値を渡すことでクライアント側の負荷分散を有効にできます。例えば、`--host 172.16.4.1,172.16.4.2 --port 4000,4001`を指定すると、プログラムは172.16.4.1:4000、172.16.4.1:4001、172.16.4.2:4000、および172.16.4.2:4001にラウンドロビン方式で接続します。
- `--conn-params`は[クエリ文字列](https://en.wikipedia.org/wiki/Query_string)の形式に従う必要があります。異なるデータベースには異なるパラメータがあるかもしれません。例:
    - `--conn-params tidb_isolation_read_engines='tiflash'`はTiDBにTiFlashから読み取るように強制します。
    - `--conn-params sslmode=disable`はPostgreSQLへの接続時にSSLを無効にします。
- CH-benCHmarkを実行する際、`--ap-host`、`--ap-port`、および`--ap-conn-params`を使用してOLAPクエリ用のスタンドアロンTiDBサーバを指定できます。

以下のセクションでは、TiUPを使用してTPC-C、TPC-H、YCSBのテストを実行する方法について説明します。

## TiUPを使用したTPC-Cテストの実行

TiUPベンチコンポーネントは次のコマンドとフラグをサポートして、TPC-Cテストを実行します。

```bash
利用可能なコマンド:
  check       ワークロードのデータ整合性を確認
  cleanup     ワークロードのデータをクリーンアップ
  prepare     ワークロードのデータを準備
  run         ワークロードを実行

フラグ:
      --check-all            すべての整合性チェックを実行
  -h, --help                 TPC-Cのヘルプ
      --partition-type int   パーティションタイプ: 1 - HASH, 2 - RANGE, 3 - LIST (HASH-like), 4 - LIST (RANGE-like) (デフォルトは1)
      --parts int            パーティション数（デフォルトは1）
      --warehouses int       倉庫の数（デフォルトは10）
```

### テスト手順

以下に、TPC-Cテストの実行手順の簡略化されたステップを示します。詳細な手順については、「[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)」を参照してください。

1. ハッシュを使用して4つの倉庫を4つの分割で作成:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpcc --warehouses 4 --parts 4 prepare
    ```

2. TPC-Cテストを実行:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpcc --warehouses 4 --time 10m run
    ```

3. 一貫性を確認:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpcc --warehouses 4 check
    ```

4. データをクリーンアップ:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpcc --warehouses 4 cleanup
    ```

SQLを使用したデータの準備は、大規模なデータセットでベンチマークを実行したいときに遅くなる場合があります。この場合、次のコマンドでCSV形式でデータを生成し、それを[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用してTiDBにインポートできます。

- CSVファイルを生成:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpcc --warehouses 4 prepare --output-dir data --output-type=csv
    ```

- 指定したテーブルのためのCSVファイルを生成:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpcc --warehouses 4 prepare --output-dir data --output-type=csv --tables history,orders
    ```

## TiUPを使用したTPC-Hテストの実行

TiUPベンチコンポーネントは、次のコマンドとパラメータをサポートしてTPC-Hテストを実行します。

```bash
利用可能なコマンド:
  cleanup     ワークロードのデータをクリーンアップ
  prepare     ワークロードのデータを準備
  run         ワークロードを実行

フラグ:
      --check            出力データをチェック、スケールファクタが1のときのみ有効
  -h, --help             tpchのヘルプ
      --queries string   すべてのクエリ（デフォルトは"q1,q2,q3,q4,q5,q6,q7,q8,q9,q10,q11,q12,q13,q14,q15,q16,q17,q18,q19,q20,q21,q22")
      --sf int           スケールファクタ
```

### テスト手順

1. データを準備:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpch --sf=1 prepare
    ```

2. 以下のコマンドのいずれかを実行してTPC-Hテストを実行します:

    - 結果をチェックする場合は、このコマンドを実行してください:

        {{< copyable "shell-regular" >}}

        ```shell
        tiup bench tpch --count=22 --sf=1 --check=true run
        ```

    - 結果をチェックしない場合は、次のコマンドを実行してください:

        {{< copyable "shell-regular" >}}

        ```shell
        tiup bench tpch --count=22 --sf=1 run
        ```

3. データをクリーンアップ:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup bench tpch cleanup
    ```

## TiUPを使用したYCSBテストの実行

YCSBを使用してTiDBおよびTiKVの両方をストレステストできます。

### TiDBをストレステストする

1. データを準備:

    ```shell
    tiup bench ycsb load tidb -p tidb.instances="127.0.0.1:4000" -p recordcount=10000
    ```

2. YCSBワークロードを実行:

    ```shell
    # デフォルトではリードライトパーセントは95%
```shell
    tiup bench ycsb run tidb -p tidb.instances="127.0.0.1:4000" -p operationcount=10000
    ```

### TiKVのストレステスト

1. データの準備：

    ```shell
    tiup bench ycsb load tikv -p tikv.pd="127.0.0.1:2379" -p recordcount=10000
    ```

2. YCSBワークロードの実行：

    ```shell
    # デフォルトでは、読み書きの割合は95％です
    tiup bench ycsb run tikv -p tikv.pd="127.0.0.1:2379" -p operationcount=10000
    ```

## TiUPを使用したRawSQLテストの実行

SQLファイルに任意のクエリを記述し、次のように`tiup bench rawsql`を実行してテストに使用できます：

1. データとクエリの準備：

    ```sql
    -- データの準備
    CREATE TABLE t (a int);
    INSERT INTO t VALUES (1), (2), (3);

    -- SQLファイルにクエリを保存します。たとえば、次のクエリを `demo.sql` に保存できます。
    SELECT a, sleep(rand()) FROM t WHERE a < 4*rand();
    ```

2. RawSQLテストの実行：

    ```shell
    tiup bench rawsql run --count 60 --query-files demo.sql
    ```