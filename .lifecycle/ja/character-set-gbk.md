---
title: GBK
summary: このドキュメントでは、TiDBがGBK文字セットをサポートする詳細について提供します。

# GBK

v5.4.0以降、TiDBはGBK文字セットをサポートしています。このドキュメントでは、TiDBがGBK文字セットをサポートし、互換性情報を提供します。

```sql
SHOW CHARACTER SET WHERE CHARSET = 'gbk';
+---------+-------------------------------------+-------------------+--------+
| Charset | Description                         | Default collation | Maxlen |
+---------+-------------------------------------+-------------------+--------+
| gbk     | Chinese Internal Code Specification | gbk_bin           |      2 |
+---------+-------------------------------------+-------------------+--------+
1 行のセット (0.00 秒)

SHOW COLLATION WHERE CHARSET = 'gbk';
+----------------+---------+------+---------+----------+---------+
| Collation      | Charset | Id   | Default | Compiled | Sortlen |
+----------------+---------+------+---------+----------+---------+
| gbk_bin        | gbk     |   87 |         | Yes      |       1 |
+----------------+---------+------+---------+----------+---------+
1 行のセット (0.00 秒)
```

## MySQL 互換性

このセクションでは、MySQLとTiDBの互換性情報を提供します。

### 照合

MySQLにおけるGBK文字セットのデフォルト照合は `gbk_chinese_ci` です。一方、TiDBにおけるGBK文字セットのデフォルト照合は `gbk_bin` です。さらに、TiDBはGBKをUTF8MB4に変換してからバイナリ照合を使用するため、TiDBにおける `gbk_bin` 照合はMySQLにおける `gbk_bin` 照合とは異なります。

<TiDB用のカスタムコンテンツ>

