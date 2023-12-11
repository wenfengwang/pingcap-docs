---
title: TiDB 4.0.0 Beta.2リリースノート
aliases: ['/docs/dev/releases/release-4.0.0-beta.2/','/docs/dev/releases/4.0.0-beta.2/']
---

# TiDB 4.0.0 Beta.2リリースノート

リリース日: 2020年3月18日

TiDBバージョン: 4.0.0-beta.2

TiDB Ansibleバージョン: 4.0.0-beta.2

## 互換性の変更

+ ツール
    - TiDB Binlog
        - `disable-dispatch`と`disable-causality`がDrainerで構成された場合にシステムがエラーを返し、終了する問題を修正 [#915](https://github.com/pingcap/tidb-binlog/pull/915)

## 新機能

+ TiKV
    - 動的に更新された構成をハードウェアディスクに永続化するサポート [#6684](https://github.com/tikv/tikv/pull/6684)

+ PD
    - 動的に更新された構成をハードウェアディスクに永続化するサポート [#2153](https://github.com/pingcap/pd/pull/2153)

+ ツール
    - TiDB Binlog
        - TiDBクラスタ間の双方向データレプリケーションをサポート [#879](https://github.com/pingcap/tidb-binlog/pull/879) [#903](https://github.com/pingcap/tidb-binlog/pull/903)
    - TiDB Lightning
        - TLS構成をサポート [#40](https://github.com/tikv/importer/pull/40) [#270](https://github.com/pingcap/tidb-lightning/pull/270)
    - TiCDC
        - 変更データキャプチャ（CDC）の初期リリースをサポートし、以下の機能を提供:
            - TiKVからの変更データのキャプチャをサポート
            - TiKVからMySQL互換データベースに変更データをレプリケートし、最終的なデータ整合性を保証
            - 変更データをKafkaにレプリケートし、最終的なデータ整合性または行レベルの整序性を保証
            - プロセスレベルの高可用性を提供
    - バックアップ＆リストア（BR）
        - インクリメンタルバックアップやファイルのAmazon S3へのバックアップなどの実験機能を有効にする [#175](https://github.com/pingcap/br/pull/175)

+ TiDB Ansible
    - ノード情報をetcdに注入するサポート [#1196](https://github.com/pingcap/tidb-ansible/pull/1196)
    - ARMプラットフォームにTiDBサービスを展開するサポート [#1204](https://github.com/pingcap/tidb-ansible/pull/1204)

## バグ修正

+ TiKV
    - バックアップ時に空の短い値に遭遇した場合に発生する可能性があるパニックの問題を修正 [#6718](https://github.com/tikv/tikv/pull/6718)
    - 一部のケースでHibernate Regionsが正しく起動しない問題を修正 [#6772](https://github.com/tikv/tikv/pull/6672) [#6648](https://github.com/tikv/tikv/pull/6648) [#6376](https://github.com/tikv/tikv/pull/6736)

+ PD
    - ルールチェッカーがストアをリージョンに割り当てる際に失敗する可能性があるパニックの問題を修正 [#2160](https://github.com/pingcap/pd/pull/2160)
    - 動的構成が有効になった後、Leaderが切り替わる際にレプリケーション遅延が発生する可能性がある問題を修正 [#2154](https://github.com/pingcap/pd/pull/2154)

+ ツール
    - バックアップ＆リストア（BR）
        - PDが大きなサイズのデータを処理できないためにBRがデータを復元できなくなる問題を修正 [#167](https://github.com/pingcap/br/pull/167)
        - BRバージョンがTiDBバージョンと互換性がないために発生するBRの失敗を修正 [#186](https://github.com/pingcap/br/pull/186)
        - BRバージョンがTiFlashと互換性がないために発生するBRの失敗を修正 [#194](https://github.com/pingcap/br/pull/194)