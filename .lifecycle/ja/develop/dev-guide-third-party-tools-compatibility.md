---
title: サードパーティツールとの既知の非互換性に関する問題
summary: テスト中に発見されたTiDBのサードパーティツールとの非互換性について説明します。
---

# サードパーティツールとの既知の非互換性に関する問題

> **Note:**
>
> [未サポートの機能](/mysql-compatibility.md#unsupported-features) セクションには、以下が含まれます:
>
> - ストアドプロシージャとファンクション
> - トリガ
> - イベント
> - ユーザ定義ファンクション
> - `SPATIAL` ファンクション、データ型、およびインデックス
> - `XA` 構文
>
> 上記の未サポートの機能は想定された動作であり、このドキュメントにはリストされていません。詳細については、[MySQL 互換性](/mysql-compatibility.md) を参照してください。

本文書にリストされている非互換性の問題は、TiDB でサポートされる一部の[サードパーティツール](/develop/dev-guide-third-party-tools-compatibility.md)で見つかります。

## 一般的な非互換性

### `SELECT CONNECTION_ID()` は TiDB で 64 ビット整数を返します

**内容**

`SELECT CONNECTION_ID()` 関数は TiDB で 2199023260887 のような 64 ビット整数を返しますが、MySQL では 391650 のような 32 ビット整数を返します。

**回避方法**

TiDB アプリケーションでは、データオーバーフローを避けるため、`SELECT CONNECTION_ID()` の結果を 64 ビット整数または文字列型で保存する必要があります。例えば、Java では `Long` または `String` を使用し、JavaScript または TypeScript では `string` を使用できます。

### TiDB は `Com_*` カウンタを維持しません

**内容**

MySQL は [Com_ で始まるサーバーステータス変数](https://dev.mysql.com/doc/refman/8.0/en/server-status-variables.html#statvar_Com_xxx) のシリースを維持して、データベースで実行した操作の総数を追跡します。例えば、`Com_select` は、データベースが最後に起動されてから実行された `SELECT` ステートメントの総数を記録します（ステートメントが成功していなくても記録されます）。TiDB はこれらの変数を維持しません。TiDB と MySQL の間の違いを確認するには、ステートメント [<code>SHOW GLOBAL STATUS LIKE 'Com_%'</code>](/sql-statements/sql-statement-show-status.md) を使用できます。

**回避方法**

<CustomContent platform="tidb">

これらの変数を使用しないでください。共通のシナリオは監視です。TiDB は見やすく、サーバーステータス変数のクエリは不要です。カスタム監視ツールについては、[TiDB 監視フレームワーク概要](/tidb-monitoring-framework.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

これらの変数を使用しないでください。共通のシナリオは監視です。TiDB Cloud は見やすく、サーバーステータス変数のクエリは不要です。詳細については、[TiDB クラスタの監視](/tidb-cloud/monitor-tidb-cluster.md) を参照してください。

</CustomContent>

### TiDB はエラーメッセージで `TIMESTAMP` と `DATETIME` を区別します

**内容**

TiDB のエラーメッセージは `TIMESTAMP` と `DATETIME` を区別しますが、MySQL は区別せず、すべてを `DATETIME` として返します。つまり、MySQL は `TIMESTAMP` 型のエラーメッセージを誤って `DATETIME` 型に変換します。

**回避方法**

<CustomContent platform="tidb">

エラーメッセージを文字列マッチングに使用しないでください。トラブルシューティングには[エラーコード](/error-codes.md)を使用してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

エラーメッセージを文字列マッチングに使用しないでください。トラブルシューティングには[エラーコード](https://docs.pingcap.com/tidb/stable/error-codes)を使用してください。

</CustomContent>

### TiDB は `CHECK TABLE` ステートメントをサポートしません

**内容**

`CHECK TABLE` ステートメントは TiDB ではサポートされていません。

**回避方法**

データの整合性とそれに対応するインデックスを確認するには、TiDB で [`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md) ステートメントを使用できます。

## MySQL JDBC との互換性

テストバージョンは MySQL Connector/J 8.0.29 です。

### デフォルトの照合順序が一貫していません

**内容**

MySQL Connector/J の照合順序はクライアント側に保存され、サーバーバージョンで区別されます。

以下の表は、キャラクタセットのクライアント側のデフォルト照合順序とサーバー側のデフォルト照合順序の一貫性に関する既知の不一致を示しています:

| キャラクタ | クライアント側のデフォルト照合順序 | サーバー側のデフォルト照合順序 |
| ------- | -------------------- | ------------- |
| `ascii`   | `ascii_general_ci`   | `ascii_bin`   |
| `latin1`  | `latin1_swedish_ci`  | `latin1_bin`  |
| `utf8mb4` | `utf8mb4_0900_ai_ci` | `utf8mb4_bin` |

**回避方法**

照合順序を手動で設定し、クライアント側のデフォルト照合順序に依存しないでください。クライアント側のデフォルト照合順序は、MySQL Connector/J 設定ファイルにより保存されます。

### `NO_BACKSLASH_ESCAPES` パラメータが効果を持たない

**内容**

TiDB では、`NO_BACKSLASH_ESCAPES` パラメータを `\` 文字をエスケープせずに使用できません。詳細は、この[issue](https://github.com/pingcap/tidb/issues/35302)を追跡してください。

**回避方法**

TiDB では、`\` をエスケープせずに `NO_BACKSLASH_ESCAPES` を使用しないでください。SQL ステートメントでは `\\` を使用してください。

### `INDEX_USED` 関連のパラメータがサポートされていません

**内容**

TiDB はプロトコルで `SERVER_QUERY_NO_GOOD_INDEX_USED` および `SERVER_QUERY_NO_INDEX_USED` パラメータを設定しません。これにより、以下のパラメータが実際の状況と不整合して返される可能性があります:

- `com.mysql.cj.protocol.ServerSession.noIndexUsed()`
- `com.mysql.cj.protocol.ServerSession.noGoodIndexUsed()`

**回避方法**

TiDB で `noIndexUsed()` および `noGoodIndexUsed()` 関数を使用しないでください。

### `enablePacketDebug` パラメータはサポートされていません

**内容**

TiDB は [enablePacketDebug](https://dev.mysql.com/doc/connector-j/en/connector-j-connp-props-debugging-profiling.html) パラメータをサポートしません。これは、データパケットのバッファを保持するために使用される MySQL Connector/J パラメータであり、接続が予期せず閉じられる可能性があります。**ON にしないで**ください。

**回避方法**

TiDB で `enablePacketDebug` パラメータを設定しないでください。

### UpdatableResultSet はサポートされていません

**内容**

TiDB では、`UpdatableResultSet` はサポートされていません。 `ResultSet.CONCUR_UPDATABLE` パラメータを指定しないでくださいし、`ResultSet` 内でデータを更新しないでください。

**回避方法**

トランザクションによってデータの整合性を確保するため、データを更新するには `UPDATE` ステートメントを使用できます。

## MySQL JDBC のバグ

### `useLocalTransactionState` と `rewriteBatchedStatements` を同時に true に設定すると、トランザクションがコミットまたはロールバックに失敗する可能性があります

**内容**

`useLocalTransactionState` および `rewriteBatchedStatements` パラメータが同時に `true` に設定されると、トランザクションがコミットに失敗する可能性があります。[このコード](https://github.com/Icemap/tidb-java-gitpod/tree/reproduction-local-transaction-state-txn-error)で再現できます。

**回避方法**

> **Note:**
>
> このバグは MySQL JDBC に報告されています。プロセスを追跡するためには、この[バグレポート](https://bugs.mysql.com/bug.php?id=108643) をフォローできます。

`useLocalTransactionState` をオンにしないでください。これにより、トランザクションがコミットまたはロールバックされない可能性があります。

### コネクタが 5.7.5 より前のサーバーバージョンと互換性がない

**内容**

MySQL Connector/J 8.0.29 を MySQL server < 5.7.5 または MySQL server < 5.7.5 プロトコルを使用しているデータベース（TiDB v6.3.0 より前など）と使用すると、特定の条件下でデータベース接続が停止する可能性があります。詳細については、[Bug Report](https://bugs.mysql.com/bug.php?id=106252) を参照してください。

**回避方法**

これは既知の問題です。2022 年 10 月 12 日現在、MySQL Connector/J でこの問題は修正されていません。

TiDB では以下の方法で修正しています:

- クライアント側: このバグは **pingcap/mysql-connector-j** で修正されており、公式の MySQL Connector/J の代わりに [pingcap/mysql-connector-j](https://github.com/pingcap/mysql-connector-j) を使用できます。
- サーバー側: この互換性の問題は TiDB v6.3.0 以降で修正されていますので、サーバーを v6.3.0 またはそれ以降のバージョンにアップグレードできます。

## Sequelize との互換性

このセクションに記載されている互換性情報は、[Sequelize v6.32.1](https://www.npmjs.com/package/sequelize/v/6.32.1) に基づいています。

テスト結果によると、TiDB は Sequelize の大部分の機能をサポートしています（[dialect として `MySQL` を使用](https://sequelize.org/docs/v6/other-topics/dialect-specific-things/#mysql)）。

未サポートの機能は次のとおりです:

- [`GEOMETRY`](https://github.com/pingcap/tidb/issues/6347) はサポートされていません。
- 整数主キーの変更はサポートされていません。
- `PROCEDURE`はサポートされていません。
- `READ-UNCOMMITTED`および`SERIALIZABLE`な分離レベルはサポートされていません。[隔離レベル](/system-variables.md#transaction_isolation)を参照してください。
- デフォルトではカラムの`AUTO_INCREMENT`属性の変更は許可されていません。
- `FULLTEXT`、`HASH`、`SPATIAL`インデックスはサポートされていません。
- `sequelize.queryInterface.showIndex(Model.tableName);`はサポートされていません。
- `sequelize.options.databaseVersion`はサポートされていません。
- [`queryInterface.addColumn`](https://sequelize.org/api/v6/class/src/dialects/abstract/query-interface.js~queryinterface#instance-method-addColumn)を使用して外部キー参照を追加することはサポートされていません。

### 整数主キーの変更はサポートされていません

**説明**

整数主キーの変更はサポートされていません。TiDBは、主キーが整数型の場合、データの組織化のために主キーを使用します。詳細については、[Issue #18090](https://github.com/pingcap/tidb/issues/18090)および[Clustered Indexes](/clustered-indexes.md)を参照してください。

### `READ-UNCOMMITTED`および`SERIALIZABLE`の分離レベルはサポートされていません

**説明**

TiDBは`READ-UNCOMMITTED`および`SERIALIZABLE`の分離レベルをサポートしていません。分離レベルが`READ-UNCOMMITTED`または`SERIALIZABLE`に設定されている場合、TiDBはエラーをスローします。

**回避方法**

TiDBがサポートする分離レベル`REPEATABLE-READ`または`READ-COMMITTED`のみを使用してください。

TiDBを`SERIALIZABLE`分離レベルを設定する他のアプリケーションとの互換性を持つが、`SERIALIZABLE`に依存しないようにする場合は、[`tidb_skip_isolation_level_check`](/system-variables.md#tidb_skip_isolation_level_check)を`1`に設定できます。その場合、TiDBはサポートされていない分離レベルエラーを無視します。

### デフォルトではカラムの`AUTO_INCREMENT`属性の変更は許可されていません

**説明**

`ALTER TABLE MODIFY`または`ALTER TABLE CHANGE`コマンドを使用してカラムの`AUTO_INCREMENT`属性を追加または削除することはデフォルトでは許可されていません。

**回避方法**

[`AUTO_INCREMENT`の制限事項](/auto-increment.md#restrictions)を参照してください。

`AUTO_INCREMENT`属性の削除を許可するには、`@@tidb_allow_remove_auto_inc`を`true`に設定してください。

### `FULLTEXT`、`HASH`、および`SPATIAL`インデックスはサポートされていません

**説明**

`FULLTEXT`、`HASH`、および`SPATIAL`インデックスはサポートされていません。