MySQLのGBK文字セットの照合にTiDBを互換性があるようにするには、TiDBクラスターを初期化する際、TiDBオプション [`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap) を `true` に設定し、[照合の新しいフレームワーク](/character-set-and-collation.md#new-framework-for-collations) を有効にする必要があります。

</TiDB用のカスタムコンテンツ>

<TiDB-Cloud用のカスタムコンテンツ>

MySQLのGBK文字セットの照合にTiDBを互換性を持たせるためには、TiDBクラスターを初期化する際、TiDB Cloudはデフォルトで [照合の新しいフレームワーク](/character-set-and-collation.md#new-framework-for-collations) を有効にします。

</TiDB-Cloud用のカスタムコンテンツ>

新しい照合フレームワークを有効にした後、GBK文字セットに対応する照合を確認すると、TiDBのGBKのデフォルト照合が `gbk_chinese_ci` に変更されていることが確認できます。

```sql
SHOW CHARACTER SET WHERE CHARSET = 'gbk';
+---------+-------------------------------------+-------------------+--------+
| Charset | Description                         | Default collation | Maxlen |
+---------+-------------------------------------+-------------------+--------+
| gbk     | Chinese Internal Code Specification | gbk_chinese_ci    |      2 |
+---------+-------------------------------------+-------------------+--------+
1 行のセット (0.00 秒)

SHOW COLLATION WHERE CHARSET = 'gbk';
+----------------+---------+------+---------+----------+---------+
| Collation      | Charset | Id   | Default | Compiled | Sortlen |
+----------------+---------+------+---------+----------+---------+
| gbk_bin        | gbk     |   87 |         | Yes      |       1 |
| gbk_chinese_ci | gbk     |   28 | Yes     | Yes      |       1 |
+----------------+---------+------+---------+----------+---------+
2 行のセット (0.00 秒)
```

### 不正な文字の互換性

* システム変数 [`character_set_client`](/system-variables.md#character_set_client) と [`character_set_connection`](/system-variables.md#character_set_connection) が同時に `gbk` に設定されていない場合、TiDBはMySQLと同様に不正な文字を処理します。
* `character_set_client` と `character_set_connection` が共に `gbk` に設定されている場合、TiDBはMySQLとは異なる方法で不正な文字を処理します。

    - MySQLは読み取りおよび書き込み操作において、不正なGBK文字セットを異なる方法で処理します。
    - TiDBはSQL strictモードでは、不正なGBK文字を読み取りまたは書き込み時にエラーを報告します。非strictモードでは、TiDBは不正なGBK文字を読み取りまたは書き込み時に `?` で置換します。

たとえば、`SET NAMES gbk` の後、MySQLとTiDBそれぞれで `CREATE TABLE gbk_table(a VARCHAR(32) CHARACTER SET gbk)` 文を使用してテーブルを作成し、次の表のSQLステートメントを実行すると、詳細な違いが確認できます。

| データベース |    設定されたSQLモードが `STRICT_ALL_TABLES` または `STRICT_TRANS_TABLES` を含んでいる場合                                               | 設定されたSQLモードが `STRICT_ALL_TABLES` または `STRICT_TRANS_TABLES` を含まない場合                                                                     |
|-------|-------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| MySQL | `SELECT HEX('一a');` <br /> `e4b88061`<br /><br />`INSERT INTO gbk_table values('一a');`<br /> `不正なエラー`       | `SELECT HEX('一a');` <br /> `e4b88061`<br /><br />`INSERT INTO gbk_table VALUES('一a');`<br />`SELECT HEX(a) FROM gbk_table;`<br /> `e4b8` |
| TiDB  | `SELECT HEX('一a');` <br /> `不正なエラー`<br /><br />`INSERT INTO gbk_table VALUES('一a');`<br /> `不正なエラー` | `SELECT HEX('一a');` <br /> `e4b83f`<br /><br />`INSERT INTO gbk_table VALUES('一a');`<br />`SELECT HEX(a) FROM gbk_table;`<br /> `e4b83f`  |

上記の表では、`utf8mb4` バイトセットにおける `SELECT HEX('a');` の結果は `e4b88061` です。

### その他のMySQL互換性

- 現在、TiDBは他の文字セットタイプを `gbk` に変換するための `ALTER TABLE` ステートメント、および `gbk` から他の文字セットタイプに変換することをサポートしていません。

* TiDBは `_gbk` の使用をサポートしていません。例：

  ```sql
  CREATE TABLE t(a CHAR(10) CHARSET BINARY);
  クエリが実行されました。影響を受けた行はありません (0.00 秒)
  INSERT INTO t VALUES (_gbk'啊');
  ERROR 1115 (42000): サポートされていない文字紹介: 'gbk'
  ```

- 現在、`ENUM` および `SET` タイプのバイナリ文字に対して、TiDBはこれらを `utf8mb4` 文字セットとして処理します。

- 文字列の接頭辞に対して `LIKE 'prefix%'` のような条件が含まれ、対象の列がGBK照合（`gbk_bin`または`gbk_chinese_ci`）に設定されている場合、オプティマイザはこの条件を範囲スキャンに変換できません。代わりに、完全スキャンを実行します。その結果、このようなSQLクエリは予期しないリソース消費を引き起こす可能性があります。

## コンポーネントの互換性

- 現在、TiFlashはGBK文字セットをサポートしていません。

- TiDBデータ移行（DM）は、v5.4.0より前のTiDBクラスターにおける `charset=GBK` テーブルの移行をサポートしていません。

- TiDB Lightningは、v5.4.0より前のTiDBクラスターにおける `charset=GBK` テーブルのインポートをサポートしていません。

- v6.1.0より前のTiCDCバージョンは、`charset=GBK` テーブルのレプリケーションをサポートしていません。TiCDCのどのバージョンも、v6.1.0より前のTiDBクラスターにおける `charset=GBK` テーブルのレプリケーションをサポートしていません。

- バックアップ＆リストア（BR）のv5.4.0より前のバージョンは、`charset=GBK` テーブルの復元をサポートしていません。BRのどのバージョンも、v5.4.0より前のTiDBクラスターにおける `charset=GBK` テーブルの復元をサポートしていません。