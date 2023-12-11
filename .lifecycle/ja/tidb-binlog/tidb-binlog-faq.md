---
title: TiDBビンログFAQ
summary: TiDBビンログに関するよくある質問（FAQ）とその回答を学びます。
aliases: ['/docs/dev/tidb-binlog/tidb-binlog-faq/','/docs/dev/reference/tidb-binlog/faq/','/docs/dev/reference/tools/tidb-binlog/faq/']
---

# TiDBビンログFAQ

この文書は、TiDBビンログに関するよくある質問（FAQ）をまとめたものです。

## TiDBビンログを有効にするとTiDBのパフォーマンスにどのような影響がありますか？

- クエリには影響がありません。

- `INSERT`、`DELETE`、`UPDATE`トランザクションにはわずかなパフォーマンス影響があります。最終的には、トランザクションがコミットされる前にTiKVの事前書き込み段階でp-binlogが同時に書き込まれます。一般的に、binlogの書き込みはTiKVの事前書き込みよりも速いため、レイテンシは増加しません。Pumpのモニタリングパネルでbinlogの書き込みの応答時間をチェックできます。

## TiDBビンログのレプリケーション遅延はどのくらいですか？

TiDBビンログのレプリケーションのレイテンシは秒単位であり、通常は混雑時でも約3秒です。

## DrainerがダウンストリームのMySQLまたはTiDBクラスタにデータをレプリケートするためにどのような権限が必要ですか？

ダウンストリームのMySQLまたはTiDBクラスタにデータをレプリケートするために、Drainerには以下の権限が必要です。

* Insert
* Update
* Delete
* Create
* Drop
* Alter
* Execute
* Index
* Select
* Create View

## Pumpのディスクがほぼいっぱいの場合、どうすればよいですか？

1. PumpのGCが正常に動作しているかどうかを確認します。

    - Pumpのモニタリングパネルで**gc_tso**の時刻が構成ファイルの時刻と同じかどうかを確認します。

2. GCが正常に動作している場合、次の手順を実行して単一のPumpに必要なスペースの量を減らします。

    - Pumpの**GC**パラメータを変更して、データを保持する日数を減らします。

    - Pumpインスタンスを追加します。

## Drainerのレプリケーションが中断した場合、どうすればよいですか？

次のコマンドを実行して、Pumpの状態が正常であり、`offline`状態でないすべてのPumpインスタンスが実行中であるかどうかを確認します。

{{< copyable "shell-regular" >}}

```bash
binlogctl -cmd pumps
```

次に、Drainerのモニタまたは対応するエラーが出力されているかどうかを確認します。それがある場合は、それに応じて解決します。

## DrainerがダウンストリームのMySQLまたはTiDBクラスタにデータを遅延してレプリケートする場合、どうすればよいですか？

次のモニタリング項目をチェックします。

- **ドレイナーイベント**モニタリングメトリクスでは、ドレイナーがダウンストリームに対して1秒あたりに`INSERT`、`UPDATE`、`DELETE`トランザクションをレプリケートするスピードをチェックします。

- **SQLクエリ時間**モニタリングメトリクスでは、ドレイナーがダウンストリームでSQLステートメントを実行するのにかかる時間をチェックします。

遅延の原因と解決策：

- レプリケートされたデータベースにプライマリキーまたはユニークインデックスのないテーブルがある場合、そのテーブルにプライマリキーを追加します。

- Drainerとダウンストリーム間のレイテンシが高い場合、Drainerの`worker-count`パラメータの値を増やします。データセンタ間のレプリケーションの場合、Drainerをダウンストリームにデプロイすることをお勧めします。

- ダウンストリームの負荷が高くない場合、Drainerの`worker-count`パラメータの値を増やします。

## Pumpインスタンスがクラッシュした場合、どうすればよいですか？

Pumpインスタンスがクラッシュすると、Drainerはこのインスタンスのデータを取得できないため、ダウンストリームにデータをレプリケートできません。このPumpインスタンスが正常な状態に戻る場合、Drainerはレプリケーションを再開します。そうでない場合は、次の手順を実行します。

1. [binlogctlを使用して、このPumpインスタンスの状態を`offline`に変更](/tidb-binlog/maintain-tidb-binlog-cluster.md)して、このPumpインスタンスのデータを破棄します。

