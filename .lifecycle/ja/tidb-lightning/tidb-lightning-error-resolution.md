---
title: TiDB Lightningエラーの解決
summary: データのインポート中に発生する型変換および重複エラーの解決方法を学びます。

# TiDB Lightningエラーの解決

v5.4.0から、TiDB Lightningを構成して無効な型変換および一意キーの競合などのエラーをスキップし、これらの誤った行データが存在しないかのようにデータ処理を続行できるようにできます。レポートが生成され、手動でエラーを修正できます。これは、エラーを手動で特定するのが難しいわずかに汚れているデータソースからのインポートに最適であり、TiDB Lightningを出会うたびに再起動するコストがかかります。

このドキュメントでは、TiDB Lightningのエラータイプ、エラーのクエリ方法、および例を紹介します。次の構成項目が関係しています。

- `lightning.max-error`：型エラーの許容閾値
- `conflict.strategy`、`conflict.threshold`、および`conflict.max-record-rows`：競合するデータに関連する構成
- `tikv-importer.duplicate-resolution`：物理インポートモードでのみ使用できる競合ハンドリング構成
- `lightning.task-info-schema-name`：TiDB Lightningが競合を検出したときに競合データが保存されるデータベース

詳細については、[TiDB Lightning（タスク）](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)を参照してください。

## 型エラー

`lightning.max-error`構成を使用して、データ型に関連するエラーの許容度を高めることができます。この構成が*N*に設定されている場合、TiDB Lightningは* N *の型エラーをデータソースから許可してスキップします。デフォルト値の `0` は、エラーを許可しないことを意味します。

これらのエラーはデータベースに記録されます。インポートが完了すると、データベース内のエラーを表示し、手動で処理できます。詳細については、「エラーレポート」を参照してください。

{{< copyable "" >}}

```toml
[lightning]
max-error = 0
```

上記の構成は、次のエラーをカバーしています。

- 無効な値（例：INT列に `'Text'` を設定する）
- 数値のオーバーフロー（例：TINYINT列に `500` を設定する）
- 文字列のオーバーフロー（例：VARCHAR(5)列に `'Very Long Text'` を設定する）
- ゼロの日時（つまり、 `'0000-00-00'` および `'2021-12-00'`）
- NOT NULL列にNULLを設定する
- 生成された列式の評価に失敗する
- 列数の不一致。行の値の数がテーブルの列数と一致しない。
- その他のSQLエラー

以下のエラーは常に致命的であり、`lightning.max-error`を変更してもスキップすることはできません。

- オリジナルのCSV、SQL、またはParquetファイルでの構文エラー（クォーテーションマークが閉じられていないなど）
- I/O、ネットワーク、またはシステム権限のエラー

## 競合エラー

[`conflict.threshold`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)構成項目を使用して、データ競合に関連するエラーの許容度を高めることができます。この構成項目が*N*に設定されている場合、TiDB Lightningは* N *の競合エラーをデータソースから許可してスキップします。デフォルト値は、`9223372036854775807`であり、ほとんどすべてのエラーが許容されることを意味します。

これらのエラーはテーブルに記録されます。インポートが完了すると、データベース内のエラーを表示し、手動で処理できます。詳細については、「エラーレポート」を参照してください。

## エラーレポート

TiDB Lightningがインポート中にエラーに遭遇すると、統計の概要を端末およびログファイルに出力します。

* 端末のエラーレポートは次の表のように似ています：

    | # | エラータイプ | エラー数 | エラーデータテーブル |
    | - | --- | --- | ------ |
    | 1 | データタイプ | 1000 | `lightning_task_info`.`type_error_v1` |

* TiDB Lightningログファイルのエラーレポートは次のようになります：

    ```shell
    [2022/03/13 05:33:57.736 +08:00] [WARN] [errormanager.go:459] ["合計1000件のデータ型エラーが検出されました。詳細については、テーブル `lightning_task_info`.`type_error_v1` を参照してください"]
    ```

すべてのエラーは、ダウンストリームのTiDBクラスター内の `lightning_task_info` データベースのテーブルに書き込まれます。インポートが完了したら、エラーデータが収集された場合、データベース内のエラーを表示し、手動で処理できます。

`lightning.task-info-schema-name`を設定することでデータベース名を変更できます。

{{< copyable "" >}}

```toml
[lightning]
task-info-schema-name = 'lightning_task_info'
```

TiDB Lightningはこのデータベースに3つのテーブルを作成します：

