---
title: TiDB 4.0 Beta リリースノート
aliases: ['/docs/dev/releases/release-4.0.0-beta/','/docs/dev/releases/4.0.0-beta/']
---

# TiDB 4.0 Beta リリースノート

リリース日: 2020年1月17日

TiDB バージョン: 4.0.0-beta

TiDB Ansible バージョン: 4.0.0-beta

## TiDB

+ `INSERT`/`REPLACE`/`DELETE`/`UPDATE` の実行中に使用されるメモリが `MemQuotaQuery` 構成項目で指定された制限を超えると、`OOMAction` 構成によって実行をキャンセルするか、ログの出力を行うようになりました。実際の動作は `OOMAction` 構成に依存します。 [#14179](https://github.com/pingcap/tidb/pull/14179) [#14289](https://github.com/pingcap/tidb/pull/14289) [#14299](https://github.com/pingcap/tidb/pull/14299)
+ `Index Join` のコストを計算する際に、ドライブテーブルとドリブンテーブルの行数を考慮して、精度を向上させました。[#12085](https://github.com/pingcap/tidb/pull/12085)
+ オプティマイザーの動作を制御し、オプティマイザーをより安定させるために、15個のSQLヒントを追加しました
    - [#11253](https://github.com/pingcap/tidb/pull/11253) [#11364](https://github.com/pingcap/tidb/pull/11364) [#11673](https://github.com/pingcap/tidb/pull/11673) [#11740](https://github.com/pingcap/tidb/pull/11740) [#11746](https://github.com/pingcap/tidb/pull/11746)
    - [#11809](https://github.com/pingcap/tidb/pull/11809) [#11996](https://github.com/pingcap/tidb/pull/11996) [#12043](https://github.com/pingcap/tidb/pull/12043) [#12059](https://github.com/pingcap/tidb/pull/12059) [#12246](https://github.com/pingcap/tidb/pull/12246)
    - [#12382](https://github.com/pingcap/tidb/pull/12382)
+ クエリに関与する列をインデックスで完全にカバーできる場合のパフォーマンスを改善しました [#12022](https://github.com/pingcap/tidb/pull/12022)
+ インデックスマージ機能をサポートし、表クエリのパフォーマンスを向上させました [#10121](https://github.com/pingcap/tidb/pull/10121) [#10512](https://github.com/pingcap/tidb/pull/10512) [#11245](https://github.com/pingcap/tidb/pull/11245) [#12225](https://github.com/pingcap/tidb/pull/12225) [#12248](https://github.com/pingcap/tidb/pull/12248) [#12305](https://github.com/pingcap/tidb/pull/12305) [#12843](https://github.com/pingcap/tidb/pull/12843)
+ レンジの計算のパフォーマンスを向上させ、CPUオーバーヘッドを削減するために、インデックス結果をキャッシュし、重複する結果を排除することでレンジの計算のパフォーマンスを向上させました [#12856](https://github.com/pingcap/tidb/pull/12856)
+ 遅いログのレベルを通常のログのレベルから分離しました [#12359](https://github.com/pingcap/tidb/pull/12359)
+ 単一のSQL文の実行中にメモリ使用量が `mem-quota-query` を超え、SQLに `Hash Join` が含まれる場合、一時ファイルを使用して中間結果をキャッシュするかどうかを制御するために `oom-use-tmp-storage` パラメータ（デフォルトでは `true`）を追加しました [#11832](https://github.com/pingcap/tidb/pull/11832) [#11937](https://github.com/pingcap/tidb/pull/11937) [#12116](https://github.com/pingcap/tidb/pull/12116) [#12067](https://github.com/pingcap/tidb/pull/12067)
+ `create index`/`alter table` を使用して、式インデックスを作成し、`drop index` を使用して式インデックスを削除することをサポートしました [#14117](https://github.com/pingcap/tidb/pull/14117)
+ `query-log-max-len` パラメータのデフォルト値を `4096` に増やし、切り捨てられるSQLの出力数を減らしました。このパラメータは動的に調整できます。[#12491](https://github.com/pingcap/tidb/pull/12491)
+ 主キーにシーケンシャルな整数を自動的に割り当てる問題によるホットスポット問題を回避するために、カラム属性で `AutoRandom` キーワードを追加することをサポートしました [#13127](https://github.com/pingcap/tidb/pull/13127)
+ テーブルロックをサポートしました [#11038](https://github.com/pingcap/tidb/pull/11038)
+ `ADMIN SHOW DDL JOBS` で `LIKE` や `WHERE` 句を使用して条件によるフィルタリングを行うことをサポートしました [#12484](https://github.com/pingcap/tidb/pull/12484)
+ `information_schema.tables` テーブルに `TIDB_ROW_ID_SHARDING_INFO` 列を追加し、`RowID` の散在情報（例: テーブル `A` の `SHARD_ROW_ID_BITS` 列の値は `"SHARD_BITS={bit_number}"` ）を出力するようになりました [#13418](https://github.com/pingcap/tidb/pull/13418)
+ SQLエラーメッセージのエラーコードを最適化し、`ERROR 1105 (HY000)` コードが複数のエラーメッセージで使用される状況を避けるようにしました
    - [#14002](https://github.com/pingcap/tidb/pull/14002) [#13874](https://github.com/pingcap/tidb/pull/13874) [#13733](https://github.com/pingcap/tidb/pull/13733) [#13654](https://github.com/pingcap/tidb/pull/13654) [#13646](https://github.com/pingcap/tidb/pull/13646)
    - [#13540](https://github.com/pingcap/tidb/pull/13540) [#13366](https://github.com/pingcap/tidb/pull/13366) [#13329](https://github.com/pingcap/tidb/pull/13329) [#13300](https://github.com/pingcap/tidb/pull/13300) [#13233](https://github.com/pingcap/tidb/pull/13233)
    - [#13033](https://github.com/pingcap/tidb/pull/13033) [#12866](https://github.com/pingcap/tidb/pull/12866) [#14054](https://github.com/pingcap/tidb/pull/14054)
+ 離散型の狭いデータ範囲を `point set` に変換し、行数の推定精度を向上させるためにCM-Sketchを使用するようにしました [#11524](https://github.com/pingcap/tidb/pull/11524)
+ 通常の `Analyze` から `TopN` 情報をCM-Sketchから抽出し、頻繁に発生する値を別々に保持するようにしました [#11409](https://github.com/pingcap/tidb/pull/11409)
+ CM-Sketchの深さと幅、および `TopN` 情報の数を動的に調整することをサポートしました [#11278](https://github.com/pingcap/tidb/pull/11278)
+ SQLバインディングを自動的にキャプチャし、進化させることをサポートしました [#13199](https://github.com/pingcap/tidb/pull/13199) [#12434](https://github.com/pingcap/tidb/pull/12434)
+ 通信のエンコード形式を最適化し、 `Chunk` を使用してTiKVとの通信パフォーマンスを向上させるようにしました [#12023](https://github.com/pingcap/tidb/pull/12023) [#12536](https://github.com/pingcap/tidb/pull/12536) [#12613](https://github.com/pingcap/tidb/pull/12613) [#12621](https://github.com/pingcap/tidb/pull/12621) [#12899](https://github.com/pingcap/tidb/pull/12899) [#13060](https://github.com/pingcap/tidb/pull/13060) [#13349](https://github.com/pingcap/tidb/pull/13349)
+ ワイドテーブルのパフォーマンスを改善するために、新しい行ストア形式をサポートしました [#12634](https://github.com/pingcap/tidb/pull/12634)
+ `Recover Binlog` インターフェースを最適化し、クライアントに戻る前にすべてのトランザクションが完了するのを待つようにしました [#13740](https://github.com/pingcap/tidb/pull/13740)
+ TiDBサーバーで有効になっているbinlogステータスをHTTP `info/all` インターフェースを使用してクエリすることをサポートしました [#13025](https://github.com/pingcap/tidb/pull/13025)
+ ペシミスティックトランザクションモードを使用する場合に、MySQL互換の `Read Committed` トランザクション分離レベルをサポートしました [#14087](https://github.com/pingcap/tidb/pull/14087)
+ 大規模なトランザクションをサポートします。トランザクションサイズは物理メモリのサイズによって制限されます。
    - [#11999](https://github.com/pingcap/tidb/pull/11999) [#11986](https://github.com/pingcap/tidb/pull/11986) [#11974](https://github.com/pingcap/tidb/pull/11974) [#11817](https://github.com/pingcap/tidb/pull/11817) [#11807](https://github.com/pingcap/tidb/pull/11807)
    - [#12133](https://github.com/pingcap/tidb/pull/12133) [#12223](https://github.com/pingcap/tidb/pull/12223) [#12980](https://github.com/pingcap/tidb/pull/12980) [#13123](https://github.com/pingcap/tidb/pull/13123) [#13299](https://github.com/pingcap/tidb/pull/13299)
    - [#13432](https://github.com/pingcap/tidb/pull/13432) [#13599](https://github.com/pingcap/tidb/pull/13599)
+ `Kill`の安定性を向上させます [#10841](https://github.com/pingcap/tidb/pull/10841)
+ `LOAD DATA`での16進数および2進数表現をセパレータとしてサポートします [#11029](https://github.com/pingcap/tidb/pull/11029)
+ `IndexLookupJoin`のパフォーマンスを向上させ、実行中のメモリ消費を削減するために`IndexLookupJoin`を`IndexHashJoin`と`IndexMergeJoin`に分割します [#8861](https://github.com/pingcap/tidb/pull/8861) [#12139](https://github.com/pingcap/tidb/pull/12139) [#12349](https://github.com/pingcap/tidb/pull/12349) [#13238](https://github.com/pingcap/tidb/pull/13238) [#13451](https://github.com/pingcap/tidb/pull/13451) [#13714](https://github.com/pingcap/tidb/pull/13714)
+ RBAC関連の複数の問題を修正します [#13896](https://github.com/pingcap/tidb/pull/13896) [#13820](https://github.com/pingcap/tidb/pull/13820) [#13940](https://github.com/pingcap/tidb/pull/13940) [#14090](https://github.com/pingcap/tidb/pull/14090) [#13940](https://github.com/pingcap/tidb/pull/13940) [#13014](https://github.com/pingcap/tidb/pull/13014)
+ `VIEW`が`UNION`を含む`SELECT`ステートメントのために作成できない問題を修正します [#12595](https://github.com/pingcap/tidb/pull/12595)
+ `CAST`関数に関する複数の問題を修正します
    - [#12858](https://github.com/pingcap/tidb/pull/12858) [#11968](https://github.com/pingcap/tidb/pull/11968) [#11640](https://github.com/pingcap/tidb/pull/11640) [#11483](https://github.com/pingcap/tidb/pull/11483) [#11493](https://github.com/pingcap/tidb/pull/11493)
    - [#11376](https://github.com/pingcap/tidb/pull/11376) [#11355](https://github.com/pingcap/tidb/pull/11355) [#11114](https://github.com/pingcap/tidb/pull/11114) [#14405](https://github.com/pingcap/tidb/pull/14405) [#14323](https://github.com/pingcap/tidb/pull/14323)
    - [#13837](https://github.com/pingcap/tidb/pull/13837) [#13401](https://github.com/pingcap/tidb/pull/13401) [#13334](https://github.com/pingcap/tidb/pull/13334) [#12652](https://github.com/pingcap/tidb/pull/12652) [#12864](https://github.com/pingcap/tidb/pull/12864)
    - [#12623](https://github.com/pingcap/tidb/pull/12623) [#11989](https://github.com/pingcap/tidb/pull/11989)
+ TiKV RPCの遅いログにTiKV RPCの詳細な`backoff`情報を出力し、トラブルシューティングを容易にします [#13770](https://github.com/pingcap/tidb/pull/13770)
+ コストのかかるログでメモリ統計のフォーマットを最適化し統一します [#12809](https://github.com/pingcap/tidb/pull/12809)
+ `EXPLAIN`の明示的なフォーマットを最適化し、オペレータのメモリおよびディスクの使用状況に関する情報を出力するようにサポートします [#13914](https://github.com/pingcap/tidb/pull/13914) [#13692](https://github.com/pingcap/tidb/pull/13692) [#13686](https://github.com/pingcap/tidb/pull/13686) [#11415](https://github.com/pingcap/tidb/pull/11415) [#13927](https://github.com/pingcap/tidb/pull/13927) [#13764](https://github.com/pingcap/tidb/pull/13764) [#13720](https://github.com/pingcap/tidb/pull/13720)
+ `LOAD DATA`で重複値のチェックをトランザクションサイズに基づいて最適化し、`tidb_dml_batch_size`パラメータを設定してトランザクションサイズをサポートします [#11132](https://github.com/pingcap/tidb/pull/11132)
+ `LOAD DATA`のパフォーマンスを最適化し、データ準備ルーチンとコミットルーチンを分離して作業量を異なるワーカーに割り当てます [#11533](https://github.com/pingcap/tidb/pull/11533) [#11284](https://github.com/pingcap/tidb/pull/11284)

## TiKV

+ RocksDBのバージョンを6.4.6にアップグレード
+ ディスクスペースが使用されるとTiKVが自動的に2GBの空ファイルを作成してしまい、システムがコンパクションタスクを正常に実行できない問題を修正します [#6321](https://github.com/tikv/tikv/pull/6321)
+ クイックバックアップと復元をサポートします
    - [#6462](https://github.com/tikv/tikv/pull/6462) [#6395](https://github.com/tikv/tikv/pull/6395) [#6378](https://github.com/tikv/tikv/pull/6378) [#6374](https://github.com/tikv/tikv/pull/6374) [#6349](https://github.com/tikv/tikv/pull/6349)
    - [#6339](https://github.com/tikv/tikv/pull/6339) [#6308](https://github.com/tikv/tikv/pull/6308) [#6295](https://github.com/tikv/tikv/pull/6295) [#6286](https://github.com/tikv/tikv/pull/6286) [#6283](https://github.com/tikv/tikv/pull/6283)
    - [#6261](https://github.com/tikv/tikv/pull/6261) [#6222](https://github.com/tikv/tikv/pull/6222) [#6209](https://github.com/tikv/tikv/pull/6209) [#6204](https://github.com/tikv/tikv/pull/6204) [#6202](https://github.com/tikv/tikv/pull/6202)
    - [#6198](https://github.com/tikv/tikv/pull/6198) [#6186](https://github.com/tikv/tikv/pull/6186) [#6177](https://github.com/tikv/tikv/pull/6177) [#6146](https://github.com/tikv/tikv/pull/6146) [#6071](https://github.com/tikv/tikv/pull/6071)
    - [#6042](https://github.com/tikv/tikv/pull/6042) [#5877](https://github.com/tikv/tikv/pull/5877) [#5806](https://github.com/tikv/tikv/pull/5806) [#5803](https://github.com/tikv/tikv/pull/5803) [#5800](https://github.com/tikv/tikv/pull/5800)
    - [#5781](https://github.com/tikv/tikv/pull/5781) [#5772](https://github.com/tikv/tikv/pull/5772) [#5689](https://github.com/tikv/tikv/pull/5689) [#5683](https://github.com/tikv/tikv/pull/5683)
+ Follower レプリカからのデータ読み取りをサポート
    - [#5051](https://github.com/tikv/tikv/pull/5051) [#5118](https://github.com/tikv/tikv/pull/5118) [#5213](https://github.com/tikv/tikv/pull/5213) [#5316](https://github.com/tikv/tikv/pull/5316) [#5401](https://github.com/tikv/tikv/pull/5401)
    - [#5919](https://github.com/tikv/tikv/pull/5919) [#5887](https://github.com/tikv/tikv/pull/5887) [#6340](https://github.com/tikv/tikv/pull/6340) [#6348](https://github.com/tikv/tikv/pull/6348) [#6396](https://github.com/tikv/tikv/pull/6396)
+ TiDB を介したデータ読み取りのパフォーマンスを改善 [#5682](https://github.com/tikv/tikv/pull/5682)
+ TiKV と TiDB で `CAST` 関数の挙動が一貫していない問題を修正
    - [#6459](https://github.com/tikv/tikv/pull/6459) [#6461](https://github.com/tikv/tikv/pull/6461) [#6458](https://github.com/tikv/tikv/pull/6458) [#6447](https://github.com/tikv/tikv/pull/6447) [#6440](https://github.com/tikv/tikv/pull/6440)
    - [#6425](https://github.com/tikv/tikv/pull/6425) [#6424](https://github.com/tikv/tikv/pull/6424) [#6390](https://github.com/tikv/tikv/pull/6390) [#5842](https://github.com/tikv/tikv/pull/5842) [#5528](https://github.com/tikv/tikv/pull/5528)
    - [#5334](https://github.com/tikv/tikv/pull/5334) [#5199](https://github.com/tikv/tikv/pull/5199) [#5167](https://github.com/tikv/tikv/pull/5167) [#5146](https://github.com/tikv/tikv/pull/5146) [#5141](https://github.com/tikv/tikv/pull/5141)
    - [#4998](https://github.com/tikv/tikv/pull/4998) [#5029](https://github.com/tikv/tikv/pull/5029) [#5099](https://github.com/tikv/tikv/pull/5099) [#5006](https://github.com/tikv/tikv/pull/5006) [#5095](https://github.com/tikv/tikv/pull/5095)
    - [#5093](https://github.com/tikv/tikv/pull/5093) [#5090](https://github.com/tikv/tikv/pull/5090) [#4987](https://github.com/tikv/tikv/pull/4987) [#5066](https://github.com/tikv/tikv/pull/5066) [#5038](https://github.com/tikv/tikv/pull/5038)
    - [#4962](https://github.com/tikv/tikv/pull/4962) [#4890](https://github.com/tikv/tikv/pull/4890) [#4727](https://github.com/tikv/tikv/pull/4727) [#6060](https://github.com/tikv/tikv/pull/6060) [#5761](https://github.com/tikv/tikv/pull/5761)
    - [#5793](https://github.com/tikv/tikv/pull/5793) [#5468](https://github.com/tikv/tikv/pull/5468) [#5540](https://github.com/tikv/tikv/pull/5540) [#5548](https://github.com/tikv/tikv/pull/5548) [#5455](https://github.com/tikv/tikv/pull/5455)
    - [#5543](https://github.com/tikv/tikv/pull/5543) [#5433](https://github.com/tikv/tikv/pull/5433) [#5431](https://github.com/tikv/tikv/pull/5431) [#5423](https://github.com/tikv/tikv/pull/5423) [#5179](https://github.com/tikv/tikv/pull/5179)
    - [#5134](https://github.com/tikv/tikv/pull/5134) [#4685](https://github.com/tikv/tikv/pull/4685) [#4650](https://github.com/tikv/tikv/pull/4650) [#6463](https://github.com/tikv/tikv/pull/6463)

## PD

+ ストレージノードの負荷情報に基づいてホットスポットのスケジューリングを最適化するサポート
    - [#1870](https://github.com/pingcap/pd/pull/1870) [#1982](https://github.com/pingcap/pd/pull/1982) [#1998](https://github.com/pingcap/pd/pull/1998) [#1843](https://github.com/pingcap/pd/pull/1843) [#1750](https://github.com/pingcap/pd/pull/1750)
+ 異なるスケジューリングルールを組み合わせて任意のデータ範囲のレプリカ数、ストレージ場所、ストレージホストのタイプおよびロールを制御する Placement Rules 機能を追加
    - [#2051](https://github.com/pingcap/pd/pull/2051) [#1999](https://github.com/pingcap/pd/pull/1999) [#2042](https://github.com/pingcap/pd/pull/2042) [#1917](https://github.com/pingcap/pd/pull/1917) [#1904](https://github.com/pingcap/pd/pull/1904)
    - [#1897](https://github.com/pingcap/pd/pull/1897) [#1894](https://github.com/pingcap/pd/pull/1894) [#1865](https://github.com/pingcap/pd/pull/1865) [#1855](https://github.com/pingcap/pd/pull/1855) [#1834](https://github.com/pingcap/pd/pull/1834)
+ プラグインの使用をサポート (実験的) [#1799](https://github.com/pingcap/pd/pull/1799)
+ スケジューラがカスタマイズされた構成およびキー範囲をサポート (実験的) [#1735](https://github.com/pingcap/pd/pull/1735) [#1783](https://github.com/pingcap/pd/pull/1783) [#1791](https://github.com/pingcap/pd/pull/1791)
+ クラスタ負荷情報に基づいてスケジューリング速度を自動的に調整するサポート (実験中、初期設定では無効) [#1875](https://github.com/pingcap/pd/pull/1875) [#1887](https://github.com/pingcap/pd/pull/1887) [#1902](https://github.com/pingcap/pd/pull/1902)

## Tools

+ TiDB Lightning
    - 下流データベースのパスワードを設定するコマンドラインツールにパラメータを追加 [#253](https://github.com/pingcap/tidb-lightning/pull/253)

## TiDB Ansible

+ パッケージのチェックサム検証を追加してダウンロードしたパッケージが不完全な場合の対処をサポート [#1002](https://github.com/pingcap/tidb-ansible/pull/1002)
+ 必要システムデーモンのバージョンを確認するサポート (systemd-219-52 以上が必要) [#1020](https://github.com/pingcap/tidb-ansible/pull/1020) [#1074](https://github.com/pingcap/tidb-ansible/pull/1074)
+ TiDB Lightning の開始時にログディレクトリが誤って作成される問題を修正 [#1103](https://github.com/pingcap/tidb-ansible/pull/1103)
+ TiDB Lightning のカスタムポートが無効な問題を修正 [#1107](https://github.com/pingcap/tidb-ansible/pull/1107)
+ TiFlash の展開とメンテナンスをサポート [#1119](https://github.com/pingcap/tidb-ansible/pull/1119)