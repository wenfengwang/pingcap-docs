---
title: TiFlash クエリ結果のマテリアライズ
summary: TiFlash のクエリ結果をトランザクションで保存する方法について学びます。
---

# TiFlash クエリ結果のマテリアライズ

このドキュメントでは、TiFlash のクエリ結果を指定された TiDB テーブルに `INSERT INTO SELECT` トランザクションで保存する方法について紹介します。

v6.5.0 から TiDB は、TiFlash クエリ結果をテーブルに保存すること、つまり TiFlash クエリ結果のマテリアライズをサポートしています。`INSERT INTO SELECT` 文の実行中に、TiDB が `SELECT` サブクエリを TiFlash にプッシュダウンした場合、TiFlash クエリ結果を `INSERT INTO` 句で指定された TiDB テーブルに保存できます。v6.5.0 より古い TiDB のバージョンでは、TiFlash クエリ結果は読み取り専用なので、TiFlash クエリ結果を保存するにはアプリケーションレベルで取得し、別のトランザクションまたはプロセスで保存する必要があります。

> **ノート:**
>
> デフォルトでは（[`tidb_allow_mpp = ON`](/system-variables.md#tidb_allow_mpp-new-in-v50)）、オプティマイザーは [SQL モード](/sql-mode.md) と TiFlash レプリカのコスト見積もりに基づいてクエリを TiFlash にプッシュダウンするかどうかをインテリジェントに決定します。
>
> - 現在のセッションの [SQL モード](/sql-mode.md) が厳格でない場合（つまり、`sql_mode` の値が `STRICT_TRANS_TABLES` および `STRICT_ALL_TABLES` を含まない場合）、オプティマイザーは TiFlash レプリカのコスト見積もりに基づいて `INSERT INTO SELECT` の `SELECT` サブクエリを TiFlash にプッシュダウンするかどうかをインテリジェントに決定します。このモードでは、オプティマイザーのコスト見積もりを無視し、クエリを強制的に TiFlash にプッシュダウンするには、[`tidb_enforce_mpp`](/system-variables.md#tidb_enforce_mpp-new-in-v51) システム変数を `ON` に設定できます。
> - 現在のセッションの [SQL モード](/sql-mode.md) が厳格である場合（つまり、`sql_mode` の値が `STRICT_TRANS_TABLES` または `STRICT_ALL_TABLES` を含む場合）、`INSERT INTO SELECT` の `SELECT` サブクエリは TiFlash にプッシュダウンできません。

`INSERT INTO SELECT` の構文は次のようになります。

```sql
INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    SELECT ...
    [ON DUPLICATE KEY UPDATE assignment_list]value:
    {expr | DEFAULT}

assignment:
    col_name = valueassignment_list:
    assignment [, assignment] ...
```

例えば、次の `INSERT INTO SELECT` 文を使用して、テーブル `t1` の `SELECT` 句からのクエリ結果をテーブル `t2` に保存できます。

```sql
INSERT INTO t2 (name, country)
SELECT app_name, country FROM t1;
```

## 典型的で推奨される使用シナリオ

- 効率的な BI ソリューション

    多くの BI アプリケーションでは、分析クエリリクエストが非常に重くなります。たとえば、多くのユーザーが同時にレポートにアクセスして更新する場合、BI アプリケーションは多くの同時クエリリクエストを処理する必要があります。このような状況に効果的に対処するために、`INSERT INTO SELECT` を使用して、レポートのクエリ結果を TiDB テーブルに保存できます。その後エンドユーザーは、レポートが更新される際に直接結果テーブルからデータをクエリできるため、複数回の反復計算と分析を回避できます。同様に、履歴的な分析結果を保存することで、長期間の履歴データ分析の計算量をさらに削減できます。たとえば、1 日の売上利益を分析するためのレポート `A` がある場合、`INSERT INTO SELECT` を使用してレポート `A` の結果を結果テーブル `T` に保存できます。その後、前月の売上利益を分析するためのレポート `B` を生成する必要がある場合は、テーブル `T` の日次分析結果を直接使用できます。これにより計算量が大幅に削減されるだけでなく、クエリ応答速度が向上し、システム負荷が軽減されます。

- TiFlash でオンラインアプリケーションを提供する

    TiFlash がサポートする同時リクエスト数はデータ量とクエリの複雑さに依存しますが、通常は 100 QPS を超えません。TiFlash クエリ結果を保存し、その結果テーブルを使用して高い同時リクエストをサポートできます。結果テーブルのデータはバックグラウンドで低頻度で更新できます（たとえば、0.5 秒の間隔で）、これは TiFlash の同時性制限を大幅に下回りつつ、データの新鮮さを維持します。

## 実行プロセス

* `INSERT INTO SELECT` 文の実行中、TiFlash はまず `SELECT` 句のクエリ結果をクラスタ内の TiDB サーバーに返し、その後結果をターゲットテーブル（TiFlash レプリカを持つことができる）に書き込みます。
* `INSERT INTO SELECT` 文の実行は ACID 特性を保証します。

## 制限事項

<CustomContent platform="tidb">

* `INSERT INTO SELECT` 文の TiDB メモリ制限は、システム変数 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を使用して調整できます。v6.5.0 から、トランザクションメモリサイズを制御するために [`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit) を使用することは推奨されません。

    詳細については、[TiDB メモリ制御](/configure-memory-usage.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

* `INSERT INTO SELECT` 文の TiDB メモリ制限は、システム変数 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を使用して調整できます。v6.5.0 から、トランザクションメモリサイズを制御するために[`txn-total-size-limit`](https://docs.pingcap.com/tidb/stable/tidb-configuration-file#txn-total-size-limit) を使用することは推奨されません。

    詳細については、[TiDB メモリ制御](https://docs.pingcap.com/tidb/stable/configure-memory-usage)を参照してください。

</CustomContent>

* TiDB は `INSERT INTO SELECT` 文の同時性にハード制限はありませんが、次の実践を検討することをお勧めします:

    * 「書き込みトランザクション」が大きい場合（たとえば 1 GiB に近い）、同時性を 10 までに制御することを推奨します。
    * 「書き込みトランザクション」が小さい場合（たとえば 100 MiB より少ない場合）、同時性を 30 までに制御することを推奨します。
    * テスト結果と具体的な状況に基づいて同時性を決定します。

## 例

データ定義:

```sql
CREATE TABLE detail_data (
    ts DATETIME,                -- 手数料発生時刻
    customer_id VARCHAR(20),    -- 顧客 ID
    detail_fee DECIMAL(20,2));  -- 手数料の金額

CREATE TABLE daily_data (
    rec_date DATE,              -- データ収集日
    customer_id VARCHAR(20),    -- 顧客 ID
    daily_fee DECIMAL(20,2));   -- 1 日あたりの手数料

ALTER TABLE detail_data SET TIFLASH REPLICA 1;
ALTER TABLE daily_data SET TIFLASH REPLICA 1;

-- ... (detail_data テーブルの更新が続く)
INSERT INTO detail_data(ts,customer_id,detail_fee) VALUES
('2023-1-1 12:2:3', 'cus001', 200.86),
('2023-1-2 12:2:3', 'cus002', 100.86),
('2023-1-3 12:2:3', 'cus002', 2200.86),
('2023-1-4 12:2:3', 'cus003', 2020.86),
('2023-1-5 12:2:3', 'cus003', 1200.86),
('2023-1-6 12:2:3', 'cus002', 20.86);
```

日次分析結果を保存:

```sql
SET @@tidb_enable_tiflash_read_for_write_stmt=ON;

INSERT INTO daily_data (rec_date, customer_id, daily_fee)
SELECT DATE(ts), customer_id, sum(detail_fee) FROM detail_data WHERE DATE(ts) = CURRENT_DATE() GROUP BY DATE(ts), customer_id;
```

日次分析データを基に月次データを分析:

```sql
SELECT MONTH(rec_date), customer_id, sum(daily_fee) FROM daily_data GROUP BY MONTH(rec_date), customer_id;
```

上記の例では、日次分析の結果を材料化し、それを日次結果テーブルに保存し、その基に月次データ分析を加速することで、データ分析の効率を向上させています。