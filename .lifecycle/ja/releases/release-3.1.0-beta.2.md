---
title: TiDB 3.1 Beta.2 リリースノート
aliases: ['/docs/dev/releases/release-3.1.0-beta.2/','/docs/dev/releases/3.1.0-beta.2/']
---

# TiDB 3.1 Beta.2 リリースノート

リリース日: 2020年3月9日

TiDB バージョン: 3.1.0-beta.2

TiDB Ansible バージョン: 3.1.0-beta.2

> **警告:**
>
> このバージョンにはいくつかの既知の問題が見つかっており、これらの問題は新しいバージョンで修正されています。最新の3.1.xバージョンを使用することをお勧めします。

## 互換性の変更

+ ツール
    - TiDB Lightning
        - [TiDB Lightning構成](/tidb-lightning/tidb-lightning-configuration.md) で指定されていない特定の項目には構成ファイルで指定されたデフォルトの構成を使用するように修正[#255](https://github.com/pingcap/tidb-lightning/pull/255)
        - `--tidb-password` CLI パラメータを追加してTiDBパスワードを設定するように修正[#253](https://github.com/pingcap/tidb-lightning/pull/253)

## 新機能

+ TiDB
    - 列属性に `AutoRandom` キーワードを追加してTiDBがプライマリキーに自動的にランダムな整数を割り当てる機能をサポートし、`AUTO_INCREMENT` プライマリキーによる書き込みホットスポットを回避する[#14555](https://github.com/pingcap/tidb/pull/14555)
    - DDL ステートメントを使用して列ストアレプリカの作成または削除をサポートする[#14537](https://github.com/pingcap/tidb/pull/14537)
    - オプティマイザが独自に異なるストレージエンジンを選択できる機能を追加[#14537](https://github.com/pingcap/tidb/pull/14537)
    - SQLヒントが異なるストレージエンジンをサポートする機能を追加[#14537](https://github.com/pingcap/tidb/pull/14537)
    - `tidb_replica_read` システム変数を使用してフォロワーからデータを読み取る機能をサポート[#13464](https://github.com/pingcap/tidb/pull/13464)
+ TiKV
    - Raftstore
        - 他のノードに接続するための `peer_address` パラメータを追加[#6491](https://github.com/tikv/tikv/pull/6491)
        - `ReadIndex` リクエストの数を監視するための `read_index` および `read_index_resp` モニタリングメトリクスを追加[#6610](https://github.com/tikv/tikv/pull/6610)
+ PD クライアント
    - ローカルスレッドの統計をPDに報告する機能をサポート[#6605](https://github.com/tikv/tikv/pull/6605)
+ バックアップ
    - ファイルのバックアップ時に余分なメモリコピーを排除するために `RocksIOLimiter` フローコントロールライブラリをRustの `async-speed-limit` フローコントロールライブラリで置換[#6462](https://github.com/tikv/tikv/pull/6462)
+ PD
    - 位置ラベル名におけるバックスラッシュを許容する機能を追加[#2084](https://github.com/pingcap/pd/pull/2084)
+ TiFlash
    - 初回リリース
+ TiDB Ansible
    - クラスタ内に複数のGrafana/Prometheus/Alertmanagerをデプロイする機能をサポート[#1143](https://github.com/pingcap/tidb-ansible/pull/1143)
    - TiFlashコンポーネントのデプロイをサポート[#1148](https://github.com/pingcap/tidb-ansible/pull/1148)
    - TiFlashコンポーネントに関連する監視メトリクスを追加[#1152](https://github.com/pingcap/tidb-ansible/pull/1152)

## バグ修正

+ TiKV
    - Raftstore
        - ハイバネートリージョンから正しくデータを読み取っていないため読み取りリクエストを処理できない問題を修正[#6450](https://github.com/tikv/tikv/pull/6450)
        - リーダー転送プロセス中の `ReadIndex` リクエストによって発生したパニックの問題を修正[#6613](https://github.com/tikv/tikv/pull/6613)
        - 特定の条件下でハイバネートリージョンが正しく起動されない問題を修正[#6730](https://github.com/tikv/tikv/pull/6730) [#6737](https://github.com/tikv/tikv/pull/6737) [#6972](https://github.com/tikv/tikv/pull/6972)
    - バックアップ
        - バックアップによって復元時に生じる不整合なデータインデックスの問題を修正[#6659](https://github.com/tikv/tikv/pull/6659)
        - 不正な削除値の処理によって発生したパニックの問題を修正[#6726](https://github.com/tikv/tikv/pull/6726)
+ PD
    - ルールチェッカーがストアをリージョンに割り当てるのに失敗したために発生したパニックの問題を修正[#2161](https://github.com/pingcap/pd/pull/2161)
+ ツール
    - TiDB Lightning
        - サーバーモード外でWebインターフェースが機能しないバグを修正[#259](https://github.com/pingcap/tidb-lightning/pull/259)
    - BR（バックアップおよびリストア）
        - データを復元する際に遭遇した回復不能なエラーによってBRがタイムリーに終了しない問題を修正[#152](https://github.com/pingcap/br/pull/152)
+ TiDB Ansible
    - いくつかのシナリオでPDリーダーを取得できないためにローリングアップデートコマンドが失敗する問題を修正[#1122](https://github.com/pingcap/tidb-ansible/pull/1122)