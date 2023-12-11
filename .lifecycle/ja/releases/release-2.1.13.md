---
title: TiDB 2.1.13のリリースノート
aliases: ['/docs/dev/releases/release-2.1.13/','/docs/dev/releases/2.1.13/']
---

# TiDB 2.1.13のリリースノート

リリース日: 2019年6月21日

TiDBバージョン: 2.1.13

TiDB Ansibleバージョン: 2.1.13

## TiDB

- `AUTO_INCREMENT`属性を含む列で`SHARD_ROW_ID_BITS`を使用して行IDを散らす機能を追加し、ホットスポットの問題を緩和する[#10788](https://github.com/pingcap/tidb/pull/10788)
- TiDBクラスタをアップグレードした後にDDL操作の正常な実行を高速化するために無効なDDLメタデータの寿命を最適化する[#10789](https://github.com/pingcap/tidb/pull/10789)
- `execdetails.ExecDetails`ポインタから引き起こされるCoprocessorリソースの迅速な解放に失敗したことによる高並行シナリオでのOOMの問題を修正する[#10833](https://github.com/pingcap/tidb/pull/10833)
- 統計情報を更新するかどうかを制御する`update-stats`設定項目を追加する[#10772](https://github.com/pingcap/tidb/pull/10772)
- ホットスポットの問題を解決するためにリージョンのプレスプリットをサポートするために以下のTiDB固有の構文を追加する:
    - `PRE_SPLIT_REGIONS`テーブルオプションを追加[#10863](https://github.com/pingcap/tidb/pull/10863)
    - `SPLIT TABLE table_name INDEX index_name`構文を追加[#10865](https://github.com/pingcap/tidb/pull/10865)
    - `SPLIT TABLE [table_name] BETWEEN (min_value...) AND (max_value...) REGIONS [region_num]`構文を追加[#10882](https://github.com/pingcap/tidb/pull/10882)
- 一部のケースで`KILL`構文によって引き起こされるパニックの問題を修正する[#10879](https://github.com/pingcap/tidb/pull/10879)
- 一部のケースでMySQLの`ADD_DATE`との互換性を向上させる[#10718](https://github.com/pingcap/tidb/pull/10718)
- インデックス結合における内部テーブル選択の選択性率の誤った推定を修正する[#10856](https://github.com/pingcap/tidb/pull/10856)

## TiKV

- イテレータが状態をチェックしないためにシステムで不完全なスナップショットが生成される問題を修正する[#4940](https://github.com/tikv/tikv/pull/4940)
- `block-size`構成の有効性をチェックする機能を追加する[#4930](https://github.com/tikv/tikv/pull/4930)

## Tools

- TiDB Binlog
    - データの書き込みに失敗した場合にPumpが返された値をチェックしないために発生する誤ったオフセットの問題を修正する[#640](https://github.com/pingcap/tidb-binlog/pull/640)
    - コンテナ環境でブリッジモードをサポートするためにDrainerに`advertise-addr`設定を追加する[#634](https://github.com/pingcap/tidb-binlog/pull/634)