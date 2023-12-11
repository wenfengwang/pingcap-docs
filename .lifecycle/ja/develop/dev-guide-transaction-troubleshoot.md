---
title: トランザクションエラーの処理
summary: デッドロックやアプリケーションの再試行エラーなど、トランザクションエラーの処理方法について学びます。

# トランザクションエラーの処理

このドキュメントでは、デッドロックやアプリケーションの再試行エラーなどのトランザクションエラーの処理方法について紹介します。

## デッドロック

アプリケーションで以下のエラーが表示される場合、デッドロックの問題が発生しています。

```sql
ERROR 1213: Deadlock found when trying to get lock; try restarting transaction
```

デッドロックは、2つ以上のトランザクションが互いの獲得済みロックを待っているか、不整合なロックの順序がループし、ロックリソースの待機に繋がることで発生します。

以下は、[`bookshop`](/develop/dev-guide-bookshop-schema-design.md) データベースのテーブル `books` を使用したデッドロックの例です。

まず、テーブル `books` に2つの行を挿入します:

```sql
INSERT INTO books (id, title, stock, published_at) VALUES (1, 'book-1', 10, now()), (2, 'book-2', 10, now());
```

TiDBの悲観的トランザクションモードでは、2つのクライアントがそれぞれ次のステートメントを実行すると、デッドロックが発生します:

| クライアントA                                       | クライアントB                                           |
| -------------------------------------------------- | --------------------------------------------------------|
| BEGIN;                                             |                                                           |
|                                                    | BEGIN;                                                    |
| UPDATE books SET stock=stock-1 WHERE id=1;         |                                                           |
|                                                    | UPDATE books SET stock=stock-1 WHERE id=2;                |
| UPDATE books SET stock=stock-1 WHERE id=2; -- この実行はブロックされる |                                                           |
|                                                    | UPDATE books SET stock=stock-1 WHERE id=1; -- デッドロックエラーが発生 |

クライアントBがデッドロックエラーに遭遇した後、TiDB は自動的にクライアントBのトランザクションをロールバックします。クライアントAの`id=2`の更新は正常に実行されます。その後、`COMMIT` を実行してトランザクションを終了します。

### 解決策1: デッドロックを回避

パフォーマンスを向上させるために、ビジネスロジックやスキーマ設計を調整することで、アプリケーションレベルでデッドロックを回避することができます。前述の例では、クライアントBもクライアントAと同じ更新順序を使用すれば、`id=1`の本をまず更新し、その後 `id=2`の本を更新することでデッドロックを回避できます:

| クライアントA                                                  | クライアントB                                                    |
| ---------------------------------------------------------- | ---------------------------------------------------------------|
| BEGIN;                                                     |                                                                 |
|                                                            | BEGIN;                                                          |
| UPDATE books SET stock=stock-1 WHERE id=1;                 |                                                                 |
|                                                            | UPDATE books SET stock=stock-1 WHERE id=1; -- ブロックされる   |
| UPDATE books SET stock=stock-1 WHERE id=2;                 |                                                                 |
| COMMIT;                                                    |                                                                 |
|                                                            | UPDATE books SET stock=stock-1 WHERE id=2;                      |
|                                                            | COMMIT;                                                         |

または、1つのSQLステートメントで2冊の本を更新することもできます。これによりデッドロックが回避され、より効率的に実行されます:

```sql
UPDATE books SET stock=stock-1 WHERE id IN (1, 2);
```

### 解決策2: トランザクションの粒度を減らす

1回のトランザクションで1冊の本のみを更新すると、デッドロックも回避できます。ただし、トランザクションの粒度が小さすぎるとパフォーマンスに影響する可能性があります。

### 解決策3: 楽観的トランザクションを使用

