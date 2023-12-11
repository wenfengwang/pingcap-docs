---
title: TiDBコントロールユーザーガイド
summary: TiDBコントロールを使用してデバッグ用のTiDBステータス情報を取得します。
aliases: ['/docs/dev/tidb-control/','/docs/dev/reference/tools/tidb-control/']
---

# TiDBコントロールユーザーガイド

TiDBコントロールはTiDBのコマンドラインツールであり、通常はデバッグ用のTiDBステータス情報を取得するために使用されます。このドキュメントでは、TiDBコントロールの機能とこれらの機能の使用方法について説明します。

> **注意:**
>
> TiDBコントロールはデバッグ目的に特化しており、将来のTiDBで導入される機能と完全に互換性があるとは限らない可能性があります。情報を取得するためにこのツールをアプリケーションやユーティリティの開発に含めることは推奨されません。

## TiDBコントロールを取得する

TiUPを使用してインストールするか、ソースコードからコンパイルしてTiDBコントロールを取得できます。

> **注意:**
>
> 使用するコントロールツールのバージョンがクラスタのバージョンと一致することをお勧めします。

### TiUPを使用してTiDBコントロールをインストールする

TiUPをインストールした後、 `tiup ctl:v<CLUSTER_VERSION> tidb` コマンドを使用してTiDBコントロールを取得および実行できます。

### ソースコードからコンパイルする

