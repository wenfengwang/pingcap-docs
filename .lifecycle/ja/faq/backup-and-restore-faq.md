---
title: バックアップとリストアのFAQ
summary: バックアップとリストアに関するよくある質問（FAQ）およびその解決策について学ぶ。
aliases: ['/docs/dev/br/backup-and-restore-faq/','/tidb/dev/pitr-troubleshoot/','/tidb/dev/pitr-known-issues/']
---

# バックアップとリストアのFAQ

このドキュメントには、TiDB Backup & Restore (BR) のよくある質問（FAQ）とその解決策がリストされています。

## 誤ってデータを削除または更新した後、データを迅速に回復するにはどうすればよいですか？

TiDB v6.4.0 では、フラッシュバック機能が導入されています。この機能を使用して、GC時間内にデータを指定した時点に迅速に回復できます。したがって、誤操作が発生した場合は、この機能を使用してデータを回復できます。詳細は、[フラッシュバック クラスター](/sql-statements/sql-statement-flashback-to-timestamp.md) および [フラッシュバック データベース](/sql-statements/sql-statement-flashback-database.md)を参照してください。

## TiDB v5.4.0 以降で、クラスターに対するバックアップタスクを重いワークロードの状態で実行すると、バックアップタスクの速度が遅くなるのはなぜですか？

TiDB v5.4.0 から、BR はバックアップタスクのための自動調整機能を導入しました。v5.4.0 以降のクラスターでは、この機能がデフォルトで有効になっています。クラスターのワークロードが重い場合、この機能はバックアップタスクが使用するリソースを制限し、オンラインクラスターへの影響を軽減します。詳細については、[バックアップ 自動調整](/br/br-auto-tune.md)を参照してください。

