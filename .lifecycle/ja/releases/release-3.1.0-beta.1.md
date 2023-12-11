---
title: TiDB 3.1 Beta.1のリリースノート
aliases: ['/docs/dev/releases/release-3.1.0-beta.1/','/docs/dev/releases/3.1.0-beta.1/']
---

# TiDB 3.1 Beta.1のリリースノート

リリース日: 2020年1月10日

TiDBバージョン: 3.1.0-beta.1

TiDB Ansibleバージョン: 3.1.0-beta.1

## TiKV

+ バックアップ
    - バックアップファイルの名前を `start_key` のハッシュ値に変更してファイル名を短くし、読みやすくする [#6198](https://github.com/tikv/tikv/pull/6198)
    - 一貫性チェックでの誤検出を避けるためにRocksDBの `force_consistency_checks` チェックを無効にする [#6249](https://github.com/tikv/tikv/pull/6249)
    - 増分バックアップ機能を追加する [#6286](https://github.com/tikv/tikv/pull/6286)

+ sst_importer
    - リストア中にSSTファイルがMVCCプロパティを持たない問題を修正する [#6378](https://github.com/tikv/tikv/pull/6378)
    - `tikv_import_download_duration`、`tikv_import_download_bytes`、`tikv_import_ingest_duration`、`tikv_import_ingest_bytes`、`tikv_import_error_counter` などの監視項目を追加して、SSTファイルのダウンロードとインジェストのオーバーヘッドを観察する [#6404](https://github.com/tikv/tikv/pull/6404)

+ raftstore
    - フォロワーリードでの問題を修正し、リーダーが変更されたときにフォロワーが古いデータを読むことでトランザクションの分離が壊れる問題を修正する [#6343](https://github.com/tikv/tikv/pull/6343)

## ツール

+ BR（バックアップとリストア）
    - 不正確なバックアップの進捗情報を修正する [#127](https://github.com/pingcap/br/pull/127)
    - Regionの分割のパフォーマンスを向上させる [#122](https://github.com/pingcap/br/pull/122)
    - パーティションテーブルのバックアップとリストア機能を追加する [#137](https://github.com/pingcap/br/pull/137)
    - 自動的にPDスケジューラをスケジュールする機能を追加する [#123](https://github.com/pingcap/br/pull/123)
    - `PKIsHandle` でないテーブルがリストアされた後にデータが上書きされる問題を修正する [#139](https://github.com/pingcap/br/pull/139)

## TiDB Ansible

- 初期化フェーズ中にオペレーティングシステムでTransparent Huge Pages（THP）を自動的に無効にする機能を追加する [#1086](https://github.com/pingcap/tidb-ansible/pull/1086)
- BRコンポーネントのためのGrafana監視を追加する [#1093](https://github.com/pingcap/tidb-ansible/pull/1093)
- TiDB Lightningのデプロイを自動的に関連ディレクトリを作成することで最適化する [#1104](https://github.com/pingcap/tidb-ansible/pull/1104)