---
title: TiFlashのレイトマテリアリゼーション
summary: OLAPシナリオでクエリの高速化のためにTiFlashのレイトマテリアリゼーション機能の使用方法を説明します。

# TiFlashのレイトマテリアリゼーション

> **ノート:**
>
> TiFlashのレイトマテリアリゼーションは[高速スキャンモード](/tiflash/use-fastscan.md)で有効になりません。

TiFlashのレイトマテリアリゼーションはOLAPシナリオでクエリの高速化を図るための最適化手法です。[`tidb_opt_enable_late_materialization`](/system-variables.md#tidb_opt_enable_late_materialization-new-in-v700)システム変数を使用して、TiFlashのレイトマテリアリゼーションを有効または無効にするかを制御できます。

- 無効にすると、`WHERE`句を含む`SELECT`ステートメントを処理する際、TiFlashはクエリに必要なすべての列からデータを読み取り、その後クエリ条件に基づいてデータをフィルタおよび集計します。
- 有効にすると、TiFlashは一部のフィルタ条件をTableScan演算子にプッシュダウンします。すなわち、TiFlashはまずTableScan演算子にプッシュダウンされたフィルタ条件に関連する列データをスキャンし、条件を満たす行をフィルタし、その後これらの行の他の列データをスキャンしてさらなる計算を行うことで、IOスキャンとデータ処理の計算を削減します。

OLAPシナリオで特定のクエリのパフォーマンスを向上させるために、v7.1.0からTiFlashのレイトマテリアリゼーション機能はデフォルトで有効化されています。TiDBオプティマイザは統計およびフィルタ条件に基づいてプッシュダウンするフィルタ条件を決定し、高いフィルトレーション率のフィルタ条件を優先的にプッシュダウンします。詳細なアルゴリズムについては、[RFCドキュメント](https://github.com/pingcap/tidb/tree/master/docs/design/2022-12-06-support-late-materialization.md)を参照してください。

例:

```sql
EXPLAIN SELECT a, b, c FROM t1 WHERE a < 1;
```

```
+-------------------------+----------+--------------+---------------+-------------------------------------------------------+
| id                      | estRows  | task         | access object | operator info                                         |
+-------------------------+----------+--------------+---------------+-------------------------------------------------------+
| TableReader_12          | 12288.00 | root         |               | MppVersion: 1, data:ExchangeSender_11                 |
| └─ExchangeSender_11     | 12288.00 | mpp[tiflash] |               | ExchangeType: PassThrough                             |
|   └─TableFullScan_9     | 12288.00 | mpp[tiflash] | table:t1      | pushed down filter:lt(test.t1.a, 1), keep order:false |
+-------------------------+----------+--------------+---------------+-------------------------------------------------------+
```

この例では、フィルタ条件`a < 1`がTableScan演算子にプッシュダウンされています。TiFlashはまず列`a`からすべてのデータを読み取り、次に条件`a < 1`を満たす行をフィルタします。その後、これらのフィルタされた行から列`b`および`c`を読み取ります。

## TiFlashのレイトマテリアリゼーションの有効化または無効化

デフォルトでは、`tidb_opt_enable_late_materialization`システム変数はセッションレベルとグローバルレベルの両方で`ON`になっており、つまりTiFlashのレイトマテリアリゼーション機能が有効化されています。以下のステートメントを使用して対応する変数情報を表示できます:

```sql
SHOW VARIABLES LIKE 'tidb_opt_enable_late_materialization';
```

```
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| tidb_opt_enable_late_materialization | ON    |
+--------------------------------------+-------+
```

```sql
SHOW GLOBAL VARIABLES LIKE 'tidb_opt_enable_late_materialization';
```

```
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| tidb_opt_enable_late_materialization | ON    |
+--------------------------------------+-------+
```

`tidb_opt_enable_late_materialization`変数をセッションレベルまたはグローバルレベルで変更できます。

- 現在のセッションでTiFlashのレイトマテリアリゼーションを無効にするには、次のステートメントを使用します:

    ```sql
    SET SESSION tidb_opt_enable_late_materialization=OFF;
    ```

- グローバルレベルでTiFlashのレイトマテリアリゼーションを無効にするには、次のステートメントを使用します:

    ```sql
    SET GLOBAL tidb_opt_enable_late_materialization=OFF;
    ```

    この設定後、新しいセッションではデフォルトで`tidb_opt_enable_late_materialization`変数が両方のセッションとグローバルレベルで有効になります。

TiFlashのレイトマテリアリゼーションを有効にするには、次のステートメントを使用します:

```sql
SET SESSION tidb_opt_enable_late_materialization=ON;
```

```sql
SET GLOBAL tidb_opt_enable_late_materialization=ON;
```

## 実装メカニズム

フィルタ条件がTableScan演算子にプッシュダウンされる場合、TableScan演算子の実行プロセスは主に以下の手順で行われます:

1. `＜handle、del_mark、version＞`の3つの列を読み取り、マルチバージョン同時制御（MVCC）のフィルタリングを行い、次にMVCC Bitmapを生成します。
2. フィルタ条件に関連する列を読み取り、条件を満たす行をフィルタし、その後フィルタビットマップを生成します。
3. MVCCビットマップとフィルタビットマップの間で`AND`演算を実行し、最終的なビットマップを生成します。
4. 最終ビットマップに従って残りの列の対応する行を読み取ります。
5. ステップ2および4で読み取ったデータをマージし、その後結果を返します。