- コンパイル環境要件: [Go](https://golang.org/) 1.21以降
- コンパイル手順: [TiDBコントロールプロジェクト](https://github.com/pingcap/tidb-ctl)のルートディレクトリに移動し、 `make` コマンドを使用してコンパイルし、`tidb-ctl`を生成します。
- コンパイルドキュメント: `doc`ディレクトリにヘルプファイルがある場合、 `make doc` コマンドを使用してヘルプファイルを生成できます。 

## 使用方法の紹介

このセクションでは、`tidb-ctl`のコマンド、サブコマンド、オプション、およびフラグの使用方法について説明します。

- command: `-`または`--`を持たない文字
- subcommand: コマンドに続く`-`または`--`を持たない文字
- option: `-`または`--`を持つ文字
- flag: コマンド/サブコマンドまたはオプションに続く文字で、コマンド/サブコマンドまたはオプションに値を渡します

使用例: `tidb-ctl schema in mysql -n db`

- `schema`: コマンド
- `in`: `schema`のサブコマンド
- `mysql`: `in`のフラグ
- `-n`: オプション
- `db`: `-n`のフラグ

現在、TiDBコントロールには以下のサブコマンドがあります:

- `tidb-ctl base64decode`: `BASE64`デコーディングに使用されます
- `tidb-ctl decoder`: `KEY`デコーディングに使用されます
- `tidb-ctl etcd`: etcdの操作に使用されます
- `tidb-ctl log`: ログファイルを単一行のスタック情報に展開するために使用されます
- `tidb-ctl mvcc`: MVCC情報を取得するために使用されます
- `tidb-ctl region`: Region情報を取得するために使用されます
- `tidb-ctl schema`: スキーマ情報を取得するために使用されます
- `tidb-ctl table`: テーブル情報を取得するために使用されます

### ヘルプを取得する

`tidb-ctl -h/--help`を使用して使用方法情報を取得できます。

TiDBコントロールには複数のレイヤーのコマンドがあります。各コマンド/サブコマンドの後に`-h/--help`を使用して、それぞれの使用方法情報を取得できます。

以下の例では、スキーマ情報を取得する方法を示しています:

`tidb-ctl schema -h`を使用して使用詳細を取得できます。`schema`コマンド自体には`in`および`tid`の2つのサブコマンドがあります。

### グローバルオプション

`tidb-ctl`には以下の接続関連のグローバルオプションがあります:

- `--host`: TiDBサービスアドレス(デフォルトは127.0.0.1)
- `--port`: TiDBステータスポート(デフォルトは10080)
- `--pdhost`: PDサービスアドレス(デフォルトは127.0.0.1)
- `--pdport`: PDサービスポート(デフォルトは2379)
- `--ca`: TLS接続に使用されるCAファイルパス
- `--ssl-key`: TLS接続に使用されるキーファイルパス
- `--ssl-cert`: TLS接続に使用される証明書ファイルパス

`--pdhost`および`--pdport`は主に`etcd`サブコマンドで使用されます。例えば、`tidb-ctl etcd ddlinfo`。アドレスとポートを指定しない場合、以下のデフォルト値が使用されます:

- TiDBおよびPDのデフォルトのサービスアドレス: `127.0.0.1`。サービスアドレスはIPアドレスである必要があります。
- TiDBのデフォルトのサービスポート: `10080`。
- PDのデフォルトのサービスポート: `2379`。

### `schema`コマンド

#### `in`サブコマンド

`in`はデータベース名を介してデータベース内のすべてのテーブルのテーブルスキーマを取得するために使用されます。

```bash
tidb-ctl schema in <データベース名>
```

例えば、`tidb-ctl schema in mysql`を実行すると、以下の結果が返されます:

```json
[
    {
        "id": 13,
        "name": {
            "O": "columns_priv",
            "L": "columns_priv"
        },
              ...
        "update_timestamp": 399494726837600268,
        "ShardRowIDBits": 0,
        "Partition": null
    }
]
```

結果はJSON形式で表示されます (上記の出力は省略されています)。

- 特定のテーブル名を指定する場合は、 `tidb-ctl schema in <データベース> -n <テーブル名>`を使用してフィルタリングします。

たとえば、`tidb-ctl schema in mysql -n db`を実行すると、`mysql`データベースの`db`テーブルのテーブルスキーマが返されます:

```json
{
    "id": 9,
    "name": {
        "O": "db",
        "L": "db"
    },
    ...
    "Partition": null
}
```

(上記の出力も省略されています)。

デフォルトのTiDBサービスアドレスとポートを使用したくない場合は、`--host`および`--port`オプションを構成するために使用します。たとえば、 `tidb-ctl --host 172.16.55.88 --port 8898 schema in mysql -n db`。

#### `tid`サブコマンド

`tid`はデータベース全体の一意の`table_id`を使用してテーブルスキーマを取得するために使用されます。`in`サブコマンドを使用して特定のスキーマのすべてのテーブルIDを取得し、`tid`サブコマンドを使用して詳細なテーブル情報を取得できます。

例えば、 `mysql.stat_meta`のテーブルIDは `21` です。`tidb-ctl schema tid -i 21`を使用して、 `mysql.stat_meta`の詳細を取得できます。

```json
{
 "id": 21,
 "name": {
  "O": "stats_meta",
  "L": "stats_meta"
 },
 "charset": "utf8mb4",
 "collate": "utf8mb4_bin",
  ...
}
```

`in`サブコマンドと同様に、デフォルトのTiDBサービスアドレスとステータスポートを使用したくない場合は、 `--host`および `--port`オプションを使用してホストとポートを指定します。

#### `base64decode`コマンド

`base64decode`は`base64`データをデコードするために使用されます。

```shell
tidb-ctl base64decode [base64_data]
tidb-ctl base64decode [db_name.table_name] [base64_data]
tidb-ctl base64decode [table_id] [base64_data]
```

1. 環境を準備するために、次のSQLステートメントを実行します:

    ```sql
    use test;
    create table t (a int, b varchar(20),c datetime default current_timestamp , d timestamp default current_timestamp, unique index(a));
    insert into t (a,b,c) values(1,"哈哈 hello",NULL);
    alter table t add column e varchar(20);
    ```

2. HTTP APIインタフェースを使用してMVCCデータを取得します:

    ```shell
    $ curl "http://$IP:10080/mvcc/index/test/t/a/1?a=1"
    {
     "info": {
      "writes": [
       {
        "start_ts": 407306449994645510,
        "commit_ts": 407306449994645513,
        "short_value": "AAAAAAAAAAE="    # 一意のインデックス aは、対応する行のハンドルIDを格納しています。
       }
      ]
     }
    }%

    $ curl "http://$IP:10080/mvcc/key/test/t/1"
    {
     "info": {
      "writes": [
       {
        "start_ts": 407306588892692486,
        "commit_ts": 407306588892692489,
        "short_value": "CAIIAggEAhjlk4jlk4ggaGVsbG8IBgAICAmAgIDwjYuu0Rk="  # ハンドルIDが1である行データ。
       }
      ]
     }
    }%
    ```

3. ```ハンドルID (uint64)を `base64decode`を使用してデコードします```。
    $ tidb-ctl base64decode AAAAAAAAAAE=
    hex: 0000000000000001
    uint64: 1
    ```

4. `base64decode`を使用して行データをデコードします。

    ```shell
    $ ./tidb-ctl base64decode test.t CAIIAggEAhjlk4jlk4ggaGVsbG8IBgAICAmAgIDwjYuu0Rk=
    a:      1
    b:      哈哈 hello
    c はNULLです
    d:      2019-03-28 05:35:30
    e データ内で見つかりません

    # test.tのtable idが60の場合、同じことを行うには以下のコマンドも使用できます。
    $ ./tidb-ctl base64decode 60 CAIIAggEAhjlk4jlk4ggaGVsbG8IBgAICAmAgIDwjYuu0Rk=
    a:      1
    b:      哈哈 hello
    c はNULLです
    d:      2019-03-28 05:35:30
    e データ内で見つかりません
    ```

### `decoder`コマンド

- 次の例は、行のキーをデコードする方法を示しています。インデックスキーをデコードするのと同様です。

    ```shell
    $ ./tidb-ctl decoder "t\x00\x00\x00\x00\x00\x00\x00\x1c_r\x00\x00\x00\x00\x00\x00\x00\xfa"
    format: table_row
    table_id: -9223372036854775780      table_id: -9223372036854775780
    row_id: -9223372036854775558        row_id: -9223372036854775558
    ```

- 次の例は、`value`をデコードする方法を示しています。

    ```shell
    $ ./tidb-ctl decoder AhZoZWxsbyB3b3JsZAiAEA==
    format: index_value
    type: bigint, value: 1024       index_value[0]: {type: bytes, value: hello world}
    index_value[1]: {type: bigint, value: 1024}
    ```

### `etcd`コマンド

- `tidb-ctl etcd ddlinfo`はDDL情報を取得するために使用されます。
- `tidb-ctl etcd putkey KEY VALUE`は、KEY VALUEをetcdに追加するために使用されます（すべてのKEYは`/tidb/ddl/all_schema_versions/`ディレクトリに追加されます）。

    ```shell
    tidb-ctl etcd putkey "foo" "bar"
    ```

    実際には、KEYが`/tidb/ddl/all_schema_versions/foo`であり、VALUEが`bar`であるetcdにキーバリューが追加されます。

- `tidb-ctl etcd delkey`はetcd内のKEYを削除します。`/tidb/ddl/fg/owner/`または`/tidb/ddl/all_schema_versions/`プレフィックスのKEYのみが削除できます。

    ```shell
    tidb-ctl etcd delkey "/tidb/ddl/fg/owner/foo"
    tidb-ctl etcd delkey "/tidb/ddl/all_schema_versions/bar"
    ```

### `log`コマンド

TiDBエラーログのスタック情報は1行の形式です。`tidb-ctl log`を使用して、複数行の形式に変更できます。

### `keyrange`コマンド

`keyrange`サブコマンドは、グローバルまたはテーブル関連のキーレンジ情報を問い合わせるために使用され、16進形式で出力されます。

* `tidb-ctl keyrange`コマンドを実行してグローバルのキーレンジ情報を確認します：

    {{< copyable "shell-regular" >}}

    ```shell
    tidb-ctl keyrange
    ```

    ```
    global ranges:
      meta: (6d, 6e)
      table: (74, 75)
    ```

* エンコードされたキーを表示するには、`--encode`オプションを追加します（これはTiKVとPDの形式と同じ形式で表示されます）：

    {{< copyable "shell-regular" >}}

    ```shell
    tidb-ctl keyrange --encode
    ```

    ```
    global ranges:
      meta: (6d00000000000000f8, 6e00000000000000f8)
      table: (7400000000000000f8, 7500000000000000f8)
    ```

* `tidb-ctl keyrange --database={db} --table={tbl}`コマンドを実行して、グローバルとテーブル関連のキーレンジ情報を確認します：

    {{< copyable "shell-regular" >}}

    ```shell
    tidb-ctl keyrange --database test --table ttt
    ```

    ```
    global ranges:
      meta: (6d, 6e)
      table: (74, 75)
    table ttt ranges: (注意：DDL後にキーレンジは変更される可能性があります)
      table: (74800000000000002f, 748000000000000030)
      table indexes: (74800000000000002f5f69, 74800000000000002f5f72)
        index c2: (74800000000000002f5f698000000000000001, 74800000000000002f5f698000000000000002)
        index c3: (74800000000000002f5f698000000000000002, 74800000000000002f5f698000000000000003)
        index c4: (74800000000000002f5f698000000000000003, 74800000000000002f5f698000000000000004)
      table rows: (74800000000000002f5f72, 748000000000000030)
    ```