2. DrainerがこのPumpインスタンスのデータを取得できないため、ダウンストリームとアップストリームのデータが一貫していません。この状況で次の手順を実行します。

    1. Drainerを停止します。

    2. アップストリームでフルバックアップを実行します。

    3. `tidb_binlog.checkpoint`テーブルを含む、ダウンストリームのデータを削除します。

    4. ダウンストリームにフルバックアップを復元します。

    5. Drainerをデプロイし、`initialCommitTs`（`initialCommitTs`にはフルバックアップのスナップショットのタイムスタンプを設定）を初期レプリケーションの開始ポイントとして使用します。

## チェックポイントとは何ですか？

チェックポイントは、Drainerがダウンストリームにレプリケートする`commit-ts`を記録します。Drainerが再起動すると、チェックポイントを読み取り、対応する`commit-ts`からダウンストリームにデータをレプリケートします。Drainerログの`["write save point"] [ts=411222863322546177]`は、対応するタイムスタンプでチェックポイントを保存していることを意味します。

異なる種類のダウンストリームプラットフォームでは、チェックポイントが異なる方法で保存されます。

- MySQL/TiDBの場合、`tidb_binlog.checkpoint`テーブルに保存されます。

- Kafka/fileの場合、対応する構成ディレクトリのファイルに保存されます。

kafka/fileのデータには`commit-ts`が含まれているため、チェックポイントが失われた場合は、ダウンストリームの最新の`commit-ts`を消費して確認できます。

Drainerは起動時にチェックポイントを読み取ります。Drainerがチェックポイントを読み取れない場合は、設定された`initialCommitTs`を初期レプリケーションの開始ポイントとして使用します。

## ダウンストリームのデータが残っている状態で、Drainerが失敗した場合に新しいマシンにDrainerを再デプロイするにはどうすればよいですか？

ダウンストリームのデータに影響がない場合は、対応するチェックポイントからデータをレプリケートできる限り、新しいマシンにDrainerを再デプロイできます。

- チェックポイントが失われていない場合は、次の手順を実行します。

    1. 新しいDrainerをデプロイして起動します（Drainerはチェックポイントを読み取り、レプリケーションを再開します）。

    2. [binlogctlを使用して、古いDrainerの状態を`offline`に変更](/tidb-binlog/maintain-tidb-binlog-cluster.md)します。

- チェックポイントが失われている場合は、次の手順を実行します。

    1. 新しいDrainerをデプロイするために、古いDrainerの`commit-ts`を取得し、それを新しいDrainerの`initialCommitTs`として使用します。

    2. [binlogctlを使用して、古いDrainerの状態を`offline`に変更](/tidb-binlog/maintain-tidb-binlog-cluster.md)します。

## フルバックアップとビンログバックアップファイルを使用してクラスタのデータを復元するにはどうすればよいですか？

1. クラスタをクリーンアップし、フルバックアップを復元します。

2. バックアップファイルの最新データを復元するには、Reparoを使用して`start-tso`を{フルバックアップのスナップショットタイムスタンプ + 1}に設定し、`end-ts`を0に設定します（または特定の時点を指定することもできます）。

## `ignore-error`を有効にした際に、プライマリセカンダリレプリケーションで`update-pump`または`update-drainer`コマンドを使用してPumpまたはDrainerサービスを一時停止できますか？
いいえ。`update-pump`または`update-drainer`コマンドは、対応する操作を実行するようPumpまたはDrainerに通知せずに、PDに保存されている状態情報を直接変更します。これらのコマンドを誤って使用すると、データレプリケーションが中断され、データ損失の原因となる可能性があります。

## binlogctlで`update-pump`または`update-drainer`コマンドを使用してPumpまたはDrainerサービスをクローズできますか？

いいえ。`update-pump`または`update-drainer`コマンドは、PDに保存されている状態情報を直接変更しますが、対応する操作をPumpやDrainerに実行することなしに更新します。こうしたコマンドを誤って使用すると、データレプリケーションが中断され、上流と下流のデータの整合性に問題が生じる可能性があります。例：

