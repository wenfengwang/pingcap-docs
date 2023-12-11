---
title: TiDB 6.5.1 リリースノート
summary: TiDB 6.5.1 の互換性の変更、改善、およびバグ修正についての情報をご覧ください。

# TiDB 6.5.1 リリースノート

リリース日: 2023年3月10日

TiDB バージョン: 6.5.1

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.5/quick-start-with-tidb) | [本番環境の展開](https://docs.pingcap.com/tidb/v6.5/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.5.1#version-list)

## 互換性の変更

- 2023年2月20日以降、新しいバージョンの TiDB および TiDB Dashboard（v6.5.1 を含む）では、[テレメトリ機能](/telemetry.md) がデフォルトで無効になり、使用情報は PingCAP と共有されません。これらのバージョンにアップグレードする前に、クラスタがデフォルトのテレメトリ構成を使用している場合、アップグレード後にテレメトリ機能が無効になります。詳細なバージョンについては、[TiDB リリースタイムライン](/releases/release-timeline.md) を参照してください。

    - [`tidb_enable_telemetry`](/system-variables.md#tidb_enable_telemetry-new-in-v402) システム変数のデフォルト値が `ON` から `OFF` に変更されました。
    - TiDB [`enable-telemetry`](/tidb-configuration-file.md#enable-telemetry-new-in-v402) 構成項目のデフォルト値が `true` から `false` に変更されました。
    - PD [`enable-telemetry`](/pd-configuration-file.md#enable-telemetry) 構成項目のデフォルト値が `true` から `false` に変更されました。

- v1.11.3 以降、新しく展開された TiUP では、テレメトリ機能がデフォルトで無効になり、使用情報は収集されません。v1.11.3 より前の TiUP バージョンから v1.11.3 以上のバージョンにアップグレードする場合、テレメトリ機能はアップグレード前と同じ状態に保たれます。

- パーティション化されたテーブルの列タイプの変更をサポートしなくなりました。これは潜在的な正しさの問題のためです。[#40620](https://github.com/pingcap/tidb/issues/40620) @[mjonss](https://github.com/mjonss)

- TiKV [`advance-ts-interval`](/tikv-configuration-file.md#advance-ts-interval) 構成項目のデフォルト値が `1s` から `20s` に変更されました。この構成項目を変更すると、Stale Read データのレイテンシを低減し、タイムリネスを向上させることができます。詳細については、[Stale Read レイテンシの低減](/stale-read.md#reduce-stale-read-latency) を参照してください。

## 改善点

+ TiDB

    - v6.5.1 から、TiDB Operator v1.4.3 以降で展開された TiDB クラスタは IPv6 アドレスをサポートします。これにより、TiDB はより大きなアドレス空間をサポートし、より優れたセキュリティとネットワークパフォーマンスを提供できます。

        - IPv6 アドレッシングの完全サポート: TiDB は、クライアント接続、ノード間の内部通信、外部システムとの通信を含むすべてのネットワーク接続で IPv6 アドレスの使用をサポートしています。
        - デュアルスタックのサポート: IPv6 に完全に切り替える準備ができていない場合、TiDB はデュアルスタックネットワークをサポートします。つまり、同じ TiDB クラスタで IPv4 と IPv6 アドレスの両方を使用し、IPv6 を優先するネットワーク展開モードを選択できます。

      IPv6 展開に関する詳細情報については、[TiDB on Kubernetes documentation](https://docs.pingcap.com/tidb-in-kubernetes/stable/configure-a-tidb-cluster#ipv6-support) を参照してください。

    - TiDB クラスタの初期化時に実行される SQL スクリプトを指定できるようになりました。[#35624](https://github.com/pingcap/tidb/issues/35624) @[morgo](https://github.com/morgo)

        TiDB v6.5.1 では、新しい構成項目 [`initialize-sql-file`](https://docs.pingcap.com/tidb/v6.5/tidb-configuration-file#initialize-sql-file-new-in-v651) が追加されました。TiDB クラスタを初めて起動するときに、`--initialize-sql-file` コマンドラインパラメーターを設定して実行する SQL スクリプトを指定できます。これは、システム変数の値の変更、ユーザーの作成、権限の付与などの操作が必要な場合に使用できます。詳細については、[documentation](https://docs.pingcap.com/tidb/v6.5/tidb-configuration-file#initialize-sql-file-new-in-v651) を参照してください。

    - メモリリージョンキャッシュを定期的にクリアしてメモリリークやパフォーマンス低下を回避しました。[#40461](https://github.com/pingcap/tidb/issues/40461) @[sticnarf](https://github.com/sticnarf)
    - PROXY プロトコルのフォールバックモードを制御する新しい構成項目 `--proxy-protocol-fallbackable` を追加しました。このパラメーターが `true` に設定されていると、TiDB は PROXY クライアント接続と PROXY プロトコルヘッダーなしのクライアント接続を受け入れます。[#41409](https://github.com/pingcap/tidb/issues/41409) @[blacktear23](https://github.com/blacktear23)
    - メモリトラッカーの精度を向上させました。[#40900](https://github.com/pingcap/tidb/issues/40900) [#40500](https://github.com/pingcap/tidb/issues/40500) @[wshwsh12](https://github.com/wshwsh12)
    - プランキャッシュが効果を上げない場合、システムは警告として理由を返します。[#40210](https://github.com/pingcap/tidb/pull/40210) @[qw4990](https://github.com/qw4990)
    - 範囲外推定のための最適化戦略を改善しました。[#39008](https://github.com/pingcap/tidb/issues/39008) @[time-and-fate](https://github.com/time-and-fate)

+ TiKV

    - 1コア未満のCPUで TiKV を起動することをサポートしました。[#13586](https://github.com/tikv/tikv/issues/13586) [#13752](https://github.com/tikv/tikv/issues/13752) [#14017](https://github.com/tikv/tikv/issues/14017) @[andreid-db](https://github.com/andreid-db)
    - 統合リードプール (`readpool.unified.max-thread-count`) のスレッド制限をCPUクォータの10倍に増やし、高並行クエリをよりよく処理できるようにしました。[#13690](https://github.com/tikv/tikv/issues/13690) @[v01dstar](https://github.com/v01dstar)
    - `resolved-ts.advance-ts-interval` のデフォルト値を `"1s"` から `"20s"` に変更し、クロスリージョントラフィックを減らしました。[#14100](https://github.com/tikv/tikv/issues/14100) @[overvenus](https://github.com/overvenus)

+ TiFlash

    - データボリュームが大きい場合の TiFlash の起動を大幅に高速化しました。[#6395](https://github.com/pingcap/tiflash/issues/6395) @[hehechen](https://github.com/hehechen)

+ ツール

    + バックアップ＆リストア (BR)

        - TiKV 側でログバックアップファイルのダウンロードの並行性を最適化し、通常のシナリオでのPITRのパフォーマンスを向上させました。[#14206](https://github.com/tikv/tikv/issues/14206) @[YuJuncen](https://github.com/YuJuncen)

    + TiCDC

        - システムスループットを最適化するために、プルベースのシンクを有効にしました。[#8232](https://github.com/pingcap/tiflow/issues/8232) @[hi-rustin](https://github.com/hi-rustin)
        - redoログを GCS 互換または Azure 互換のオブジェクトストレージに保存することをサポートしました。[#7987](https://github.com/pingcap/tiflow/issues/7987) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - 非同期モードで MQシンクと MySQLシンクを実装し、シンクスループットを向上させました。[#5928](https://github.com/pingcap/tiflow/issues/5928) @[amyangfei](https://github.com/amyangfei) @[CharlesCheung96](https://github.com/CharlesCheung96)

## バグ修正

+ TiDB

    - [`pessimistic-auto-commit`](/tidb-configuration-file.md#pessimistic-auto-commit-new-in-v600) 構成項目がポイント取得クエリに対して効果を発揮しない問題を修正しました。[#39928](https://github.com/pingcap/tidb/issues/39928) @[zyguan](https://github.com/zyguan)
    - 長時間のセッション接続で `INSERT` または `REPLACE` ステートメントがパニックを起こす可能性がある問題を修正しました。[#40351](https://github.com/pingcap/tidb/issues/40351) @[winoros](https://github.com/winoros)
    - `auto analyze` がグレースフルシャットダウンに長い時間を要する問題を修正しました。[#40038](https://github.com/pingcap/tidb/issues/40038) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
- DDLのインジェクション中にデータ競合が発生する可能性がある問題を修正 [#40970](https://github.com/pingcap/tidb/issues/40970) @[tangenta](https://github.com/tangenta)
- インデックスが追加される際にデータ競合が発生する可能性がある問題を修正 [#40879](https://github.com/pingcap/tidb/issues/40879) @[tangenta](https://github.com/tangenta)
- テーブル内に多くのRegionが存在する場合、無効なRegionキャッシュがあるため、インデックス追加操作が非効率である問題を修正 [#38436](https://github.com/pingcap/tidb/issues/38436) @[tangenta](https://github.com/tangenta)
- 初期化中にTiDBがデッドロックする可能性がある問題を修正 [#40408](https://github.com/pingcap/tidb/issues/40408) @[Defined2014](https://github.com/Defined2014)
- `NULL`値を構築する際にTiDBが`NULL`値を誤って処理したため、予期しないデータが読み取られる可能性がある問題を修正 [#40158](https://github.com/pingcap/tidb/issues/40158) @[tiancaiamao](https://github.com/tiancaiamao)
- メモリ再利用によってシステム変数の値が一部のケースで誤って変更される可能性がある問題を修正 [#40979](https://github.com/pingcap/tidb/issues/40979) @[lcwangchao](https://github.com/lcwangchao)
- テーブルの主キーが`ENUM`列を含む場合、TTLタスクが失敗する可能性がある問題を修正 [#40456](https://github.com/pingcap/tidb/issues/40456) @[lcwangchao](https://github.com/lcwangchao)
- ユニークインデックスを追加する際にTiDBがパニックを起こす可能性がある問題を修正 [#40592](https://github.com/pingcap/tidb/issues/40592) @[tangenta](https://github.com/tangenta)
- 同じテーブルを同時に切り詰める際にMDLによって一部の切り詰め操作がブロックされない可能性がある問題を修正 [#40484](https://github.com/pingcap/tidb/issues/40484) @[wjhuang2016](https://github.com/wjhuang2016)
- 動的トリミングモードでパーティションテーブルにグローバルバインディングが作成された後にTiDBが再起動できない可能性がある問題を修正 [#40368](https://github.com/pingcap/tidb/issues/40368) @[Yisaer](https://github.com/Yisaer)
- "カーソルリード"メソッドを使用してデータを読む際に、GCのためにエラーが返される可能性がある問題を修正 [#39447](https://github.com/pingcap/tidb/issues/39447) @[zyguan](https://github.com/zyguan)
- `SHOW PROCESSLIST`の結果において`EXECUTE`情報が`NULL`である可能性がある問題を修正 [#41156](https://github.com/pingcap/tidb/issues/41156) @[YangKeao](https://github.com/YangKeao)
- `globalMemoryControl`がクエリをキャンセルしようとする際、`KILL`操作が終了しない可能性がある問題を修正 [#41057](https://github.com/pingcap/tidb/issues/41057) @[wshwsh12](https://github.com/wshwsh12)
- `indexMerge`でエラーが発生した後にTiDBがパニックを起こす可能性がある問題を修正 [#41047](https://github.com/pingcap/tidb/issues/41047) [#40877](https://github.com/pingcap/tidb/issues/40877) @[guo-shaoge](https://github.com/guo-shaoge) @[windtalker](https://github.com/windtalker)
- `KILL`によって`ANALYZE`文が終了する可能性がある問題を修正[#41825](https://github.com/pingcap/tidb/issues/41825) @[XuHuaiyu](https://github.com/XuHuaiyu)
- `indexMerge`でゴルーチンリークが発生する可能性がある問題を修正 [#41545](https://github.com/pingcap/tidb/issues/41545) [#41605](https://github.com/pingcap/tidb/issues/41605) @[guo-shaoge](https://github.com/guo-shaoge)
- `TINYINT`/`SMALLINT`/`INT`の符号なしの値と`DECIMAL`/`FLOAT`/`DOUBLE`の値 `<` `0` を比較する際に、間違った結果が生成される可能性がある問題を修正 [#41736](https://github.com/pingcap/tidb/issues/41736) @[LittleFall](https://github.com/LittleFall)
- `tidb_enable_reuse_chunk`を有効にするとメモリリークが発生する可能性がある問題を修正 [#40987](https://github.com/pingcap/tidb/issues/40987) @[guo-shaoge](https://github.com/guo-shaoge)
- タイムゾーンのデータ競合によってデータとインデックスの整合性が損なわれる可能性がある問題を修正 [#40710](https://github.com/pingcap/tidb/issues/40710) @[wjhuang2016](https://github.com/wjhuang2016)
- `batch cop`の実行中にスキャンの詳細情報が不正確になる可能性がある問題を修正 [#41582](https://github.com/pingcap/tidb/issues/41582) @[you06](https://github.com/you06)
- `cop`の上限並行性が制限されていない問題を修正 [#41134](https://github.com/pingcap/tidb/issues/41134) @[you06](https://github.com/you06)
- `cursor read`における`statement context`が誤ってキャッシュされる問題を修正 [#39998](https://github.com/pingcap/tidb/issues/39998) @[zyguan](https://github.com/zyguan)
- メモリリークや性能の低下を避けるために古いRegionキャッシュを定期的にクリーンアップする問題を修正 [#40355](https://github.com/pingcap/tidb/issues/40355) @[sticnarf](https://github.com/sticnarf)
- `year <cmp> const`を含むクエリでプランキャッシュを使用すると誤った結果が得られる可能性がある問題を修正 [#41626](https://github.com/pingcap/tidb/issues/41626) @[qw4990](https://github.com/qw4990)
- 大きな範囲と多くのデータ変更を使用してクエリを行う際に大きな推定エラーが発生する問題を修正 [#39593](https://github.com/pingcap/tidb/issues/39593) @[time-and-fate](https://github.com/time-and-fate)
- プランキャッシュを使用する際にJoin演算子を通して一部の条件が押し下げられない問題を修正 [#40093](https://github.com/pingcap/tidb/issues/40093) [#38205](https://github.com/pingcap/tidb/issues/38205) @[qw4990](https://github.com/qw4990)
- `SET`タイプの列に対してIndexMergeプランが不正確な範囲を生成する可能性がある問題を修正 [#41273](https://github.com/pingcap/tidb/issues/41273) [#41293](https://github.com/pingcap/tidb/issues/41293) @[time-and-fate](https://github.com/time-and-fate)
- `int_col <cmp> decimal`条件を処理する際にPlan CacheがFullScanプランをキャッシュし、不正な結果を返す可能性がある問題を修正 [#40679](https://github.com/pingcap/tidb/issues/40679) [#41032](https://github.com/pingcap/tidb/issues/41032) @[qw4990](https://github.com/qw4990)
- `int_col in (decimal...)`条件を処理する際にPlan CacheがFullScanプランをキャッシュし、不正な結果を返す可能性がある問題を修正 [#40224](https://github.com/pingcap/tidb/issues/40224) @[qw4990](https://github.com/qw4990)
- `INSERT`文に対して`ignore_plan_cache`ヒントが機能しない可能性がある問題を修正 [#40079](https://github.com/pingcap/tidb/issues/40079) [#39717](https://github.com/pingcap/tidb/issues/39717) @[qw4990](https://github.com/qw4990)
- Auto AnalyzeがTiDBの終了を妨げる可能性がある問題を修正 [#40038](https://github.com/pingcap/tidb/issues/40038) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
- パーティションテーブルの符号なしのプライマリキーで不正なアクセス間隔が作成される可能性がある問題を修正 [#40309](https://github.com/pingcap/tidb/issues/40309) @[winoros](https://github.com/winoros)
- Plan CacheがShuffle演算子をキャッシュし、不正な結果を返す可能性がある問題を修正 [#38335](https://github.com/pingcap/tidb/issues/38335) @[qw4990](https://github.com/qw4990)
- パーティションテーブルに対してGlobal Bindingを作成するとTiDBが失敗する可能性がある問題を修正 [#40368](https://github.com/pingcap/tidb/issues/40368) @[Yisaer](https://github.com/Yisaer)
- スローログにおいてクエリプラン演算子が欠落する可能性がある問題を修正 [#41458](https://github.com/pingcap/tidb/issues/41458) @[time-and-fate](https://github.com/time-and-fate)
- TopN演算子に仮想カラムが誤ってTiKVまたはTiFlashにプッシュダウンされたときに誤った結果が返される可能性がある問題を修正する[#41355](https://github.com/pingcap/tidb/issues/41355) @[Dousir9](https://github.com/Dousir9)
    - インデックスの追加時にデータの一貫性の問題を修正する[#40698](https://github.com/pingcap/tidb/issues/40698) [#40730](https://github.com/pingcap/tidb/issues/40730) [#41459](https://github.com/pingcap/tidb/issues/41459) [#40464](https://github.com/pingcap/tidb/issues/40464) [#40217](https://github.com/pingcap/tidb/issues/40217) @[tangenta](https://github.com/tangenta)
    - インデックスの追加時に`Pessimistic lock not found`エラーが発生する問題を修正する[#41515](https://github.com/pingcap/tidb/issues/41515) @[tangenta](https://github.com/tangenta)
    - ユニークインデックスの追加時に重複キーエラーが誤って報告される問題を修正する[#41630](https://github.com/pingcap/tidb/issues/41630) @[tangenta](https://github.com/tangenta)
    - TiDBで`paging`を使用した際のパフォーマンス劣化の問題を修正する[#40741](https://github.com/pingcap/tidb/issues/40741) @[solotzg](https://github.com/solotzg)

+ TiKV

    - 解決したTSがネットワークトラフィックを増加させる問題を修正する[#14092](https://github.com/tikv/tikv/issues/14092) @[overvenus](https://github.com/overvenus)
    - 失敗した悲観的DMLの実行後にTiDBとTiKV間のネットワーク障害によるデータの不一致の問題を修正する[#14038](https://github.com/tikv/tikv/issues/14038) @[MyonKeminta](https://github.com/MyonKeminta)
    - `const Enum`型を他の型にキャストする際に発生するエラーを修正する[#14156](https://github.com/tikv/tikv/issues/14156) @[wshwsh12](https://github.com/wshwsh12)
    - copタスク内のpagingが正確でない問題を修正する[#14254](https://github.com/tikv/tikv/issues/14254) @[you06](https://github.com/you06)
    - `batch_cop`モードでの`scan_detail`フィールドが正確でない問題を修正する[#14109](https://github.com/tikv/tikv/issues/14109) @[you06](https://github.com/you06)
    - Raft Engine内の潜在的なエラーを修正し、TiKVがRaftデータの破損を検出して再起動に失敗する可能性がある問題を修正する[#14338](https://github.com/tikv/tikv/issues/14338) @[tonyxuqqi](https://github.com/tonyxuqqi)

+ PD

    - 特定の条件下で実行の`replace-down-peer`が遅くなる問題を修正する[#5788](https://github.com/tikv/pd/issues/5788) @[HundunDM](https://github.com/HunDunDM)
    - PDが予期せず複数のLearnerをリージョンに追加する問題を修正する[#5786](https://github.com/tikv/pd/issues/5786) @[HundunDM](https://github.com/HunDunDM)
    - リージョンScatterタスクが予期しなく不要なレプリカを生成する問題を修正する[#5909](https://github.com/tikv/pd/issues/5909) @[HundunDM](https://github.com/HunDunDM)
    - `ReportMinResolvedTS`の呼び出しが頻繁すぎるとPD OOM問題が発生する問題を修正する[#5965](https://github.com/tikv/pd/issues/5965) @[HundunDM](https://github.com/HunDunDM)
    - リージョンScatterがリーダーの不均衡分布を引き起こす可能性がある問題を修正する[#6017](https://github.com/tikv/pd/issues/6017) @[HundunDM](https://github.com/HunDunDM)

+ TiFlash

    - セミジョインが直積演算を計算する際に過剰なメモリを消費する問題を修正する[#6730](https://github.com/pingcap/tiflash/issues/6730) @[gengliqi](https://github.com/gengliqi)
    - TiFlashのログ検索が遅すぎる問題を修正する[#6829](https://github.com/pingcap/tiflash/issues/6829) @[hehechen](https://github.com/hehechen)
    - 繰り返しの再起動後にファイルが誤って削除され、TiFlashが開始できなくなる問題を修正する[#6486](https://github.com/pingcap/tiflash/issues/6486) @[JaySon-Huang](https://github.com/JaySon-Huang)
    - 新しい列を追加した後のクエリ実行時にTiFlashがエラーを報告する可能性がある問題を修正する[#6726](https://github.com/pingcap/tiflash/issues/6726) @[JaySon-Huang](https://github.com/JaySon-Huang)
    - TiFlashがIPv6構成をサポートしていない問題を修正する[#6734](https://github.com/pingcap/tiflash/issues/6734) @[ywqzzy](https://github.com/ywqzzy)

+ Tools

    + バックアップ＆リストア（BR）

        - PDとtidb-server間の接続障害によりPITRバックアップの進行が進まなくなる問題を修正する[#41082](https://github.com/pingcap/tidb/issues/41082) @[YuJuncen](https://github.com/YuJuncen)
        - PDとTiKV間の接続障害によりTiKVがPITRタスクのリスニングを行えない問題を修正する[#14159](https://github.com/tikv/tikv/issues/14159) @[YuJuncen](https://github.com/YuJuncen)
        - PITRがPDクラスタの構成変更をサポートしていない問題を修正する[#14165](https://github.com/tikv/tikv/issues/14165) @[YuJuncen](https://github.com/YuJuncen)
        - PITRがCAバンドルをサポートしていない問題を修正する[#38775](https://github.com/pingcap/tidb/issues/38775) @[3pointer](https://github.com/3pointer)
        - PITRバックアップタスクが削除されると、残存するバックアップデータが新しいタスクでデータの不一致を引き起こす問題を修正する[#40403](https://github.com/pingcap/tidb/issues/40403) @[joccau](https://github.com/joccau)
        - BRが`backupmeta`ファイルを解析する際にパニックが発生する問題を修正する[#40878](https://github.com/pingcap/tidb/issues/40878) @[MoCuishle28](https://github.com/MoCuishle28)
        - リージョンサイズを取得する際の失敗によりリストアが中断する問題を修正する[#36053](https://github.com/pingcap/tidb/issues/36053) @[YuJuncen](https://github.com/YuJuncen)
        - TiDBクラスタにPITRバックアップタスクがない場合、`resolve lock`の頻度が高すぎる問題を修正する[#40759](https://github.com/pingcap/tidb/issues/40759) @[joccau](https://github.com/joccau)
        - ログバックアップが実行されるクラスタにデータをリストアすると、ログバックアップファイルをリストアできなくなる問題を修正する[#40797](https://github.com/pingcap/tidb/issues/40797) @[Leavrth](https://github.com/Leavrth)
        - フルバックアップの失敗後にチェックポイントからバックアップを再開しようとした際にパニックが発生する問題を修正する[#40704](https://github.com/pingcap/tidb/issues/40704) @[Leavrth](https://github.com/Leavrth)
        - PITRエラーが上書きされる問題を修正する[#40576](https://github.com/pingcap/tidb/issues/40576) @[Leavrth](https://github.com/Leavrth)
        - advance ownerとgc ownerが異なる場合にPITRバックアップタスクのチェックポイントが進まない問題を修正する[#41806](https://github.com/pingcap/tidb/issues/41806) @[joccau](https://github.com/joccau)

    + TiCDC

        - changefeedがTiKVまたはTiCDCノードのスケーリングインまたはスケーリングアウトなどの特殊なシナリオで動けなくなる問題を修正する[#8174](https://github.com/pingcap/tiflow/issues/8174) @[hicqu](https://github.com/hicqu)
        - redoログのストレージパスに対する事前チェックが実行されない問題を修正する[#6335](https://github.com/pingcap/tiflow/issues/6335) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - S3ストレージの障害に対するredoログの耐えられる時間の不足問題を修正する[#8089](https://github.com/pingcap/tiflow/issues/8089) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - イシューを修正する：`transaction_atomicity`および`protocol`が構成ファイルを介して更新できない問題を修正[#7935](https://github.com/pingcap/tiflow/issues/7935) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - イシューを修正する：TiCDCが過度に多くのテーブルを複製すると、チェックポイントが進まない問題を修正[#8004](https://github.com/pingcap/tiflow/issues/8004) @[overvenus](https://github.com/overvenus)
        - イシューを修正する：レプリケーション遅延が過度に高い場合に、リドゥログの適用がOOMを引き起こす可能性がある問題を修正[#8085](https://github.com/pingcap/tiflow/issues/8085) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - イシューを修正する：リドゥログがメタを書き込むために有効になっている場合にパフォーマンスが低下する問題を修正[#8074](https://github.com/pingcap/tiflow/issues/8074) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - TiCDCが大規模なトランザクションを分割せずにデータを複製すると、コンテキストの締め切りが超過するバグを修正[#7982](https://github.com/pingcap/tiflow/issues/7982) @[hi-rustin](https://github.com/hi-rustin)
        - PDが異常な場合にchangefeedを一時停止すると、不正確な状態が発生する問題を修正[#8330](https://github.com/pingcap/tiflow/issues/8330) @[sdojjy](https://github.com/sdojjy)
        - TiDBまたはMySQLシンクにデータを複製し、非NULLのユニークインデックスを持つ列に`CHARACTER SET`が指定されている場合に発生するデータの不一致を修正する問題を修正[#8420](https://github.com/pingcap/tiflow/issues/8420) @[asddongmen](https://github.com/asddongmen)
        - テーブルスケジューリングとブラックホールシンクでのパニック問題を修正する[#8024](https://github.com/pingcap/tiflow/issues/8024) [#8142](https://github.com/pingcap/tiflow/issues/8142) @[hicqu](https://github.com/hicqu)

    + TiDB Data Migration (DM)

        - `binlog-schema delete`コマンドの実行に失敗する問題を修正[#7373](https://github.com/pingcap/tiflow/issues/7373) @[liumengya94](https://github.com/liumengya94)
        - 最後のバイナリログがスキップされたDDLである場合に、チェックポイントが進まない問題を修正[#8175](https://github.com/pingcap/tiflow/issues/8175) @[D3Hunter](https://github.com/D3Hunter)
        - 1つのテーブルで"update"および"non-update"タイプの式フィルタが両方指定された場合、すべての`UPDATE`ステートメントがスキップされるバグを修正[#7831](https://github.com/pingcap/tiflow/issues/7831) @[lance6716](https://github.com/lance6716)

    + TiDB Lightning

        - TiDB Lightningの事前チェックで、以前に失敗したインポートで残された不正データを見つけられない問題を修正[#39477](https://github.com/pingcap/tidb/issues/39477) @[dsdashun](https://github.com/dsdashun)
        - 分割リージョンフェーズでTiDB Lightningがパニックする問題を修正[#40934](https://github.com/pingcap/tidb/issues/40934) @[lance6716](https://github.com/lance6716)
        - 衝突解決ロジック（`duplicate-resolution`）が不一致のチェックサムにつながる可能性がある問題を修正[#40657](https://github.com/pingcap/tidb/issues/40657) @[gozssky](https://github.com/gozssky)
        - ローカルデータベースモードでデータをインポートする場合、インポート対象テーブルの復合主キーが`auto_random`列を持ち、ソースデータで列の値が指定されていない場合に自動的にデータを生成しない問題を修正[#41454](https://github.com/pingcap/tidb/issues/41454) @[D3Hunter](https://github.com/D3Hunter)