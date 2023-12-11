---
title: TiDB 4.0.0 Beta.1 リリースノート
aliases: ['/docs/dev/releases/release-4.0.0-beta.1/','/docs/dev/releases/4.0.0-beta.1/']
---

# TiDB 4.0.0 Beta.1 リリースノート

リリース日: 2020年2月28日

TiDB バージョン: 4.0.0-beta.1

TiDB Ansible バージョン: 4.0.0-beta.1

## 互換性の変更

* TiDB
    + `log.enable-slow-log` 構成項目のタイプを整数からブール値に変更 [#14864](https://github.com/pingcap/tidb/pull/14864)
    + `mysql.user` システムテーブル内の `password` フィールド名を `authentication_string` に変更し、MySQL 5.7 と一貫性を保つ（**この互換性の変更により、以前のバージョンにロールバックできなくなります。**） [#14598](https://github.com/pingcap/tidb/pull/14598)
    + `txn-total-size-limit` 構成項目のデフォルト値を `1GB` から `100MB` に調整 [#14522](https://github.com/pingcap/tidb/pull/14522)
    + PD から読み取った構成項目を動的に変更または更新するサポートを追加 [#14750](https://github.com/pingcap/tidb/pull/14750) [#14303](https://github.com/pingcap/tidb/pull/14303) [#14830](https://github.com/pingcap/tidb/pull/14830)

* TiKV
    + コプロセッサと同じスレッドを使用するかどうかを制御する `readpool.unify-read-pool` 構成項目（デフォルトは `True`）を追加 [#6375](https://github.com/tikv/tikv/pull/6375) [#6401](https://github.com/tikv/tikv/pull/6401) [#6534](https://github.com/tikv/tikv/pull/6534) [#6582](https://github.com/tikv/tikv/pull/6582) [#6585](https://github.com/tikv/tikv/pull/6585) [#6593](https://github.com/tikv/tikv/pull/6593) [#6597](https://github.com/tikv/tikv/pull/6597) [#6677](https://github.com/tikv/tikv/pull/6677)

* PD
    + HTTP API を最適化して構成マネージャと互換性があるようにする [#2080](https://github.com/pingcap/pd/pull/2080)

* TiDB Lightning
    + 構成ファイルで構成されていない特定の項目について、ドキュメントで指定されたデフォルト構成を使用するように変更 [#255](https://github.com/pingcap/tidb-lightning/pull/255)

* TiDB Ansible
    + `theflash` を `tiflash` に名前を変更 [#1130](https://github.com/pingcap/tidb-ansible/pull/1130)
    + TiFlash の構成ファイル内のデフォルト値と関連する構成を最適化 [#1138](https://github.com/pingcap/tidb-ansible/pull/1138)

## 新機能

* TiDB
    + `SLOW_QUERY / CLUSTER_SLOW_QUERY` システムテーブルでいつでも遅いクエリをクエリするサポートを追加 [#14840](https://github.com/pingcap/tidb/pull/14840) [#14878](https://github.com/pingcap/tidb/pull/14878)
    + SQL パフォーマンス診断をサポート
        - [#14843](https://github.com/pingcap/tidb/pull/14843) [#14810](https://github.com/pingcap/tidb/pull/14810) [#14835](https://github.com/pingcap/tidb/pull/14835) [#14801](https://github.com/pingcap/tidb/pull/14801) [#14743](https://github.com/pingcap/tidb/pull/14743)
        - [#14718](https://github.com/pingcap/tidb/pull/14718) [#14721](https://github.com/pingcap/tidb/pull/14721) [#14670](https://github.com/pingcap/tidb/pull/14670) [#14663](https://github.com/pingcap/tidb/pull/14663) [#14668](https://github.com/pingcap/tidb/pull/14668)
        - [#14896](https://github.com/pingcap/tidb/pull/14896)
    + `Sequence` 関数をサポート [#14731](https://github.com/pingcap/tidb/pull/14731) [#14589](https://github.com/pingcap/tidb/pull/14589) [#14674](https://github.com/pingcap/tidb/pull/14674) [#14442](https://github.com/pingcap/tidb/pull/14442) [#14303](https://github.com/pingcap/tidb/pull/14303) [#14830](https://github.com/pingcap/tidb/pull/14830)
    + PD から読み取った構成項目を動的に変更または更新するサポートを追加 [#14750](https://github.com/pingcap/tidb/pull/14750) [#14303](https://github.com/pingcap/tidb/pull/14303) [#14830](https://github.com/pingcap/tidb/pull/14830)
    + 負荷分散ポリシーに従って異なる役割のデータを自動的に読み取る機能を追加し、これを可能にする `leader-and-follower` システム変数を追加 [#14761](https://github.com/pingcap/tidb/pull/14761)
    + `Coercibility` 関数を追加 [#14739](https://github.com/pingcap/tidb/pull/14739)
    + パーティション化されたテーブルで TiFlash レプリカを設定するサポートを追加 [#14735](https://github.com/pingcap/tidb/pull/14735) [#14713](https://github.com/pingcap/tidb/pull/14713) [#14644](https://github.com/pingcap/tidb/pull/14644)
    + `SLOW_QUERY` テーブルの権限チェックを改善 [#14451](https://github.com/pingcap/tidb/pull/14451)
    + SQL ジョイン時にメモリが不足する場合、中間結果を自動的にディスクファイルに書き込む機能をサポート [#14708](https://github.com/pingcap/tidb/pull/14708) [#14279](https://github.com/pingcap/tidb/pull/14279)
    + `information_schema.PARTITIONS` システムテーブルをクエリしてテーブルパーティションを確認するサポートを追加 [#14347](https://github.com/pingcap/tidb/pull/14347)
    + `json_objectagg` 集約関数を追加 [#11154](https://github.com/pingcap/tidb/pull/11154)
    + オーディットログで接続拒否の試みをログに記録するサポートを追加 [#14594](https://github.com/pingcap/tidb/pull/14594)
    + 1つのサーバーへの接続数を制御するための `max-server-connections` 構成項目（デフォルトは `4096`）を追加 [#14409](https://github.com/pingcap/tidb/pull/14409)
    + サーバーレベルで複数のストレージエンジンを指定して分離読み取りをサポート [#14440](https://github.com/pingcap/tidb/pull/14440)
    + `Apply` オペレーターと `Sort` オペレーターのコストモデルを最適化して安定性を向上させる [#13550](https://github.com/pingcap/tidb/pull/13550) [#14708](https://github.com/pingcap/tidb/pull/14708)

* TiKV
    + ステータスポートからの構成項目の取得を HTTP API でサポート [#6480](https://github.com/tikv/tikv/pull/6480)
    + コプロセッサ内の `Chunk Encoder` のパフォーマンスを最適化 [#6341](https://github.com/tikv/tikv/pull/6341)

* PD
    + ダッシュボード UI を介してクラスター内のホットスポットの分布にアクセスするサポートを追加 [#2086](https://github.com/pingcap/pd/pull/2086)
    + クラスターコンポーネントの `START_TIME` および `UPTIME` をキャプチャし表示するサポートを追加 [#2116](https://github.com/pingcap/pd/pull/2116)
    + `member` API の返されたメッセージにデプロイパスとコンポーネントバージョンの情報を追加 [#2130](https://github.com/pingcap/pd/pull/2130)
    + 他のコンポーネントの構成を修正および確認するための pd-ctl に `component` サブコマンドを追加（実験的）[#2092](https://github.com/pingcap/pd/pull/2092)

* TiDB Binlog
    + コンポーネント間の TLS をサポート [#904](https://github.com/pingcap/tidb-binlog/pull/904) [#894](https://github.com/pingcap/tidb-binlog/pull/894)
    + Drainer における Kafka のクライアント ID を設定するための `kafka-client-id` 構成項目を追加 [#902](https://github.com/pingcap/tidb-binlog/pull/902)
    + Drainerの増分バックアップデータのパージをサポート [#885](https://github.com/pingcap/tidb-binlog/pull/885)

* TiDB Ansible
    + 1つのクラスターで複数のGrafana/Prometheus/Alertmanagerのデプロイをサポート [#1142](https://github.com/pingcap/tidb-ansible/pull/1142)
    + TiFlashの構成ファイルに`metric_port`設定項目（デフォルトで`8234`）を追加 [#1145](https://github.com/pingcap/tidb-ansible/pull/1145)
    + TiFlashの構成ファイルに`flash_proxy_status_port`設定項目（デフォルトで`20292`）を追加 [#1141](https://github.com/pingcap/tidb-ansible/pull/1141)
    + TiFlashモニタリングダッシュボードを追加 [#1147](https://github.com/pingcap/tidb-ansible/pull/1147) [#1151](https://github.com/pingcap/tidb-ansible/pull/1151)

## バグ修正

* TiDB
    + 64文字を超えるカラム名で`view`を作成するとエラーが報告される問題を修正 [#14850](https://github.com/pingcap/tidb/pull/14850)
    + `create or replace view`ステートメントが誤って処理されるため、`information_schema.views`に重複したデータが存在する問題を修正 [#14832](https://github.com/pingcap/tidb/pull/14832)
    + `plan cache`が有効な場合、`BatchPointGet`の結果が正しくない問題を修正 [#14855](https://github.com/pingcap/tidb/pull/14855)
    + タイムゾーンが変更された後にデータが間違ったパーティション化されたテーブルに挿入される問題を修正 [#14370](https://github.com/pingcap/tidb/pull/14370)
    + アウタージョインの簡略化中に`IsTrue`関数の無効な名前を使用して式を再構築しようとした際にパニックが発生する問題を修正 [#14515](https://github.com/pingcap/tidb/pull/14515)
    + `show binding`ステートメントの誤った権限チェックを修正 [#14443](https://github.com/pingcap/tidb/pull/14443)

* TiKV
    + TiDBとTiKVで`CAST`関数の振る舞いが一貫していない問題を修正 [#6463](https://github.com/tikv/tikv/pull/6463) [#6461](https://github.com/tikv/tikv/pull/6461) [#6459](https://github.com/tikv/tikv/pull/6459) [#6474](https://github.com/tikv/tikv/pull/6474) [#6492](https://github.com/tikv/tikv/pull/6492) [#6569](https://github.com/tikv/tikv/pull/6569)

* TiDB Lightning
    + Serverモード外でウェブインターフェースが機能しないバグを修正 [#259](https://github.com/pingcap/tidb-lightning/pull/259)