```sql
CREATE TABLE type_error_v1 (
    task_id     bigint NOT NULL,
    create_time datetime(6) NOT NULL DEFAULT now(6),
    table_name  varchar(261) NOT NULL,
    path        varchar(2048) NOT NULL,
    offset      bigint NOT NULL,
    error       text NOT NULL,
    row_data    text NOT NULL
);

CREATE TABLE conflict_error_v1 (
    task_id     bigint NOT NULL,
    create_time datetime(6) NOT NULL DEFAULT now(6),
    table_name  varchar(261) NOT NULL,
    index_name  varchar(128) NOT NULL,
    key_data    text NOT NULL,
    row_data    text NOT NULL,
    raw_key     mediumblob NOT NULL,
    raw_value   mediumblob NOT NULL,
    raw_handle  mediumblob NOT NULL,
    raw_row     mediumblob NOT NULL,
    KEY (task_id, table_name)
);
CREATE TABLE conflict_records (
    task_id     bigint NOT NULL,
    create_time datetime(6) NOT NULL DEFAULT now(6),
    table_name  varchar(261) NOT NULL,
    path        varchar(2048) NOT NULL,
    offset      bigint NOT NULL,
    error           text NOT NULL,
    row_id        bigint NOT NULL COMMENT '競合する行の行ID',
    row_data    text NOT NULL COMMENT '競合する行の行データ',
    KEY (task_id, table_name)
);

`type_error_v1`は`lightning.max-error`で管理される[型エラー](#型エラー)をすべて記録します。各エラーにつき1行が対応します。

`conflict_error_v1`は物理インポートモードでの `tikv-importer.duplicate-resolution`で管理されるすべての一意キーおよび主キーの競合を記録します。各競合ペアにつき2行が対応します。

`conflict_records`は論理インポートモードおよび物理インポートモードの `conflict`構成グループによって管理されるすべての一意キーおよび主キーの競合を記録します。各エラーにつき1行が対応します。

| 列       | 構文 | タイプ | 競合 | 説明 |
| ------------ | ------ | ---- | -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| task_id      | ✓      | ✓    | ✓        | このエラーを生成したTiDB LightningタスクID                                                        |
| create_time | ✓      | ✓    | ✓       | エラーが記録された時刻                                   |
| table_name   | ✓      | ✓    | ✓        | エラーを含むテーブルの名前、 ``'`db`.`tbl`'`` の形式                                           |
| path         | ✓      | ✓    |          | エラーを含むファイルのパス                                                              |
| offset       | ✓      | ✓    |          | エラーが見つかったファイル内のバイト位置                                                        |
| error        | ✓      | ✓    |          | エラーメッセージ                                                              |
| context      | ✓      |      |          | エラーを取り巻くテキスト                                                              |
| index_name   |        |      | ✓        | 競合するユニークキーの名前。主キーの競合の場合は、 'PRIMARY' です。                                                         |
| key_data     |        |      | ✓        | エラーを引き起こす行の形式化されたキーハンドル。内容は人間の参照用であり、機械読み取りは意図されていません。 |
| row_data     |        | ✓    | ✓        | エラーを引き起こす行データの形式化されたデータ。内容は人間の参照用であり、機械読み取りは意図されていません。              |
| raw_key      |        |      | ✓        | 競合するKVペアのキー                                                       |
| raw_value    |        |      | ✓        | 競合するKVペアの値                                                     |
| raw_handle   |        |      | ✓        | 競合する行の行ハンドル                                                    |
| raw_row      |        |      | ✓        | 競合する行のエンコードされた値                                                    |

> **注意:**
>
> エラーレポートは行/列番号ではなくファイルオフセットを記録し、取得することが非効率です。以下のコマンドを使用してファイル内のバイト位置に素早く移動できます(例：183を使用)：
>
> * shell、最初の数行を印刷。
>
>     ```shell
>     head -c 183 file.csv | tail
>     ```
>
> * shell、次の数行を印刷：
>
>     ```shell
>     tail -c +183 file.csv | head
>     ```
>
> * vim — `:goto 183`または`183go`

## 例

この例では、いくつかの既知のエラーを含むデータソースが準備されています。

1. データベースとテーブルスキーマを準備します。

    {{< copyable "shell-regular" >}}

    ```shell
    mkdir example && cd example
```
```japanese
    echo 'CREATE SCHEMA example;' > example-schema-create.sql
    echo 'CREATE TABLE t(a TINYINT PRIMARY KEY, b VARCHAR(12) NOT NULL UNIQUE);' > example.t-schema.sql
    ```

2. データを準備します。

    {{< copyable "shell-regular" >}}

    ```shell
    cat <<EOF > example.t.1.sql

        INSERT INTO t (a, b) VALUES
        (0, NULL),              -- 列はNOT NULLです
        (1, 'one'),
        (2, 'two'),
        (40, 'forty'),          -- 他の40と競合します
        (54, 'fifty-four'),     -- 他の'fifty-four'と競合します
        (77, 'seventy-seven'),  -- 文字列が12文字を超えています
        (600, 'six hundred'),   -- 数値がTINYINTをオーバーフローします
        (40, 'forty'),         -- 他の40と競合します
        (42, 'fifty-four');     -- 他の'fifty-four'と競合します

    EOF
    ```

3. TiDB Lightningを設定して、strict SQLモードを有効にし、データのインポートにLocal-backendを使用し、重複を削除し、最大10のエラーをスキップします。

    {{< copyable "shell-regular" >}}

    ```shell
    cat <<EOF > config.toml

        [lightning]
        max-error = 10

        [tikv-importer]
        backend = 'local'
        sorted-kv-dir = '/tmp/lightning-tmp/'
        duplicate-resolution = 'remove'

        [mydumper]
        data-source-dir = '.'
        [tidb]
        host = '127.0.0.1'
        port = 4000
        user = 'root'
        password = ''
        sql-mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE'

    EOF
    ```

4. TiDB Lightningを実行します。このコマンドはすべてのエラーがスキップされるため、正常に終了します。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup tidb-lightning -c config.toml
    ```