- Pumpノードが正常に実行されているか`一時停止`状態にある場合、`update-pump`コマンドを使用してPumpの状態を`offline`に設定すると、Drainerノードは`offline`になったPumpからbinlogデータを取得できなくなります。この状況では、最新のbinlogがDrainerノードにレプリケートされなくなり、上流と下流のデータの整合性に問題が生じます。
- Drainerノードが正常に実行されている場合、`update-drainer`コマンドを使用してDrainerの状態を`offline`に設定すると、新しく開始されたPumpノードは`online`状態のDrainerノードにのみ通知します。この状況では、`offline`状態のDrainerはPumpノードからbinlogデータを時々に取得できず、上流と下流のデータの整合性に問題が発生します。

## binlogctlで`update-pump`コマンドを使用し、Pumpの状態を`paused`に設定できるのはいつですか？

いくつかの異常な状況では、Pumpが正常に状態を維持できなくなります。その場合、`update-pump`コマンドを使用して状態を変更します。

例えば、Pumpプロセスが異常終了した場合（パニックが発生したときにプロセスを直接終了させるか、`kill -9`コマンドを誤って使用した場合）、PDに保存されたPumpの状態情報はまだ`online`のままとなります。この状況では、サービスを即座に再起動してPumpを復旧させなくてもよい場合、`update-pump`コマンドを使用してPumpの状態を`paused`に更新します。その後、TiDBがbinlogを書き込み、Drainerがbinlogを取得する際に中断を回避することができます。

## binlogctlで`update-drainer`コマンドを使用し、Drainerの状態を`paused`に設定できるのはいつですか？

いくつかの異常な状況では、Drainerノードが正常に状態を維持できず、レプリケーションタスクに影響を及ぼす場合があります。その場合、`update-drainer`コマンドを使用して状態を変更します。

例えば、Drainerプロセスが異常終了した場合（パニックが発生したときにプロセスを直接終了させるか、`kill -9`コマンドを誤って使用した場合）、PDに保存されたDrainerの状態情報はまだ`online`のままとなります。Pumpノードが起動されると、終了したDrainerノードに通知されない（`notify drainer ...`エラーが発生する）ため、Pumpノードの失敗が起こります。この状況では、`update-drainer`コマンドを使用してDrainerの状態を`paused`に更新し、Pumpノードを再起動します。

## PumpまたはDrainerノードを閉じる方法は？

現在、PumpまたはDrainerノードを閉じるには、binlogctlで`offline-pump`または`offline-drainer`コマンドを使用するだけです。

## binlogctlで`update-pump`コマンドを使用し、Pumpの状態を`offline`に設定できるのはいつですか？

以下の状況で`update-pump`コマンドを使用してPumpの状態を`offline`に設定できます：

- Pumpプロセスが異常終了し、サービスを回復できない場合、レプリケーションタスクが中断されます。レプリケーションを回復し、binlogデータの一部の損失を受け入れたい場合、`update-pump`コマンドを使用してPumpの状態を`offline`に設定します。その後、DrainerノードはPumpノードからbinlogを取得するのを停止し、データのレプリケーションを続行します。
- いくつかの古いPumpノードが歴史的なタスクの残り物として残っています。これらのプロセスは終了し、サービスはもはや必要とされていません。その場合、`update-pump`コマンドを使用してその状態を`offline`に設定します。

その他の状況については、Pumpサービスを閉じるために`offline-pump`コマンドを使用してください。これが通常の手順です。

> **警告:**
>
> > `update-pump`コマンドは、binlogデータの損失と上流と下流のデータの整合性に寛容であり、Pumpノードに格納されたbinlogデータが不要でない場合にのみ使用してください。

## `update-pump`コマンドを使用して、終了したPumpノードの状態を`一時停止`に設定した後に、Pumpの状態を`offline`に設定できますか？

Pumpプロセスが終了し、ノードが`一時停止`状態の場合、ノード内のすべてのbinlogデータが下流のDrainerノードで消費されていない可能性があります。したがって、これを行うと上流と下流のデータの整合性にリスクが生じる場合があります。この状況では、Pumpを再起動し、`offline-pump`コマンドを使用してPumpノードを閉じてください。

## `update-drainer`コマンドを使用して、Drainerの状態を`offline`に設定できるのはいつですか？

