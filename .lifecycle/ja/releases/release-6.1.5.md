---
title: TiDB 6.1.5 リリースノート
summary: TiDB 6.1.5 での互換性の変更、改善、およびバグ修正について学びます。

# TiDB 6.1.5 リリースノート

リリース日: 2023年2月28日

TiDB バージョン: 6.1.5

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [本番展開](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.5#version-list)

## 互換性の変更

- 2023年2月20日以降、新しい TiDB および TiDB ダッシュボードのバージョン（v6.1.5 を含む）では、[テレメトリ機能](/telemetry.md) がデフォルトで無効になり、使用状況情報は PingCAP と共有されません。これらのバージョンにアップグレードする前に、クラスタがデフォルトのテレメトリ構成を使用している場合、アップグレード後にテレメトリ機能が無効になります。特定のバージョンについては、[TiDB リリースタイムライン](/releases/release-timeline.md) を参照してください。

    - [`tidb_enable_telemetry`](/system-variables.md#tidb_enable_telemetry-new-in-v402) システム変数のデフォルト値が `ON` から `OFF` に変更されました。
    - TiDB の [`enable-telemetry`](/tidb-configuration-file.md#enable-telemetry-new-in-v402) 構成項目のデフォルト値が `true` から `false` に変更されました。
    - PD の [`enable-telemetry`](/pd-configuration-file.md#enable-telemetry) 構成項目のデフォルト値が `true` から `false` に変更されました。

- v1.11.3 以降、新たに展開される TiUP では、デフォルトでテレメトリ機能が無効になり、使用状況情報は収集されません。v1.11.3 よりも前の TiUP バージョンから v1.11.3 またはそれ以降のバージョンにアップグレードする場合、テレメトリ機能はアップグレード前と同じ状態を維持します。

## 改善

- TiDB

    - クラスタ化された複合インデックスの最初の列として `AUTO_RANDOM` 列をサポート [#38572](https://github.com/pingcap/tidb/issues/38572) @[tangenta](https://github.com/tangenta)

## バグ修正

- TiDB

    - データ競合によって TiDB が再起動する可能性がある問題を修正 [#27725](https://github.com/pingcap/tidb/issues/27725) @[XuHuaiyu](https://github.com/XuHuaiyu)
    - `UPDATE` ステートメントが使用される際に、Read Committed 分離レベルで最新のデータが読み取られない可能性がある問題を修正 [#41581](https://github.com/pingcap/tidb/issues/41581) @[cfzjywxk](https://github.com/cfzjywxk)

- PD

    - `ReportMinResolvedTS` の呼び出しがあまりに頻繁であると PD OOM 問題が発生する問題を修正 [#5965](https://github.com/tikv/pd/issues/5965) @[HundunDM](https://github.com/HunDunDM)

- Tools

    - TiCDC

        - レプリケーション遅延が過度に高い場合に、リドゥログの適用が OOM を引き起こす問題を修正 [#8085](https://github.com/pingcap/tiflow/issues/8085) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - リドゥログを書き込む際に性能が低下する問題を修正 [#8074](https://github.com/pingcap/tiflow/issues/8074) @[CharlesCheung96](https://github.com/CharlesCheung96)

    - TiDB データ移行（DM）

        - `binlog-schema delete` コマンドの実行に失敗する問題を修正 [#7373](https://github.com/pingcap/tiflow/issues/7373) @[liumengya94](https://github.com/liumengya94)
        - 最後のバイナリログがスキップされた DDL である場合にチェックポイントが進まない問題を修正 [#8175](https://github.com/pingcap/tiflow/issues/8175) @[D3Hunter](https://github.com/D3Hunter)