5. インポートされたテーブルが2つの通常の行のみを含んでいることを確認します：

    ```sql
    $ mysql -u root -h 127.0.0.1 -P 4000 -e 'select * from example.t'
    +---+-----+
    | a | b   |
    +---+-----+
    | 1 | one |
    | 2 | two |
    +---+-----+
    ```

6. `type_error_v1`テーブルが型変換に関連する3つの行をキャッチしているかどうかを確認します：

    ```sql
    $ mysql -u root -h 127.0.0.1 -P 4000 -e 'select * from lightning_task_info.type_error_v1;' -E

    *************************** 1. row ***************************
        task_id: 1635888701843303564
    create_time: 2021-11-02 21:31:42.620090
     table_name: `example`.`t`
           path: example.t.1.sql
         offset: 46
          error: failed to cast value as varchar(12) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin for column `b` (#2): [table:1048]Column 'b' cannot be null
       row_data: (0,NULL)

    *************************** 2. row ***************************
        task_id: 1635888701843303564
    create_time: 2021-11-02 21:31:42.627496
     table_name: `example`.`t`
           path: example.t.1.sql
         offset: 183
          error: failed to cast value as varchar(12) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin for column `b` (#2): [types:1406]Data Too Long, field len 12, data len 13
       row_data: (77,'seventy-seven')

    *************************** 3. row ***************************
        task_id: 1635888701843303564
    create_time: 2021-11-02 21:31:42.629929
     table_name: `example`.`t`
           path: example.t.1.sql
         offset: 253
          error: failed to cast value as tinyint(4) for column `a` (#1): [types:1690]constant 600 overflows tinyint
       row_data: (600,'six hundred')
    ```

7. `conflict_error_v1`テーブルが一意制約/プライマリキーの競合を含む4つの行をキャッチしているかどうかを確認します：

    ```sql
    $ mysql -u root -h 127.0.0.1 -P 4000 -e 'select * from lightning_task_info.conflict_error_v1;' --binary-as-hex -E

    *************************** 1. row ***************************
        task_id: 1635888701843303564
    create_time: 2021-11-02 21:31:42.669601
     table_name: `example`.`t`
     index_name: PRIMARY
       key_data: 40
       row_data: (40, "forty")
        raw_key: 0x7480000000000000C15F728000000000000028
      raw_value: 0x800001000000020500666F727479
     raw_handle: 0x7480000000000000C15F728000000000000028
        raw_row: 0x800001000000020500666F727479

    *************************** 2. row ***************************
        task_id: 1635888701843303564
    create_time: 2021-11-02 21:31:42.674798
     table_name: `example`.`t`
     index_name: PRIMARY
       key_data: 40
       row_data: (40, "forty")
        raw_key: 0x7480000000000000C15F728000000000000028
      raw_value: 0x800001000000020600666F75727479
     raw_handle: 0x7480000000000000C15F728000000000000028
        raw_row: 0x800001000000020600666F75727479

    *************************** 3. row ***************************
        task_id: 1635888701843303564
    create_time: 2021-11-02 21:31:42.680332
     table_name: `example`.`t`
     index_name: b
       key_data: 54
       row_data: (54, "fifty-four")
        raw_key: 0x7480000000000000C15F6980000000000000010166696674792D666FFF7572000000000000F9
      raw_value: 0x0000000000000036
     raw_handle: 0x7480000000000000C15F728000000000000036
        raw_row: 0x800001000000020A0066696674792D666F7572

    *************************** 4. row ***************************
        task_id: 1635888701843303564
    create_time: 2021-11-02 21:31:42.681073
     table_name: `example`.`t`
     index_name: b
       key_data: 42
       row_data: (42, "fifty-four")
        raw_key: 0x7480000000000000C15F6980000000000000010166696674792D666FFF7572000000000000F9
      raw_value: 0x000000000000002A
     raw_handle: 0x7480000000000000C15F72800000000000002A
        raw_row: 0x800001000000020A0066696674792D666F7572
    ```