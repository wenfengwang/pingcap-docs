---
title: 楽観的トランザクションでの書き込み競合のトラブルシューティング
summary: 楽観的トランザクションでの書き込み競合の原因と解決策について学びます。

# 楽観的トランザクションでの書き込み競合のトラブルシューティング

このドキュメントでは、楽観的トランザクションでの書き込み競合の原因と解決策について紹介します。

TiDB v3.0.8より前は、TiDBはデフォルトで楽観的トランザクションモデルを使用していました。このモデルでは、トランザクションの実行中に競合をチェックしません。代わりに、トランザクションが最終的にコミットされる際に、2段階コミット（2PC）がトリガーされ、TiDBが書き込み競合をチェックします。書き込み競合が存在し、自動リトライメカニズムが有効になっている場合、TiDBは限られた回数内でトランザクションを再試行します。再試行が成功した場合または再試行回数の上限に達した場合、TiDBはトランザクションの実行結果をクライアントに返します。したがって、TiDBクラスタに多くの書き込み競合が存在する場合、処理時間が長くなる可能性があります。

## 書き込み競合の原因

TiDBは、[Percolator](https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Peng.pdf)トランザクションモデルを使用してトランザクションを実装しています。`percolator` は一般的に2PCの実装です。詳細な2PCのプロセスについては、[TiDB楽観的トランザクションモデル](/optimistic-transaction.md)を参照してください。

クライアントがTiDBに`COMMIT`リクエストを送信すると、TiDBは2PCプロセスを開始します:

1. TiDBは、トランザクション内のすべてのキーから1つのキーをトランザクションの主キーとして選択します。
2. TiDBは、このコミットに関与するすべてのTiKVリージョンに`prewrite`リクエストを送信します。TiKVはすべてのキーが正常にプレビューできるかどうかを判断します。
3. TiDBは、すべての`prewrite`リクエストが成功した結果を受け取ります。
4. TiDBは、PDから`commit_ts`を取得します。
5. TiDBは、トランザクションの主キーを含むTiKVリージョンに`commit`リクエストを送信します。TiKVは`commit`リクエストを受け取った後、データの有効性を確認し、`prewrite`段階で残されたロックをクリアします。
6. `commit`リクエストが成功すると、TiDBはクライアントに成功を返します。

書き込み競合は`prewrite`段階で発生します。トランザクションは、別のトランザクションが現在のキーを書き込んでいることを見つけると（`data.commit_ts` > `txn.start_ts`）、書き込み競合が発生します。

## 書き込み競合の検出

TiDB Grafanaパネルで、**KV Errors**の下で次の監視メトリクスをチェックしてください:

* **KV Backoff OPS**: TiKVが返すエラーメッセージのカウント（1秒あたり）を示します。

    ![kv-backoff-ops](/media/troubleshooting-write-conflict-kv-backoff-ops.png)

    `txnlock`メトリックは`書き込み-書き込み`競合を示します。`txnLockFast`メトリックは`読み取り-書き込み`競合を示します。

* **Lock Resolve OPS**：秒ごとのトランザクション競合に関連するアイテムのカウントを示します:

    ![lock-resolve-ops](/media/troubleshooting-write-conflict-lock-resolve-ops.png)

    - `not_expired`：ロックのTTLが期限切れでないことを示します。期限切れまで、競合トランザクションはロックを解決できません。
    - `wait_expired`：トランザクションはロックの期限切れまで待機する必要があります。
    - `expired`：ロックのTTLが期限切れであることを示します。その後、競合トランザクションはこのロックを解決できます。

* **KV Retry Duration**：KVリクエストを再送する時間を示します:

    ![kv-retry-duration](/media/troubleshooting-write-conflict-kv-retry-duration.png)

TiDBログで`[kv:9007]Write conflict`をキーワードとして検索することもできます。このキーワードも、クラスタ内に書き込み競合が存在することを示します。

## 書き込み競合の解決

クラスタに多くの書き込み競合が存在する場合は、書き込み競合のキーと原因を特定し、アプリケーションロジックを変更して書き込み競合を回避することがお勧めです。書き込み競合がクラスタに存在する場合、TiDBログファイルに以下のようなログを見ることができます:

```log
[2020/05/12 15:17:01.568 +08:00] [WARN] [session.go:446] ["commit failed"] [conn=3] ["finished txn"="Txn{state=invalid}"] [error="[kv:9007]Write conflict, txnStartTS=416617006551793665, conflictStartTS=416617018650001409, conflictCommitTS=416617023093080065, key={tableID=47, indexID=1, indexValues={string, }} primary={tableID=47, indexID=1, indexValues={string, }} [try again later]"]
```

上記ログの説明は次のとおりです:

* `[kv:9007]Write conflict`: 書き込み-書き込み競合を示します。
* `txnStartTS=416617006551793665`: 現在のトランザクションの`start_ts`を示します。`pd-ctl`ツールを使用して`start_ts`を物理的な時間に変換できます。
* `conflictStartTS=416617018650001409`: 書き込み競合トランザクションの`start_ts`を示します。
* `conflictCommitTS=416617023093080065`: 書き込み競合トランザクションの`commit_ts`を示します。
* `key={tableID=47, indexID=1, indexValues={string, }}`: 書き込み競合のキーを示します。`tableID`は書き込み競合テーブルのIDを示します。`indexID`は書き込み競合インデックスのIDを示します。書き込み競合キーがレコードキーの場合、ログには`handle=x`が印刷され、競合が発生しているレコード（行）が示されます。`indexValues`は競合しているインデックスの値を示します。
* `primary={tableID=47, indexID=1, indexValues={string, }}`: 現在のトランザクションの主キー情報を示します。

`pd-ctl`ツールを使用してタイムスタンプを読みやすい時間に変換できます:

{{< copyable "" >}}

```shell
tiup ctl:v<CLUSTER_VERSION> pd -u https://127.0.0.1:2379 tso {TIMESTAMP}
```

`tableID`を使用して関連するテーブルの名前を検索できます:

{{< copyable "" >}}

```shell
curl http://{TiDBIP}:10080/db-table/{tableID}
```

`indexID`およびテーブル名を使用して関連するインデックスの名前を検索できます:

{{< copyable "sql" >}}

```sql
SELECT * FROM INFORMATION_SCHEMA.TIDB_INDEXES WHERE TABLE_SCHEMA='{db_name}' AND TABLE_NAME='{table_name}' AND INDEX_ID={indexID};
```

また、TiDB v3.0.8以降では、悲観的トランザクションがデフォルトモードとなりました。悲観的トランザクションモードでは、各DMLステートメントが実行中に関連するキーに悲観的ロックを書き込みます。この悲観的ロックは、他のトランザクションが同じキーを変更するのを防ぐため、トランザクションの`prewrite`段階で書き込み競合が発生しないようにします。