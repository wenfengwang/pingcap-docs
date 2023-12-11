---
title: TiDB 6.5.5リリースノート
summary: TiDB 6.5.5の改善点やバグ修正について学びましょう。

# TiDB 6.5.5リリースノート

リリース日: 2023年9月21日

TiDBバージョン: 6.5.5

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.5/quick-start-with-tidb) | [本番展開](https://docs.pingcap.com/tidb/v6.5/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.5.5#version-list)

## 改善点

+ TiDB

    - [`NO_MERGE_JOIN()`](/optimizer-hints.md#no_merge_joint1_name--tl_name-)、[`NO_INDEX_JOIN()`](/optimizer-hints.md#no_index_joint1_name--tl_name-)、[`NO_INDEX_MERGE_JOIN()`](/optimizer-hints.md#no_index_merge_joint1_name--tl_name-)、[`NO_HASH_JOIN()`](/optimizer-hints.md#no_hash_joint1_name--tl_name-)、[`NO_INDEX_HASH_JOIN()`](/optimizer-hints.md#no_index_hash_joint1_name--tl_name-)などの新しい最適化ヒントを追加しました [#45520](https://github.com/pingcap/tidb/issues/45520) @[qw4990](https://github.com/qw4990)
    - コプロセッサに関連するリクエストソース情報を追加しました [#46514](https://github.com/pingcap/tidb/issues/46514) @[you06](https://github.com/you06)

+ TiKV

    - PDクライアントの接続リトライ処理にバックオフメカニズムを追加し、エラーリトライ中にリトライ間隔を徐々に増やしてPDへの負荷を軽減するようにしました [#15428](https://github.com/tikv/tikv/issues/15428) @[nolouch](https://github.com/nolouch)
    - スナップショットのモニタリングメトリクスを追加しました [#15401](https://github.com/tikv/tikv/issues/15401) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - リーダー移行中のPITRチェックポイント遅延の安定性を向上させました [#13638](https://github.com/tikv/tikv/issues/13638) @[YuJuncen](https://github.com/YuJuncen)
    - `safe-ts`に関連するログとモニタリングメトリクスを追加しました [#15082](https://github.com/tikv/tikv/issues/15082) @[ekexium](https://github.com/ekexium)
    - `resolved-ts`に関連するログとモニタリングメトリクスを提供するようにしました [#15082](https://github.com/tikv/tikv/issues/15082) @[ekexium](https://github.com/ekexium)
    - 地域が分割された場合、分割するキーが存在しない場合は、過剰なMVCCバージョンを除去するためにコンパクションがトリガーされるように圧縮メカニズムを最適化しました [#15282](https://github.com/tikv/tikv/issues/15282) @[SpadeA-Tang](https://github.com/SpadeA-Tang)

+ ツール

    + バックアップ＆リストア（BR）

        - ログバックアップのCPUオーバーヘッドを削減する`resolve lock`の問題を修正しました [#40759](https://github.com/pingcap/tidb/issues/40759) @[3pointer](https://github.com/3pointer)

## バグ修正

+ TiDB

    - ステールリードが利用不可能なレプリカを選択する問題を修正しました [#46198](https://github.com/pingcap/tidb/issues/46198) @[zyguan](https://github.com/zyguan)
    - ステールリードとスキーマキャッシュの非互換性による追加オーバーヘッドの問題を修正しました [#43481](https://github.com/pingcap/tidb/issues/43481) @[crazycs520](https://github.com/crazycs520)

+ TiKV

    - Titanが有効であり「Blob file deleted twice」エラーが発生した場合にTiKVの起動に失敗する問題を修正しました [#15454](https://github.com/tikv/tikv/issues/15454) @[Connor1996](https://github.com/Connor1996)
    - オンライン不安定リカバリがマージ中止を処理できない問題を修正しました [#15580](https://github.com/tikv/tikv/issues/15580) @[v01dstar](https://github.com/v01dstar)

+ PD

    - スケジューラが起動に長い時間がかかる問題を修正しました [#6920](https://github.com/tikv/pd/issues/6920) @[HuSharp](https://github.com/HuSharp)
    - スキャッターリージョン内のリーダーとピアを処理するロジックが一貫していない問題を修正しました [#6962](https://github.com/tikv/pd/issues/6962) @[bufferflies](https://github.com/bufferflies)
    - クラスターが再起動されるかPDリーダーが切り替わると`empty-region-count`モニタリングメトリクスが異常になる問題を修正しました [#7008](https://github.com/tikv/pd/issues/7008) @[CabinfeverB](https://github.com/CabinfeverB)
    
+ ツール

    + バックアップ＆リストア（BR）

        - PITRによる暗黙的プライマリキーの復元によって競合が発生する問題を修正しました [#46520](https://github.com/pingcap/tidb/issues/46520) @[3pointer](https://github.com/3pointer)
        - PITRがメタKVの復元時にエラーが発生する問題を修正しました [#46578](https://github.com/pingcap/tidb/issues/46578) @[Leavrth](https://github.com/Leavrth)
        - BR統合テストケースのエラーを修正しました [#45561](https://github.com/pingcap/tidb/issues/46561) @[purelind](https://github.com/purelind)
        - PITRがGCSからデータを復元できない問題を修正しました [#47022](https://github.com/pingcap/tidb/issues/47022) @[Leavrth](https://github.com/Leavrth)

    + TiCDC

        - PDノードのネットワーク分離によって引き起こされるTiCDCレプリケーション遅延の問題を修正しました [#9565](https://github.com/pingcap/tiflow/issues/9565) @[asddongmen](https://github.com/asddongmen)
        - CSV形式を使用する際にTiCDCが`UPDATE`操作を誤って`INSERT`に変更する問題を修正しました [#9658](https://github.com/pingcap/tiflow/issues/9658) @[3AceShowHand](https://github.com/3AceShowHand)
        - ユーザーパスワードが一部のログに記録される問題を修正しました [#9690](https://github.com/pingcap/tiflow/issues/9690) @[sdojjy](https://github.com/sdojjy)
        - SASL認証を使用するとTiCDCがパニックを起こす可能性がある問題を修正しました [#9669](https://github.com/pingcap/tiflow/issues/9669) @[sdojjy](https://github.com/sdojjy)
        - TiCDCレプリケーションタスクが一部の特定ケースで失敗する問題を修正しました [#9685](https://github.com/pingcap/tiflow/issues/9685) [#9697](https://github.com/pingcap/tiflow/issues/9697) [#9695](https://github.com/pingcap/tiflow/issues/9695) [#9736](https://github.com/pingcap/tiflow/issues/9736) @[hicqu](https://github.com/hicqu) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 大量の上流地域がある場合、TiCDCがTiKVノードの障害から速やかに回復できない問題を修正しました [#9741](https://github.com/pingcap/tiflow/issues/9741) @[sdojjy](https://github.com/sdojjy)
    
    + TiDB Lightning

        - TiCDCがターゲットサーバーに展開されている場合にTiDB Lightningが起動に失敗する問題を修正しました [#41040](https://github.com/pingcap/tidb/issues/41040) @[lance6716](https://github.com/lance6716)
        - PDトポロジが変更された場合にTiDB Lightningが起動に失敗する問題を修正しました [#46688](https://github.com/pingcap/tidb/issues/46688) @[lance6716](https://github.com/lance6716)
        - PDのリーダーが切り替わった後にTiDB Lightningがデータのインポートを継続できない問題を修正しました [#46540](https://github.com/pingcap/tidb/issues/46540) @[lance6716](https://github.com/lance6716)
        - ターゲットクラスターで実行中のTiCDCの存在を正確に検出できない問題を修正しました [#41040](https://github.com/pingcap/tidb/issues/41040) @[lance6716](https://github.com/lance6716)