---
title: TiDBスナップショットバックアップと復元コマンドマニュアル
summary: TiDBのスナップショットバックアップと復元のコマンドについて学びます。
---

# TiDBスナップショットバックアップと復元コマンドマニュアル

このドキュメントでは、アプリケーションシナリオに応じたTiDBスナップショットバックアップと復元のコマンドについて説明します。具体的には以下の内容が含まれます:

- [クラスタースナップショットをバックアップ](#クラスタースナップショットをバックアップ)
- [データベースまたはテーブルをバックアップ](#データベースまたはテーブルをバックアップ)
    - [データベースをバックアップ](#データベースをバックアップ)
    - [テーブルをバックアップ](#テーブルをバックアップ)
    - [テーブルフィルタで複数のテーブルをバックアップ](#テーブルフィルタで複数のテーブルをバックアップ)
- [統計情報をバックアップ](#統計情報をバックアップ)
- [バックアップデータを暗号化](#バックアップデータを暗号化)
- [クラスタースナップショットを復元](#クラスタースナップショットを復元)
- [データベースまたはテーブルを復元](#データベースまたはテーブルを復元)
    - [データベースを復元](#データベースを復元)
    - [テーブルを復元](#テーブルを復元)
    - [`mysql`スキーマから実行計画バインディングを復元](#mysqlスキーマから実行計画バインディングを復元)
- [暗号化されたスナップショットを復元](#暗号化されたスナップショットを復元)

スナップショットバックアップと復元に関する詳細情報については、以下を参照してください:

- [スナップショットバックアップと復元ガイド](/br/br-snapshot-guide.md)
- [バックアップとリストアの使用例](/br/backup-and-restore-use-cases.md)

## クラスタースナップショットをバックアップ

`br backup full`コマンドを使用して、TiDBクラスターの最新または指定されたスナップショットをバックアップできます。コマンドに関する詳細情報は、`br backup full --help`コマンドを実行してください。

```shell
br backup full \
    --pd "${PD_IP}:2379" \
    --backupts '2022-09-08 13:30:00' \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --ratelimit 128 \
    --log-file backupfull.log
```

前述のコマンドでは:

- `--backupts`: スナップショットの時点。フォーマットは[TSO](/glossary.md#tso)またはタイムスタンプであり、例えば`400036290571534337`または`2018-05-11 01:42:23`のようになります。このスナップショットのデータがガベージコレクションされた場合、`br backup`コマンドはエラーを返し、'br'が終了します。このパラメータを指定しない場合、`br`はバックアップ開始時刻に対応するスナップショットを選択します。
- `--ratelimit`: バックアップタスクを実行する**各TiKV**の最大速度。単位はMiB/sです。
- `--log-file`: `br`ログが書き込まれるターゲットファイルです。

> **注意:**
> 
> BRツールは、GCに自動適応できるようになっています。デフォルトで最新のPDタイムスタンプをPDの`safePoint`に自動的に登録し、バックアップ中にTiDBのGCセーフポイントが前に進まないようにします。これにより、手動でGC設定を行う必要がありません。

バックアップ中には、ターミナルに進行状況バーが表示され、進行状況バーが100%になると、バックアップが完了します。

```shell
Full Backup <---------/................................................> 17.12%.
```

## データベースまたはテーブルをバックアップ

Backup & Restore (BR)は、クラスタースナップショットまたは増分データバックアップから指定されたデータベースまたはテーブルの部分データをバックアップする機能をサポートしています。この機能を使用すると、スナップショットバックアップおよび増分データバックアップから不要なデータをフィルタリングし、ビジネスに重要なデータのみをバックアップできます。

### データベースをバックアップ

クラスター内のデータベースをバックアップするには、`br backup db`コマンドを実行します。

次の例では、`test`データベースをAmazon S3にバックアップしています:

```shell
br backup db \
    --pd "${PD_IP}:2379" \
    --db test \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --ratelimit 128 \
    --log-file backuptable.log
```

前述のコマンドでは、`--db`はデータベース名を指定し、他のパラメータは[クラスタースナップショットをバックアップ](#クラスタースナップショットをバックアップ)と同じです。

### テーブルをバックアップ

クラスター内のテーブルをバックアップするには、`br backup table`コマンドを実行します。

次の例では、`test.usertable`テーブルをAmazon S3にバックアップしています:

```shell
br backup table \
    --pd "${PD_IP}:2379" \
    --db test \
    --table usertable \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --ratelimit 128 \
    --log-file backuptable.log
```

前述のコマンドでは、`--db`および `--table`はそれぞれデータベース名とテーブル名を指定し、他のパラメータは[クラスタースナップショットをバックアップ](#クラスタースナップショットをバックアップ)と同じです。

### テーブルフィルタで複数のテーブルをバックアップ

より詳細な条件で複数のテーブルをバックアップするには、`br backup full`コマンドを実行し、`--filter`または`-f`で[テーブルフィルタ](/table-filter.md)を指定します。

次の例では、`db*.tbl*`フィルタルールに一致するテーブルをAmazon S3にバックアップしています:

```shell
br backup full \
    --pd "${PD_IP}:2379" \
    --filter 'db*.tbl*' \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --ratelimit 128 \
    --log-file backupfull.log
```

## 統計情報をバックアップ

TiDB v7.5.0以降、`br`コマンドラインツールには`--ignore-stats`パラメータが導入されました。このパラメータを`false`に設定すると、`br`コマンドラインツールは列、インデックス、およびテーブルの統計情報をバックアップおよび復元する機能をサポートします。この場合、バックアップから復元されたTiDBデータベースで統計情報収集タスクを手動で実行する必要はありません。また、自動収集タスクの完了を待つ必要もありません。この機能により、データベースのメンテナンス作業が簡素化され、クエリのパフォーマンスが向上します。

このパラメータを`false`に設定しない場合、`--ignore-stats=true`がデフォルト設定となり、データバックアップ中に統計情報はバックアップされません。

次の例は、クラスタースナップショットデータのバックアップと`--ignore-stats=false`を使用してテーブル統計情報をバックアップしています:

```shell
br backup full \
--storage local:///br_data/ --pd "${PD_IP}:2379" --log-file restore.log \
--ignore-stats=false
```

前述の構成でデータをバックアップした後にデータを復元する場合、バックアップされたテーブル統計情報は自動的に復元されます:

```shell
br restore full \
--storage local:///br_data/ --pd "${PD_IP}:2379" --log-file restore.log
```

バックアップとリストア機能がデータをバックアップすると、統計情報は`backupmeta`ファイル内でJSONフォーマットで保存されます。データを復元する際には、統計情報がJSONフォーマットでクラスターに読み込まれます。詳細については、[LOAD STATS](/sql-statements/sql-statement-load-stats.md)を参照してください。

## バックアップデータを暗号化

> **警告:**
>
> これは実験的機能です。本番環境での使用はお勧めしません。

BRは、バックアップ側および[Amazon S3にバックアップする際のストレージ側でバックアップデータを暗号化する機能](/br/backup-and-restore-storages.md#amazon-s3-server-side-encryption)をサポートしています。必要に応じて、どちらかの暗号化方法を選択できます。

TiDB v5.3.0以降、以下のパラメータを設定することでバックアップデータを暗号化できます:

- `--crypter.method`: 暗号化アルゴリズム。`aes128-ctr`、`aes192-ctr`、または`aes256-ctr`が指定できます。デフォルト値は`plaintext`であり、データは暗号化されません。
- `--crypter.key`: 16バイトの128ビット（`aes128-ctr`の場合）、24バイトのキー（`aes192-ctr`の場合）、または32バイトのキー（`aes256-ctr`の場合）の16進数文字列形式の暗号化キーです。
- `--crypter.key-file`: キーファイル。キーが保存されているファイルのパスをパラメータとして渡すことができます。

次の例は:

```shell
br backup full\
    --pd ${PD_IP}:2379 \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --crypter.method aes128-ctr \
    --crypter.key 0123456789abcdef0123456789abcdef
```

> **注意:**
>
> - キーを失った場合、バックアップデータはクラスターに復元できません。
> - 暗号化機能は`br`およびTiDBクラスターv5.3.0以降で使用する必要があります。暗号化されたバックアップデータは、v5.3.0より古いクラスターには復元できません。

## クラスタースナップショットを復元
TiDBクラスタースナップショットは、`br restore full`コマンドを実行することで復元できます。

```shell
br restore full \
    --pd "${PD_IP}:2379" \
    --with-sys-table \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --ratelimit 128 \
    --log-file restorefull.log
```

前述のコマンドでは:

- `--with-sys-table`: BRは、アカウント権限データやSQLバインド、統計など、**一部のシステムテーブルのデータ**を復元します（詳細は[統計のバックアップ](/br/br-snapshot-manual.md#back-up-statistics)を参照）。ただし、統計テーブル（`mysql.stat_*`）やシステム変数テーブル（`mysql.tidb`および`mysql.global_variables`）は復元されません。詳細は、[`mysql`スキーマのテーブルを復元する](/br/br-snapshot-guide.md#restore-tables-in-the-mysql-schema)を参照してください。
- `--ratelimit`: バックアップタスクを実行する際の、**TiKVあたりの最大速度**を指定します。単位はMiB/sです。
- `--log-file`: `br`ログが書き込まれるターゲットファイルです。

復元中には進捗バーがターミナルに表示され、以下のように進捗バーが100%に達すると復元タスクが完了します。その後、`br`は復元されたデータを検証してデータのセキュリティを確認します。

```shell
フルリストア <---------/...............................................> 17.12%.
```

## データベースまたはテーブルの復元

バックアップデータから特定のデータベースやテーブルの部分データを復元するには、`br`を使用できます。この機能を使用すると、復元中に不要なデータをフィルタリングできます。

### データベースの復元

クラスターにデータベースを復元するには、`br restore db`コマンドを実行します。

次の例では、バックアップデータから`test`データベースをターゲットクラスターに復元します:

```shell
br restore db \
    --pd "${PD_IP}:2379" \
    --db "test" \
    --ratelimit 128 \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --log-file restore_db.log
```

前述のコマンドでは、`--db`は復元するデータベースの名前を指定し、他のパラメータは[ティDBクラスタースナップショットの復元](#restore-cluster-snapshots)と同じです。

> **注記:**
>
> バックアップデータを復元する際、`--db`で指定するデータベース名は、バックアップコマンドの`-- db`で指定するデータベース名と同じである必要があります。そうでない場合は、復元に失敗します。これは、バックアップデータのメタファイル（`backupmeta`ファイル）にデータベース名が記録されており、同じ名前のデータベースにのみデータを復元できるからです。推奨される方法は、バックアップデータを別のクラスターの同じ名前のデータベースに復元することです。

### テーブルの復元

単一のテーブルをクラスターに復元するには、`br restore table`コマンドを実行します。

次の例では、Amazon S3からターゲットクラスターに`test.usertable`テーブルを復元します:

```shell
br restore table \
    --pd "${PD_IP}:2379" \
    --db "test" \
    --table "usertable" \
    --ratelimit 128 \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --log-file restore_table.log
```

前述のコマンドでは、`--table`は復元するテーブルの名前を指定し、他のパラメータは[データベースの復元](#restore-a-database)と同じです。

### テーブルフィルターを使用して複数のテーブルを復元する

より複雑なフィルタールールで複数のテーブルを復元するには、`br restore full`コマンドを実行し、`--filter`または`-f`で[テーブルフィルター](/table-filter.md)を指定します。

次の例では、「db*.tbl*」フィルタールールに一致するテーブルをAmazon S3からターゲットクラスターに復元します:

```shell
br restore full \
    --pd "${PD_IP}:2379" \
    --filter 'db*.tbl*' \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --log-file restorefull.log
```

### `mysql`スキーマから実行計画バインドを復元する

クラスターの実行計画バインドを復元するには、`br restore full`コマンドを実行し、`--with-sys-table`オプションと`--filter`または`-f`オプションで`mysql`スキーマを指定します。

以下は`mysql.bind_info`テーブルを復元する例です:

```shell
br restore full \
    --pd "${PD_IP}:2379" \
    --filter 'mysql.bind_info' \
    --with-sys-table \
    --ratelimit 128 \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --log-file restore_system_table.log
```

復元が完了すると、[`SHOW GLOBAL BINDINGS`](/sql-statements/sql-statement-show-bindings.md)で実行計画バインディング情報を確認できます:

```sql
SHOW GLOBAL BINDINGS;
```

復元後の実行計画バインディングの動的読み込みは引き続き最適化作業中です（関連する問題は[#46527](https://github.com/pingcap/tidb/issues/46527)および[#46528](https://github.com/pingcap/tidb/issues/46528)です）。復元後に実行計画バインディングを手動で再ロードする必要があります。

```sql
-- `mysql.bind_info`テーブルにbuiltin_pseudo_sql_for_bind_lockのレコードが1つだけ存在することを確認してください。複数のレコードがある場合は手動で削除する必要があります。
SELECT count(*) FROM mysql.bind_info WHERE original_sql = 'builtin_pseudo_sql_for_bind_lock';
DELETE FROM bind_info WHERE original_sql = 'builtin_pseudo_sql_for_bind_lock' LIMIT 1;

-- バインド情報を強制的にリロードします。
ADMIN RELOAD BINDINGS;
```

## 暗号化されたスナップショットの復元

> **警告:**
>
> これは実験的な機能です。本番環境での使用はお勧めしません。

バックアップデータを暗号化した後、復元するために対応する復号化パラメータを渡す必要があります。復号化アルゴリズムとキーが正しいことを確認してください。復号化アルゴリズムまたはキーが間違っていると、データを復元できません。以下は例です:

```shell
br restore full\
    --pd "${PD_IP}:2379" \
    --storage "s3://${backup_collection_addr}/snapshot-${date}?access-key=${access-key}&secret-access-key=${secret-access-key}" \
    --crypter.method aes128-ctr \
    --crypter.key 0123456789abcdef0123456789abcdef
```