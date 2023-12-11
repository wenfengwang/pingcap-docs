---
title: TiDB 3.0.11リリースノート
aliases: ['/docs/dev/releases/release-3.0.11/','/docs/dev/releases/3.0.11/']
---

# TiDB 3.0.11リリースノート

リリース日: 2020年3月4日

TiDBバージョン: 3.0.11

TiDB Ansibleバージョン: 3.0.11

> **警告:**
>
> このバージョンにはいくつかの既知の問題があり、これらの問題は新しいバージョンで修正されています。最新の3.0.xバージョンを使用することをお勧めします。

## 互換性の変更

* TiDB
    + `max-index-length`構成項目を追加して、最大インデックス長を制御するようにしました。これは、TiDBのバージョン3.0.7以前やMySQLの動作と互換性があります。[#15057](https://github.com/pingcap/tidb/pull/15057)

## 新機能

* TiDB
    + `information_schema.PARTITIONS`テーブルでパーティションテーブルのメタ情報を表示する機能をサポートしました。[#14849](https://github.com/pingcap/tidb/pull/14849)

* TiDB Binlog
    + TiDBクラスタ間の双方向データレプリケーションをサポートしました。[#884](https://github.com/pingcap/tidb-binlog/pull/884) [#909](https://github.com/pingcap/tidb-binlog/pull/909)

* TiDB Lightning
    + TLS構成をサポートしました。[#44](https://github.com/tikv/importer/pull/44) [#270](https://github.com/pingcap/tidb-lightning/pull/270)

* TiDB Ansible
    + `create_users.yml`のロジックを変更して、制御マシンのユーザーが`ansible_user`と一貫性がなくてもよくなりました。[#1184](https://github.com/pingcap/tidb-ansible/pull/1184)

## バグ修正

* TiDB
    + `Union`を使用するクエリが読み取り専用にマークされていないため、楽観的トランザクションを再試行する際にGoroutineがリークする問題を修正しました。[#15076](https://github.com/pingcap/tidb/pull/15076)
    + `SHOW TABLE STATUS`が`tidb_snapshot`パラメータの値が正しく使用されないために、スナップショット時のテーブルステータスを正しく出力できない問題を修正しました。[#14391](https://github.com/pingcap/tidb/pull/14391)
    + `Sort Merge Join`と`ORDER BY DESC`を含むSQLステートメントによって引き起こされる誤った結果を修正しました。[#14664](https://github.com/pingcap/tidb/pull/14664)
    + サポートされていない式を使用してパーティションテーブルを作成する際にTiDBサーバーがパニックに陥る問題を修正しました。このパニックを修正すると、エラー情報「このパーティション関数は許可されていません」が返されます。[#14769](https://github.com/pingcap/tidb/pull/14769)
    + `Union`を含むサブクエリを含む`select max() from subquery`ステートメントを実行したときに発生する不正なクエリ結果を修正しました。[#14944](https://github.com/pingcap/tidb/pull/14944)
    + 実行バインディングを解除する`DROP BINDING`を実行した後に`SHOW BINDINGS`ステートメントを実行したときにエラーメッセージが返される問題を修正しました。[#14865](https://github.com/pingcap/tidb/pull/14865)
    + クエリ結果が不正になる可能性がある`DIV`で文字列型を使用した場合の問題を修正しました。たとえば、現在は`select 1 / '2007' div 1`ステートメントを正しく実行できます。[#14098](https://github.com/pingcap/tidb/pull/14098)

* TiKV
    + 不要なログを削除することでログの出力を最適化しました。[#6657](https://github.com/tikv/tikv/pull/6657)
    + 高負荷下でピアが削除されると発生する可能性があるパニックを修正しました。[#6704](https://github.com/tikv/tikv/pull/6704)
    + いくつかのケースでHibernate Regionsが起動しない問題を修正しました。[#6732](https://github.com/tikv/tikv/pull/6732) [#6738](https://github.com/tikv/tikv/pull/6738)

* TiDB Ansible
    + `tidb-ansible`内の古いドキュメントリンクを更新しました。[#1169](https://github.com/pingcap/tidb-ansible/pull/1169)
    + `wait for region replication complete`タスクで未定義の変数が発生する可能性がある問題を修正しました。[#1173](https://github.com/pingcap/tidb-ansible/pull/1173)