楽観的トランザクションモデルではデッドロックは発生しません。ただしアプリケーションで失敗した場合の楽観的トランザクション再試行ロジックを追加する必要があります。詳細は、[アプリケーションの再試行とエラーの処理](#アプリケーションの再試行とエラーの処理)を参照してください。

### 解決策4: 再試行

エラーメッセージで提案されるように、アプリケーションに再試行ロジックを追加します。詳細は、[アプリケーションの再試行とエラーの処理](#アプリケーションの再試行とエラーの処理)を参照してください。

## アプリケーションの再試行とエラーの処理

TiDB は可能な限り MySQL と互換性がありますが、その分散システムの性質によりいくつかの違いが生じます。その1つはトランザクションモデルです。

開発者がデータベースに接続するために使用するアダプターやORMは、MySQL や Oracle などの伝統的なデータベース用に調整されています。これらのデータベースでは、通常、デフォルトの分離レベルでトランザクションがコミットに失敗することは稀であり、再試行メカニズムは必要ありません。トランザクションがコミットに失敗すると、これらのクライアントはエラーが発生し例外として扱われ、中止されます。

TiDB は MySQL などの伝統的なデータベースとは異なり、楽観的トランザクションモデルを使用しており、コミットの失敗を避けるためには、関連する例外を処理するメカニズムをアプリケーションに追加する必要があります。

以下は、アプリケーションレベルでの再試行を実装するPython擬似コードの例です。ドライバーやORMが高度な再試行ロジックを実装する必要はありません。どのプログラミング言語や環境でも使用できます。

再試行ロジックは以下のルールに従う必要があります:

- 失敗した再試行の回数が `max_retries` 制限に達した場合はエラーをスローします。
- SQL実行の例外をキャッチするために `try ... catch ...` を使用します。以下のエラーが発生した場合に再試行し、その他のエラーが発生した場合はロールバックします。
    - `Error 8002: can not retry select for update statement`: SELECT FOR UPDATE の書き込み競合エラー
    - `Error 8022: Error: KV error safe to retry`: トランザクションコミットに失敗したエラー。
    - `Error 8028: Information schema is changed during the execution of the statement`: テーブルスキーマがDDL操作によって変更され、トランザクションコミットでエラーが発生した。
    - `Error 9007: Write conflict`: 書き込み競合エラー。楽観的トランザクションモードを使用する際に、複数のトランザクションが同じデータ行を変更した場合に通常発生する。
- `try` ブロックの最後でトランザクションを `COMMIT` します。

<CustomContent platform="tidb">

エラーコードの詳細については、[エラーコードとトラブルシューティング](/error-codes.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

エラーコードの詳細については、[エラーコードとトラブルシューティング](https://docs.pingcap.com/tidb/stable/error-codes) を参照してください。

</CustomContent>

```python
while True:
    n++
    if n == max_retries:
        raise("did not succeed within #{n} retries")
    try:
        connection.execute("your sql statement here")
        connection.exec('COMMIT')
        break
    catch error:
        if (error.code != "9007" && error.code != "8028" && error.code != "8002" && error.code != "8022"):
            raise error
        else:
            connection.exec('ROLLBACK')

            # アプリケーション側の再試行が必要なエラーをキャプチャし、
            # 短い期間待機し、各トランザクションの失敗ごとに待機時間を指数関数的に増やします
            sleep_ms = int(((1.5 ** n) + rand) * 100)
            sleep(sleep_ms) # sleep() がミリ秒単位であることを確認してください
```

> **注意:**
>
> `Error 9007: Write conflict` が頻繁に発生する場合は、スキーマ設計やワークロードのデータアクセスパターンをチェックし、競合の原因を特定し、よりよい設計で競合を回避する必要があるかもしれません。

<CustomContent platform="tidb">

トランザクションの競合をトラブルシューティングして解決する方法については、[ロック競合のトラブルシューティング](/troubleshoot-lock-conflicts.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

トランザクションの競合をトラブルシューティングして解決する方法については、[ロック競合のトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-lock-conflicts) を参照してください。

</CustomContent>

## 関連情報

<CustomContent platform="tidb">

- [楽観的トランザクションの書き込み競合のトラブルシューティング](/troubleshoot-write-conflicts.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

- [楽観的トランザクションの書き込み競合のトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-write-conflicts)

</CustomContent>