いくつかの古いDrainerノードが歴史的なタスクの残り物として残っています。これらのプロセスは終了し、サービスはもはや必要とされていません。その場合、`update-drainer`コマンドを使用してその状態を`offline`に設定します。

## `change pump`や`change drainer`などのSQL操作を使用してPumpまたはDrainerサービスを一時停止または閉じることはできますか？

いいえ。これらのSQL操作はPDに保存されている状態情報を直接変更し、binlogctlの`update-pump`および`update-drainer`コマンドと機能的に同等です。PumpまたはDrainerサービスを一時停止または閉じるには、binlogctlツールを使用してください。

## 上流データベースがサポートするDDL文が実行されたときに、下流データベースでエラーが発生する場合、どのように対処すればよいですか？

問題を解決するには、次の手順に従います：

1. `drainer.log`を確認します。Drainerプロセスがダウンする前の、最後に失敗したDDL操作について`exec failed`を検索します。
2. 下流データベースでDDLバージョンを下流と互換性のあるバージョンに変更します。この手順は手動で行います。
3. `drainer.log`を確認します。失敗したDDL操作を検索し、この操作の`commit-ts`を見つけます。例：

    ```
    [2020/05/21 09:51:58.019 +08:00] [INFO] [syncer.go:398] ["add ddl item to syncer, you can add this commit ts to `ignore-txn-commit-ts` to skip this ddl if needed"] [sql="ALTER TABLE `test` ADD INDEX (`index1`)"] ["commit ts"=416815754209656834].
    ```

4. `drainer.toml`設定ファイルを変更します。`ignore-txn-commit-ts`項目に`commit-ts`を追加し、Drainerノードを再起動します。

## TiDBはbinlogに書き込めず、`listener stopped, waiting for manual stop`がログに表示されます

TiDB v3.0.12およびそれ以前のバージョンでは、binlogの書き込みに失敗すると、TiDBが致命的エラーを報告します。TiDBは自動的に終了するわけではなく、サービスを停止しますが、スタックしたように見えます。ログに`listener stopped, waiting for manual stop`エラーが表示されます。

binlogの書き込み失敗の具体的な原因を特定する必要があります。失敗が下流にゆっくり書き込まれるためであれば、Pumpのスケーリングアウトまたはbinlogの書き込みのタイムアウト時間の増加を検討できます。

v3.0.13以降では、エラー報告ロジックが最適化されています。binlogの書き込み失敗はトランザクション実行に失敗し、TiDB Binlogはエラーを返しますが、TiDBをスタックさせることはありません。

## TiDBは重複したbinlogをPumpに書き込みます

この問題は下流やレプリケーションロジックに影響しません。

binlogの書き込みが失敗したり、タイムアウトになった場合、TiDBは次の利用可能なPumpノードにbinlogを書き込むまでリトライします。そのため、binlogがPumpノードに遅れて書き込まれ、TiDBがタイムアウト（デフォルトは15秒）になっても、TiDBは書き込みが失敗したと判断し、次のPumpノードに書き込みを試みます。実際にはタイムアウトの原因となったPumpノードにbinlogが正常に書き込まれた場合、同じbinlogが複数のPumpノードに書き込まれます。Drainerがbinlogを処理する際、同じTSOを持つbinlogは自動的に重複されますので、この重複書き込みは下流やレプリケーションロジックに影響しません。

## Reparoがバックアップと差分復元プロセス中に中断された。ログ中の最後のTSOを使用してレプリケーションを再開できますか？

はい。Reparoは開始時に自動的にセーフモードを有効にしません。次の手順を手動で実行する必要があります：

1. Reparoが中断された後、ログ中の最後のTSOを`checkpoint-tso`として記録します。
2. Reparoの設定ファイルを変更し、`start-tso`を`checkpoint-tso + 1`に設定し、`stop-tso`を`checkpoint-tso + 80,000,000,000`（`checkpoint-tso`よりも約5分後）に設定し、`safe-mode`を`true`に設定します。Reparoを開始し、Reparoは`stop-tso`までデータをレプリケートし、その後自動的に停止します。
```
3. After Reparo stops automatically, set `start-tso` to `checkpoint tso + 80,000,000,001`, set `stop-tso` to `0`, and set `safe-mode` to `false`. Start Reparo to resume replication.
```