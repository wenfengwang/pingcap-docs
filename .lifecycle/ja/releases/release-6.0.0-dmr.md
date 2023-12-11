---
title: TiDB 6.0.0リリースノート
---

# TiDB 6.0.0リリースノート

リリース日: 2022年4月7日

TiDBバージョン: 6.0.0-DMR

> **注意:**
>
> TiDB 6.0.0-DMRのドキュメントは[アーカイブ](https://docs-archive.pingcap.com/tidb/v6.0/)されました。PingCAPではTiDBデータベースの[最新LTSバージョン](https://docs.pingcap.com/tidb/stable)を使用することをお勧めします。

6.0.0-DMRでは、主な新機能または改善点は以下の通りです:

- SQLで配置ルールをサポートし、データ配置の柔軟な管理を提供します。
- カーネルレベルでデータとインデックスの整合性チェックを追加し、非常に低いリソースオーバーヘッドでシステムの安定性と堅牢性を向上させます。
- 非専門家向けの自己サービス型のデータベースパフォーマンス監視および診断機能であるTop SQLを提供します。
- クラスターパフォーマンスデータを常に収集するContinuous Profilingをサポートし、テクニカルエキスパートのMTTRを削減します。
- メモリ内でホットスポットとなる小さいテーブルをキャッシュし、アクセスパフォーマンスを大幅に向上させ、スループットを向上させ、アクセスの待ち時間を短縮します。
- パフォーマンスボトルネックによる悲観的ロックの最適化を実施し、悲観的ロックのメモリ最適化により待ち時間を10%削減し、QPSを10%増加させます。
- 実行計画を共有するためにプリペアドステートメントを強化し、CPUリソース消費を減らし、SQL実行効率を向上させます。

**【以下略】**
SQLの実行計画を再利用すると、SQLステートメントの解析時間が効果的に短縮され、CPUリソース消費が減少し、SQLの実行効率が向上します。SQLのチューニングにおける重要な方法の1つは、SQLの実行計画を効果的に再利用することです。TiDBはプリペアドステートメントで実行計画を共有する機能をサポートしています。ただし、プリペアドステートメントがクローズされると、TiDBは自動的に対応するプランキャッシュをクリアします。その後、TiDBは繰り返しのSQLステートメントを不必要に解析する可能性があり、実行効率に影響を与えることがあります。v6.0.0以降、TiDBは`tidb_ignore_prepared_cache_close_stmt`パラメータを介して`COM_STMT_CLOSE`コマンドを無視するかどうかを制御する機能をサポートしています（デフォルトでは無効）。このパラメータを有効にすると、TiDBはプリペアドステートメントのクローズコマンドを無視し、実行計画をキャッシュに保持し、実行計画の再利用率を向上させます。

[ユーザードキュメント](/sql-prepared-plan-cache.md#ignore-the-com_stmt_close-command-and-the-deallocate-prepare-statement)、[#31056](https://github.com/pingcap/tidb/issues/31056)

- クエリプッシュダウンの改善

TiDBのコンピューティングとストレージのネイティブアーキテクチャにより、オペレータをプッシュダウンして無効なデータをフィルタリングすることができます。これにより、TiDBとTiKV間のデータ転送が大幅に削減され、クエリの効率が向上します。v6.0.0では、TiDBはさらに多くの式と`BIT`データ型をTiKVにプッシュダウンする機能をサポートしており、これにより、式とデータ型の計算時のクエリ効率を向上させています。

[ユーザードキュメント](/functions-and-operators/expressions-pushed-down.md)、[#30738](https://github.com/pingcap/tidb/issues/30738)

- ホットスポットインデックスの最適化

連続増加データをセカンダリインデックスにバッチで書き込むと、インデックスホットスポットが発生し、全体の書き込みスループットに影響を与えます。v6.0.0以降、TiDBは`tidb_shard`関数を使用してインデックスホットスポットを散開する機能をサポートしており、これにより書き込み性能が向上します。現在、`tidb_shard`はユニークセカンダリインデックスにのみ効果があります。このアプリケーション向けのソリューションは、元のクエリ条件を変更する必要はありません。このソリューションは高い書き込みスループット、ポイントクエリ、バッチポイントクエリのシナリオで使用することができます。ただし、アプリケーションで範囲クエリで散開されたデータを使用すると、パフォーマンスが低下する可能性がありますので、検証なしにこの機能を使用しないでください。

[ユーザードキュメント](/functions-and-operators/tidb-functions.md#tidb_shard)、[#31040](https://github.com/pingcap/tidb/issues/31040)

- TiFlash MPPエンジンのパーティションテーブルのダイナミックプルーニングモードのサポート（実験的）

このモードでは、TiDBはTiFlashのMPPエンジンを使用してパーティションテーブルのデータを読み取りおよび計算できるため、パーティションテーブルのクエリパフォーマンスが大幅に向上します。

[ユーザードキュメント](/tiflash/use-tiflash-mpp-mode.md#access-partitioned-tables-in-the-mpp-mode)

- MPPエンジンの計算パフォーマンスの向上

    - MPPエンジンにより、より多くの関数と演算子をプッシュダウンする機能がサポートされました

        - 論理関数: `IS`、`IS NOT`
        - 文字列関数: `REGEXP()`、`NOT REGEXP()`
        - 数学関数: `GREATEST(int/real)`、`LEAST(int/real)`
        - 日付関数: `DAYNAME()`、`DAYOFMONTH()`、`DAYOFWEEK()`、`DAYOFYEAR()`、`LAST_DAY()`、`MONTHNAME()`
        - 演算子: Anti Left Outer Semi Join、Left Outer Semi Join

        [ユーザードキュメント](/tiflash/tiflash-supported-pushdown-calculations.md)

    - エラスティックスレッドプール（デフォルトで有効）がGAになりました。この機能はCPU利用率を向上させることを目的としています。

        [ユーザードキュメント](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

### 安定性

- 実行計画のベースラインキャプチャを強化

    テーブル名、頻度、およびユーザー名などの次元を持つブロックリストを追加することで、実行計画のベースラインキャプチャの利便性を向上させます。メモリ管理を最適化する新しいアルゴリズムを導入し、ベースラインキャプチャが有効になると、システムは自動的にほとんどのOLTPクエリのバインディングを作成します。バインドステートメントの実行計画は固定されるため、実行計画の変更によるパフォーマンスの問題を回避できます。ベースラインキャプチャは、メジャーバージョンのアップグレードやクラスターの移行などのシナリオに適用できます。また、実行計画の回帰によるパフォーマンスの問題を軽減するのに役立ちます。

    [ユーザードキュメント](/sql-plan-management.md#baseline-capturing)、[#32466](https://github.com/pingcap/tidb/issues/32466)

- TiKVクォータリミッターのサポート（実験的）

    TiKVが展開されたマシンがリソースに制限があり、前景タスクが過剰なリクエストによってリソースを占有し、バックグラウンドのCPUリソースが消費されることにより、TiKVのパフォーマンスが不安定になる可能性があります。TiDB v6.0.0では、前景で使用されるリソース（CPU、読み書き帯域幅など）を制限するためのクォータ関連の構成項目を使用できます。これにより、長期間の重いワークロード下でクラスタの安定性が向上します。

    [ユーザードキュメント](/tikv-configuration-file.md#quota)、[#12131](https://github.com/tikv/tikv/issues/12131)

- TiFlashでのzstd圧縮アルゴリズムのサポート

    TiFlashは、`profiles.default.dt_compression_method`および`profiles.default.dt_compression_level`という2つのパラメータを導入し、パフォーマンスと容量のバランスに基づいて最適な圧縮アルゴリズムを選択できるようにしています。

    [ユーザードキュメント](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)

- デフォルトですべてのI/Oチェック（チェックサム）を有効化

    この機能はv5.4.0で実験的に導入されました。これにより、データの正確さとセキュリティが向上しますが、ユーザーのビジネスに顕著な影響が出ることはありません。

    注意: データ形式の新しいバージョンをv5.4.0以前のバージョンに対して場所を変更してダウングレードすることはできません。そのようなダウングレードが必要な場合、TiFlashのレプリカを削除し、ダウングレード後にデータをレプリケートする必要があります。または、[dttool migrate](/tiflash/tiflash-command-line-flags.md#dttool-migrate)を参照してダウングレードを実行できます。

    [ユーザードキュメント](/tiflash/tiflash-data-validation.md)

- スレッドの利用率の改善

    TiFlashは非同期gRPCおよびMin-TSOスケジューリングメカニズムを導入しました。これにより、スレッドの効率的な使用が確保され、過剰なスレッドによるシステムのクラッシュを回避できます。

    [ユーザードキュメント](/tiflash/monitor-tiflash.md#coprocessor)

### データ移行

#### TiDBデータ移行（DM）

- WebUIの追加（実験的）

    WebUIを使用すると、多数の移行タスクを簡単に管理できます。WebUIでは、以下のことができます:

    - ダッシュボードで移行タスクを表示する
    - 移行タスクを管理する
    - 上流設定を構成する
    - レプリケーション状況をクエリする
    - マスターおよびワーカーの情報を表示する

    WebUIはまだ実験的であり、開発中です。そのため、試験的な使用のみを推奨します。WebUIとdmctlを使用して同じタスクを操作すると問題が発生する可能性があります。この問題は、後のバージョンで解決される予定です。

    [ユーザードキュメント](/dm/dm-webui-guide.md)

- エラーハンドリングメカニズムの追加

    移行タスクを中断する問題に対処するための新しいコマンドが導入されました。例:

    - スキーマエラーの場合、`binlog-schema update`コマンドの`--from-source/--from-target`パラメータを使用してスキーマファイルを更新し、個別にスキーマファイルを編集する代わりにします。
    - DDLステートメントを注入、置換、スキップ、revertするためのバイナリログポジションを指定することができます。

    [ユーザードキュメント](/dm/dm-manage-schema.md)

- Amazon S3への完全データストレージのサポート

    DMでは、上流からの全データまたは完全データ移行タスクを実行する際、上流からの完全データを保存するために十分なハードディスク容量が必要です。EBSに比べて、Amazon S3はより低コストでほぼ無限のストレージを提供しています。現在、DMはダンプディレクトリとしてAmazon S3を構成することをサポートしています。つまり、全データまたは完全データ移行タスクを実行する際にS3を使用してデータを保存できます。

    [ユーザードキュメント](/dm/task-configuration-file-full.md#task-configuration-file-template-advanced)

- 指定した時間からの移行タスクの開始のサポート

    移行タスクに`--start-time`という新しいパラメータが追加されました。`--start-time`パラメータでは、'2021-10-21 00:01:00'または'2021-10-21T00:01:00'の形式で時間を定義できます。

    この機能は、シャードMySQLインスタンスからの増分データの移行とマージするシナリオで特に有用です。具体的には、増分移行タスクの`safe-mode`で`--start-time`パラメータを使用することで、各ソースに対して個別のバイナリログの開始点を設定する必要はありません。

    [ユーザードキュメント](/dm/dm-create-task.md#flags-description)

#### TiDB Lightning

- 許容できるエラーの最大数を設定する機能のサポート

    `lightning.max-error`という構成項目が追加されました。デフォルト値は0です。値が0より大きい場合、max-error機能が有効になります。エンコード中にエラーが発生した場合、この行を含むレコードが`lightning_task_info.type_error_v1`に追加され、この行が無視されます。エラーのある行がしきい値を超えると、TiDB Lightningは直ちに終了します。

    `lightning.max-error`構成に一致するように、`lightning.task-info-schema-name`という構成項目はデータ保存エラーを報告しているデータベースの名前を記録します。
この機能はすべての種類のエラーをカバーしているわけではなく、たとえば構文エラーは適用されません。

[ユーザードキュメント](/tidb-lightning/tidb-lightning-error-resolution.md#type-error)

### TiDBデータ共有サブスクリプション

- 100,000個のテーブルを同時にレプリケートできます

    データ処理フローを最適化することで、TiCDCは各テーブルの増分データ処理のリソース消費を減らし、大規模なクラスターでのデータレプリケーションの安定性と効率を大幅に改善します。 内部テストの結果から、TiCDCは安定して100,000個のテーブルを同時にレプリケートできることが示されています。

### デプロイおよびメンテナンス

- 新コーレーションルールをデフォルトで有効にします

    v4.0以降、TiDBはMySQLと同じように大文字と小文字を区別せず、アクセントを無視し、パディングのルールを適用する新しいコーレーションルールをサポートしています。 新しいコーレーションルールは、`new_collations_enabled_on_first_bootstrap`パラメータで制御されており、デフォルトでは無効になっていました。 v6.0以降、TiDBは新しいコーレーションルールをデフォルトで有効にします。 この設定は、TiDBクラスターの初期化時のみ有効です。

    [ユーザードキュメント](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap)

- TiKVノードの再起動後にリーダーバランシングを加速します

    TiKVノードの再起動後、不均衡に分散しているリーダーは負荷分散のために再分散する必要があります。 大規模なクラスターでは、リーダーバランシング時間が領域数と正の相関関係にあります。 たとえば、100,000個のリージョンのリーダーバランシングには20〜30分かかることがあり、負荷の不均衡によりパフォーマンスの問題や安定性のリスクが発生しやすくなります。 TiDB v6.0.0では、バランシング並行性を制御するパラメータを提供し、デフォルト値を4倍に拡大し、リーダーバランシング時間を大幅に短縮し、TiKVノードの再起動後のビジネスリカバリを加速します。

    [ユーザードキュメント](/pd-control.md#scheduler-config-balance-leader-scheduler), [#4610](https://github.com/tikv/pd/issues/4610)

- 統計情報の自動更新のキャンセルをサポートします

    統計情報はSQLのパフォーマンスに影響を与える最も重要な基本データの1つです。 統計情報の完全性とタイムリネスを確保するために、TiDBは定期的にオブジェクト統計情報を自動的に更新していました。 ただし、自動統計情報の更新はリソースの競合を引き起こし、SQLのパフォーマンスに影響を与える可能性があります。 この問題に対処するために、v6.0以降、自動統計情報の更新を手動でキャンセルすることができます。

    [ユーザードキュメント](/statistics.md#automatic-update)

- PingCAP Clinic診断サービス（技術プレビューバージョン）

    PingCAP ClinicはTiDBクラスターの診断サービスです。 このサービスはクラスターの問題をリモートでトラブルシューティングし、クラスターの状態をローカルで素早くチェックするのに役立ちます。 PingCAP Clinicを使用すると、TiDBクラスターの安定した動作をフルライフサイクル中に確保し、潜在的な問題を予測し、問題発生の確率を減らし、クラスターの問題を迅速にトラブルシューティングすることができます。

    PingCAPのテクニカルサポートにクラスターの問題をリモートでトラブルシューティングする場合、PingCAP Clinicサービスを使用して診断データを収集およびアップロードし、トラブルシューティングの効率を向上させることができます。

    [ユーザードキュメント](/clinic/clinic-introduction.md)

- 企業レベルのデータベース管理プラットフォーム、TiDBエンタープライズマネージャー

    TiDB Enterprise Manager（TiEM）は、TiDBデータベースを基盤とした企業レベルのデータベース管理プラットフォームで、自己ホスト型またはパブリッククラウド環境でのTiDBクラスターの管理を支援します。

    TiEMはTiDBクラスターのフルライフサイクルの視覚管理を提供するだけでなく、パラメータの管理、バージョンのアップグレード、クラスターの複製、アクティブスタンバイクラスターの切り替え、データのインポートとエクスポート、データのレプリケーション、およびデータのバックアップとリストアサービスをワンストップで提供します。 TiEMはTiDB上のDevOpsの効率を向上し、企業のDevOpsコストを削減できます。

    現在、TiEMは[TiDB Enterprise](https://en.pingcap.com/tidb-enterprise/)エディションでのみ提供されています。 TiEMを取得するには、[TiDB Enterprise](https://en.pingcap.com/tidb-enterprise/)ページからお問い合わせください。

- モニタリングコンポーネントの設定のカスタマイズをサポートします

    TiUPを使用してTiDBクラスターをデプロイする際、TiUPは自動的にPrometheus、Grafana、Alertmanagerなどのモニタリングコンポーネントをデプロイし、スケールアウト後に新しいノードを監視範囲に自動的に追加します。 `topology.yaml`ファイルに構成項目を追加することで、モニタリングコンポーネントの構成をカスタマイズできます。

    [ユーザードキュメント](/tiup/customized-montior-in-tiup-environment.md)

## 互換性の変更

> **注意：**
>
> v6.0.0に既存のTiDBバージョンからアップグレードする場合、中間バージョンの互換性変更ノートをすべて知りたい場合は、対応するバージョンの[リリースノート](/releases/release-notes.md)を確認できます。

### システム変数

<table>
<thead>
  <tr>
    <th>変数名</th>
    <th>変更タイプ</th>
    <th>説明</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td><code>placement_checks</code></td>
    <td>削除</td>
    <td>DDLステートメントが<a href="https://docs.pingcap.com/tidb/dev/placement-rules-in-sql">SQLの配置ルール</a>で指定された配置ルールを検証するかどうかを制御します。 <code>tidb_placement_mode</code>に置き換えられました。</td>
  </tr>
  <tr>
    <td><code>tidb_enable_alter_placement</code></td>
    <td>削除</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/placement-rules-in-sql">SQLの配置ルール</a>を有効にするかどうかを制御します。</td>
  </tr>
  <tr>
    <td>
      <code>tidb_mem_quota_hashjoin</code><br/>
      <code>tidb_mem_quota_indexlookupjoin</code><br/>
      <code>tidb_mem_quota_indexlookupreader</code><br/>
      <code>tidb_mem_quota_mergejoin</code><br/>
      <code>tidb_mem_quota_sort</code><br/>
      <code>tidb_mem_quota_topn</code>
    </td>
    <td>削除</td>
    <td>v5.0以降、これらの変数は<code>tidb_mem_quota_query</code>に置き換わり、<a href="https://docs.pingcap.com/tidb/dev/system-variables">システム変数</a>ドキュメントから削除されています。 互換性を保証するために、これらの変数はソースコードに保持されていました。 TiDB 6.0.0以降、これらの変数はコードから削除されています。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_enable_mutation_checker-new-in-v600"><code>tidb_enable_mutation_checker</code></a></td>
    <td>新規追加</td>
    <td>変異チェッカーを有効にするかどうかを制御します。デフォルト値は<code>ON</code>です。 v6.0.0より前のバージョンからアップグレードする既存のクラスターでは、変異チェッカーはデフォルトで無効になっています。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_ignore_prepared_cache_close_stmt-new-in-v600"><code>tidb_ignore_prepared_cache_close_stmt</code></a></td>
    <td>新規追加</td>
    <td>プリペアドステートメントを閉じるコマンドを無視するかどうかを制御します。デフォルト値は<code>OFF</code>です。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_mem_quota_binding_cache-new-in-v600"><code>tidb_mem_quota_binding_cache</code></a></td>
    <td>新規追加</td>
    <td>バインディングを保持するキャッシュのメモリ使用量のしきい値を設定します。デフォルト値は<code>67108864</code>（64 MiB）です。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_placement_mode-new-in-v600"><code>tidb_placement_mode</code></a></td>
    <td>新規追加</td>
    <td>DDLステートメントが<a href="https://docs.pingcap.com/tidb/dev/placement-rules-in-sql">SQLの配置ルール</a>で指定された配置ルールを無視するかどうかを制御します。 デフォルト値は<code>strict</code>で、DDLステートメントは配置ルールを無視しません。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_rc_read_check_ts-new-in-v600"><code>tidb_rc_read_check_ts</code></a></td>
    <td>新規追加</td>
    <td>
      <ul>
        <li>トランザクション内での読み取りステートメントのレイテンシーを最適化します。読み書きの競合がより深刻な場合、この変数をオンにすると追加のオーバーヘッドとレイテンシーが発生し、パフォーマンスに影響を与える可能性があります。<code>off</code>がデフォルト値です。</li>
<ul>
  <li>この変数は、<a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_replica_read-new-in-v40">replica-read</a>とまだ互換性がありません。読み取りリクエストに<code>tidb_rc_read_check_ts</code>がオンになっている場合、replica-readを使用できない場合があります。両方の変数を同時にオンにしないでください。</li>
</ul>

<table>
<thead>
  <tr>
    <th>設定ファイル</th>
    <th>設定</th>
    <th>変更タイプ</th>
    <th>説明</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>TiDB</td>
    <td>
      <code>stmt-summary.enable</code><br/>
      <code>stmt-summary.enable-internal-query</code><br/>
      <code>stmt-summary.history-size</code><br/>
      <code>stmt-summary.max-sql-length</code><br/>
      <code>stmt-summary.max-stmt-count</code><br/>
      <code>stmt-summary.refresh-interval</code>
    </td>
    <td>削除</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/statement-summary-tables">ステートメントサマリーテーブル</a>に関連する構成。これらの構成項目はすべて削除されます。ステートメントサマリーテーブルを制御するには、SQL変数を使用する必要があります。</td>
  </tr>
  <tr>
    <td>TiDB</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tidb-configuration-file#new_collations_enabled_on_first_bootstrap"><code>new_collations_enabled_on_first_bootstrap</code></a></td>
    <td>変更済み</td>
    <td>新しい照合順序のサポートを有効にするかどうかを制御します。v6.0以降、デフォルト値が<code>false</code>から<code>true</code>に変更されました。この構成項目は、クラスターが最初に初期化されたときにのみ効果があります。最初のブートストラップ後、この構成項目を使用して新しい照合順序フレームワークを有効または無効にはできません。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#num-threads-1"><code>backup.num-threads</code></a></td>
    <td>変更済み</td>
    <td>値の範囲が<code>[1, CPU]</code>に変更されます。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#apply-max-batch-size"><code>raftstore.apply-max-batch-size</code></a></td>
    <td>変更済み</td>
    <td>最大値が<code>10240</code>に変更されました。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#raft-max-size-per-msg"><code>raftstore.raft-max-size-per-msg</code></a></td>
    <td>変更済み</td>
    <td>最小値が<code>0</code>から0より大きい値に変更されます。<br/>最大値は<code>3GB</code>に設定されます。<br/>単位が<code>MB</code>から<code>KB|MB|GB</code>に変更されます。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#store-max-batch-size"><code>raftstore.store-max-batch-size</code></a></td>
    <td>変更済み</td>
    <td>最大値が<code>10240</code>に設定されます。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#max-thread-count"><code>readpool.unified.max-thread-count</code></a></td>
    <td>変更済み</td>
    <td>調整可能な範囲が<code>[min-thread-count, MAX(4, CPU)]</code>に変更されます。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#enable-pipelined-write"><code>rocksdb.enable-pipelined-write</code></a></td>
    <td>変更済み</td>
    <td>デフォルト値が<code>true</code>から<code>false</code>に変更されました。この構成が有効な場合、以前のパイプラインライティングが使用されます。この構成が無効になっている場合、新しいパイプラインコミットメカニズムが使用されます。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#max-background-flushes"><code>rocksdb.max-background-flushes</code></a></td>
    <td>変更済み</td>
    <td>CPUコア数が10の場合、デフォルト値は<code>3</code>です。<br/>CPUコア数が8の場合、デフォルト値は<code>2</code>です。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#max-background-jobs"><code>rocksdb.max-background-jobs</code></a></td>
    <td>変更済み</td>
    <td>CPUコア数が10の場合、デフォルト値は<code>9</code>です。<br/>CPUコア数が8の場合、デフォルト値は<code>7</code>です。</td>
  </tr>
  <tr>
    <td>TiFlash</td>
  </tr>
</tbody>
</table>
```
    <td><a href="https://docs.pingcap.com/tidb/dev/tiflash-configuration#configure-the-tiflashtoml-file"><code>profiles.default.dt_enable_logical_split</code></a></td>
    <td>変更</td>
    <td>DeltaTree Storage Engineのセグメントが論理スプリットを使用するかどうかを決定します。デフォルト値が<code>true</code>から<code>false</code>に変更されました。</td>
  </tr>
  <tr>
    <td>TiFlash</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tiflash-configuration#configure-the-tiflashtoml-file"><code>profiles.default.enable_elastic_threadpool</code></a></td>
    <td>変更</td>
    <td>弾力的なスレッドプールを有効にするかどうかを制御します。デフォルト値が<code>false</code>から<code>true</code>に変更されました。</td>
  </tr>
  <tr>
    <td>TiFlash</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tiflash-configuration#configure-the-tiflashtoml-file"><code>storage.format_version</code></a></td>
    <td>変更</td>
    <td>TiFlashのデータ検証機能を制御します。デフォルト値が<code>2</code>から<code>3</code>に変更されました。<br/><code>format_version</code>が<code>3</code>に設定されている場合、すべてのTiFlashデータの読み取り操作で一貫性チェックが行われ、ハードウェアの故障による不正確な読み取りを回避します。<br/>新しいフォーマットバージョンはv5.4より前のバージョンにインプレースでダウングレードすることはできません。</td>
  </tr>
  <tr>
    <td>TiDB</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tidb-configuration-file#pessimistic-auto-commit-new-in-v600"><code>pessimistic-txn.pessimistic-auto-commit</code></a></td>
    <td>新規追加</td>
    <td>悲観的トランザクションモードがグローバルに有効になっている場合（<code>tidb_txn_mode='pessimistic'</code>）、自動コミットトランザクションが使用するトランザクションモードを決定します。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#in-memory-new-in-v600"><code>pessimistic-txn.in-memory</code></a></td>
    <td>新規追加</td>
    <td>インメモリ悲観的ロックを有効にするかどうかを制御します。この機能を有効にすると、悲観的なトランザクションは可能な限りTiKVメモリに悲観的なロックを保存し、ディスクに書き込んだり他のレプリカにレプリケートしたりする必要がなくなります。これにより悲観的なトランザクションのパフォーマンスが向上しますが、悲観的なロックが失われる可能性が低いため、悲観的なトランザクションのコミットに失敗する可能性があります。デフォルト値は<code>true</code>です。</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tikv-configuration-file#quota"><code>quota</code></a></td>
    <td>新規追加</td>
    <td>フロントエンドリクエストが占有するリソースを制限するQuota Limiterに関連する設定項目を追加します。Quota Limiterは実験的な機能であり、デフォルトでは無効になっています。新しいクォータ関連の設定項目は<code>foreground-cpu-time</code>、<code>foreground-write-bandwidth</code>、<code>foreground-read-bandwidth</code>、<code>max-delay-duration</code>です。</td>
  </tr>
  <tr>
    <td>TiFlash</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tiflash-configuration#configure-the-tiflashtoml-file"><code>profiles.default.dt_compression_method</code></a></td>
    <td>新規追加</td>
    <td>TiFlashの圧縮アルゴリズムを指定します。オプションの値は<code>LZ4</code>、<code>zstd</code>、<code>LZ4HC</code>です（大文字小文字を区別しません）。デフォルト値は<code>LZ4</code>です。</td>
  </tr>
  <tr>
    <td>TiFlash</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/tiflash-configuration#configure-the-tiflashtoml-file"><code>profiles.default.dt_compression_level</code></a></td>
    <td>新規追加</td>
    <td>TiFlashの圧縮レベルを指定します。デフォルト値は<code>1</code>です。</td>
  </tr>
  <tr>
    <td>DM</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/task-configuration-file-full#task-configuration-file-template-advanced"><code>loaders.&lt;name&gt;.import-mode</code></a></td>
    <td>新規追加</td>
    <td>フルインポートフェーズ中のインポートモードを指定します。v6.0以降、DMはフルインポートフェーズ中にTiDB LightningのTiDBバックエンドモードを使用します。以前のLoaderコンポーネントは使用されなくなりました。これは内部的な置換であり、日常の操作に明らかな影響を与えるものではありません。<br/>デフォルト値は<code>sql</code>で、tidbバックエンドモードを意味します。まれなケースではtidbバックエンドが完全に互換性がないことがあります。このパラメータを<code>loader</code>に設定することでLoaderモードに戻ることができます。</td>
  </tr>
  <tr>
    <td>DM</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/task-configuration-file-full#task-configuration-file-template-advanced"><code>loaders.&lt;name&gt;.on-duplicate</code></a></td>
    <td>新規追加</td>
    <td>フルインポートフェーズ中の競合解決メソッドを指定します。デフォルト値は<code>replace</code>で、既存データを新しいデータで置き換えます。</td>
  </tr>
  <tr>
    <td>TiCDC</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/ticdc-sink-to-kafka#configure-sink-uri-for-kafka"><code>dial-timeout</code></a></td>
    <td>新規追加</td>
    <td>下流のKafkaとの接続確立のタイムアウト時間を指定します。デフォルト値は<code>10s</code>です。</td>
  </tr>
  <tr>
    <td>TiCDC</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/ticdc-sink-to-kafka#configure-sink-uri-for-kafka"><code>read-timeout</code></a></td>
    <td>新規追加</td>
    <td>下流のKafkaから返されるレスポンスを取得するタイムアウト時間を指定します。デフォルト値は<code>10s</code>です。</td>
  </tr>
  <tr>
    <td>TiCDC</td>
    <td><a href="https://docs.pingcap.com/tidb/dev/ticdc-sink-to-kafka#configure-sink-uri-for-kafka"><code>write-timeout</code></a></td>
    <td>新規追加</td>
    <td>下流のKafkaにリクエストを送信するタイムアウト時間を指定します。デフォルト値は<code>10s</code>です。</td>
  </tr>
</tbody>
</table>

### その他

- データ配置ポリシーには次の互換性の変更があります：
    - Bindingはサポートされていません。直接配置オプションは構文から削除されました。
    - `CREATE PLACEMENT POLICY`および`ALTER PLACEMENT POLICY`ステートメントは、もはや`VOTERS`および`VOTER_CONSTRAINTS`配置オプションをサポートしません。
    - TiDBマイグレーションツール（TiDB Binlog、TiCDC、BR）は今や配置ルールと互換性があります。配置オプションはTiDB Binlogの特別なコメントに移動されました。
    - `information_schema.placement_rules`システムテーブルは`information_schema.placement_policies`に名前が変更されました。このテーブルでは今後は配置ポリシーに関する情報のみが表示されます。
    - `placement_checks`システム変数は`tidb_placement_mode`に置き換えられました。
    - TiFlashレプリカを持つテーブルに配置ルールでパーティションを追加することは禁止されています。
    - `INFORMATION_SCHEMA`テーブルから`TIDB_DIRECT_PLACEMENT`列が削除されました。
- SQLプラン管理（SPM）バインディングの`status`値が変更されました：
    - `using`が削除されました。
    - `enabled`（利用可能）が`using`を置き換えるために追加されました。
    - `disabled`（利用不可）が追加されました。
- DMはOpenAPIインターフェースを変更しました
    - 内部のメカニズムの変更により、タスク管理に関連するインターフェースが以前の実験バージョンと互換性がなくなりました。適応するためには新しい[DM OpenAPI documentation](/dm/dm-open-api.md)を参照する必要があります。
- DMはフルインポートフェーズ中の競合解決メソッドを変更しました
    - パラメータ`loader.<name>.on-duplicate`が追加されました。デフォルト値は`replace`で、これは新しいデータを使用して既存のデータを置き換えることを意味します。以前の動作を維持したい場合は、値を`error`に設定できます。このパラメータはフルインポートフェーズ中の動作のみを制御します。
- DMを使用するには、対応するバージョンの`dmctl`を使用する必要があります
    - 内部メカニズムの変更により、DMをv6.0.0にアップグレードした後は、`dmctl`もv6.0.0にアップグレードする必要があります。
- v5.4（v5.4のみ）では、TiDBはいくつかのnoopシステム変数に対して不正な値を許可していました。v6.0.0からは、TiDBはシステム変数に不正な値を設定することを許可しません。[#31538](https://github.com/pingcap/tidb/issues/31538)

## 改善点

+ TiDB

    - `FLASHBACK`または`RECOVER`ステートメントを使用してテーブルを復元した後、テーブルの配置ルール設定を自動的にクリアします  [#31668](https://github.com/pingcap/tidb/issues/31668)
    - 典型的なクリティカル パスのコア パフォーマンス メトリクスを示すパフォーマンス概要ダッシュボードを追加し、TiDBでメトリクス解析がしやすくなりました [#31676](https://github.com/pingcap/tidb/issues/31676)
    - `LOAD DATA LOCAL INFILE`ステートメントで`REPLACE`キーワードを使用できるようにサポートしました [#24515](https://github.com/pingcap/tidb/issues/24515)
    - Rangeパーティションテーブルの組み込み`IN`式のパーティションプルーニングをサポートしました [#26739](https://github.com/pingcap/tidb/issues/26739)
    - MPP集約クエリで潜在的に冗長なExchange操作を除去することにより、クエリ効率を改善しました [#31762](https://github.com/pingcap/tidb/issues/31762)
    - MySQLとの互換性を向上させるため、`TRUNCATE PARTITION`および`DROP PARTITION`ステートメントで重複するパーティション名を許可しました [#31681](https://github.com/pingcap/tidb/issues/31681)
    - `ADMIN SHOW DDL JOBS`ステートメントの結果に`CREATE_TIME`情報を表示するサポートを追加しました [#23494](https://github.com/pingcap/tidb/issues/23494)
    - 新しい組み込み関数`CHARSET()`をサポートしました [#3931](https://github.com/pingcap/tidb/issues/3931)
    - ユーザ名によってベースラインキャプチャのブロックリストをフィルタリングするサポートを追加しました [#32558](https://github.com/pingcap/tidb/issues/32558)
    - ワイルドカードをベースラインキャプチャのブロックリストで使用するサポートを追加しました [#32714](https://github.com/pingcap/tidb/issues/32714)
    - `ADMIN SHOW DDL JOBS`および`SHOW TABLE STATUS`ステートメントの結果を、現在の`time_zone`に従って時間を表示することで最適化しました [#26642](https://github.com/pingcap/tidb/issues/26642)
    - `DAYNAME()`および`MONTHNAME()`関数をTiFlashにプッシュダウンするサポートを追加しました [#32594](https://github.com/pingcap/tidb/issues/32594)
    - `REGEXP`関数をTiFlashにプッシュダウンするサポートを追加しました [#32637](https://github.com/pingcap/tidb/issues/32637)
    - `DAYOFMONTH()`および`LAST_DAY()`関数をTiFlashにプッシュダウンするサポートを追加しました [#33012](https://github.com/pingcap/tidb/issues/33012)
    - `DAYOFWEEK()`および`DAYOFYEAR()`関数をTiFlashにプッシュダウンするサポートを追加しました [#33130](https://github.com/pingcap/tidb/issues/33130)
    - `IS_TRUE`、`IS_FALSE`、`IS_TRUE_WITH_NULL`関数をTiFlashにプッシュダウンするサポートを追加しました [#33047](https://github.com/pingcap/tidb/issues/33047)
    - `GREATEST`および`LEAST`関数をTiFlashにプッシュダウンするサポートを追加しました [#32787](https://github.com/pingcap/tidb/issues/32787)
    - `UnionScan`オペレーターの実行をトラッキングするサポートを追加しました [#32631](https://github.com/pingcap/tidb/issues/32631)
    - `_tidb_rowid`列を読むクエリに対してPointGet計画を使用するサポートを追加しました [#31543](https://github.com/pingcap/tidb/issues/31543)
    - `EXPLAIN`ステートメントの出力で元のパーティション名を表示し、名前を小文字に変換せずに表示するサポートを追加しました [#32719](https://github.com/pingcap/tidb/issues/32719)
    - RANGE COLUMNSパーティショニングのCONDITIONSおよび文字列型列に対するパーティションプルーニングを有効にしました [#32626](https://github.com/pingcap/tidb/issues/32626)
    - システム変数がNULLに設定された場合にエラーメッセージを返すようにしました [#32850](https://github.com/pingcap/tidb/issues/32850)
    - MPPモードでのBroadcast Joinを削除しました [#31465](https://github.com/pingcap/tidb/issues/31465)
    - ダイナミックプルーニングモードでパーティションテーブルでMPPプランを実行するサポートを追加しました [#32347](https://github.com/pingcap/tidb/issues/32347)
    - 共通表式（CTE）に対するプッシュダウン述語をサポートしました [#28163](https://github.com/pingcap/tidb/issues/28163)
    - `Statement Summary`および`Capture Plan Baselines`の構成をグローバルベースでのみ利用可能にするように構成を簡素化しました  [#30557](https://github.com/pingcap/tidb/issues/30557)
    - macOS 12でバイナリのビルド時に報告されたアラームに対処するために、gopsutilをv3.21.12に更新しました [#31607](https://github.com/pingcap/tidb/issues/31607)

+ TiKV

    - 複数のキー範囲を持つバッチのRaftstoreのサンプリング精度を向上しました [#12327](https://github.com/tikv/tikv/issues/12327)
    - Profileをより簡単に識別できるように、`debug/pprof/profile`の正しい "Content-Type"を追加しました[#11521](https://github.com/tikv/tikv/issues/11521)
    - Raftstoreがハートビートまたは読み取りリクエストを処理すると、リーダーのリース時間を無制限に更新し、レイテンシの揺らぎを減らすのに役立ちます[#11579](https://github.com/tikv/tikv/issues/11579)
    - リーダーを切り替える際に最もコストが少ないストアを選択し、性能の安定性を高めるためのサポートを追加しました[#10602](https://github.com/tikv/tikv/issues/10602)
    - Raftログを非同期で取得し、Raftstoreをブロックすることによる性能の揺れを低減するサポートを追加しました[#11320](https://github.com/tikv/tikv/issues/11320)
    - ベクター演算で`QUARTER`関数をサポートしました[#5751](https://github.com/tikv/tikv/issues/5751)
    - `BIT`データ型をTiKVにプッシュダウンするサポートを追加しました[#30738](https://github.com/pingcap/tidb/issues/30738)
    - `MOD`関数と`SYSDATE`関数をTiKVにプッシュダウンするサポートを追加しました[#11916](https://github.com/tikv/tikv/issues/11916)
    - リージョンの解決ロックステップを必要とするリージョンの数を減らすことで、TiCDCのリカバリ時間を短縮するサポートを追加しました[#11993](https://github.com/tikv/tikv/issues/11993)
    - `raftstore.raft-max-inflight-msgs`を動的に変更するサポートを追加しました[#11865](https://github.com/tikv/tikv/issues/11865)
    - 動的プルーニングモードを有効にする`EXTRA_PHYSICAL_TABLE_ID_COL_ID`をサポートするサポートを追加しました[#11888](https://github.com/tikv/tikv/issues/11888)
    - バケツ内での計算をサポートしました[#11759](https://github.com/tikv/tikv/issues/11759)
    - RawKV API V2のキーを`user-key` + `memcomparable-padding` + `timestamp`としてエンコードしました[#11965](https://github.com/tikv/tikv/issues/11965)
    - RawKV API V2の値を`user-value` + `ttl` + `ValueMeta`としてエンコードし、`ValueMeta`で`delete`をエンコードしました[#11965](https://github.com/tikv/tikv/issues/11965)
    - `raftstore.raft-max-size-per-msg`を動的に変更するサポートを追加しました[#12017](https://github.com/tikv/tikv/issues/12017)
    - Grafanaで複数のk8sを監視するサポートを追加しました[#12014](https://github.com/tikv/tikv/issues/12014)
    - レイテンシの揺れを低減するために、リーダーシップをCDCオブザーバーに転送するサポートを追加しました[#12111](https://github.com/tikv/tikv/issues/12111)
    - `raftstore.apply_max_batch_size`および`raftstore.store_max_batch_size`を動的に変更するサポートを追加しました[#11982](https://github.com/tikv/tikv/issues/11982)
    - RawKV V2は、`raw_get`または`raw_scan`リクエストを受信した際に最新バージョンを返します[#11965](https://github.com/tikv/tikv/issues/11965)
    - RCCheckTS一貫性リードをサポートしました[#12097](https://github.com/tikv/tikv/issues/12097)
    - `storage.scheduler-worker-pool-size`（スケジューラプールのスレッドカウント）を動的に変更するサポートを追加しました[#12067](https://github.com/tikv/tikv/issues/12067)
- TiKV

    - グローバルフォアグラウンドフローコントローラを使用してCPUと帯域幅の使用を制御し、TiKVのパフォーマンス安定性を向上させる [#11855](https://github.com/tikv/tikv/issues/11855)
    - `readpool.unified.max-thread-count`（UnifyReadPoolのスレッド数）を動的に変更するサポート [#11781](https://github.com/tikv/tikv/issues/11781)
    - TiKV内部パイプラインを使用してRocksDBパイプラインを置き換え、「rocksdb.enable-multibatch-write」パラメータを非推奨化する [#12059](https://github.com/tikv/tikv/issues/12059)

+ PD

    - リーダーを追い出す際に最速のオブジェクトを自動的に選択し、追い出しプロセスを高速化するサポートを追加 [#4229](https://github.com/tikv/pd/issues/4229)
    - リージョンが利用できなくなった場合、2レプリカRaftグループから投票者を削除することを禁止する [#4564](https://github.com/tikv/pd/issues/4564)
    - バランスリーダーのスケジュールを高速化する [#4652](https://github.com/tikv/pd/issues/4652)

+ TiFlash

    - TiFlashファイルの論理的な分割を禁止し（`profiles.default.dt_enable_logical_split`のデフォルト値を`false`に調整）、TiFlashコラムストレージのスペース使用効率を向上させ、TiFlashに同期されたテーブルのスペース占有量をTiKVのテーブルのスペース占有量と類似にする[ユーザードキュメント](/tiflash/tiflash-configuration.md#tiflash-configuration-parameters)参照） [#29924](https://github.com/pingcap/tidb/issues/29924)

    - 小さなテーブル向けに以前のクラスタ管理モジュールをTiDBに統合し、TiFlashのクラスタ管理とレプリカ複製メカニズムを最適化し、レプリカの作成を加速する [#29924](https://github.com/pingcap/tidb/issues/29924)

+ Tools

    + バックアップ＆リストア（BR）

        - バックアップデータの復元速度を改善。BRが15ノード（各ノードに16 CPUコア）のTiKVクラスタに16 TBのデータを復元するシミュレーションテストでは、スループットが2.66 GiB/sに達した [#27036](https://github.com/pingcap/tidb/issues/27036)
        - プレースメントルールのインポートとエクスポートをサポートする。データのインポート時にプレースメントルールを無視するかどうかを制御する`--with-tidb-placement-mode`パラメータを追加する [#32290](https://github.com/pingcap/tidb/issues/32290)

    + TiCDC

        - Grafanaに`Lag analyze`パネルを追加する [#4891](https://github.com/pingcap/tiflow/issues/4891)
        - プレースメントルールをサポートする [#4846](https://github.com/pingcap/tiflow/issues/4846)
        - HTTP APIハンドリングを同期する [#1710](https://github.com/pingcap/tiflow/issues/1710)
        - クローンフィードを再起動するための指数バックオフメカニズムを追加する [#3329](https://github.com/pingcap/tiflow/issues/3329)
        - MySQLシンクのデフォルト分離レベルを読み取りコミットに設定して、MySQLでデッドロックを減らす [#3589](https://github.com/pingcap/tiflow/issues/3589)
        - クローンフィードパラメータを作成時に検証し、エラーメッセージを洗練させる [#1716](https://github.com/pingcap/tiflow/issues/1716) [#1718](https://github.com/pingcap/tiflow/issues/1718) [#1719](https://github.com/pingcap/tiflow/issues/1719) [#4472](https://github.com/pingcap/tiflow/issues/4472)
        - Kafkaプロデューサの設定パラメータを公開し、TiCDCで構成可能にする [#4385](https://github.com/pingcap/tiflow/issues/4385)

    + TiDBデータ移行（DM）

        - 上流のテーブルスキーマが不整合であり、オプティミスティックモードでタスクを開始するのをサポートする [#3629](https://github.com/pingcap/tiflow/issues/3629) [#3708](https://github.com/pingcap/tiflow/issues/3708) [#3786](https://github.com/pingcap/tiflow/issues/3786)
        - `stopped`状態でタスクを作成するのをサポートする [#4484](https://github.com/pingcap/tiflow/issues/4484)
        - SyncerがDM-workerの作業ディレクトリを`/tmp`ではなく使用して内部ファイルを書き込み、タスク終了後にディレクトリをクリーニングするのをサポートする [#4107](https://github.com/pingcap/tiflow/issues/4107)
        - Precheckが改善された。重要なチェックはもはやスキップされない [#3608](https://github.com/pingcap/tiflow/issues/3608)

    + TiDBライトニング

        - より多くの再試行可能なエラータイプを追加する [#31376](https://github.com/pingcap/tidb/issues/31376)
        - base64形式のパスワード文字列をサポートする [#31194](https://github.com/pingcap/tidb/issues/31194)
        - エラーコードとエラー出力を標準化する [#32239](https://github.com/pingcap/tidb/issues/32239)

## バグ修正

+ TiDB

    - `SCHEDULE = majority_in_primary`、`PrimaryRegion`、および`Regions`の値が同じ場合にテーブルのプレースメントルールでテーブルを作成できないバグを修正する [#31271](https://github.com/pingcap/tidb/issues/31271)
    - インデックスルックアップ結合を使用してクエリを実行する際に `invalid transaction`エラーを修正する [#30468](https://github.com/pingcap/tidb/issues/30468)
    - 2つ以上の権限が付与される場合に`show grants`が正しくない結果を返すバグを修正する[#30855](https://github.com/pingcap/tidb/issues/30855)
    - `INSERT INTO t1 SET timestamp_col = DEFAULT` が `CURRENT_TIMESTAMP`にデフォルト設定されたフィールドのタイムスタンプをゼロのタイムスタンプに設定してしまうバグを修正する [#29926](https://github.com/pingcap/tidb/issues/29926)
    - 文字列型の非null値の最大値と最小値をエンコードすることを避けることで、結果の読み込み時にエラーが発生する問題を修正する [#31721](https://github.com/pingcap/tidb/issues/31721)
    - エスケープ文字の破損したデータでのLoadデータパニックを修正する [#31589](https://github.com/pingcap/tidb/issues/31589)
    - プライマリーキーにおいて合成照合と`greatest`または`least`関数が誤った結果を返す問題を修正する [#31789](https://github.com/pingcap/tidb/issues/31789)
    - date_addとdate_sub関数が誤ったデータ型を返す可能性があるバグを修正する [#31809](https://github.com/pingcap/tidb/issues/31809)
    - 挿入ステートメントを使用して仮想的に生成された列にデータを挿入する際に、パニックが発生する可能性がある問題を修正する [#31735](https://github.com/pingcap/tidb/issues/31735)
    - 生成リストパーティションに重複する列が存在する場合に、エラーが報告されないバグを修正する [#31784](https://github.com/pingcap/tidb/issues/31784)
    - `select for update union select`が誤ったスナップショットを使用している場合に誤った結果を返す問題を修正する [#31530](https://github.com/pingcap/tidb/issues/31530)
    - リストア操作が完了した後にリージョンが均等に分布されない可能性がある問題を修正する [#31034](https://github.com/pingcap/tidb/issues/31034)
    - `json`タイプのCOERCIBILITYが間違っているバグを修正する [#31541](https://github.com/pingcap/tidb/issues/31541)
    - `json`タイプにおいて組み込み関数を使用する際に、照合が誤っている問題を修正する [#31320](https://github.com/pingcap/tidb/issues/31320)
    - `TiFlash`レプリカの数が0に設定されてもPDルールが削除されないバグを修正する [#32190](https://github.com/pingcap/tidb/issues/32190)
    - `alter column set default`が誤ってテーブルスキーマを更新する問題を修正する [#31074](https://github.com/pingcap/tidb/issues/31074)
    - `date_format`がMySQL非互換の方法で `'\n'`を処理する問題を修正する [#32232](https://github.com/pingcap/tidb/issues/32232)
    - 結合を使用してパーティションテーブルを更新する際にエラーが発生する可能性があるバグを修正する [#31629](https://github.com/pingcap/tidb/issues/31629)
    - Enum値のNulleq関数に対する誤った範囲計算結果を修正する[#32428](https://github.com/pingcap/tidb/issues/32428)
    - `upper()`と`lower()`関数でパニックが発生する可能性があるバグを修正する [#32488](https://github.com/pingcap/tidb/issues/32488)
    - 他のタイプの列をタイムスタンプ型の列に変更する際に、タイムゾーンの問題を修正する [#29585](https://github.com/pingcap/tidb/issues/29585)
    - ChunkRPCを使用してデータをエクスポートする際にTiDB OOMが発生する問題を修正する [#31981](https://github.com/pingcap/tidb/issues/31981) [#30880](https://github.com/pingcap/tidb/issues/30880)
    - ダイナミックパーティションプルーニングモードでサブSELECT LIMITが期待どおりに機能しない問題を修正する [#32516](https://github.com/pingcap/tidb/issues/32516)
- `INFORMATION_SCHEMA.COLUMNS` テーブルの `bit` デフォルト値のフォーマットが誤っているか一貫していない場合の修正 [#32655](https://github.com/pingcap/tidb/issues/32655)
- サーバー再起動後にパーティションテーブルのリスト表示でパーティションテーブルのプルーニングが機能しない可能性のあるバグを修正 [#32416](https://github.com/pingcap/tidb/issues/32416)
- `SET timestamp` 実行後に `add column` が誤ったデフォルトタイムスタンプを使用する可能性があるバグを修正 [#31968](https://github.com/pingcap/tidb/issues/31968)
- MySQL 5.5 または 5.6 クライアントから TiDB のパスワードレスアカウントに接続すると失敗する可能性があるバグを修正 [#32334](https://github.com/pingcap/tidb/issues/32334)
- トランザクション内でダイナミックモードのパーティションテーブルを読み取る際に誤った結果が返される可能性があるバグを修正 [#29851](https://github.com/pingcap/tidb/issues/29851)
- TiDB が TiFlash に重複したタスクを送信する可能性があるバグを修正 [#32814](https://github.com/pingcap/tidb/issues/32814)
- `timdiff` 関数の入力にミリ秒が含まれる場合に誤った結果が返される可能性があるバグを修正 [#31680](https://github.com/pingcap/tidb/issues/31680)
- 明示的にパーティションを読み取り、IndexJoin プランを使用する場合に誤った結果が返される可能性があるバグを修正 [#32007](https://github.com/pingcap/tidb/issues/32007)
- 列の型を同時に変更する際に列の名前変更に失敗する可能性があるバグを修正 [#31075](https://github.com/pingcap/tidb/issues/31075)
- TiFlash プランのネットコストを計算するための式が TiKV プランと整合していない可能性があるバグを修正 [#30103](https://github.com/pingcap/tidb/issues/30103)
- アイドル接続に対して `KILL TIDB` が即座に効果を発揮しない可能性があるバグを修正 [#24031](https://github.com/pingcap/tidb/issues/24031)
- 生成列を持つテーブルをクエリする際に誤った結果が返される可能性があるバグを修正 [#33038](https://github.com/pingcap/tidb/issues/33038)
- `left join` を使用して複数のテーブルのデータを削除する際に誤った結果が返される可能性があるバグを修正 [#31321](https://github.com/pingcap/tidb/issues/31321)
- オーバーフローの場合、`SUBTIME` 関数が誤った結果を返す可能性があるバグを修正 [#31868](https://github.com/pingcap/tidb/issues/31868)
- 集約クエリに `having` 条件が含まれる場合、`selection` 演算子をプッシュダウンできない可能性があるバグを修正 [#33166](https://github.com/pingcap/tidb/issues/33166)
- クエリでエラーが報告されたときに CTE がブロックされる可能性があるバグを修正 [#31302](https://github.com/pingcap/tidb/issues/31302)
- 念入りモードでテーブルを作成する際に varbinary または varchar 列の長さが過大でエラーが発生する可能性があるバグを修正 [#30328](https://github.com/pingcap/tidb/issues/30328)
- `information_schema.placement_policies` における followers の間違った数を修正するバグを修正 [#31702] (https://github.com/pingcap/tidb/issues/31702)
- インデックス作成時に列のプレフィックス長を 0 として指定できる問題を修正 [#31972](https://github.com/pingcap/tidb/issues/31972)
- パーティション名の最後にスペースが付いていることを許可する問題を修正 [#31535](https://github.com/pingcap/tidb/issues/31535)
- `RENAME TABLE` ステートメントのエラーメッセージを修正する [#29893](https://github.com/pingcap/tidb/issues/29893)

+ TiKV

    - ピアステータスが `Applying` の状態でスナップショットファイルが削除された際に発生するパニックの問題を修正 [#11746](https://github.com/tikv/tikv/issues/11746)
    - フロー制御が有効になっており、`level0_slowdown_trigger` が明示的に設定されている場合に QPS が低下する問題を修正 [#11424](https://github.com/tikv/tikv/issues/11424)
    - ピアの破棄が高いレイテンシを引き起こす可能性がある問題を修正 [#10210](https://github.com/tikv/tikv/issues/10210)
    - GC ワーカーがビジーな場合にデータ範囲を削除できない TiKV のバグを修正 [#11903](https://github.com/tikv/tikv/issues/11903)
    - 一部の特定のケースで `StoreMeta` のデータが意図せず削除された際に TiKV がパニックを引き起こす可能性があるバグを修正 [#11852](https://github.com/tikv/tikv/issues/11852)
    - ARM プラットフォームでプロファイリングを実行する際に TiKV がパニックを引き起こす可能性があるバグを修正 [#10658](https://github.com/tikv/tikv/issues/10658)
    - TiKV が 2年以上動作している場合にパニックを起こす可能性があるバグを修正 [#11940](https://github.com/tikv/tikv/issues/11940)
    - ARM64 アーキテクチャでのコンパイルの問題を修正（SSE 命令セットが不足していた） [#12034](https://github.com/tikv/tikv/issues/12034)
    - 未初期化のレプリカを削除すると古いレプリカが再作成される可能性があるバグを修正 [#10533](https://github.com/tikv/tikv/issues/10533)
    - 古いメッセージが TiKV にパニックを引き起こす可能性があるバグを修正 [#12023](https://github.com/tikv/tikv/issues/12023)
    - TsSet 変換で未定義の動作（UB）が発生する可能性がある問題を修正 [#12070](https://github.com/tikv/tikv/issues/12070)
    - レプリカの読み取りが線形性を守らない可能性があるバグを修正 [#12109](https://github.com/tikv/tikv/issues/12109)
    - Ubuntu 18.04 で TiKV がプロファイリングを行う際に発生するパニックの可能性を修正 [#9765](https://github.com/tikv/tikv/issues/9765)
    - tikv-ctl が誤った文字列の一致により間違った結果を返す問題を修正 [#12329](https://github.com/tikv/tikv/issues/12329)
    - メモリメトリクスのオーバーフローによる断続的なパケット損失およびメモリ不足の問題を修正 [#12160](https://github.com/tikv/tikv/issues/12160)
    - TiKV を終了するときに誤って TiKV パニックが報告される可能性がある問題を修正 [#12231](https://github.com/tikv/tikv/issues/12231)

+ PD

    - PD が意味のない Joint Consensus ステップのオペレータを生成する問題を修正 [#4362](https://github.com/tikv/pd/issues/4362)
    - PD クライアントを閉じる際に TSO の取り消し処理がスタックする可能性がある問題を修正 [#4549](https://github.com/tikv/pd/issues/4549)
    - リージョンスキャッタラーのスケジューリングでいくつかのピアが失われる問題を修正 [#4565](https://github.com/tikv/pd/issues/4565)
    - `dr-autosync` の `Duration` フィールドを動的に構成できない問題を修正 [#4651](https://github.com/tikv/pd/issues/4651)

+ TiFlash

    - メモリ制限が有効な場合に発生する TiFlash パニックの問題を修正 [#3902](https://github.com/pingcap/tiflash/issues/3902)
    - 期限切れのデータが遅くリサイクルされる問題を修正 [#4146](https://github.com/pingcap/tiflash/issues/4146)
    - `Snapshot` を複数の DDL 操作と同時に適用すると TiFlash パニックが発生する可能性がある問題を修正 [#4072](https://github.com/pingcap/tiflash/issues/4072)
    - 高負荷のリード作業で列を追加した後にクエリエラーが発生する可能性がある問題を修正 [#3967](https://github.com/pingcap/tiflash/issues/3967)
    - 負の引数を持つ `SQRT` 関数が `NaN` 而 全く `Null` を返す問題を修正 [#3598](https://github.com/pingcap/tiflash/issues/3598)
    - `INT` を `DECIMAL` に変換する際にオーバーフローが発生する可能性がある問題を修正 [#3920](https://github.com/pingcap/tiflash/issues/3920)
    - マルチバリュー式で `IN` の結果が正しくない問題を修正 [#4016](https://github.com/pingcap/tiflash/issues/4016)
    - 日付形式が `'\n'` を無効なセパレータとして識別する問題を修正 [#4036](https://github.com/pingcap/tiflash/issues/4036)
    - ラーナー読み取りプロセスが高い並行処理シナリオで長時間を要する問題を修正 [#3555](https://github.com/pingcap/tiflash/issues/3555)
    - `DATETIME` を `DECIMAL` にキャストすると誤った結果が返される問題を修正 [#4151](https://github.com/pingcap/tiflash/issues/4151)
    - クエリのキャンセル時にメモリリークが発生する問題を修正 [#4098](https://github.com/pingcap/tiflash/issues/4098)
    - エラスティックスレッドプールを有効にするとメモリリークが発生する問題を修正 [#4098](https://github.com/pingcap/tiflash/issues/4098)
- ローカルトンネルが有効になっている場合、MPPクエリがキャンセルされると、タスクが永遠にハングする場合のバグを修正しました[#4229](https://github.com/pingcap/tiflash/issues/4229)
    - HashJoinのビルド側の失敗がMPPクエリが永遠にハングする原因になるバグを修正しました[#4195](https://github.com/pingcap/tiflash/issues/4195)
    - MPPタスクが永遠にスレッドをリークする可能性があるバグを修正しました[#4238](https://github.com/pingcap/tiflash/issues/4238)

+ ツール

    + バックアップ＆リストア（BR）

        - リストア操作が回復不能なエラーに遭遇した場合、BRがスタックするバグを修正しました[#33200](https://github.com/pingcap/tidb/issues/33200)
        - バックアップの再試行中に暗号情報が失われた場合に、リストア操作が失敗する原因となるバグを修正しました[#32423](https://github.com/pingcap/tidb/issues/32423)

    + TiCDC

        - `batch-replace-enable`が無効になっている場合、MySQL Sinkが重複した `replace` SQL ステートメントを生成するバグを修正しました[#4501](https://github.com/pingcap/tiflow/issues/4501)
        - TiCDCノードがPDリーダーが停止した場合に異常終了するバグを修正しました[#4248](https://github.com/pingcap/tiflow/issues/4248)
        - 特定のMySQLバージョンで `Unknown system variable 'transaction_isolation'` エラーを修正しました[#4504](https://github.com/pingcap/tiflow/issues/4504)
        - `Canal-JSON` が `string` を誤って処理することでTiCDCパニックが発生する問題を修正しました[#4635](https://github.com/pingcap/tiflow/issues/4635)
        - 特定の条件下でシーケンスが誤ってレプリケーションされるバグを修正しました[#4563](https://github.com/pingcap/tiflow/issues/4552)
        - `Canal-JSON` が `nil` をサポートしていないことでTiCDCパニックが発生する問題を修正しました[#4736](https://github.com/pingcap/tiflow/issues/4736)
        - `Enum/Set`および`TinyText/MediumText/Text/LongText`のavroコーデックのデータマッピングが間違っている問題を修正しました[#4454](https://github.com/pingcap/tiflow/issues/4454)
        - Avroが `NOT NULL` 列をNULL可能フィールドに変換するバグを修正しました[#4818](https://github.com/pingcap/tiflow/issues/4818)
        - TiCDCが正常に終了できない問題を修正しました[#4699](https://github.com/pingcap/tiflow/issues/4699)

    + TiDB データ移行（DM）

        - ステータスをクエリするときにのみ、syncerメトリクスが更新される問題を修正しました[#4281](https://github.com/pingcap/tiflow/issues/4281)
        - セーフモードで更新ステートメントの実行エラーがDMワーカーを異常終了させる可能性がある問題を修正しました[#4317](https://github.com/pingcap/tiflow/issues/4317)
        - 長いvarcharがエラー `Column length too big` を報告するバグを修正しました[#4637](https://github.com/pingcap/tiflow/issues/4637)
        - 同じ上流からデータを書き込む複数のDMワーカーによって引き起こされる競合問題を修正しました[#3737](https://github.com/pingcap/tiflow/issues/3737)
        - ログに数百回の "checkpoint has no change, skip sync flush checkpoint" が表示され、レプリケーションが非常に遅いという問題を修正しました[#4619](https://github.com/pingcap/tiflow/issues/4619)
        - 悲観的モードでシャードをマージし、上流からの増分データをレプリケートする際にDMLが損失する問題を修正しました[#5002](https://github.com/pingcap/tiflow/issues/5002)

    + TiDB ライトニング

        - いくつかのインポートタスクにソースファイルが含まれていない場合に、TiDBライトニングがメタデータスキーマを削除しない場合があるバグを修正しました[#28144](https://github.com/pingcap/tidb/issues/28144)
        - ソースファイルとターゲットクラスターのテーブル名が異なる場合に発生するパニックを修正しました[#31771](https://github.com/pingcap/tidb/issues/31771)
        - チェックサムエラー "GC life time is shorter than transaction duration" を修正しました[#32733](https://github.com/pingcap/tidb/issues/32733)
        - 空のテーブルをチェックする際にTiDBライトニングがスタックする問題を修正しました[#31797](https://github.com/pingcap/tidb/issues/31797)

    + Dumpling

        - `dumpling --sql $query` を実行する際に表示される進行状況が正確でない問題を修正しました[#30532](https://github.com/pingcap/tidb/issues/30532)
        - Amazon S3が圧縮データのサイズを正しく計算できない問題を修正しました[#30534](https://github.com/pingcap/tidb/issues/30534)

    + TiDB Binlog

        - 大規模な上流書き込みトランザクションがKafkaにレプリケートされるとTiDB Binlogがスキップされる可能性がある問題を修正しました[#1136](https://github.com/pingcap/tidb-binlog/issues/1136)