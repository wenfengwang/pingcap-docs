---
title: TiDB 5.4.3 リリースノート
---

# TiDB 5.4.3 リリースノート

リリース日: 2022年10月13日

TiDB バージョン: 5.4.3

## 改善点

+ TiKV

    - `RocksDB` の書き込みスタール設定のフロー制御閾値よりも小さい値に設定するサポート [#13467](https://github.com/tikv/tikv/issues/13467)
    - Raftstore が1つのピアが到達不能になった後に多くのメッセージをブロードキャストすることを避けるために `unreachable_backoff` アイテムを設定するサポート [#13054](https://github.com/tikv/tikv/issues/13054)

+ ツール

    + TiDB Lightning

        - `Scatter Region` をバッチモードに最適化して `Scatter Region` プロセスの安定性を向上させるためのサポート [#33618](https://github.com/pingcap/tidb/issues/33618)

    + TiCDC

        - 複数のリージョンシナリオでの実行時コンテキスト切り替えによるパフォーマンスオーバーヘッドを削減するサポート [#5610](https://github.com/pingcap/tiflow/issues/5610)

## バグ修正

+ TiDB

    - `SHOW CREATE PLACEMENT POLICY` の出力が不正である問題を修正 [#37526](https://github.com/pingcap/tidb/issues/37526)
    - クラスターの PD ノードが置換された後、一部の DDL ステートメントが一定期間スタックする可能性がある問題を修正 [#33908](https://github.com/pingcap/tidb/issues/33908)
    - アイドル接続に即座に `KILL TIDB` が有効にならない問題を修正 [#24031](https://github.com/pingcap/tidb/issues/24031)
    - `INFORMSTION_SCHEMA.COLUMNS` システムテーブルをクエリする際に `DATA_TYPE` および `COLUMN_TYPE` 列で不正な結果が返される問題を修正 [#36496](https://github.com/pingcap/tidb/issues/36496)
    - TiDB Binlog が有効になっている場合、`ALTER SEQUENCE` ステートメントを実行すると誤ったメタデータバージョンが発生し、Drainer が終了する可能性がある問題を修正 [#36276](https://github.com/pingcap/tidb/issues/36276)
    - `UNION` 演算子が予期しない空の結果を返す可能性がある問題を修正 [#36903](https://github.com/pingcap/tidb/issues/36903)
    - 静的パーティションプルーンモードで、テーブルが空の場合に集約条件を持つ SQL ステートメントが誤った結果を返す可能性がある問題を修正 [#35295](https://github.com/pingcap/tidb/issues/35295)
    - `UPDATE` ステートメントを実行する際に TiDB がパニックを起こす可能性がある問題を修正 [#32311](https://github.com/pingcap/tidb/issues/32311)
    - `UnionScan` 演算子が順序を維持できないため、誤ったクエリ結果が発生する問題を修正 [#33175](https://github.com/pingcap/tidb/issues/33175)
    - `BIT` 型のインデックスを使用した場合に TiDB が誤った結果を返す可能性がある問題を修正 [#33067](https://github.com/pingcap/tidb/issues/33067)

+ TiKV

    - PD リーダーが切り替わるか、PD が再起動された後にクラスターで連続した SQL 実行エラーが発生する問題を修正 [#12934](https://github.com/tikv/tikv/issues/12934)
        - 原因: この問題は TiKV のバグによるもので、TiKV が PD クライアントに対するハートビート情報の送信に失敗した後、TiKV が PD クライアントに再接続するまでハートビート情報の再送信をリトライしないために発生します。その結果、失敗した TiKV ノードのリージョン情報が古くなり、TiDB が最新のリージョン情報を取得できなくなり、SQL 実行エラーが発生します。
        - 影響を受けるバージョン: v5.3.2 および v5.4.2。この問題は v5.3.3 および v5.4.3 で修正されています。v5.4.2 を使用している場合、クラスターを v5.4.3 にアップグレードできます。
        - 回避策: アップグレードの他に、PD にリージョンハートビートを送信できない TiKV ノードを再起動することで、リージョンハートビートを送信しない状態になるまで再起動してください。
    - TiKV がウェブアイデンティティプロバイダからエラーを受け取り、デフォルトプロバイダにフォールバックする際にアクセスが拒否されるエラーを修正 [#13122](https://github.com/tikv/tikv/issues/13122)
    - PD クライアントがデッドロックを引き起こす可能性がある問題を修正 [#13191](https://github.com/tikv/tikv/issues/13191)
    - Raftstore がビジーな場合にリージョンが重複する可能性がある問題を修正 [#13160](https://github.com/tikv/tikv/issues/13160)

+ PD

    - PD がダッシュボードプロキシリクエストを正しく処理できない問題を修正 [#5321](https://github.com/tikv/pd/issues/5321)
    - PD リーダーの移行後に削除された墓石ストアが再度表示される問題を修正 [#4941](https://github.com/tikv/pd/issues/4941)
    - TiFlash の learner レプリカが作成されない問題を修正 [#5401](https://github.com/tikv/pd/issues/5401)

+ TiFlash

    - `format` 関数が `Data truncated` エラーを返す可能性がある問題を修正 [#4891](https://github.com/pingcap/tiflash/issues/4891)
    - パラレル集約にエラーがあるため TiFlash がクラッシュする可能性がある問題を修正 [#5356](https://github.com/pingcap/tiflash/issues/5356)
    - `NULL` 値を含む列で主キーを作成した後に発生するパニックを修正 [#5859](https://github.com/pingcap/tiflash/issues/5859)

+ ツール

    + TiDB Lightning

        - `BIGINT` 型のオートインクリメント列が範囲外になる可能性がある問題を修正 [#27397](https://github.com/pingcap/tidb/issues/27937)
        - 極端なケースで重複が原因で TiDB Lightning がパニックを起こす可能性がある問題を修正 [#34163](https://github.com/pingcap/tidb/issues/34163)
        - Parquet ファイルでスラッシュ、数字、または非ASCII文字で始まる列をサポートしない問題を修正 [#36980](https://github.com/pingcap/tidb/issues/36980)
        - TiDB が IPv6 ホストを使用する場合に TiDB Lightning が TiDB に接続できない問題を修正 [#35880](https://github.com/pingcap/tidb/issues/35880)

    + TiDB データ移行 (DM)

        - DB Conn を取得する際に DM Worker がスタックする可能性がある問題を修正 [#3733](https://github.com/pingcap/tiflow/issues/3733)
        - `Specified key was too long` エラーを報告する問題を修正 [#5315](https://github.com/pingcap/tiflow/issues/5315)
        - ラテン1データがレプリケーション中に破損する可能性がある問題を修正 [#7028](https://github.com/pingcap/tiflow/issues/7028)
        - TiDB が IPv6 ホストを使用する場合に DM が起動しない問題を修正 [#6249](https://github.com/pingcap/tiflow/issues/6249)
        - `query-status` でのデータ競合の可能性がある問題を修正 [#4811](https://github.com/pingcap/tiflow/issues/4811)
        - リレーでエラーが発生した場合の goroutine リークを修正 [#6193](https://github.com/pingcap/tiflow/issues/6193)

    + TiCDC
        - `enable-old-value = false` を設定した場合に TiCDC パニックの問題を修正 [#6198](https://github.com/pingcap/tiflow/issues/6198)

    + バックアップ & リストア (BR)

        - 外部ストレージの認証キーに特殊文字が存在すると、バックアップとリストアの失敗につながる可能性がある問題を修正 [#37469](https://github.com/pingcap/tidb/issues/37469)
        - リストア中に並行性が高すぎるためにリージョンのバランスが取れない問題を修正 [#37549](https://github.com/pingcap/tidb/issues/37549)

    + Dumpling

        - GetDSN が IPv6 をサポートしていない問題を修正 [#36112](https://github.com/pingcap/tidb/issues/36112)