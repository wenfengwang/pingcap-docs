---
title: データとインデックスの不整合をトラブルシューティングする
summary: データとインデックスの整合性チェックで報告されたエラーの対処法を学びます。
---

# データとインデックスの不整合をトラブルシューティングする

TiDB はトランザクションの実行または [`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md) 文の実行時にデータとインデックスの整合性をチェックします。チェックでレコードのキーと値、および対応するインデックスのキーと値が不整合である（たとえば、インデックスが不足しているなど）場合、TiDB はデータの不整合エラーを報告し、関連するエラーをエラーログにプリントします。

<CustomContent platform="tidb">

このドキュメントではデータの不整合エラーの意味を説明し、整合性チェックをバイパスするためのいくつかの方法を提供します。データの不整合エラーが発生した場合は、PingCAP またはコミュニティから[サポートを取得](/support.md)できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

このドキュメントではデータの不整合エラーの意味を説明し、整合性チェックをバイパスするためのいくつかの方法を提供します。データの不整合エラーが発生した場合は、[TiDB Cloud サポートに連絡](/tidb-cloud/tidb-cloud-support.md)できます。

</CustomContent>

## エラーの説明

データとインデックスの不整合が発生した場合、TiDB のエラーメッセージをチェックして、行データとインデックスデータの間で何が不整合かを知ることができます。また、詳細な調査のために関連するエラーログを確認できます。

### トランザクションの実行中に報告されるエラー

このセクションでは、TiDB がトランザクションを実行する際に報告されるデータの不整合エラーを一覧にし、これらのエラーの意味と例を説明します。

#### エラー 8133

`ERROR 8133 (HY000): data inconsistency in table: t, index: k2, index-count:1 != record-count:0`

このエラーは、表 `t` の `k2` インデックスについて、テーブル内のインデックス数が 1 であり、行レコード数が 0 であることを示しています。数が不整合です。

#### エラー 8138

`ERROR 8138 (HY000): writing inconsistent data in table: t, expected-values:{KindString green} != record-values:{KindString GREEN}`

このエラーは、トランザクションが誤った行値を書こうとしていたことを示しています。書き込もうとしているデータは、エンコード前のデータと一致しないエンコード後のデータです。

#### エラー 8139

`ERROR 8139 (HY000): writing inconsistent data in table: t, index: i1, index-handle:4 != record-handle:3, index: tables.mutation{key:kv.Key{...}}`

このエラーは、書き込もうとしているデータのハンドル（つまり、行データのキー）が不整合であることを示しています。表 `t` のインデックス `i1` において、トランザクションが書き込もうとしている行は、インデックスのキーと値のペアにおいてはハンドルが 4 であり、行レコードのキーと値のペアにおいてはハンドルが 3 です。この行のデータは書き込まれません。

#### エラー 8140

`ERROR 8140 (HY000): writing inconsistent data in table: t, index: i2, col: c1, indexed-value:{KindString hellp} != record-value:{KindString hello}`

このエラーは、トランザクションが書き込もうとしている行のデータがインデックスのデータと一致しないことを示しています。表 `t` のインデックス `i2` において、トランザクションが書き込もうとしている行は、インデックスのキーと値のペアには `hellp` というデータがあり、行レコードのキーと値のペアには `hello` というデータがあります。この行のデータは書き込まれません。

#### エラー 8141

`ERROR 8141 (HY000): assertion failed: key: 7480000000000000405f72013300000000000000f8, assertion: NotExist, start_ts: 430590532931813377, existing start ts: 430590532931551233, existing commit ts: 430590532931551234`

このエラーは、トランザクションがコミットされる際にアサーションが失敗したことを示しています。データとインデックスが整合していると仮定し、TiDB はキー `7480000000000000405f720133000000000000000000f8` が存在しないとアサートしました。しかし、トランザクションがコミットされる際に、TiDB は `start ts` が `430590532931551233` で書き込まれたキーが存在することを見つけました。このキーの Multi-Version Concurrency Control（MVCC）履歴をログに出力します。

### `ADMIN CHECK` で報告されるエラー

このセクションでは、[`ADMIN CHECK`](/sql-statements/sql-statement-admin-check-table-index.md) 文を実行した際に TiDB で発生する可能性のあるデータの不整合エラーを一覧にし、これらのエラーの意味と例を説明します。

#### エラー 8003

`ERROR 8003 (HY000): table count 3 != index(idx) count 2`

このエラーは、[`ADMIN CHECK`](/sql-statements/sql-statement-admin-check-table-index.md) 文を実行したテーブルに、3 つの行キーと値のペアがありながら、インデックスキーと値のペアが 2 つしかないことを示しています。

#### エラー 8134

`ERROR 8134 (HY000): data inconsistency in table: t, index: c2, col: c2, handle: "2", index-values:"KindInt64 13" != record-values:"KindInt64 12", compare err:<nil>`

このエラーは、表 `t` のインデックス `c2` において、列 `c2` の値に次の不整合があることを示しています：

- ハンドルが `2` である行のインデックスキーと値のペアにおいて、列 `c2` の値は `13` です。
- 行レコードのキーと値のペアにおいて、列 `c2` の値は `12` です。

#### エラー 8223

`ERROR 8223 (HY000): data inconsistency in table: t2, index: i1, handle: {hello, hello}, index-values:"" != record-values:"handle: {hello, hello}, values: [KindString hello KindString hello]"`

このエラーは、`index-values` が空であり `record-values` が空でないことを示しています。つまり、行に対応するインデックスが存在しないことを意味します。

## 解決方法

<CustomContent platform="tidb">

データの不整合エラーに遭遇した場合は、自分でエラーを処理する代わりに、すぐにトラブルシューティングのために PingCAP から[サポートを取得](/support.md)してください。アプリケーションが緊急にこのようなエラーをスキップする必要がある場合は、次の方法を使用してチェックをバイパスできます。

</CustomContent>

<CustomContent platform="tidb-cloud">

データの不整合エラーに遭遇した場合は、自分でエラーを処理する代わりに、すぐに[TiDB Cloud サポートに連絡](/tidb-cloud/tidb-cloud-support.md)してください。アプリケーションが緊急にこのようなエラーをスキップする必要がある場合は、次の方法を使用してチェックをバイパスできます。

</CustomContent>

### SQL の書き直し

特定の SQL ステートメントでデータの不整合エラーが発生する場合は、異なる実行オペレータを使用して SQL ステートメントを別の同等の形式に書き直すことで、このエラーをバイパスできます。

### エラーチェックの無効化

トランザクションの実行時に報告される以下のエラーについて、対応するチェックをバイパスできます：

```markdown
- To bypass the checks of errors 8138, 8139, and 8140, configure `set @@tidb_enable_mutation_checker=0`.
- To bypass the checks of error 8141, configure `set @@tidb_txn_assertion_level=OFF`.

> **Note:**
>
> Disabling `tidb_enable_mutation_checker` and `tidb_txn_assertion_level` will bypass the corresponding checks of all SQL statements.

For other errors reported in transaction execution and all errors reported during the execution of the [`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md) statement, you cannot bypass the corresponding check, because the data is already inconsistent.
```

```markdown
- エラー8138、8139、および8140のチェックをバイパスするには、「set @@tidb_enable_mutation_checker=0」を構成してください。
- エラー8141のチェックをバイパスするには、「set @@tidb_txn_assertion_level=OFF」を構成してください。

> **注意:**
>
> `tidb_enable_mutation_checker`と`tidb_txn_assertion_level`を無効にすると、すべてのSQL文の対応するチェックがバイパスされます。

トランザクション実行中に報告されたその他のエラーや[`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md)ステートメントの実行中に報告されたすべてのエラーについては、データが既に不整合であるため、対応するチェックをバイパスすることはできません。
```