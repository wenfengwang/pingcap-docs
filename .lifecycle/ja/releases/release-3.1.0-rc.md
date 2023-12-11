---
title: TiDB 3.1 RCリリースノート
aliases: ['/docs/dev/releases/release-3.1.0-rc/','/docs/dev/releases/3.1.0-rc/']
---

# TiDB 3.1 RCリリースノート

リリース日: 2020年4月2日

TiDBバージョン: 3.1.0-rc

TiDB Ansibleバージョン: 3.1.0-rc

> **警告:**
>
> このバージョンには既知の問題があります。これらの問題は新しいバージョンで修正されています。最新の3.1.xバージョンを使用することを推奨します。

## 新機能

+ TiDB

    - パーティションプルーニングの再実装にバイナリサーチを使用して、パフォーマンスを向上させる[#15678](https://github.com/pingcap/tidb/pull/15678)
    - `RECOVER`構文を使用して、切り捨てられたテーブルを復元する機能をサポート[#15460](https://github.com/pingcap/tidb/pull/15460)
    - リトライステートメントとテーブルのリカバリーのために`AUTO_RANDOM` IDキャッシュを追加する[#15393](https://github.com/pingcap/tidb/pull/15393)
    - `recover table`ステートメントを使用して、`AUTO_RANDOM` IDアロケーターの状態を復元する機能をサポート[#15393](https://github.com/pingcap/tidb/pull/15393)
    - `Hash`パーティションテーブルのパーティションキーとして`YEAR`、`MONTH`、および`TO_DAY`関数をサポート[#15619](https://github.com/pingcap/tidb/pull/15619)
    - `SELECT... FOR UPDATE`ステートメントでロックが必要な場合にのみ、スキーマ変更関連のテーブルにテーブルIDを追加する[#15708](https://github.com/pingcap/tidb/pull/15708)
    - 自動的にロードバランシングポリシーに従って異なる役割からデータを読み取る機能を追加し、この機能を有効にするための`leader-and-follower`システム変数を追加する[#15721](https://github.com/pingcap/tidb/pull/15721)
    - TiDBが新しい接続を確立するたびにTLS証明書を動的に更新する機能をサポートし、期限切れのクライアント証明書を再起動せずに更新する[#15163](https://github.com/pingcap/tidb/pull/15163)
    - PDクライアントをアップグレードして、TiDBが新しい接続を確立するたびに最新の証明書をロードする機能をサポートする[#15425](https://github.com/pingcap/tidb/pull/15425)
    - `cluster-ssl-*`が構成されている場合に、TiDBサーバーとPDサーバー間、または2つのTiDBサーバー間で構成されたTLS証明書を強制的に使用する機能を追加する[#15430](https://github.com/pingcap/tidb/pull/15430)
    - クライアントが構成中にTLS認証を強制するためのMySQL互換の`--require-secure-transport`起動オプションを追加する[#15442](https://github.com/pingcap/tidb/pull/15442)
    - `cluster-verify-cn`構成項目を追加する。構成後、対応するCN証明書を使用してステータスサービスを利用できる[#15137](https://github.com/pingcap/tidb/pull/15137)

+ TiKV

    - Raw KV APIでデータのバックアップをサポート[#7051](https://github.com/tikv/tikv/pull/7051)
    - ステータスサーバーのTLS認証をサポート[#7142](https://github.com/tikv/tikv/pull/7142)
    - KVサーバーのTLS認証をサポート[#7305](https://github.com/tikv/tikv/pull/7305)
    - バックアップのパフォーマンスを向上させるためにロックを保持する時間を最適化する[#7202](https://github.com/tikv/tikv/pull/7202)

+ PD

    - `shuffle-region-scheduler`を使用してLearnerのスケジューリングをサポート[#2235](https://github.com/pingcap/pd/pull/2235)
    - pd-ctlに配置ルールを構成するためのコマンドを追加する[#2306](https://github.com/pingcap/pd/pull/2306)

+ ツール

    - TiDB Binlog

        * コンポーネント間でのTLS認証をサポートする[#931](https://github.com/pingcap/tidb-binlog/pull/931) [#937](https://github.com/pingcap/tidb-binlog/pull/937) [#939](https://github.com/pingcap/tidb-binlog/pull/939)
        * DrainerにKafkaのクライアントIDを構成する`kafka-client-id`構成項目を追加する[#929](https://github.com/pingcap/tidb-binlog/pull/929)

    - TiDB Lightning

        * TiDB Lightningのパフォーマンスを最適化する[#281](https://github.com/pingcap/tidb-lightning/pull/281) [#275](https://github.com/pingcap/tidb-lightning/pull/275)
        * TiDB LightningのTLS認証をサポートする[#270](https://github.com/pingcap/tidb-lightning/pull/270)

    - バックアップ＆リストア（BR）

        * ログ出力を最適化する[#189](https://github.com/pingcap/br/pull/189)

+ TiDB Ansible

    - TiFlashデータディレクトリの作成方法を最適化する[#1242](https://github.com/pingcap/tidb-ansible/pull/1242)
    - TiFlashに`Write Amplification`モニタリング項目を追加する[#1234](https://github.com/pingcap/tidb-ansible/pull/1234)
    - CPU epollexclusiveが利用できない場合の失敗したプレフライトチェックのエラーメッセージを最適化する[#1243](https://github.com/pingcap/tidb-ansible/pull/1243)

## バグ修正

+ TiDB

    - TiFlashレプリカを頻繁に更新することによって発生する情報スキーマのエラーを修正する[#14884](https://github.com/pingcap/tidb/pull/14884)
    - `AUTO_RANDOM`を適用する際に`last_insert_id`が誤って生成される問題を修正する[#15149](https://github.com/pingcap/tidb/pull/15149)
    - TiFlashレプリカのステータスを更新することによってDDL操作がスタックする可能性がある問題を修正する[#15161](https://github.com/pingcap/tidb/pull/15161)
    - プッシュダウンできない述語が存在する場合、`Aggregation`プッシュダウンおよび`TopN`プッシュダウンを禁止する[#15141](https://github.com/pingcap/tidb/pull/15141)
    - ネストされた`view`の作成を禁止する[#15440](https://github.com/pingcap/tidb/pull/15440)
    - `SET ROLE ALL`の後に`SELECT CURRENT_ROLE()`を実行した際にエラーが発生する問題を修正する[#15570](https://github.com/pingcap/tidb/pull/15570)
    - `view_name.col_name from view_name`ステートメントを実行した際に`view`名を識別できないエラーを修正する[#15573](https://github.com/pingcap/tidb/pull/15573)
    - binlog情報の書き込み中にDDLステートメントの前処理でエラーが発生する問題を修正する[#15444](https://github.com/pingcap/tidb/pull/15444)
    - `view`とパーティションテーブルのアクセス時にパニックが発生する問題を修正する[#15560](https://github.com/pingcap/tidb/pull/15560)
    - `bit(n)`データ型を含む`update duplicate key`ステートメントで`VALUES`関数を実行する際にエラーが発生する問題を修正する[#15487](https://github.com/pingcap/tidb/pull/15487)
    - 特定の状況下で指定された最大実行時間が影響を与えない問題を修正する[#15616](https://github.com/pingcap/tidb/pull/15616)
    - `Index Scan`を使用して実行計画を生成する際に、現在の`ReadEngine`にTiKVサーバーが含まれているかどうかを確認しない問題を修正する[#15773](https://github.com/pingcap/tidb/pull/15773)

+ TiKV

    - トランザクションに既存のキーを挿入してすぐに削除することで発生する競合チェックの失敗またはデータインデックスの不整合を修正する[#7112](https://github.com/tikv/tikv/pull/7112)
    - `TopN`が符号なし整数を比較する際の計算エラーを修正する[#7199](https://github.com/tikv/tikv/pull/7199)
    - Raftstoreにフローコントロールメカニズムを導入して、フロー制御がないと遅いログ追跡やクラスターのスタックが発生する問題や大規模なトランザクションサイズがTiKVサーバー間での頻繁な再接続を引き起こす問題を解決する[#7087](https://github.com/tikv/tikv/pull/7087) [#7078](https://github.com/tikv/tikv/pull/7078)
    - レプリカに送信された保留中の読み取りリクエストが永久的にブロックされる可能性がある問題を修正する[#6543](https://github.com/tikv/tikv/pull/6543)
    - スナップショットの適用によってレプリカ読み取りがブロックされる可能性がある問題を修正する[#7249](https://github.com/tikv/tikv/pull/7249)
    - リーダーの移行がTiKVをパニックさせる可能性がある問題を修正する[#7240](https://github.com/tikv/tikv/pull/7240)
    - S3にデータをバックアップする際に、すべてのSSTファイルがゼロで埋められる問題を修正する[#6967](https://github.com/tikv/tikv/pull/6967)
    - バックアップ中にSSTファイルのサイズが記録されず、復元後に多くの空のリージョンが発生する問題を修正する[#6983](https://github.com/tikv/tikv/pull/6983)
    - バックアップのためのAWS IAMウェブアイデンティティをサポートする[#7297](https://github.com/tikv/tikv/pull/7297)

+ PD

    - PDがリージョンのハートビートを処理する際に、データ競合によって不正確なリージョン情報が発生する問題を修正する[#2234](https://github.com/pingcap/pd/pull/2234)
    - `random-merge-scheduler`が位置ラベルと配置ルールに従わない問題を修正する[#2212](https://github.com/pingcap/pd/pull/2221)
    - 開始キーと終了キーが同じ`startKey`、`endKey`を持つ他の配置ルールによって配置ルールが上書きされる問題を修正する[#2222](https://github.com/pingcap/pd/pull/2222)
    - APIのバージョン番号がPDサーバーのバージョン番号と一貫していない問題を修正する[#2192](https://github.com/pingcap/pd/pull/2192)

+ Tools

    - TiDB Lightning

        * TiDBバックエンドで`&`文字が`EOF`文字に置換されるバグを修正する[#283](https://github.com/pingcap/tidb-lightning/pull/283)

    - バックアップ＆リストア(BR)

        * BRがTiFlashクラスターデータを復元できない問題を修正する[#194](https://github.com/pingcap/br/pull/194)