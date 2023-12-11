---
title: TiDBの増分バックアップとリストアガイド
summary: TiDBでの増分バックアップおよびリストアの実行方法について学ぶ
---

# TiDBの増分バックアップとリストアガイド

TiDBクラスタの増分データとは、開始時のスナップショットと終了時のスナップショットの間で生成された差分データおよびこの期間中に生成されたDDLとなります。増分データは完全（スナップショット）バックアップデータと比較してより小さく、したがってバックアップデータの容量を削減するための補完的なものです。増分バックアップを実行するには、指定された期間内で生成されたMVCCデータが[TiDB GCメカニズム](/garbage-collection-overview.md)によって破棄されていないことを確認してください。たとえば、1時間ごとに増分バックアップを実行するには、[`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)を1時間以上の値に設定する必要があります。

> **警告:**
>
> この機能の開発は終了しました。代替として[ログバックアップおよびPITR](/br/br-pitr-guide.md)の使用が推奨されています。

## 増分データのバックアップ

増分データをバックアップするには、`br backup`コマンドを**最後のバックアップタイムスタンプ**`--lastbackupts`を指定して実行します。これにより、brコマンドラインツールは自動的に`lastbackupts`と現在の時刻の間で生成された増分データをバックアップします。`--lastbackupts`を取得するには、`validate`コマンドを実行します。以下は例です:

```shell
LAST_BACKUP_TS=`tiup br validate decode --field="end-version" --storage "s3://backup-101/snapshot-202209081330?access-key=${access-key}&secret-access-key=${secret-access-key}"| tail -n1`
```

次のコマンドは、`(LAST_BACKUP_TS、現在のPDタイムスタンプ]`の間に生成された増分データとこの期間中に生成されたDDLをバックアップします:

```shell
tiup br backup full --pd "${PD_IP}:2379" \
--storage "s3://backup-101/snapshot-202209081330/incr?access-key=${access-key}&secret-access-key=${secret-access-key}" \
--lastbackupts ${LAST_BACKUP_TS} \
--ratelimit 128
```

- `--lastbackupts`：最後のバックアップタイムスタンプ
- `--ratelimit`：バックアップタスクを実行する各TiKVの最大速度（MiB/s）
- `storage`：バックアップデータの保存先パス。増分バックアップデータを前回のスナップショットバックアップとは異なるパスに保存する必要があります。前述の例では、増分バックアップデータは完全バックアップデータの`incr`ディレクトリに保存されます。詳細については、[外部ストレージサービスのURIフォーマット](/external-storage-uri.md)を参照してください。

## 増分データのリストア

増分データをリストアする際には、`LAST_BACKUP_TS`より前にバックアップされたすべてのデータがターゲットクラスタに復元されていることを確認してください。また、増分リストアはデータを更新するため、リストア中に他の書き込みがないことを確認する必要があります。そうでないと競合が発生する可能性があります。

次のコマンドは、`backup-101/snapshot-202209081330`ディレクトリに保存された完全バックアップデータを復元します:

```shell
tiup br restore full --pd "${PD_IP}:2379" \
--storage "s3://backup-101/snapshot-202209081330?access-key=${access-key}&secret-access-key=${secret-access-key}"
```

次のコマンドは、`backup-101/snapshot-202209081330/incr`ディレクトリに保存された増分バックアップデータを復元します:

```shell
tiup br restore full --pd "${PD_IP}:2379" \
--storage "s3://backup-101/snapshot-202209081330/incr?access-key=${access-key}&secret-access-key=${secret-access-key}"
```