TiKV では、[自動調整機能の動的な設定](/tikv-control.md#modify-the-tikv-configuration-dynamically)がサポートされています。次の方法で、機能を有効または無効にできます（クラスターを再起動する必要はありません）。

- 自動調整を無効にする: TiKV 設定項目 [`backup.enable-auto-tune`](/tikv-configuration-file.md#enable-auto-tune-new-in-v540) を `false` に設定します。
- 自動調整を有効にする: `backup.enable-auto-tune` を `true` に設定します。v5.3.x から v5.4.0 以降にアップグレードされたクラスターでは、自動調整機能はデフォルトで無効になっています。手動で有効にする必要があります。

`br` を使用して自動調整を有効または無効にする方法の詳細については、[自動調整の使用](/br/br-auto-tune.md#use-auto-tune)を参照してください。

さらに、自動調整により、バックアップタスクが使用するスレッドのデフォルト数が減少します。詳細については、`backup.num-threads`（/tikv-configuration-file.md#num-threads-1）を参照してください。そのため、Grafana ダッシュボードでは、v5.4.0 より前のバージョンよりも、バックアップタスクが使用する速度、CPU 使用率、および I/O リソース利用率が低くなります。v5.4.0 以前では、`backup.num-threads` のデフォルト値は `CPU * 0.75` であり、つまり、バックアップタスクが使用するスレッドの数は論理 CPU コアの 75% を占めます。最大値は `32` でした。v5.4.0 から、この設定項目のデフォルト値は `CPU * 0.5` になり、最大値は `8` です。

オフラインクラスターでバックアップタスクを実行する場合、`tikv-ctl` を使用して`backup.num-threads` の値を大きい数値に変更してバックアップの速度を上げることができます。

## PITR 関連の問題

### [PITR](/br/br-pitr-guide.md) と [クラスター フラッシュバック](/sql-statements/sql-statement-flashback-to-timestamp.md) の違いは何ですか？

ユースケースの観点からは、PITR は通常、クラスターが完全にサービスを停止しているか、データが破損して他のソリューションで回復できない場合に、クラスターのデータを指定された時点に復元するために使用されます。PITR を使用するには、データ復旧用の新しいクラスターが必要です。クラスター フラッシュバック機能は、ユーザーの誤操作やその他の要因によって引き起こされたデータエラーのシナリオに特化しており、そのクラスターのデータをエラーが発生する前の最新のタイムスタンプにインプレースで復元できます。

ほとんどの場合、フラッシュバックは、ヒューマンエラーによるデータエラーのための回復ソリューションとして、RPO（復旧時の最大許容データ損失）と RTO（復旧時の目標復旧時間）がほぼゼロに近いため、PITR よりも優れています。ただし、クラスターが完全に利用できない場合、フラッシュバックはその時点で実行できないため、この場合は PITR がクラスターを回復する唯一の解決策です。そのため、PITR は常にデータベースの災害復旧戦略を開発する際に必要なソリューションです。フラッシュバックよりも RPO（最大許容データ損失）と RTO（目標復旧時間）が長くなりますが、必ずしも使用できるソリューションです。

### 上流データベースが物理インポートモードを使用して TiDB Lightning でデータをインポートする場合、ログバックアップ機能が使用できなくなるのはなぜですか？

現在、ログバックアップ機能は TiDB Lightning に完全に適応されていません。そのため、TiDB Lightning の物理モードでインポートされたデータはログデータにバックアップすることができません。

ログバックアップタスクを作成する上流のクラスターでは、TiDB Lightning の物理モードを使用せず、代わりに TiDB Lightning の論理モードを使用することができます。物理モードを使用する必要がある場合は、インポートが完了した後にスナップショットバックアップを実行して、スナップショットバックアップ後の時点で PITR を復元できるようにします。

### インデックスの追加の高速化機能が PITR と非互換性があるのはなぜですか？

問題: [#38045](https://github.com/pingcap/tidb/issues/38045)

現在、[インデックスの追加](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)機能で作成されたインデックスデータは PITR によってバックアップできません。

そのため、PITR 回復が完了すると、BR はインデックスの高速化によって作成されたインデックスデータを削除し、それを再作成します。多くのインデックスが高速化によって作成されている場合やログバックアップが行われる際にインデックスデータが大きい場合は、インデックスを作成した後に完全バックアップを実行することをお勧めします。

### クラスターがネットワーク分割障害から回復したが、ログバックアップタスクの進行状況のチェックポイントがまだ再開しないのはなぜですか？

問題: [#13126](https://github.com/tikv/tikv/issues/13126)

クラスターでネットワーク分割障害が発生した後、バックアップタスクはログのバックアップを継続できません。一定の再試行時間を経過すると、タスクは `ERROR` 状態に設定されます。この時点で、バックアップタスクは停止します。

この問題を解決するには、`br log resume` コマンドを手動で実行してログバックアップタスクを再開する必要があります。

### PITR を実行する際に `execute over region id` エラーが返された場合はどうすればよいですか？

問題: [#37207](https://github.com/pingcap/tidb/issues/37207)

この問題は通常、完全なデータインポート中にログバックアップを有効にし、その後 PITR を実行すると発生します。

具体的には、非常に多くのホットスポットライトが長時間（24 時間など）維持された場合（Grafana でメトリクスを表示できます: **TiKV-Details** -> **Backup Log** -> **Handle Event Rate**）、各 TiKV ノードの OPS が 50k/s を超える場合にこの問題が発生する可能性があります。

データインポート後にスナップショットバックアップを実行し、このスナップショットバックアップに基づいて PITR を実行することをお勧めします。

## `br restore point` コマンドを使用して下流クラスターを復元した後、TiFlash からデータにアクセスできません。どうすればよいですか？

現在、PITR はデータ復元中に直接データを TiFlash に書き込むことをサポートしていません。代わりに、`br` コマンドラインツールは、`ALTER TABLE table_name SET TIFLASH REPLICA ***` DDL を実行してデータを複製します。そのため、PITR がデータ復元後、TiFlash レプリカは直ちに利用できません。代わりに、データが TiKV ノードから複製されるため、データが複製されるまで一定の時間を待つ必要があります。複製の進行状況を確認するには、`INFORMATION_SCHEMA.tiflash_replica` テーブルの `progress` 情報を確認します。

### ログバックアップタスクの `status` が `ERROR` になった場合はどうすればよいですか？

ログバックアップタスク中、タスクの状態が失敗し、再試行しても回復できない場合、タスクの状態は `ERROR` になります。次の例を参照してください：

```shell
br log status --pd x.x.x.x:2379

● Total 1 Tasks.
> #1 <
                    name: task1
                  status: ○ ERROR
                   start: 2022-07-25 13:49:02.868 +0000
                     end: 2090-11-18 14:07:45.624 +0000
                 storage: s3://tmp/br-log-backup0ef49055-5198-4be3-beab-d382a2189efb/Log
             speed(est.): 0.00 ops/s
      checkpoint[global]: 2022-07-25 14:46:50.118 +0000; gap=11h31m29s
          error[store=1]: KV:LogBackup:RaftReq
error-happen-at[store=1]: 2022-07-25 14:54:44.467 +0000; gap=11h23m35s
```
エラーメッセージ [store=1]: リトライ回数が超過しました: およびエラー、初期スナップショットの取得に失敗: スナップショットの取得に失敗しました (region_id = 94812): Raftstore のリクエスト中にエラーが発生しました: メッセージ: "read index not ready, reason can not read index due to merge, region 94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }: 初期スナップショットの取得に失敗: スナップショットの取得に失敗しました (region_id = 94812): Raftstore のリクエスト中にエラーが発生しました: メッセージ: "read index not ready, reason can not read index due to merge, region 94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }: 初期スナップショットの取得に失敗: スナップショットの取得に失敗しました (region_id = 94812): Raftstore のリクエスト中にエラーが発生しました: メッセージ: "read index not ready, reason can not read index due to merge, region 94812" read_index_not_ready { reason: "can not read index due to merge" region_id: 94812 }

この問題を解決するには、エラーメッセージを確認して原因を特定し、指示に従って操作を行います。問題が解決した後、次のコマンドを実行してタスクを再開します。

```shell
br log resume --task-name=task1 --pd x.x.x.x:2379
```

バックアップタスクが再開されたら、`br log status` を使用して状態を確認できます。タスクのステータスが `NORMAL` になると、バックアップタスクが継続します。

```shell
● 合計タスク数 1。
> #1 <
              名前: task1
               状態: ● NORMAL
             開始時刻: 2022-07-25 13:49:02.868 +0000
               終了時刻: 2090-11-18 14:07:45.624 +0000
           ストレージ: s3://tmp/br-log-backup0ef49055-5198-4be3-beab-d382a2189efb/Log
       速度(推定): 15509.75 ops/s
チェックポイント[グローバル]: 2022-07-25 14:46:50.118 +0000; 間隔=6m28s
```

> **注意:**
>
> この機能は複数のデータバージョンをバックアップします。バックアップタスクが長時間実行され、状態が `ERROR` になると、このタスクのチェックポイントデータは `セーフ ポイント` として設定され、セーフ ポイントのデータは24時間以内にガベージコレクションされません。したがって、エラーを再開した後は、最後のチェックポイントからバックアップタスクを再開します。タスクが24時間以上失敗し、最後のチェックポイントデータがガベージコレクションされている場合は、タスクを再開するとエラーが報告されます。この場合、`br log stop` コマンドを実行して最初にタスクを停止し、その後新しいバックアップタスクを開始することができます。

### 中断されたタスクを再開するために `br log resume` コマンドを使用した際に `ErrBackupGCSafepointExceeded` エラーメッセージが返された場合、対処方法は何ですか？

```shell
エラー: GC セーフポイントの確認に失敗しました。チェックポイント ts 433177834291200000: GC セーフポイント 433193092308795392 はTS 433177834291200000を超えています: [BR:Backup:ErrBackupGCSafepointExceeded]backup GC セーフポイントが超過しました
```

ログバックアップタスクを一時停止した後、MVCC データがガベージコレクションされないように、一時停止タスクプログラムは現在のチェックポイントをサービス セーフポイントに自動的に設定します。これにより、24時間以内に生成された MVCC データが保持されます。バックアップチェックポイントの MVCC データが 24 時間以上生成されている場合、チェックポイントのデータがガベージコレクションされ、バックアップタスクは再開できません。

この問題を解決するには、`br log stop` を使用して現在のタスクを削除し、`br log start` を使用してログバックアップタスクを作成します。同時に、以降の PITR に対して完全バックアップを実行できます。

## 機能の互換性に関する問題

### BR コマンドライン ツールを使用して復元されたデータを TiCDC や Drainer の上流クラスタにレプリケーションできないのはなぜですか？

+ **BR で復元されたデータは、下流にレプリケーションされません**。これは BR が SST ファイルを直接インポートするためであり、現在の下流クラスタはこれらのファイルを上流から取得できないためです。

+ v4.0.3 より前では、復元中に生成された DDL ジョブは TiCDC/Drainer で予期せぬ DDL 実行を引き起こす可能性があります。そのため、TiCDC/Drainer の上流クラスタで復元を実行する必要がある場合は、BR コマンドライン ツールを使用して復元されたすべてのテーブルを TiCDC/Drainer のブロック リストに追加してください。

TiCDC には [`filter.rules`](https://github.com/pingcap/tiflow/blob/7c3c2336f98153326912f3cf6ea2fbb7bcc4a20c/cmd/changefeed.toml#L16) を使用してブロックリストを構成し、Drainer には [`syncer.ignore-table`](/tidb-binlog/tidb-binlog-configuration-file.md#ignore-table) を使用してブロックリストを構成できます。

### `br` コマンドライン ツールを使用してデータを復元するときに `new_collations_enabled_on_first_bootstrap` の不一致が報告される原因は何ですか？

TiDB v6.0.0 以降、[`new_collations_enabled_on_first_bootstrap`](/tidb-configuration-file.md#new_collations_enabled_on_first_bootstrap) のデフォルト値が `false` から `true` に変更されました。BR は上流クラスタの `new_collations_enabled_on_first_bootstrap` 構成をバックアップし、その後この構成の値が上流と下流のクラスタで一致しているかどうかを確認します。値が一致する場合、BR は安全に上流クラスタでバックアップされたデータを下流クラスタに復元します。値が一致しない場合、BR はデータの復元を実行せずにエラーを報告します。

早期の v6.0.0 バージョンの TiDB クラスタでデータをバックアップし、このデータを v6.0.0 以降の TiDB クラスタに復元したい場合は、上流と下流のクラスタで `new_collations_enabled_on_first_bootstrap` の値が一致するかどうかを手動で確認する必要があります。

- 値が一致する場合は、復元コマンドに `--check-requirements=false` を追加してこの構成のチェックをスキップできます。
- 値が一致しない場合、復元を強制的に実行し、BR がデータ検証エラーを報告します。

### プレースメント ルールをクラスタに復元する際にエラーが発生するのはなぜですか？

v6.0.0 より前のバージョンでは、BR は[プレースメント ルール](/placement-rules-in-sql.md)をサポートしていませんでした。v6.0.0 以降、BR はプレースメント ルールをサポートし、バックアップおよび復元モードを制御するためのコマンドライン オプション `--with-tidb-placement-mode=strict/ignore` を導入しました。デフォルト値 `strict` では、BR はプレースメント ルールをインポートおよび検証しますが、値が `ignore` の場合、すべてのプレースメント ルールを無視します。

## データの復元に関する問題

### `Io(Os...)` エラーを処理するためには何をすればよいですか？

ほとんどのこれらの問題は、TiKV がデータをディスクに書き込む際に発生するシステムコールエラーです。たとえば、`Io(Os {code: 13, kind: PermissionDenied...})` や `Io(Os {code: 2, kind: NotFound...})` などです。

このような問題に対処するには、まずバックアップディレクトリのマウント方法とファイルシステムを確認し、別のフォルダや別のハードディスクにデータをバックアップしてみてください。

たとえば、`samba` によって構築されたネットワークディスクにデータをバックアップしようとした場合、`Code: 22(invalid argument)` エラーが発生することがあります。

### 復元中の `rpc error: code = Unavailable desc =...` エラーを処理するためには何をすればよいですか？

このエラーは、復元対象クラスタの容量が不足している場合に発生する可能性があります。このクラスタの監視メトリックや TiKV ログを確認することで、原因をさらに確認できます。

この問題を処理するために、クラスタリソースをスケールアウトしたり、復元中の並列処理を減らしたり、`RATE_LIMIT` オプションを有効にしたりすることができます。

### バックアップデータの復元において、エラーメッセージ `the entry too large, the max entry size is 6291456, the size of data is 7690800` が表示された場合、対処方法は何ですか？

`--ddl-batch-size` を `128` またはそれ以下の値に設定することで、一括で作成するテーブルの数を減らすことができます。

`--ddl-batch-size` の値が `1` より大きい場合、BR は TiDB がテーブルの作成のために生成した DDL ジョブを TiKV が維持する DDL ジョブキューに書き込みます。この場合、TiDB が一度に送信するすべてのテーブルスキーマの合計サイズは、デフォルトで `6 MB` です（この値を **変更しない** ことをお勧めします。詳細については、[`txn-entry-size-limit`](/tidb-configuration-file.md#txn-entry-size-limit-new-in-v50) および [`raft-entry-max-size`](/tikv-configuration-file.md#raft-entry-max-size) を参照してください）。したがって、`--ddl-batch-size` を異常に大きな値に設定した場合、一度に TiDB がバッチ処理で送信するテーブルスキーマのサイズが指定された値を超えるため、BR が `the entry too large, the max entry size is 6291456, the size of data is 7690800` エラーを報告します。

### `local` ストレージを使用するときに、バックアップされたファイルはどこに保存されますか？

> **注記:**
> **ネットワークファイルシステム（NFS）がBRまたはTiKVノードにマウントされていない場合、またはAmazon S3、GCS、またはAzure Blob Storageプロトコルをサポートする外部ストレージを使用する場合、BRによってバックアップされるデータがそれぞれのTiKVノードに生成されます。**BRの展開方法としては推奨されません**。なぜなら、バックアップデータが各ノードのローカルファイルシステムに散在しているため、バックアップデータの収集がデータの冗長性および運用・保守上の問題を引き起こす可能性があります。また、バックアップデータの収集前にデータを直接復元する場合、「SSTファイルが見つかりません」というエラーが発生します。

ローカルストレージを使用する場合、「backupmeta」はBRを実行しているノードに生成され、バックアップファイルは各リージョンのリーダーノードに生成されます。

### データの復元中に`local://...:download sst failed`というエラーメッセージが返された場合、どうすればよいですか？

データを復元する際、各ノードは**全て**のバックアップファイル（SSTファイル）にアクセスする必要があります。デフォルトでは`local`ストレージを使用する場合、異なるノードにバックアップファイルが散在しているため、データを復元することができません。そのため、各TiKVノードのバックアップファイルを他のTiKVノードにコピーする必要があります。 **Amazon S3、Google Cloud Storage（GCS）、Azure Blob Storage、またはNFSにバックアップデータを保存することをおすすめします**。

### `root`を使用しても`Permission denied`または`No such file or directory`エラーが発生した場合、どうすればよいですか？

TiKVがバックアップディレクトリにアクセスできるかを確認する必要があります。データをバックアップするには、TiKVが書き込み権限を持っているかを確認してください。データを復元するには、読み込み権限を持っているかを確認してください。

バックアップ操作中、ストレージメディアがローカルディスクまたはネットワークファイルシステム（NFS）の場合、`br`を起動するユーザーとTiKVを起動するユーザーが一致していることを確認してください（`br`とTiKVが異なるマシンにある場合、ユーザーのUIDが一致している必要があります）。そうしないと、`Permission denied`の問題が発生する可能性があります。

ディスク許可の問題のため、`root`ユーザーとして`br`を実行すると失敗する可能性があります。なぜなら、バックアップファイル（SSTファイル）はTiKVによって保存されるためです。

> **ノート:**
>
> データ復元中に同様の問題が発生する可能性があります。SSTファイルが初めて読み込まれる際に読み取り権限が確認されます。DDLの実行期間には権限をチェックすると長い間隔があるため、長い間待機した後に「Permission denied」というエラーメッセージを受け取る可能性があります。

そのため、以下の手順に従ってデータの復元前に権限を確認することをお勧めします:

1. プロセスクエリのためのLinuxコマンドを実行します:

    {{< copyable "shell-regular" >}}

    ```bash
    ps aux | grep tikv-server
    ```

    出力は次のようになります:

    ```shell
    tidb_ouo  9235 10.9  3.8 2019248 622776 ?      Ssl  08:28   1:12 bin/tikv-server --addr 0.0.0.0:20162 --advertise-addr 172.16.6.118:20162 --status-addr 0.0.0.0:20188 --advertise-status-addr 172.16.6.118:20188 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20162 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20162/log/tikv.log
    tidb_ouo  9236  9.8  3.8 2048940 631136 ?      Ssl  08:28   1:05 bin/tikv-server --addr 0.0.0.0:20161 --advertise-addr 172.16.6.118:20161 --status-addr 0.0.0.0:20189 --advertise-status-addr 172.16.6.118:20189 --pd 172.16.6.118:2379 --data-dir /home/user1/tidb-data/tikv-20161 --config conf/tikv.toml --log-file /home/user1/tidb-deploy/tikv-20161/log/tikv.log
    ```

    または、次のコマンドを実行できます:

    {{< copyable "shell-regular" >}}

    ```bash
    ps aux | grep tikv-server | awk '{print $1}'
    ```

    出力は次のようになります:

    ```shell
    tidb_ouo
    tidb_ouo
    ```

2. `tiup`コマンドを使用してクラスターの起動情報をクエリします:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster list
    ```

    出力は次のようになります:

    ```shell
    [root@Copy-of-VM-EE-CentOS76-v1 br]# tiup cluster list
    Starting component `cluster`: /root/.tiup/components/cluster/v1.5.2/tiup-cluster list
    Name          User      Version  Path                                               PrivateKey
    ----          ----      -------  ----                                               ----------
    tidb_cluster  tidb_ouo  v5.0.2   /root/.tiup/storage/cluster/clusters/tidb_cluster  /root/.tiup/storage/cluster/clusters/tidb_cluster/ssh/id_rsa
    ```

3. バックアップディレクトリの権限を確認します。例えば、`backup`はバックアップデータのためのものです:

    {{< copyable "shell-regular" >}}

    ```bash
    ls -al backup
    ```

    出力は次のようになります:

    ```shell
    [root@Copy-of-VM-EE-CentOS76-v1 user1]# ls -al backup
    total 0
    drwxr-xr-x  2 root root   6 Jun 28 17:48 .
    drwxr-xr-x 11 root root 310 Jul  4 10:35 ..
    ```

    ステップ2の出力から、`tikv-server`インスタンスがユーザー`tidb_ouo`によって起動されていることがわかります。しかし、ユーザー`tidb_ouo`は`backup`に対して書き込み権限を持っていないため、バックアップに失敗します。

### `mysql`スキーマのテーブルが復元されないのはなぜですか？

BR v5.1.0以降では、フルバックアップを実行すると、BRは**`mysql`スキーマのテーブル**をバックアップします。BR v6.2.0未満では、デフォルトの構成ではBRはユーザーデータのみを復元し、**`mysql`スキーマ**のテーブルは復元しません。

`mysql`スキーマ内のユーザーによって作成されたテーブルを復元するには、[テーブルフィルタ](/table-filter.md#syntax)を使用してテーブルを明示的に含めることができます。次の例では、BRが通常の復元を実行する際に`mysql.usertable`テーブルをどのように復元するかが示されています。

{{< copyable "shell-regular" >}}

```shell
br restore full -f '*.*' -f '!mysql.*' -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

前述のコマンドでは、

- `-f '*.*'`はデフォルトのルールを上書きするために使用されます
- `-f '!mysql.*'`は`mysql`のテーブルを復元しないようBRに指示します
- `-f 'mysql.usertable'`は`mysql.usertable`を復元することを示します

`mysql.usertable`の復元のみが必要な場合は、次のコマンドを実行してください:

{{< copyable "shell-regular" >}}

```shell
br restore full -f 'mysql.usertable' -s $external_storage_url --with-sys-table
```

なお、[テーブルフィルタ](/table-filter.md#syntax)を構成していても、**BRは以下のシステムテーブルを復元しません**:

- 統計テーブル（`mysql.stat_*`）。ただし、統計情報は復元できます。[統計情報のバックアップ](/br/br-snapshot-manual.md#back-up-statistics)を参照してください。
- システム変数テーブル（`mysql.tidb`、`mysql.global_variables`）
- [その他のシステムテーブル](https://github.com/pingcap/tidb/blob/master/br/pkg/restore/systable_restore.go#L31)

## バックアップと復元に関して知っておくべきその他のこと

### バックアップデータのサイズはどのくらいですか？バックアップにはレプリカがありますか？

データのバックアップ中、バックアップファイルは各リージョンのリーダーノードに生成されます。バックアップのサイズはデータサイズと等しく、冗長なレプリカはありません。そのため、総データサイズはおおよそTiKVデータの合計数をレプリカの数で割ったものになります。

しかし、ローカルストレージからデータを復元する場合、レプリカの数はTiKVノードの数と等しくなります。なぜなら、各TiKVノードが全てのバックアップファイルにアクセスする必要があるためです。

### BRを使用してバックアップまたは復元を行った後に監視ノードで表示されるディスク使用量が一貫していないのはなぜですか？

この不一致は、バックアップで使用されるデータの圧縮率が復元で使用されるデフォルトの圧縮率と異なることにより引き起こされます。チェックサムに成功すれば、この問題を無視してかまいません。

### BRがバックアップデータを復元した後、テーブルの統計情報を更新するために`ANALYZE`ステートメントを実行する必要がありますか？

BRは統計情報をバックアップしません（v4.0.9では除く）。従って、バックアップデータを復元した後は、手動で`ANALYZE TABLE`を実行するか、TiDBが自動的に`ANALYZE`を実行するのを待つ必要があります。

v4.0.9では、BRはデフォルトで統計情報をバックアップしますが、これによって多くのメモリが消費されます。バックアッププロセスが正常に進行するために、v4.0.10からは統計情報のバックアップがデフォルトで無効になりました。

テーブルで`ANALYZE`を実行しないと、不正確な統計情報のためにTiDBが最適な実行プランを選択できず失敗します。クエリのパフォーマンスが重要でない場合は`ANALYZE`を無視できます。

### 同一クラスターのデータを復元するために複数の復元タスクを同時に開始することはできますか？

同一クラスターのデータを復元するために複数の復元タスクを同時に開始することは**強くお勧めしません**。その理由は以下の通りです：

- BRがデータを復元する際、PDの一部のグローバル設定を変更します。そのため、同時に複数の復元タスクを開始すると、これらの設定が誤って上書きされ、クラスターの状態が異常になる可能性があります。
- BRは多くのクラスターリソースを消費してデータを復元します。そのため、実際には複数の復元タスクを並行して実行しても、復元速度がわずかに向上するだけです。
- データを復元するために複数の復元タスクを並行して実行することについては、テストが行われていないため、成功することは保証されていません。

### BRはテーブルの`SHARD_ROW_ID_BITS`および`PRE_SPLIT_REGIONS`情報をバックアップするのですか？復元されたテーブルは複数のリージョンを持ちますか？

はい。BRはテーブルの[`SHARD_ROW_ID_BITS`および`PRE_SPLIT_REGIONS`](/sql-statements/sql-statement-split-region.md#pre_split_regions)情報をバックアップします。復元されたテーブルのデータは複数のリージョンに分割されます。